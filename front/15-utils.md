# Utils

Ce document présente les fonctions utilitaires disponibles dans l'application.

## Manipulation de données

### getAvatar

Fonction qui retourne le chemin vers l'avatar par défaut correspondant au genre de l'utilisateur. Utilisée principalement dans :
- Le popover utilisateur du header (desktop et mobile)
- La page de profil pour l'affichage par défaut quand aucune image n'est téléchargée

```tsx
import { getAvatar } from '@/utils/avatar';
import { UserProfileDTOGender } from '@/api/generated/schemas';

// Utilisation
const avatarPath = getAvatar(UserProfileDTOGender.male); // '/assets/images/avatar/avatar-man.svg'
```

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
import { makeValidationErrorResponse } from '@/utils/errors';

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

// Utilisation avec React Hook Form
if ('violations' in validationError) {
  validationError.violations.forEach((violation) => {
    setError(violation.field as keyof FormInputs, {
      type: 'server',
      message: violation.message
    });
  });
}
```

### parseAxiosError

Parse une erreur Axios et la convertit en type d'erreur spécifique pour un traitement typé. Cette fonction :
- Extrait les données de l'erreur Axios de manière typée
- Fournit une structure d'erreur cohérente même si la réponse est incomplète
- Permet de traiter différents types d'erreurs API (validation, authentification, serveur) de manière unifiée

**Important** : Vous devez spécifier en paramètre de type générique les types d'erreurs attendus générés par Orval. Ces types sont disponibles dans le dossier `@/api/generated/schemas`.

```tsx
import { parseAxiosError } from '@/utils/errors';
import { 
  // Types d'erreurs générés par Orval
  ValidationExceptionResponseDTO,
  UnauthorizedResponseDTO,
  TooManyRequestResponseDTO,
  InternalServerErrorExceptionResponseDTO
} from '@/api/generated/schemas';

// Utilisation avec mutate (utilisé dans ce projet pour les formulaires)
mutate_login(
  { data },
  {
    onSuccess: (response) => {
      // Traitement du succès...
    },
    onError: (error) => {
      const errData = parseAxiosError<LoginErrorResponse>(error, 'Erreur');
      // Traitement des erreurs...
    }
  }
);
```

**Note** : Dans ce projet, les formulaires utilisent `mutate` avec les callbacks `onSuccess`/`onError`. Pour les cas où vous avez besoin d'attendre le résultat (ex: dans un contexte, un guard, etc.), on utilise `mutateAsync` :

```tsx
// Utilisation avec mutateAsync (pour les cas non-formulaires)
try {
  const response = await mutateAsync_login({ data });
  // Traitement du succès...
} catch (error) {
  const errData = parseAxiosError<LoginErrorResponse>(error, 'Erreur');
  // Traitement des erreurs...
}
```

## Métadonnées (SEO)

Les utilitaires de métadonnées permettent de gérer le SEO de manière cohérente dans toute l'application Next.js.

### generateLayoutMetadata

Génère les métadonnées de base pour le layout racine de l'application. Cette fonction :
- Définit les métadonnées par défaut (titre, description, mots-clés)
- Configure les langues alternatives
- Définit les règles pour les robots d'indexation
- Utilise les variables d'environnement pour les URLs

```tsx
import { generateLayoutMetadata } from '@/utils/metadata';
import { Metadata } from 'next';

// Dans layout.tsx
export const metadata: Metadata = generateLayoutMetadata();
```

### generateMetadata

Génère des métadonnées personnalisées pour une page spécifique, avec URL et balises Open Graph. Cette fonction :
- Étend les métadonnées de base avec des informations spécifiques à la page
- Gère les URLs pour éviter le contenu dupliqué
- Configure les balises Open Graph pour les partages sur les réseaux sociaux
- Permet de personnaliser les règles d'indexation par page

```tsx
import { generateMetadata } from '@/utils/metadata';
import { Metadata } from 'next';

// Dans une page.tsx
export const metadata: Metadata = generateMetadata({
  title: 'Titre de la page',
  description: 'Description détaillée de la page pour le SEO',
  canonical: '/chemin-de-page',
  ogImage: '/images/og-image.jpg',
  ogTitle: 'Titre pour les réseaux sociaux'
});
```

### generateNoIndexMetadata

Génère des métadonnées pour les pages qui ne doivent pas être indexées par les moteurs de recherche. Particulièrement utile pour :
- Les pages privées (profil, tableau de bord)
- Les pages de processus (paiement, inscription)
- Les pages temporaires ou en développement

```tsx
import { generateNoIndexMetadata } from '@/utils/metadata';
import { Metadata } from 'next';

// Dans une page privée
export const metadata: Metadata = generateNoIndexMetadata({
  title: 'Page privée',
  description: 'Cette page ne doit pas être indexée',
  canonical: '/page-privee'
});
```

### getCanonicalUrl

Convertit un chemin relatif en URL absolue, en utilisant la base URL configurée.

```tsx
import { getCanonicalUrl } from '@/utils/metadata';

// Obtenir l'URL complète
const canonicalUrl = getCanonicalUrl('/ma-page'); // https://example.com/ma-page
```

Pour plus d'informations sur l'utilisation des métadonnées, consultez le guide [17-seo-metadata.md](./17-seo-metadata.md).