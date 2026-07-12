# Kitchen Timers — Sezam&Co
> Last update: 2026-07-12

App de minuteurs cuisine pour tablettes (Android + iPad, web app). Fichier unique : `index.html`.

## Décisions validées (12/07/2026)
- Voix en **anglais** (Web Speech API, en-US), phrase éditable par timer (défaut : "The {name} is ready")
- Après arrêt d'une alarme → le timer revient à sa durée initiale
- Hébergement cible : GitHub Pages (pas encore fait) ; v2 : sync Firebase
- Persistance : localStorage (`sezam-timers-v1` défs, `sezam-timers-run-v1` état en cours)

## Fonctionnalités v1
- CRUD timers (nom, durée, ringtone ×3 synthétisées WebAudio, phrase vocale)
- Grille auto-scale : les tuiles se redimensionnent pour que tout tienne sans scroll
- Tap = start ; tuile en cours : boutons −10s / +10s / ✕ (tap sur le corps ignoré, anti fausse manip)
- Alarme : ringtone + voix, répétition à ~7s puis fréquence croissante (gap ×0.72, min 2s), compteur de dépassement (+m:ss), pulsation rouge
- Mode Edit : wobble, tap = éditer, drag pointer-based = réordonner
- Timestamps (endsAt) → survit au reload / onglet en arrière-plan ; Wake Lock ; unlock audio au 1er tap ; vibration si supportée (Android oui, iPad non)

## Lancer en local
```
python -m http.server 8734 -d C:\Users\serge\Claude\topics\kitchen-timers
```
→ http://localhost:8734

## Design
Thème **light** (demandé par Serge) : fond papier chaud `#f5f1e8`, texte charbon, accent ember `#e8600f`. Fonts : Big Shoulders Display (titres/noms) + Chivo Mono (chiffres, tabular).

## Changelog
- 2026-07-12 — v1 initiale (single-file) thème light, servie en localhost
- 2026-07-12 — fix: overlay `#empty` (position:absolute inset:0) interceptait tous les clics sur les tuiles car `#empty{display:flex}` (spécificité #id) écrasait l'attribut `hidden`. Ajout `#empty[hidden]{display:none}` + `pointer-events:none`. Vérifié headless (idle→running au clic). + favicon inline (supprime 404).
- 2026-07-12 — batch retours Serge (tous vérifiés headless, 0 erreur) :
  - **Voix féminine** : `scoreVoice()` sélectionne une voix anglaise féminine par nom (Zira/Aria/Samantha…), évite les voix masculines (David/Mark…). Machine actuelle → Zira.
  - **File vocale** (`speechQueue`) : les annonces prennent leur tour au lieu de se couper ; arrêter un timer ne coupe que SA voix, pas celle d'un autre timer encore en train de sonner.
  - **Interaction timer actif** : tap simple = **pause/reprise** (statut `paused` : temps figé, anneau gris, respiration) ; **double-tap** = **reset**. Désambiguïsation via fenêtre 260 ms.
  - **Anneau** épaissi (`--ring-w:10px`, `--tile-r:20px`).
  - **Bouton ✕** écarté des ±10s (`margin-left:10cqw` ≈ 36px) + bordure rouge distinctive (anti fausse manip).
  - **Nom du plat** agrandi (11cqh → 14cqh) ; décompte reste dominant. Vérifié en capture (nom long « Chicken Shawarma » non tronqué).
- 2026-07-12 — **timers par défaut** (SEED_VERSION=2) = Fries 4:00 · Mozza/elastics 2:15 · Manoushe 0:45 · Crispy chicken 6:00 · Dough making 12:00 · Cooking chicken 6:00 · Donut 1:30. Bump SEED_VERSION = ré-applique la liste (remplace les timers existants une fois).
- 2026-07-12 — **PWA iPad/Android** : manifest.webmanifest (standalone, paysage), icônes PNG 180/192/512, service worker (`sw.js`, offline shell), meta iOS (apple-touch-icon, status-bar, title), theme-color, viewport-fit=cover + safe-area, anti-zoom/anti-callout, layout robuste au padding. Haptique : Android via Vibration API ; iOS via hack `<switch>` (best-effort, **non garanti** — iOS Safari n'a pas d'API Vibration). Vérifié headless (7 timers, manifest valide, SW enregistré, comportements OK, 0 erreur).

## Déploiement
GitHub Pages, repo `sergemio/sezam-timers` → https://sergemio.github.io/sezam-timers/
Pour mettre à jour : commit + push sur `main` (bumper `CACHE` dans `sw.js` à chaque changement pour forcer le refresh des clients).
