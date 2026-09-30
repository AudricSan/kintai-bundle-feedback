# Changelog

Tous les changements notables de ce bundle sont documentés dans ce fichier.

Le format suit [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/).
Le schéma de version (X.Y.Z, canaux alpha/beta/main) est décrit dans
`.github/workflows/release.yml`.

## [Unreleased]

### Changed

- Garde-fou contre les handlers inline, en prévision de la Content-Security-Policy stricte de Kintai Core 0.3.0 (`script-src 'self' 'nonce-…'`, sans `'unsafe-inline'`) : `tests.yml` échoue désormais si un attribut `onclick=`/`onchange=`/`onsubmit=`/`oninput=`, un lien `javascript:` ou un `<script>` sans nonce apparaît dans `Views/` ou `src/` — le navigateur les bloquerait en silence, sans aucune erreur côté serveur. **Aucun changement fonctionnel** : les vues de ce bundle n'utilisent déjà aucun handler inline ni `<script>` inline exécutable. La règle est documentée dans `CONTRIBUTING.md` et `CLAUDE.md`.

## [1.1.0] - 2026-09-29

### Changed

- Aucun changement fonctionnel — bump de version pour aligner ce bundle sur la ligne 1.1.0 commune à tous les bundles officiels.
- La notification envoyée aux managers à la soumission d'un feedback ne disait rien de son contenu et ne menait nulle part au clic. Le corps précise désormais la catégorie, l'auteur (ou "Anonyme") et le magasin (`notif_feedback_submitted_body`, côté Kintai Core, gagne les placeholders `:category`/`:author`/`:store`), et le clic renvoie vers `/admin/feedbacks`. **Nécessite** la version de Kintai Core introduisant le paramètre `$link` sur `notify()`/`notifyMany()`.
- Le CSS (formulaire + modale, `.fb-*`) et le JS (`feedback.js`) vivaient dans Kintai Core — y compris le bloc modale (`.fb-overlay`/`.fb-modal*`), auparavant mélangé au composant partagé `modals.css`. Tout vit maintenant dans `public/css/feedback.css`/`public/js/feedback.js`, fournis par ce bundle via `Bundle::loadAssetsFrom()`/`bundle_asset()` ; le partial Core `layout/partials/feedback-modal.php` (qui reste dans le Core, inclus pour tout utilisateur authentifié derrière `$feedback_enabled`) ne fournit plus que le HTML. **Nécessite** `kintai_core.min: "0.2.0"`.

## [1.0.0] - 2026-09-19

### Added

- Extraction initiale depuis Kintai (`src/Bundles/Feedback`).
