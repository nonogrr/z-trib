# Commande : /component

Génère les fichiers frontend React pour une nouvelle interface Z-Trib.

## Ce que cette commande produit

Selon le besoin :
1. **Composant réutilisable** dans `resources/js/Components/{Domaine}/`
2. **Page Inertia** dans `resources/js/Pages/{Domaine}/`
3. **Hook personnalisé** dans `resources/js/hooks/` si temps réel

## Instructions pour Claude

Quand l'utilisateur invoque `/component [Domaine] [NomComposant]` :

### Étape 1 — Identifier le type de fichier à créer

- **Page** : si le composant est lié à une route (`/evenements`, `/evenements/:id`...)
  → créer dans `resources/js/Pages/{Domaine}/{NomComposant}.jsx`
- **Composant** : si réutilisable dans plusieurs pages
  → créer dans `resources/js/Components/{Domaine}/{NomComposant}.jsx`
- **Hook** : si gestion de WebSocket ou état partagé complexe
  → créer dans `resources/js/hooks/use{NomComposant}.js`

### Étape 2 — Structure d'une Page Inertia

```jsx
import AppLayout from '@/Layouts/AppLayout';
import { Head } from '@inertiajs/react';

export default function {NomPage}({ /* props Inertia */ }) {
    return (
        <AppLayout>
            <Head title="{Titre de la page}" />

            <div className="max-w-7xl mx-auto px-4 py-8">
                {/* contenu */}
            </div>
        </AppLayout>
    );
}
```

**Pour les pages publiques QR code (sans auth) :**
```jsx
import PublicLayout from '@/Layouts/PublicLayout';
import { Head } from '@inertiajs/react';

export default function {NomPage}({ evenement, token }) {
    return (
        <PublicLayout evenement={evenement}>
            <Head title={evenement.nom} />
            {/* contenu sans navigation connectée */}
        </PublicLayout>
    );
}
```

### Étape 3 — Structure d'un Composant réutilisable

```jsx
import { useState } from 'react';

/**
 * {Description du composant}
 *
 * @param {Object} props
 * @param {string} props.prop1 - description
 * @param {Function} props.onAction - callback
 */
export default function {NomComposant}({ prop1, onAction }) {
    const [state, setState] = useState(null);

    return (
        <div className="">
            {/* contenu */}
        </div>
    );
}
```

### Étape 4 — Hook WebSocket temps réel (Reverb)

Pour les scores et classements en temps réel :

```js
import { useEffect, useState } from 'react';
import Echo from 'laravel-echo';

/**
 * Écoute les mises à jour en temps réel pour un match ou un événement.
 *
 * @param {string} channel - nom du channel Reverb (ex: 'match.{id}')
 * @param {string} event - nom de l'event Laravel (ex: 'ScoreUpdated')
 * @param {*} initialData - données initiales (props Inertia)
 */
export function use{NomHook}(channel, event, initialData) {
    const [data, setData] = useState(initialData);

    useEffect(() => {
        const echo = window.Echo;

        echo.channel(channel)
            .listen(event, (payload) => {
                setData(payload);
            });

        return () => {
            echo.leaveChannel(channel);
        };
    }, [channel, event]);

    return data;
}
```

---

## Conventions Tailwind Z-Trib

Utiliser les classes Tailwind cohérentes avec l'identité visuelle Z-Trib :

```jsx
// Couleurs principales
'bg-indigo-600'   // primaire (boutons, liens actifs)
'bg-emerald-500'  // succès, scores positifs
'bg-amber-400'    // avertissements, à venir
'bg-red-500'      // erreurs, forfaits

// Cartes
'bg-white rounded-xl shadow-sm border border-gray-100 p-6'

// Badges de statut événement
'px-2.5 py-0.5 rounded-full text-xs font-semibold'

// Bouton primaire
'bg-indigo-600 hover:bg-indigo-700 text-white font-medium px-4 py-2 rounded-lg transition'

// Bouton outline
'border border-indigo-600 text-indigo-600 hover:bg-indigo-50 font-medium px-4 py-2 rounded-lg transition'
```

---

## Composants existants à réutiliser

Avant de créer un nouveau composant, vérifier si l'un de ces composants existants convient :

| Composant                                    | Usage                                         |
|----------------------------------------------|-----------------------------------------------|
| `Components/Match/MatchCard.jsx`             | Afficher un match (live, terminé, à venir)    |
| `Components/Match/ScoreInput.jsx`            | Saisir un score (adapté au type de sport)     |
| `Components/Classement/TableauClassement.jsx`| Tableau de classement filtrable               |
| `Components/Evenement/EventWizard.jsx`       | Wizard création d'événement multi-étapes      |
| `Components/UI/`                             | Boutons, modals, badges, toasts               |

---

## Gestion des props Inertia

Les props sont typées côté PHP dans le Controller :

```php
// Controller
return Inertia::render('Evenement/Show', [
    'evenement' => EvenementResource::make($evenement),
    'matchs'    => MatchResource::collection($matchs),
    'canEdit'   => $request->user()->can('update', $evenement),
]);
```

Côté React, les recevoir directement en props de la Page :

```jsx
export default function Show({ evenement, matchs, canEdit }) {
    // ...
}
```

---

## Gestion des formulaires Inertia

```jsx
import { useForm } from '@inertiajs/react';

export default function CreateForm() {
    const { data, setData, post, processing, errors } = useForm({
        nom: '',
        type: 'tribu',
        sport_id: '',
    });

    const submit = (e) => {
        e.preventDefault();
        post(route('evenements.store'));
    };

    return (
        <form onSubmit={submit}>
            <input
                value={data.nom}
                onChange={e => setData('nom', e.target.value)}
            />
            {errors.nom && <p className="text-red-500 text-sm">{errors.nom}</p>}

            <button type="submit" disabled={processing}>
                {processing ? 'Enregistrement…' : 'Créer'}
            </button>
        </form>
    );
}
```

---

## Exemples d'usage

```
/component Evenement EventCard
```
→ `resources/js/Components/Evenement/EventCard.jsx`

```
/component Match Show
```
→ `resources/js/Pages/Match/Show.jsx` (page Inertia)

```
/component Public Resultats
```
→ `resources/js/Pages/Public/Resultats.jsx` (avec PublicLayout)

```
/component hooks useScoreTempsReel
```
→ `resources/js/hooks/useScoreTempsReel.js` (hook Reverb)
