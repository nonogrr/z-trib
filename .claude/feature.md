# Commande : /feature

Génère tous les fichiers nécessaires pour une nouvelle fonctionnalité Z-Trib côté backend.

## Ce que cette commande produit

1. **Controller** dans `app/Http/Controllers/{Domaine}/`
2. **Form Requests** dans `app/Http/Requests/{Domaine}/`
3. **Routes** à ajouter dans `routes/web.php` ou `routes/public.php`
4. **Feature Test** dans `tests/Feature/{Domaine}/`

## Instructions pour Claude

Quand l'utilisateur invoque `/feature [Domaine] [Action]` :

### Étape 1 — Créer le Controller
Fichier : `app/Http/Controllers/{Domaine}/{Action}Controller.php`

```php
<?php

namespace App\Http\Controllers\{Domaine};

use App\Http\Controllers\Controller;
use App\Http\Requests\{Domaine}\Store{Action}Request;
use App\Http\Requests\{Domaine}\Update{Action}Request;
use App\Models\{Model};
use App\Services\{Action}Service;
use Inertia\Inertia;
use Inertia\Response;

class {Action}Controller extends Controller
{
    public function __construct(
        private readonly {Action}Service $service
    ) {}

    public function index(): Response
    {
        $this->authorize('viewAny', {Model}::class);

        return Inertia::render('{Domaine}/Index', [
            // props passées à React
        ]);
    }

    public function show({Model} ${model}): Response
    {
        $this->authorize('view', ${model});

        return Inertia::render('{Domaine}/Show', [
            '{model}' => ${model},
        ]);
    }

    public function store(Store{Action}Request $request): \Illuminate\Http\RedirectResponse
    {
        $this->authorize('create', {Model}::class);

        $result = $this->service->create($request->validated());

        return redirect()->route('{domaine}.show', $result)
            ->with('success', '{Action} créé avec succès.');
    }

    public function update(Update{Action}Request $request, {Model} ${model}): \Illuminate\Http\RedirectResponse
    {
        $this->authorize('update', ${model});

        $this->service->update(${model}, $request->validated());

        return redirect()->back()->with('success', 'Mis à jour.');
    }

    public function destroy({Model} ${model}): \Illuminate\Http\RedirectResponse
    {
        $this->authorize('delete', ${model});

        $this->service->delete(${model});

        return redirect()->route('{domaine}.index')
            ->with('success', 'Supprimé.');
    }
}
```

**Règles Controller Z-Trib :**
- Toujours appeler `$this->authorize()` en première ligne de chaque méthode
- Toujours déléguer la logique métier au Service correspondant
- Retourner `Inertia::render()` pour les GET, `redirect()` pour les POST/PUT/DELETE
- Ne jamais écrire de logique SQL dans un Controller

### Étape 2 — Créer les Form Requests
Fichier : `app/Http/Requests/{Domaine}/Store{Action}Request.php`

```php
<?php

namespace App\Http\Requests\{Domaine};

use Illuminate\Foundation\Http\FormRequest;

class Store{Action}Request extends FormRequest
{
    public function authorize(): bool
    {
        return true; // autorisation gérée dans le Controller via Policy
    }

    public function rules(): array
    {
        return [
            // règles de validation
        ];
    }

    public function messages(): array
    {
        return [
            // messages d'erreur en français
        ];
    }
}
```

**Règles de validation Z-Trib fréquentes :**
```php
// Événement
'nom'            => ['required', 'string', 'max:100'],
'type'           => ['required', 'in:prive,tribu,public'],
'statut'         => ['required', 'in:brouillon,ouvert,en_cours,cloture,annule'],
'sport_id'       => ['required', 'uuid', 'exists:sports,id'],
'tribu_id'       => ['nullable', 'uuid', 'exists:tribus,id'],
'date_debut'     => ['required', 'date', 'after_or_equal:today'],
'date_fin'       => ['nullable', 'date', 'after_or_equal:date_debut'],
'nb_max_participants' => ['nullable', 'integer', 'min:2', 'max:10000'],

// Score
'score_global'   => ['required', 'integer', 'min:0'],
'resultat'       => ['required', 'in:victoire,nul,defaite'],
'chrono_secondes'=> ['nullable', 'integer', 'min:0'],

// Membre
'pseudo'         => ['required', 'string', 'max:30', 'unique:membres,pseudo'],
'email'          => ['required', 'email', 'unique:membres,email'],
'visibilite'     => ['required', 'in:public,tribu,prive'],
```

### Étape 3 — Déclarer les routes
Selon le type d'accès, ajouter dans le bon fichier :

**`routes/web.php`** (routes authentifiées Inertia) :
```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::resource('{domaine}', {Action}Controller::class);
    // ou routes nommées spécifiques
});
```

**`routes/public.php`** (pages QR code sans auth) :
```php
// Uniquement pour les pages publiques des événements
Route::prefix('p/{token}')->group(function () {
    Route::get('/resultats', [PublicResultatsController::class, 'index'])->name('public.resultats');
    Route::get('/inscription', [PublicInscriptionController::class, 'index'])->name('public.inscription');
    Route::get('/jour-j', [PublicJourJController::class, 'index'])->name('public.jour-j');
});
```

### Étape 4 — Créer le Feature Test
Fichier : `tests/Feature/{Domaine}/{Action}Test.php`

```php
<?php

namespace Tests\Feature\{Domaine};

use App\Models\Membre;
use App\Models\{Model};
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class {Action}Test extends TestCase
{
    use RefreshDatabase;

    private Membre $membre;

    protected function setUp(): void
    {
        parent::setUp();
        $this->membre = Membre::factory()->create();
    }

    /** @test */
    public function un_membre_peut_creer_un_{model}(): void
    {
        $this->actingAs($this->membre);

        $response = $this->post(route('{domaine}.store'), [
            // données valides
        ]);

        $response->assertRedirect();
        $this->assertDatabaseHas('{table}', [
            // vérifier les données en base
        ]);
    }

    /** @test */
    public function un_membre_non_autorise_ne_peut_pas_modifier(): void
    {
        $autreMembre = Membre::factory()->create();
        ${model} = {Model}::factory()->create(['membre_id' => $this->membre->id]);

        $this->actingAs($autreMembre);

        $response = $this->put(route('{domaine}.update', ${model}), [
            // données
        ]);

        $response->assertForbidden();
    }

    /** @test */
    public function les_donnees_invalides_sont_rejetees(): void
    {
        $this->actingAs($this->membre);

        $response = $this->post(route('{domaine}.store'), [
            // données invalides
        ]);

        $response->assertSessionHasErrors(['nom_du_champ']);
    }
}
```

**Tests à toujours écrire pour Z-Trib :**
1. Cas nominal (données valides → succès)
2. Autorisation refusée (mauvais membre → 403)
3. Validation échouée (données invalides → erreurs)
4. Cas limites métier (ex: événement clôturé ne peut plus recevoir de score)

## Exemple d'usage

```
/feature Evenement Create
```

Produit :
- `app/Http/Controllers/Evenement/EvenementController.php`
- `app/Http/Requests/Evenement/StoreEvenementRequest.php`
- `app/Http/Requests/Evenement/UpdateEvenementRequest.php`
- Routes à ajouter dans `routes/web.php`
- `tests/Feature/Evenement/EvenementTest.php`

```
/feature Match SaisieScore
```

Produit :
- `app/Http/Controllers/Match/SaisieScoreController.php`
- `app/Http/Requests/Match/StoreSaisieScoreRequest.php`
- `tests/Feature/Match/SaisieScoreTest.php`
