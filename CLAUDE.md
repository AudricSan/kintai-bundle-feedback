# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is the standalone distribution repo for the official "Feedback" bundle of
[Kintai](https://github.com/AudricSan/Kintai). It used to live inside the main
Kintai monorepo at `src/Bundles/Feedback/` and was extracted so it can be
installed independently, the same way any third-party bundle would be (see
`docs/creating-a-bundle.md` in the main Kintai repo for the full bundle
distribution model — manifest, registry, installer).

There is no build or test suite in this repo (no `composer.json`, no
PHPUnit). The code is not runnable or functionally testable standalone: every
class under `src/` depends on `kintai\Core\*` (repositories, middleware,
`Request`/`Response`, `ViewRenderer`, `PermissionService`, etc.) that only
exist inside a running Kintai instance. Verifying a behavior change means
installing the bundle into a real Kintai instance, not running anything in
this repo. CI here only checks PHP syntax and manifest validity (see
"CI and branches" below) — it cannot catch logic errors.

Kintai never `git clone`/`pull`s bundles (many shared-hosting environments
have no `git` CLI available to PHP) — `BundleInstallerService` always
downloads a tagged GitHub Release's zipball. This repo's only "build output"
is therefore the GitHub Release itself; nothing here gets compiled or
packaged.

## CI and branches

This repo mirrors the branch/release model of the main Kintai repo:

- `main`, `alpha`, and `beta` are protected branches — no direct push; land
  changes via a PR (see `CONTRIBUTING.md`). New work targets `alpha` (the
  active channel); promote a line forward by merging `alpha` → `beta` → `main`.
- `.github/workflows/tests.yml` runs a `test` job (PHP syntax check via
  `php -l` on every `.php` file, plus JSON validation of `bundle.json` and
  `lang/*.json`) on every push and PR to these branches. This is the required
  status check gating merges.
- Merging into any of the three branches triggers
  `.github/workflows/release.yml`, which tags and publishes a GitHub Release
  — see "Release process" below.

## Release process

`.github/workflows/release.yml` triggers on push to `alpha`, `beta`, or
`main` (i.e. on every merge, since those branches are protected) and computes
and pushes the tag itself — never tag or `gh release create` by hand:

- Version line `X.Y` comes from `version` in `bundle.json`, which is always
  written as the placeholder `X.Y.0` and is only bumped by hand when opening a
  new release line (new `Y`).
- `alpha`/`beta` merges tag `vX.Y.Z` as a prerelease, where `Z` is the highest
  existing `vX.Y.*` tag + 1 — a counter shared and cumulative across alpha and
  beta within the same line, never reset between them.
- `main` merges tag `vX.Y.0` as the stable release for that line. If `vX.Y.0`
  already exists, the job skips cleanly (a line only ever gets one stable
  release; further fixes require opening a new line).
- Release notes are extracted from `CHANGELOG.md`: `## [Unreleased]` for
  alpha/beta (falling back to `## [X.Y.0]` if `Unreleased` is empty, i.e. the
  release commit already renamed it), or `## [X.Y.0]` directly for `main`. A
  push to a channel with no matching CHANGELOG section fails the job — always
  update `CHANGELOG.md` in your PR before merging.

## Architecture

- `bundle.json` — manifest read by Kintai's bundle installer/registry: slug,
  version, `kintai_core` compatibility range, `entry_class`.
- `src/FeedbackBundle.php` — the entry point (`kintai\Bundles\Installed\Feedback\FeedbackBundle`,
  extends `kintai\Core\BundleContract\Bundle`). Its `register()` binds
  `FeedbackRepositoryInterface` to `DatabaseFeedbackRepository` as a singleton
  in the app container, then calls `loadViewsFrom(..., 'feedback')` and
  `loadRoutesFrom(routes.php)`. `FeedbackRepositoryInterface` is owned and
  registered by this bundle, not by Kintai Core — no other Core component
  depends on it, so uninstalling/disabling this bundle removes the feature
  entirely. (Kintai Core still has a `feedback-modal` partial gated by
  `FeatureManager`/`AuthMiddleware` so the footer form disappears too when the
  bundle is absent.)
- `routes.php` — three route groups, each with different middleware:
  - `/employee/*` (`AuthMiddleware`) — feedback submission modal + past-shifts
    lookup for any authenticated employee.
  - `/admin/*` (`AuthMiddleware` + `PermissionMiddleware`, permissions
    `feedbacks.view`/`feedbacks.delete`) — admin list/delete screen.
  - `/api/v1/*` (`ApiAuthMiddleware` + `ApiPermissionMiddleware`) — REST API,
    permission-checked per route (`feedbacks.view`/`update`/`delete`), with
    `store` additionally scoped via `self: user_id`.
- `src/Controllers/Web/FeedbackController.php` — submission flow: resolves
  the user's store from `StoreUserRepositoryInterface` (Owner/global roles can
  have no store, so `store_id` is nullable), validates category/rating/shift
  ownership, saves via `FeedbackRepositoryInterface`, audit-logs, and notifies
  store managers who hold `feedbacks.view` (looked up manually per-user via
  `PermissionService::can()`, not a role check). Also serves the "past shifts"
  dropdown used by the submission modal.
- `src/Controllers/Api/FeedbackController.php` — REST CRUD. Its docblock
  references an RBAC audit fix (mirrors `ShiftSwapRequestController` in Core):
  unscoped `index()` used to leak all stores' feedback to any token holder.
  The current code always restricts `index()` results via
  `PermissionService::restrictToScope()` and single-item access via
  `requireOwnedResource()` — follow this pattern for any new bundle API
  endpoint rather than calling repository methods directly.
- `Views/feedbacks.php` — admin list view, registered under the `feedback::`
  view namespace (so it's referenced from controllers as `feedback::feedbacks`).
- `lang/{en,fr,ja}.json` — bundle-scoped translation keys, merged into
  Kintai's `__()` translator. Keys used by `FeedbackBundle` itself
  (`bundle_feedback`, `bundle_feedback_desc`) must exist in every locale file.
