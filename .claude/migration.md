# Commande : /migration

Génère tous les fichiers nécessaires pour une nouvelle entité du modèle de données Z-Trib.

## Ce que cette commande produit

Pour une entité donnée, génère dans l'ordre :

1. **Migration** numérotée dans `database/migrations/`
2. **Model Eloquent** dans `app/Models/`
3. **Factory** dans `database/factories/`
4. **Policy** dans `app/Policies/`

## Instructions pour Claude

Quand l'utilisateur invoque `/migration [NomEntite]` :

### Étape 1 — Déterminer le numéro de migration
Lire les fichiers existants dans `database/migrations/` et prendre le numéro suivant
(ex: si le dernier est `0017_`, créer `0018_`).

### Étape 2 — Créer la migration
Fichier : `database/migrations/XXXX_create_{table}_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('{table}', function (Blueprint $table) {
            $table->uuid('id')->primary();
            // colonnes spécifiques à l'entité
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('{table}');
    }
};
```

**Conventions colonnes :**
- Clé primaire : `$table->uuid('id')->primary()`
- Clés étrangères : `$table->uuid('{entite}_id')` + `$table->foreign('{entite}_id')->references('id')->on('{table}')->cascadeOnDelete()`
- Enums : `$table->enum('statut', ['val1', 'val2'])`
- Texte long : `$table->text()` — Texte court : `$table->string()`
- Booléen : `$table->boolean()->default(false)`
- Timestamps nullable : `$table->timestamp('publie_at')->nullable()`

### Étape 3 — Créer le Model
Fichier : `app/Models/{NomEntite}.php`

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class {NomEntite} extends Model
{
    use HasUuids, HasFactory;

    protected $fillable = [
        // lister les colonnes fillable
    ];

    protected $casts = [
        'id' => 'string',
        // casts des enums et dates
    ];

    // Relations Eloquent (belongsTo, hasMany, belongsToMany)
}
```

**Conventions relations :**
- `belongsTo` pour les FK portées par cette table
- `hasMany` / `hasOne` pour les entités enfants
- `belongsToMany` avec table pivot explicite

### Étape 4 — Créer la Factory
Fichier : `database/factories/{NomEntite}Factory.php`

```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class {NomEntite}Factory extends Factory
{
    public function definition(): array
    {
        return [
            // faker cohérent avec le domaine sportif Z-Trib
        ];
    }
}
```

**Données faker utiles pour Z-Trib :**
- Pseudo : `fake()->userName()`
- Région : `fake()->departmentName()` (ou ville française)
- Sport nom : parmi `['Football', 'Tennis', 'Basketball', 'Natation', 'Running']`
- Statut : `fake()->randomElement(['ouvert', 'en_cours', 'cloture'])`
- UUID ref : `Sport::factory()` / `Membre::factory()`

### Étape 5 — Créer la Policy
Fichier : `app/Policies/{NomEntite}Policy.php`

```php
<?php

namespace App\Policies;

use App\Models\Membre;
use App\Models\{NomEntite};

class {NomEntite}Policy
{
    public function viewAny(Membre $membre): bool
    {
        return true;
    }

    public function view(Membre $membre, {NomEntite} $entite): bool
    {
        return true; // à affiner selon les règles métier
    }

    public function create(Membre $membre): bool
    {
        return true;
    }

    public function update(Membre $membre, {NomEntite} $entite): bool
    {
        return $membre->id === $entite->membre_id; // exemple
    }

    public function delete(Membre $membre, {NomEntite} $entite): bool
    {
        return $membre->id === $entite->membre_id;
    }
}
```

## Règles spécifiques Z-Trib à respecter

- **Événement** : vérifier le `type` (prive/tribu/public) dans les policies `view` et `update`
- **Match / Score** : tout gestionnaire de l'événement peut `update`
- **Classement** : lecture seule, jamais de `create` / `update` direct (passe par le Job)
- **TribsHistorique** : lecture seule, jamais de `create` direct (passe par `TribsService`)
- **Sport** : `create` réservé aux admins globaux (`$membre->is_admin === true`)

## Exemple d'usage

```
/migration AnnonceEvenement
```

Produit :
- `database/migrations/0018_create_annonce_evenements_table.php`
- `app/Models/AnnonceEvenement.php`
- `database/factories/AnnonceEvenementFactory.php`
- `app/Policies/AnnonceEvenementPolicy.php`
