# Z-Trib — Contexte Claude Code

## Présentation du projet
Application web de gestion d'événements sportifs amateurs.
- Met en relation des sportifs pour organiser matchs, tournois, entraînements
- Permet à un club d'organiser des événements publics avec pages QR code
- Système de points de fidélité appelés **Tribs**

---

## Stack technique

| Couche       | Techno               | Version |
|--------------|----------------------|---------|
| Backend      | PHP / Laravel        | 8.3 / 12.x |
| Bridge SPA   | Inertia.js           | 2.x     |
| Frontend     | React + Tailwind CSS | 19.x / 4.x |
| Build        | Vite                 | 6.x     |
| Base données | MariaDB              | 12.x    |
| Cache/Queue  | Valkey               | 7.x     |
| WebSocket    | Laravel Reverb       | 1.x     |
| Infra        | Docker + Nginx + PHP-FPM + Supervisor | — |

---

## Architecture

Monolithe Laravel avec rendu côté serveur via Inertia.js + React.
Pas d'API REST dédiée : les données transitent via les props Inertia.

```
Navigateur (React SPA)
    ↓ HTTPS / WSS
Nginx (reverse proxy, SSL)
    ↓
Laravel 12 (Controllers → Inertia → React Pages)
    ↓              ↓              ↓
MariaDB        Valkey         Reverb
(données)   (cache/queues)  (WebSocket)
```

---

## Domaines métier

### Membre
- Profil : nom, pseudo, âge, région, email, avatar
- Authentification : Google, Apple, email+password
- MFA optionnel : application TOTP ou email
- Visibilité : `public` | `tribu` | `prive`
- Points **Tribs** cumulés (global et par sport)

### Sport
- Géré uniquement par les admins globaux
- 4 modes de score : `score` | `sets_jeux` | `victoire_nul_defaite` | `classement_arrivee`
- Optionnel : chrono

### Tribu
- Groupe de sportifs autour d'un ou plusieurs sports
- Rôles : `chef` (gestion membres) | `membre`

### Événement — 3 niveaux de confidentialité
| Type     | Visibilité       | Inscription | Pages publiques QR |
|----------|-----------------|-------------|-------------------|
| `prive`  | Membres only    | Libre       | Non               |
| `tribu`  | Tribu only      | Non         | Non               |
| `public` | Tout le monde   | Via lien    | Oui (4 pages)     |

Les pages publiques d'un événement public sont accessibles via `lien_public_token`
sans authentification (route dans `routes/public.php`).

### Activité (format compétitif)
- `championnat` — tout le monde rencontre tout le monde
- `match_unique` — un seul match
- `tournoi_poule_elim` — phases de poule + élimination directe
- `tournoi_poule_poule` — phases de poule + seconde phase de poule
- `montante_descendante` — montée/descente entre groupes
- `entrainement` — pas de score, juste une description

### Match / Score
- Score adapté au type de sport (score brut, sets/jeux, V/N/D, classement à l'arrivée)
- Saisie du score : tout membre autorisé dans l'événement
- Scores temps réel via Reverb (event `ScoreUpdated`)

### Classement
- Recalculé de façon **asynchrone** via Job (`RecalculerClassement`)
- Jamais calculé à la volée en requête HTTP
- Stocké en base dans `classements`

### Tribs
- Historique tracé dans `tribs_historique` à chaque attribution
- Jamais calculé à la volée
- Classement global et par sport

---

## Conventions de code

### Backend
- **1 migration par entité**, numérotée `XXXX_` (ex: `0018_add_logo_to_sports_table.php`)
- **Ne jamais modifier une migration existante** — toujours en créer une nouvelle
- Chaque Model a son **Factory** et sa **Policy**
- Les **Services** portent la logique métier (ex: `TournoiGeneratorService`)
- Les **Controllers** restent fins : ils délèguent aux Services
- Les **Form Requests** valident les entrées (`app/Http/Requests/`)
- Les **Jobs** sont dispatchés sur la queue Valkey, pas exécutés en synchrone

### Frontend
- Une **Page Inertia** par route (`resources/js/Pages/`)
- Les composants réutilisables sont dans `resources/js/Components/`
- Layout connecté : `AppLayout.jsx`
- Layout sans auth (QR) : `PublicLayout.jsx`
- WebSocket temps réel : hook `useReverb.js`

### Tests
- Feature test pour chaque Controller / endpoint
- Unit test pour chaque Service
- Toujours tester les cas d'autorisation (Policy)

---

## Commandes utiles

```bash
# Dev
docker compose up -d
php artisan migrate:fresh --seed
php artisan reverb:start
npm run dev

# Générer une entité complète
# → utilise la commande slash /migration

# Lancer les tests
php artisan test

# Vérifier les migrations en attente
php artisan migrate:status

# Prévisualiser le SQL d'une migration
php artisan migrate --pretend
```

---

## Fichiers à ne pas modifier directement

- `database/migrations/0001_` à `0017_` — migrations initiales du schéma
- `resources/js/Layouts/PublicLayout.jsx` — layout pages QR code
- `docker/supervisor/supervisord.conf` — configuration Reverb + queue worker
- `routes/public.php` — routes sans auth pour les pages QR code

---

## Variables d'environnement importantes

```
APP_ENV=local
DB_CONNECTION=mariadb
BROADCAST_CONNECTION=reverb
QUEUE_CONNECTION=valkey
CACHE_STORE=valkey
SESSION_DRIVER=valkey
REVERB_APP_ID=
REVERB_APP_KEY=
REVERB_APP_SECRET=
```

---

## Commandes slash disponibles

| Commande     | Description                                              |
|--------------|----------------------------------------------------------|
| `/migration` | Génère migration + Model + Factory + Policy              |
| `/feature`   | Génère Controller + Form Request + Feature Test          |
| `/component` | Génère composant React + Page Inertia si nécessaire      |
