# Changelog

Tous les changements notables de ce bundle sont documentés dans ce fichier.

Le format suit [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/).
Le schéma de version (X.Y.Z, canaux alpha/beta/main) est décrit dans
`.github/workflows/release.yml`.

## [Unreleased]

### Changed

- Aucun changement fonctionnel — bump de version pour aligner ce bundle sur la ligne 1.1.0 commune à tous les bundles officiels.
- La notification envoyée aux managers à la soumission d'un feedback ne disait rien de son contenu et ne menait nulle part au clic. Le corps précise désormais la catégorie, l'auteur (ou "Anonyme") et le magasin (`notif_feedback_submitted_body`, côté Kintai Core, gagne les placeholders `:category`/`:author`/`:store`), et le clic renvoie vers `/admin/feedbacks`. **Nécessite** la version de Kintai Core introduisant le paramètre `$link` sur `notify()`/`notifyMany()`.

## [1.0.0] - 2026-09-19

### Added

- Extraction initiale depuis Kintai (`src/Bundles/Feedback`).
