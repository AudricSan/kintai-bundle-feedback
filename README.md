# kintai-bundle-feedback

🌐 **English** · [Français](#français)

Official "Feedback" bundle for [Kintai](https://github.com/AudricSan/Kintai) — employee feedback (bugs, suggestions): a submission modal for employees, plus a list/delete admin screen and a REST API (`/api/v1/feedbacks`).

This bundle used to live inside the main Kintai repository (`src/Bundles/Feedback/`); it's now distributed independently, the same way any third-party bundle would be — see [docs/creating-a-bundle.md](https://github.com/AudricSan/Kintai/blob/main/docs/creating-a-bundle.md) in the main repository for the full distribution model (manifest, registry, installer).

## Installing

From a running Kintai instance, as the Owner: `/admin/bundles/market` → find "Feedback" (listed as official, from the official Kintai registry) → Install.

Manual installation isn't supported: Kintai never does `git clone`/`pull` for bundles (many shared-hosting environments have no `git` CLI available to PHP) — it always downloads a tagged GitHub Release's zipball through `BundleInstallerService`.

## Structure

```
bundle.json          # manifest — slug, version, Kintai core compatibility, entry class
src/
  FeedbackBundle.php                    # kintai\Bundles\Installed\Feedback\FeedbackBundle
  Controllers/Web/FeedbackController.php
  Controllers/Api/FeedbackController.php
Views/feedbacks.php  # admin list view
lang/{en,fr,ja}.json # bundle-specific translation keys
routes.php           # loaded by FeedbackBundle::register() via loadRoutesFrom()
```

## Releasing a new version

1. Bump `version` in `bundle.json`.
2. Tag the commit `vX.Y.Z` (matching `bundle.json`) and push the tag.
3. `.github/workflows/release.yml` creates the GitHub Release automatically — Kintai's installer reads its `zipball_url` directly, nothing else to build or upload.

## License

AGPL-3.0-only, same as Kintai itself — see [LICENSE](LICENSE).

---

## Français

Bundle officiel "Feedback" pour [Kintai](https://github.com/AudricSan/Kintai) — retours des employés (bugs, suggestions) : une modale de soumission côté employé, un écran liste/suppression côté admin, et une API REST (`/api/v1/feedbacks`).

Ce bundle vivait auparavant dans le dépôt principal de Kintai (`src/Bundles/Feedback/`) ; il est désormais distribué indépendamment, exactement comme n'importe quel bundle tiers — voir [docs/creating-a-bundle.md](https://github.com/AudricSan/Kintai/blob/main/docs/creating-a-bundle.md) dans le dépôt principal pour le modèle de distribution complet (manifest, registry, installeur).

### Installation

Depuis une instance Kintai en cours d'exécution, en tant qu'Owner : `/admin/bundles/market` → trouver "Feedback" (listé comme officiel, depuis le registry officiel Kintai) → Installer.

L'installation manuelle n'est pas prise en charge : Kintai ne fait jamais de `git clone`/`pull` pour ses bundles (de nombreux hébergements mutualisés n'exposent pas le CLI `git` à PHP) — il télécharge toujours le zipball d'une release GitHub taguée via `BundleInstallerService`.

### Publier une nouvelle version

1. Incrémenter `version` dans `bundle.json`.
2. Tagger le commit `vX.Y.Z` (correspondant à `bundle.json`) et pousser le tag.
3. `.github/workflows/release.yml` crée automatiquement la GitHub Release — l'installeur de Kintai lit directement son `zipball_url`, rien d'autre à construire ni à uploader.

### Licence

AGPL-3.0-only, comme Kintai lui-même — voir [LICENSE](LICENSE).
