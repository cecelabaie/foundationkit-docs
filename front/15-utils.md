# Utils

Ce document présente les fonctions utilitaires disponibles dans l'application.

## Manipulation de données

### hexToRgba

Convertit une couleur hexadécimale en format RGBA avec une opacité configurable. Cette fonction est essentielle pour :
- Le système de thème, particulièrement pour les boutons de type "link" qui nécessitent une transparence
- Le composant `ThemeSampleContext` pour appliquer l'opacité aux couleurs du thème
- Le composant `RhfInputColorPicker` qui permet de sélectionner des couleurs avec transparence

```tsx
import { hexToRgba } from '@/utils/color';

// Utilisation basique (opacité par défaut à 0.2)
const rgbaColor = hexToRgba('#0d275d'); // 'rgba(13, 39, 93, 0.2)'

// Avec opacité personnalisée
const buttonBg = hexToRgba('#0d275d', 0.5); // 'rgba(13, 39, 93, 0.5)'
```

### normalizeDate

Normalise une date en fixant l'heure à midi pour éviter les problèmes de fuseaux horaires. Cette fonction est cruciale pour les composants de calendrier et de date, car elle :
- Évite les problèmes de décalage de date lors des conversions entre UTC et heure locale
- Assure que les dates sélectionnées dans les calendriers sont cohérentes
- Est utilisée dans les composants `RhfDateInput` et `RhfCalendarInput` pour stabiliser les dates

```tsx
import normalizeDate from '@/utils/date';

// Crée une date normalisée (année, mois, jour)
// Note: les mois commencent à 0 (janvier = 0)
const date = normalizeDate(2023, 0, 15); // 15 janvier 2023 à 12:00:00
```

### cn (Utility de classes CSS)

Fonction provenant de shadcn/ui qui combine et optimise les classes CSS avec Tailwind, en éliminant les conflits. Elle utilise `clsx` et `tailwind-merge` pour :
- Fusionner des classes conditionnelles
- Résoudre les conflits de classes Tailwind (la dernière classe l'emporte)
- Simplifier la gestion des classes dans les composants React

```tsx
import { cn } from '@/lib/utils';

// Combine des classes avec priorité pour celles à droite
const className = cn(
  'text-red-500',           // Classe de base
  isActive && 'font-bold',  // Conditionnelle
  className                 // Classes passées en props (prioritaires)
);
```

## Gestion des erreurs

### makeValidationErrorResponse

Crée une réponse d'erreur de validation au format attendu par l'API. Cette fonction est particulièrement importante pour :
- Transformer les erreurs côté client en format compatible avec les erreurs de validation du backend
- Permettre l'affichage des erreurs de validation sur les champs de formulaire spécifiques
- Gérer les cas où l'API renvoie une erreur générique (401, 500) mais qu'on souhaite l'afficher sur des champs spécifiques

```tsx
import { makeValidationErrorResponse, mapFieldsErrors } from '@/utils/errors';

// Création d'une erreur de validation pour des champs de formulaire
const validationError = makeValidationErrorResponse(
  400,                    // Code HTTP
  'Validation échouée',   // Message
  'Bad Request',          // Type d'erreur
  [
    {
      field: 'email',     // Nom du champ concerné
      message: 'Email invalide'
    },
    {
      field: 'password',
      message: 'Mot de passe trop court'
    }
  ]
);

// Utilisation avec React Hook Form via mapFieldsErrors
mapFieldsErrors(setError, validationError);
```

### parseAxiosError

Parse une erreur Axios et la convertit en type d'erreur spécifique pour un traitement typé. Fournit une structure d'erreur cohérente même si la réponse est incomplète.

**Note** : Dans la plupart des cas, utilisez `handleServerErrors` (formulaires) ou `displayError` (hors formulaire) plutôt que `parseAxiosError` directement. Ces fonctions l'utilisent en interne.

```tsx
import { parseAxiosError } from '@/utils/errors';

onError: (error) => {
  const errData = parseAxiosError<ValidationExceptionResponseDTO | UnauthorizedResponseDTO>(
    error,
    'Erreur lors de la requête'
  );
  // errData.statusCode, errData.message, errData.violations...
}
```

### handleServerErrors

Fonction centrale pour la gestion des erreurs dans les formulaires React Hook Form. Dispatche automatiquement les erreurs selon leur type :

- **400** → erreurs de champ via `mapFieldsErrors` (affichage sous chaque input)
- **401** → `root` error uniquement si `includeUnauthorizedAlert: true` + callback `onUnauthorized` optionnel (sinon silencieux car le `QueryClient` gère le refresh)
- **Autres** → `root` error (affiché dans l'`Alert` du formulaire)

```tsx
import { handleServerErrors } from '@/utils/errors';

const { setError } = useForm();
const { mutate } = useSomeControllerMutation();

mutate(
  { data },
  {
    onSuccess: () => { /* ... */ },
    onError: (error) => {
      handleServerErrors(error, setError);
    },
  }
);
```

Avec options :

```tsx
// Cas où on veut afficher l'erreur 401 (ex: formulaire de vérification de compte)
handleServerErrors(error, setError, {
  includeUnauthorizedAlert: true,
  onUnauthorized: () => router.push(APP_PATHS.LOGIN),
});
```

### displayError

Affiche une erreur dans un toast Sonner. Ignore les 401 par défaut car le `QueryClient` se charge du refresh et du retry automatiquement. A utiliser hors formulaire (guards, callbacks sans formulaire).

```tsx
import { displayError } from '@/utils/errors';

onError: (error) => {
  displayError(error);
}

// Pour les routes dans ROUTES_WITHOUT_RETRY (ex: verifyAccountGuard)
// où le QueryClient ne gère pas le retry, on inclut le 401 :
onError: (error) => {
  displayError(error, { includeUnauthorizedError: true });
}
```

### mapFieldsErrors

Mappe les violations de validation d'une réponse 400 sur les champs d'un formulaire React Hook Form. Appelée automatiquement par `handleServerErrors` sur les erreurs 400, mais peut aussi être utilisée directement.

```tsx
import { mapFieldsErrors } from '@/utils/errors';

const errData = parseAxiosError<ValidationExceptionResponseDTO>(error, 'Erreur');
mapFieldsErrors(setError, errData);
// Applique violation.message sur chaque violation.field dans le formulaire
// Si aucune violation, applique le message sur root
```

## Métadonnées (SEO)

Toutes les métadonnées sont centralisées dans `src/utils/metadata.ts`. Ce fichier exporte la configuration du site, les metadata de chaque page et les fonctions utilitaires.

### getPageMetadata

Fonction principale à utiliser dans chaque `page.tsx`. Récupère les metadata depuis `PAGES_METADATA`. Lève une erreur si la clé est inconnue.

```tsx
import { getPageMetadata } from '@/utils/metadata';

export const metadata = getPageMetadata('login');
```

> Une règle ESLint bloquante vérifie que chaque `page.tsx` sous `src/app/` appelle `getPageMetadata`. Le build échoue si absent.

### SITE_CONFIG

Constante exportée contenant les valeurs globales du site (nom, titre, description, locale, ogImage, authors).

### generateLayoutMetadata

Réservé au `layout.tsx` racine. Génère les métadonnées de base sans URL canonique.

```tsx
import { generateLayoutMetadata } from '@/utils/metadata';

// Dans layout.tsx uniquement
export const metadata: Metadata = generateLayoutMetadata();
```

Pour plus d'informations, consultez le guide [17-seo-metadata.md](./17-seo-metadata.md).