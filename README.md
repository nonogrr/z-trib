# Z-Trib 🏆

> Organise tes événements sportifs. Rejoins une tribu. Gagne des Tribs.

Z-Trib est une application web pour les sportifs amateurs qui veulent organiser des matchs, tournois et entraînements — entre amis, en club ou en ouvert au public.

---

## Fonctionnalités

### 👤 Profil membre
- Inscription via Google, Apple ou email/password
- Authentification multi-facteur optionnelle (application TOTP ou email)
- Profil avec sports pratiqués, diplômes sportifs, statistiques de participation
- Visibilité configurable : public, membres de mes tribus, ou privé
- Classement général et par sport basé sur les **Tribs** (points de fidélité)

### 🏘️ Tribus
- Groupes de sportifs réunis autour d'un ou plusieurs sports
- Rôles : chef de tribu (invitation, révocation, gestion) et membre
- Chaque tribu peut organiser ses propres événements

### 📅 Événements — 3 niveaux
| Type | Accès | Inscription | Pages publiques |
|------|-------|-------------|-----------------|
| **Privé** | Membres invités | Libre | ✗ |
| **Tribu** | Membres de la tribu | Automatique | ✗ |
| **Public** | Tout le monde | Via lien ou QR code | ✓ |

Configuration complète : date, lieu, sport, règlement, vestiaires, restauration, logo, bandeau.

### 🏆 Formats de compétition
- **Championnat** — tout le monde rencontre tout le monde
- **Match unique**
- **Tournoi poule + élimination directe**
- **Tournoi poule + seconde phase de poule** (pour classement complet)
- **Montée / descente** entre groupes
- **Entraînement / activité ludique** — sans score ni classement

### ⚽ Modes de score adaptés au sport
- Score classique (football, basketball…)
- Sets et jeux (tennis, volleyball…)
- Victoire / Nul / Défaite uniquement (échecs…)
- Classement à l'arrivée avec ou sans chrono (running, natation…)

### 📡 Temps réel
- Scores et classements mis à jour en direct via WebSocket
- Annonces diffusées pendant l'événement

### 🔗 Pages publiques QR code (événements publics)
Sans compte, accessibles via lien partagé ou QR code :
- **Inscriptions** — formulaire pour équipes et joueurs externes
- **Jour J** — vestiaires, restauration, accès, règlement
- **Résultats** — matchs en cours, terminés, à venir + classement filtrable par équipe
- **Infos** — toutes les informations pratiques de l'événement

---

## Stack technique

| Couche | Technologie | Version |
|--------|-------------|---------|
| Backend | PHP / Laravel | 8.3 / 12.x |
| Bridge SPA | Inertia.js | 2.x |
| Frontend | React | 19.x |
| CSS | Tailwind CSS | 4.x |
| Build | Vite | 6.x |
| Base de données | MariaDB | 12.x |
| Cache / Queues | Valkey | 7.x |
| WebSocket | Laravel Reverb | 1.x |
| Infra | Docker + Nginx + PHP-FPM + Supervisor | — |

Architecture monolithique Laravel avec rendu via Inertia.js + React. Pas d'API REST séparée.

---

## Prérequis

- Docker & Docker Compose
- Node.js ≥ 20
- PHP 8.3 (si hors Docker)

---

## Installation

```bash
# Cloner le repo
git clone https://github.com/votre-org/ztrib.git
cd ztrib

# Copier la configuration d'environnement
cp .env.example .env

# Démarrer les conteneurs
docker compose up -d

# Installer les dépendances PHP
docker compose exec app composer install

# Installer les dépendances JS
npm install

# Générer la clé applicative
docker compose exec app php artisan key:generate

# Jouer les migrations et le seed de démo
docker compose exec app php artisan migrate --seed

# Compiler les assets
npm run dev
```

L'application est disponible sur [http://localhost](http://localhost).

---

## Commandes utiles

```bash
# Démarrer l'environnement complet
docker compose up -d

# Relancer les migrations depuis zéro (avec données de démo)
php artisan migrate:fresh --seed

# Démarrer le serveur WebSocket Reverb
php artisan reverb:start

# Lancer les tests
php artisan test

# Vérifier les migrations en attente
php artisan migrate:status

# Prévisualiser le SQL d'une migration sans l'exécuter
php artisan migrate --pretend
```

---

## Structure du projet

```
app/
├── Http/Controllers/     # Controllers fins (délèguent aux Services)
├── Models/               # Models Eloquent avec UUID
├── Policies/             # Autorisation par entité
└── Services/             # Logique métier

database/
├── migrations/           # Migrations numérotées (source de vérité du schéma)
├── factories/            # Factories pour les tests et le seed
└── seeders/

resources/js/
├── Components/           # Composants React réutilisables
├── Layouts/              # AppLayout (auth) + PublicLayout (QR)
├── Pages/                # Pages Inertia (1 par route)
└── hooks/                # Hooks React (WebSocket, classement…)

docs/
├── data-model.mermaid    # Modèle de données versionné
└── ARCHITECTURE.md       # Document d'architecture technique
```

---

## Contribuer

### Workflow Git

Chaque modification suit ce flux :

```
feature/nom-de-la-feature
  ├── migration numérotée si changement de schéma
  ├── model / service / controller
  └── tests feature + unit
```

**Règles importantes :**
- Ne jamais modifier une migration existante — toujours en créer une nouvelle
- Chaque nouvelle entité = migration + Model + Factory + Policy
- Les Controllers restent fins, la logique métier va dans les Services
- Tout changement de schéma doit être accompagné d'un Feature Test

### Claude Code

Ce projet est configuré pour [Claude Code](https://claude.ai/code). Le dossier `.claude/` contient le contexte projet et trois commandes slash :

| Commande | Description |
|----------|-------------|
| `/migration [Entite]` | Génère migration + Model + Factory + Policy |
| `/feature [Domaine] [Action]` | Génère Controller + Requests + Test |
| `/component [Domaine] [Nom]` | Génère composant React ou Page Inertia |

### CI / CD

La pipeline GitHub Actions vérifie à chaque PR :
- Exécution de `migrate:fresh` sur MariaDB de test
- Absence de migrations non jouées
- Suite de tests complète (`php artisan test`)

---

## Licence

MIT — voir [LICENSE](LICENSE).
