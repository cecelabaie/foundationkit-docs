# Guide IA — Contribuer au Frontend

[← Retour au sommaire](./SUMMARY.md)

Ce document est destiné aux agents IA travaillant sur le Foundation Kit. Il définit un workflow structuré et reproductible pour créer, modifier ou étendre des features du frontend tout en respectant la philosophie du projet : **architecture par sections, génération Orval, formulaires typés, traductions systématiques**.

Suivre ce guide dans l'ordre garantit une implémentation cohérente avec le reste du projet.

> **Features touchant l'API :** Si la feature modifie ou ajoute des endpoints, consulter aussi le [Guide IA API](../api/16-agent-ia.md) pour garantir la cohérence backend/frontend.

> **Important — code généré automatiquement :** Le dossier `src/api/generated/` est entièrement géré par Orval. **Ne jamais modifier ces fichiers manuellement** — ils sont écrasés à chaque regénération. Tout le code frontend (hooks, types, schemas Zod) découle du spec OpenAPI de l'API.

---

## Lecture préalable obligatoire

Avant toute action, **lire les documents de référence correspondants** dans `docs/front/`.

### Documents à lire selon l'action

| Action envisagée | Documents à lire en priorité |
|-----------------|------------------------------|
| Toute action | [Pages](./07-pages.md) · [Components](./02-components.md) |
| Créer / modifier une page | [Pages](./07-pages.md) · [Layout](./08-layout.md) · [SEO](./17-seo-metadata.md) |
| Créer / modifier un formulaire | [Formulaires](./10-formulaires.md) · [Traductions](./06-traductions.md) · [Data](./11-data.md) |
| Appeler l'API | [Generation](./09-generation.md) · [Data](./11-data.md) |
| Gérer la session / l'auth | [Session](./12-session.md) · [Contexts](./13-contexts.md) |
| Ajouter des traductions | [Traductions](./06-traductions.md) |
| Modifier le thème / les couleurs | [Theme](./01-theme.md) · [Tailwind](./03-tailwind.md) |
| Ajouter un composant UI | [Components](./02-components.md) · [Shadcn](./04-shadcn.md) |
| Ajouter une entrée de navigation | [Layout](./08-layout.md) |

### Référence complète de la documentation frontend

| # | Document | Contenu clé |
|---|----------|-------------|
| 01 | [Theme](./01-theme.md) | CSS variables, light/dark, palette, next-themes |
| 02 | [Components](./02-components.md) | Liste des 26 composants UI + 13 inputs RHF |
| 03 | [Tailwind](./03-tailwind.md) | Tailwind v4, `@theme inline`, classes custom |
| 04 | [Shadcn](./04-shadcn.md) | Installation, personnalisation, conventions |
| 05 | [Sample](./05-sample.md) | Page `/sample`, theme builder |
| 06 | [Traductions](./06-traductions.md) | i18next, namespaces, Zod i18n, `useZodI18n` |
| 07 | [Pages](./07-pages.md) | App Router, guards, structure page/view/form |
| 08 | [Layout](./08-layout.md) | Layout hiérarchique, header, navigation, footer |
| 09 | [Generation](./09-generation.md) | Orval, hooks générés, types, Zod schemas |
| 10 | [Formulaires](./10-formulaires.md) | React Hook Form, Zod, inputs RHF, erreurs |
| 11 | [Data](./11-data.md) | React Query, hooks Orval, gestion des erreurs |
| 12 | [Session](./12-session.md) | AuthContext, guards, flag_session, refresh token |
| 13 | [Contexts](./13-contexts.md) | AuthContext, ToastContext, providers |
| 14 | [Hooks](./14-hooks.md) | useAppRouter, useZodI18n, useIsScreenBelowBreakpoint |
| 15 | [Utils](./15-utils.md) | parseAxiosError, cn(), metadata, avatar, date |
| 16 | [Config](./16-config.md) | Variables d'env, constantes, scripts, next.config |
| 17 | [SEO](./17-seo-metadata.md) | generateMetadata, noIndex, sitemap, og |

---

## Vue d'ensemble du workflow

```
1. Analyser & découper la feature
         ↓
2. Régénérer le code Orval (si l'API a changé)
         ↓
3. Créer / modifier la page et sa structure
         ↓
4. Implémenter les composants (view, form, sections)
         ↓
5. Ajouter les traductions
         ↓
6. Valider la checklist finale
```

---

## Étape 1 — Analyse & découpage de la feature

### 1.1 Résumer la feature

Avant toute implémentation, répondre à ces questions :
- **Que fait cette feature ?** (comportement attendu côté utilisateur)
- **Qui l'utilise ?** (connecté, déconnecté, public)
- **Est-ce qu'elle appelle l'API ?** (quels endpoints)
- **Est-ce qu'elle modifie l'état global ?** (AuthContext, ToastContext)

### 1.2 Décomposer en sous-tâches atomiques

| Élément | Action | Notes |
|---------|--------|-------|
| Page | Nouvelle / Existante | Route App Router |
| Guard | AuthGuard / UnloggedGuard / aucun | Selon le type de page |
| Sections | Nouveaux fichiers / Mise à jour | `sections/{feature}/` |
| Formulaire | Nouveau / Existant | React Hook Form + Zod Orval |
| Appels API | Hooks Orval à utiliser | `src/api/generated/` |
| Traductions | Nouveau namespace / Nouveau fichier | `public/locales/` |
| Navigation | Ajouter dans `nav.tsx` | Si nouvelle page accessible |
| SEO | `generateMetadata` ou `generateNoIndexMetadata` | Selon indexation |

### 1.3 Lire le code existant

Si la feature touche une section ou page existante, **lire les fichiers concernés avant d'écrire quoi que ce soit** :

```
src/
├── app/{route}/
│   └── page.tsx                    ← metadata, import de la view
├── sections/{feature}/
│   ├── view/{feature}-view.tsx     ← structure visuelle
│   └── {feature}-form.tsx          ← logique formulaire
```

---

## Étape 2 — Régénération Orval (si nécessaire)

### 2.1 Quand régénérer

Régénérer dès que :
- Un nouvel endpoint a été ajouté à l'API
- Un DTO existant a été modifié (champs, types, validations)
- Un champ a changé de statut requis/optionnel dans un DTO
- Un nouveau domaine API a été créé

### 2.2 Configuration Orval

Orval est configuré via `orval.config.ts` à la racine du frontend. La variable d'environnement `SWAGGER_JSON` (dans `.env`, `.env.local` ou `.env.production`) pointe vers le spec OpenAPI de l'API (ex: URL de l'API + `/api-json` ou chemin local).

### 2.3 Lancer la génération

> **Action utilisateur requise**
>
> ```bash
> # Sur Windows
> npm run gen:win
>
> # Sur Linux/Mac
> npm run gen:linux
> ```
>
> La commande vide `src/api/generated/`, relance Orval, corrige l'ESLint et recrée le `.gitignore`.

### 2.4 Ce qui est généré

```
src/api/generated/
├── {domain}/                ← domaine = tag Swagger (auth, user, reset-password, media...)
│   └── {domain}.ts          ← hooks React Query (useAuthControllerLogin, useUserControllerUpdate...)
├── schemas/
│   └── *.ts                 ← types TypeScript (AuthLoginBodyDTO, UserProfileDTO...)
└── zod/
    └── {domain}/
        └── {domain}.ts      ← schemas Zod (authControllerLoginBody, userControllerUpdateBody...)
```

Le nom du domaine correspond au tag `@ApiTags()` du controller côté API. Exemple : controller tagué `auth` → `useAuthControllerLogin` dans `src/api/generated/auth/auth.ts`.

**Règle absolue :** ne jamais modifier ces fichiers. Toute correction passe par l'API puis une regénération.

Voir aussi : [Generation](./09-generation.md)

---

## Étape 3 — Structure de la page

### 3.1 Architecture en 3 couches

Chaque feature suit ce découpage :

```
src/app/{route}/
└── page.tsx                          ← (1) metadata + import view

src/sections/{feature}/
├── view/
│   └── {feature}-view.tsx            ← (2) structure visuelle, layout
└── {feature}-form.tsx                ← (3) logique formulaire (si formulaire)
```

**Couche 1 — `page.tsx` :** fine, uniquement la metadata et l'import de la view.

```tsx
import { generateNoIndexMetadata } from '@/utils/metadata';
import FeatureView from '@/sections/feature/view/feature-view';

export const metadata = generateNoIndexMetadata({
  title: 'Titre de la page',
  description: 'Description SEO',
  canonical: '/feature',
});

export default function FeaturePage() {
  return <FeatureView />;
}
```

**Couche 2 — `*-view.tsx` :** structure visuelle (Card, titre, sous-titre, mise en page). Pour les pages avec illustration (login, register, profile, etc.), s'inspirer des composants existants dans `src/assets/illustrations/`.

> **`'use client'` obligatoire** sur tous les fichiers `*-view.tsx` et `*-form.tsx` — ils utilisent des hooks React (useTranslation, useIsScreenBelowBreakpoint, etc.).

```tsx
'use client';

import FeatureForm from '../feature-form';

export default function FeatureView() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <Card className="w-full max-w-md">
        <CardHeader>
          <CardTitle>Titre</CardTitle>
        </CardHeader>
        <CardContent>
          <FeatureForm />
        </CardContent>
      </Card>
    </div>
  );
}
```

**Couche 3 — `*-form.tsx` :** logique React Hook Form, appel API, gestion des erreurs. Si la page n'a pas de formulaire (ex : page de lecture seule), la view contient directement le contenu — pas de fichier `*-form.tsx`.

### 3.2 Routing et guards

**Pages protégées (connecté requis) :** placer dans `src/app/(logged)/`

```
src/app/(logged)/
└── {feature}/
    └── page.tsx
```

Le layout `(logged)/layout.tsx` applique automatiquement `<AuthGuard requiredRoles={['user']}>`.

**Pages réservées aux non-connectés :** placer dans `src/app/(unlogged)/`

```
src/app/(unlogged)/
└── {feature}/
    └── page.tsx
```

Le layout `(unlogged)/layout.tsx` applique automatiquement `<UnloggedGuard>`.

**Pages publiques :** à la racine de `src/app/`.

**Page protégée par un guard spécial (ex : ForgotPasswordGuard) :** placer dans le groupe et layout approprié. La page `/forgot-password/new-password` utilise `ForgotPasswordGuard` via le layout `(unlogged)/(reset-password)/forgot-password/new-password/layout.tsx` — s'en inspirer pour des pages similaires (ex : validation de token par email).

### 3.3 Metadata SEO

| Type de page | Fonction | Indexée |
|---|---|---|
| Page publique (landing, articles) | `generateMetadata` | ✅ |
| Page auth, privée, outil | `generateNoIndexMetadata` | ❌ |
| Root layout uniquement | `generateLayoutMetadata` | — |

```tsx
// Page publique indexée — utiliser l'alias generateMeta pour éviter le conflit avec generateMetadata de Next.js
import { generateMetadata as generateMeta } from '@/utils/metadata';

export const metadata = generateMeta({
  title: 'Mon titre',
  description: 'Description',
  canonical: '/ma-page',
  ogImage: '/images/og.jpg', // optionnel
});

// Page privée / auth
import { generateNoIndexMetadata } from '@/utils/metadata';

export const metadata = generateNoIndexMetadata({
  title: 'Mon titre',
  description: 'Description',
  canonical: '/ma-page',
});
```

> **Si la page est accessible via le header**, ajouter le chemin dans `APP_PATHS` (`src/constants/constants.ts`) et utiliser cette constante dans `nav.tsx` — éviter les chemins en dur (certaines pages existantes comme `/private` n'ont pas encore été migrées vers `APP_PATHS`, c'est la cible à viser).

Voir aussi : [Pages](./07-pages.md), [SEO](./17-seo-metadata.md)

---

## Étape 4 — Implémentation des composants

### 4a. Formulaires

**Pattern complet d'un formulaire :**

```tsx
'use client';

import { AlertCircleIcon } from 'lucide-react';
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm, UseFormSetError } from 'react-hook-form';
import { useTranslation } from 'react-i18next';
import { z } from 'zod';

// 1. Schema Zod généré par Orval (préféré) ou défini manuellement
import { featureControllerActionBody } from '@/api/generated/zod/{domain}/{domain}';
// 2. Hook React Query généré par Orval
import { useFeatureControllerAction } from '@/api/generated/{domain}/{domain}';
// 3. Types d'erreur générés par Orval
import {
  InternalServerErrorExceptionResponseDTO,
  ValidationExceptionResponseDTO,
} from '@/api/generated/schemas';
// 4. Composants form
import { Form } from '@/components/form/form';
import { RhfTextInput, RhfPasswordInput } from '@/components/form/inputs';
import { LoadingButton } from '@/components/ui/loading-button';
import { Alert, AlertTitle } from '@/components/ui/alert';
// 5. Contextes et hooks
import { useAuth } from '@/contexts/AuthContext';
import { useToast } from '@/contexts/ToastContext';
import { useZodI18n } from '@/hooks/useZodI18n';
import { useAppRouter } from '@/hooks/useAppRouter';
import { parseAxiosError } from '@/utils/errors';
import { APP_PATHS } from '@/constants/constants';

const zSchema = featureControllerActionBody;
type FormInputs = z.infer<typeof zSchema>;

// Union des types d'erreur possibles pour cet endpoint — adapter selon les codes HTTP retournés
type FeatureErrorResponse =
  | ValidationExceptionResponseDTO
  | InternalServerErrorExceptionResponseDTO;

export default function FeatureForm() {
  const { t } = useTranslation(['{namespace}', 'common']);
  const { showSuccessToast, showErrorToast } = useToast();
  const router = useAppRouter();

  const { mutate, isPending } = useFeatureControllerAction();

  const methods = useForm<FormInputs>({
    resolver: zodResolver(zSchema),
    defaultValues: { field: '' },
  });

  useZodI18n(methods); // Obligatoire : re-valide les erreurs au changement de langue

  const { handleSubmit, setError, formState: { errors, isSubmitting } } = methods;

  const onSubmit = handleSubmit((input) => {
    mutate(
      { data: input },
      {
        onSuccess: () => {
          showSuccessToast(t('{namespace}:form.success', { defaultValue: '...' }));
          router.push(APP_PATHS.HOME);
        },
        onError: (error) => {
          const errData = parseAxiosError<FeatureErrorResponse>(
            error,
            t('common:error.500', { defaultValue: "Une erreur est survenue." })
          );
          if (errData.statusCode === 400) {
            mapFieldsErrors(setError, errData);
          } else {
            setError('root', { type: 'server', message: errData.message });
          }
        },
      }
    );
  });

  return (
    <Form methods={methods} onSubmit={onSubmit}>
      {/* Erreur globale — utiliser Alert pour une meilleure UX */}
      {errors.root && (
        <Alert variant="destructive">
          <AlertCircleIcon />
          <AlertTitle>{errors.root.message}</AlertTitle>
        </Alert>
      )}

      <RhfTextInput
        control={methods.control}
        name="field"
        label={t('form.field.label')}
        placeholder={t('form.field.placeholder')}
        required={!zSchema.shape.field.safeParse('').success}
      />

      <LoadingButton type="submit" loading={isSubmitting || isPending}>
        {t('form.submit')}
      </LoadingButton>
    </Form>
  );
}

// Helper local — mapper les violations serveur sur les champs du formulaire
function mapFieldsErrors(
  setError: UseFormSetError<FormInputs>,
  error: FeatureErrorResponse,
) {
  if ('violations' in error) {
    error.violations.forEach((v) => {
      setError(v.field as keyof FormInputs, { type: 'server', message: v.message });
    });
    return;
  }
}
```

**Règles importantes :**
- Toujours utiliser le schema Zod généré par Orval (`featureControllerActionBody`) — il est en sync avec les validations de l'API
- `useZodI18n(methods)` est **obligatoire** dans chaque formulaire avec Zod
- Pour la prop `required` sur les inputs, la dériver du schéma Zod plutôt que de la coder en dur : `required={!zSchema.shape.field.safeParse('').success}`. Ce pattern est fiable pour les champs `string` ; pour les champs d'un autre type (boolean, number…), `safeParse('')` échoue systématiquement — utiliser la valeur vide du type concerné (ex: `safeParse(false)` pour un boolean).
- Le message affiché en `onSuccess` vient d'une clé de traduction frontend, pas de `response.message`
- Gestion du 401 dans `onError` — deux cas :
  - **Route protégée (auth requise)** : ajouter `if (errData.statusCode === 401) return;` en premier dans `onError` — le 401 est géré silencieusement par `MutationCache` (refresh + retry automatique)
  - **Route non protégée avec 401 métier** (ex : token de réinitialisation expiré) : traiter explicitement avec redirection ou toast d'erreur
- Utiliser `mutate` (avec callbacks) pour les formulaires, `mutateAsync` uniquement si un `await` est nécessaire
- Utiliser `useAppRouter` à la place de `useRouter` (barre de progression)

**Cas particulier — formulaire de connexion :**

Un formulaire de login doit, après succès, mettre à jour la session :
1. `localStorage.setItem(FLAG_SESSION.KEY, FLAG_SESSION.VALID)` — marquer la session comme valide
2. `fetchAndSetUser()` — recharger les données utilisateur depuis l'API

```tsx
import { FLAG_SESSION } from '@/constants/constants';
import { useAuth } from '@/contexts/AuthContext';

const { fetchAndSetUser } = useAuth();

mutate(
  { data: input },
  {
    onSuccess: () => {
      localStorage.setItem(FLAG_SESSION.KEY, FLAG_SESSION.VALID);
      fetchAndSetUser();
      // Puis router.push() si redirection nécessaire
    },
    // ...
  }
);
```

### 4b. Inputs disponibles

| Composant | Cas d'usage |
|-----------|------------|
| `RhfTextInput` | Texte, email (`type="email"`) |
| `RhfPasswordInput` | Mot de passe (toggle visibilité inclus) |
| `RhfTextAreaInput` | Texte multi-lignes |
| `RhfNumberInput` | Valeurs numériques |
| `RhfSelectInput` | Liste déroulante |
| `RhfComboboxInput` | Select avec recherche |
| `RhfRadioInput` | Boutons radio |
| `RhfCheckboxInput` | Case à cocher |
| `RhfSwitchInput` | Toggle on/off |
| `RhfDateInput` | Date avec calendrier |
| `RhfCalendarInput` | Calendrier complet |
| `RhfFileInput` | Upload de fichier |
| `RhfInputColorPicker` | Sélecteur de couleur |

Tous partagent les props : `control`, `name`, `label`, `placeholder`, `required`, `disabled`, `className`.

**`RhfFileInput` — props spécifiques :**

| Prop | Type | Description |
|------|------|-------------|
| `preview` | `string` | URL de prévisualisation de l'image actuelle |
| `defaultImage` | `string` | Image affichée si aucune valeur |
| `previewWidth` | `number` | Largeur de la prévisualisation en px |
| `previewHeight` | `number` | Hauteur de la prévisualisation en px |
| `onDelete` | `() => void` | Callback déclenché au clic sur "Supprimer" |

Le fichier sélectionné est passé comme objet `File` directement à la mutation — la conversion en `FormData` est gérée en interne par le mutator Orval.

### 4c. Appels API sans formulaire

Pour les appels API hors formulaire (ex: bouton d'action, chargement de données) :

```tsx
'use client';

// Mutation (POST/PATCH/DELETE)
import { useFeatureControllerAction } from '@/api/generated/{domain}/{domain}';

const { mutate, isPending } = useFeatureControllerAction();

const handleAction = () => {
  mutate(
    { data: payload },
    {
      onSuccess: () => { showSuccessToast(t('{namespace}:form.success', { defaultValue: '...' })); },
      onError: (error) => {
        const errData = parseAxiosError<ErrorType>(error, t('common:error.500', { defaultValue: 'Une erreur est survenue.' }));
        showErrorToast(errData.message);
      },
    }
  );
};

// Query (GET)
import { useFeatureControllerGetData } from '@/api/generated/{domain}/{domain}';

const { data, isLoading, isError } = useFeatureControllerGetData();
```

### 4d. Accéder à l'utilisateur connecté

Dans n'importe quel composant client, l'utilisateur est disponible via `useAuth()` :

```tsx
'use client';

import { useAuth } from '@/contexts/AuthContext';

export default function MyComponent() {
  const { user, isLoading } = useAuth();

  if (isLoading) return <PageLoader />;
  if (!user) return null;

  return <p>Bonjour {user.firstName}</p>;
}
```

`user` est de type `UserProfileDTO` (généré par Orval). Champs disponibles : `id`, `firstName`, `lastName`, `email`, `username`, `role`, `gender`, `dateOfBirth`, `notificationEmail`, `profilePicture`, etc.

**Toutes les valeurs retournées par `useAuth()` :**

| Valeur | Type | Usage |
|--------|------|-------|
| `user` | `UserProfileDTO \| undefined` | Données de l'utilisateur connecté |
| `isLoading` | `boolean` | En cours de vérification du token |
| `isFirstLoad` | `boolean` | Premier chargement de l'app (avant que l'auth soit connue) |
| `disconnect` | `() => void` | Déconnecte l'utilisateur, vide le localStorage, redirige vers `/` |
| `fetchAndSetUser` | `() => Promise<...>` | Recharge les données utilisateur depuis l'API et met à jour le contexte — à appeler après une mise à jour du profil |
| `checkToken` | `(disconnectIfFailed: boolean) => Promise<void>` | Vérifie la validité du token (utilisé par les guards) |
| `tryRefreshToken` | `(disconnectIfFailed: boolean) => Promise<...>` | Tente un refresh du token (utilisé en interne) |

**`onSettled` — même comportement succès et erreur :**

Quand l'action doit produire le même effet qu'elle réussisse ou échoue (ex : logout) :

```tsx
mutate(undefined, {
  onSettled: () => {
    disconnect(); // appelé dans tous les cas
  },
});
```

**Validation côté client avant l'appel API :**

Pour valider des champs que Zod ne peut pas vérifier (ex : confirmation de mot de passe), construire les violations manuellement avec `makeValidationErrorResponse` puis passer à `mapFieldsErrors` :

```tsx
import { makeValidationErrorResponse } from '@/utils/errors';

const onSubmit = handleSubmit((input) => {
  const clientViolations = [];

  if (input.password !== input.confirmPassword) {
    clientViolations.push({ field: 'confirmPassword', message: t('form.confirm-password.mismatch') });
  }

  if (clientViolations.length > 0) {
    mapFieldsErrors(setError, makeValidationErrorResponse(400, 'Validation failed', 'Bad Request', clientViolations));
    return;
  }

  // Continuer avec l'appel API...
  mutate({ data: input }, { ... });
});
```

### 4e. Toasts et notifications

```tsx
import { useToast } from '@/contexts/ToastContext';

const { showSuccessToast, showErrorToast, showWarningToast, showInfoToast } = useToast();

showSuccessToast('Opération réussie');
showErrorToast('Une erreur est survenue');
showWarningToast('Attention');
showInfoToast('Information');
```

### 4f. Navigation

**Toujours utiliser `useAppRouter`** à la place de `useRouter` — il intègre la barre de progression. **Utiliser `APP_PATHS`** pour tous les chemins (y compris les liens `<a href>` et `Link`) — éviter les chemins en dur comme `href="/forgot-password"`.

```tsx
import { useAppRouter } from '@/hooks/useAppRouter';
import { APP_PATHS } from '@/constants/constants';

const router = useAppRouter();
router.push(APP_PATHS.PROFILE);

// Pour un lien
<a href={APP_PATHS.FORGOT_PASSWORD}>Mot de passe oublié</a>
```

### 4g. Responsive

Pour conditionner le rendu (pas seulement le CSS) selon la taille d'écran :

```tsx
import { useIsScreenBelowBreakpoint } from '@/hooks/useIsScreenBelowBreakpoint';

const isMobile = useIsScreenBelowBreakpoint('lg'); // < 1024px
```

Breakpoints : `sm` (640px), `md` (768px), `lg` (1024px), `xl` (1280px), `2xl` (1536px).

### 4h. Classes CSS conditionnelles

```tsx
import { cn } from '@/lib/utils';

<div className={cn(
  'base-class',
  isActive && 'active-class',
  className, // prop externe
)} />
```

Voir aussi : [Formulaires](./10-formulaires.md), [Components](./02-components.md), [Data](./11-data.md), [Hooks](./14-hooks.md)

---

## Étape 5 — Traductions

### 5.1 Choisir le namespace

Chaque page/feature a son propre namespace, correspondant à un fichier JSON dans `public/locales/`.

| Namespace existant | Usage |
|-------------------|-------|
| `login` | Page de connexion |
| `logout` | Page de déconnexion |
| `register` | Page d'inscription |
| `profile` | Page profil |
| `update-password` | Changement de mot de passe |
| `forgot-password` | Page demande de réinitialisation |
| `forgot-password-guard` | Messages du guard de réinitialisation |
| `forgot-password-new-password` | Page nouveau mot de passe |
| `home` | Page d'accueil |
| `verify-account` | Page de vérification de compte (email) |
| `private` | Page protégée de démonstration |
| `sample` | Page de preview du kit |
| `not-found` | Page 404 |
| `common` | Éléments partagés (navigation, boutons génériques, erreurs) |
| `form` | Labels et messages de formulaire génériques |
| `header` / `footer` | Layout global |
| `zod` | Messages d'erreur Zod traduits |

Pour une nouvelle feature : créer un nouveau namespace (ex: `notifications`).

### 5.2 Créer les fichiers de traduction

```
public/locales/
├── fr/
│   └── {namespace}.json    ← à créer
└── en/
    └── {namespace}.json    ← à créer
```

Structure recommandée d'un namespace :

```json
{
  "title": "Titre de la page",
  "description": "Description de la page",
  "form": {
    "field": {
      "label": "Libellé du champ",
      "placeholder": "Placeholder"
    },
    "submit": "Envoyer"
  },
  "success": "Opération réussie",
  "error": {
    "default": "Une erreur est survenue"
  }
}
```

**Toujours créer les deux langues** (`fr` et `en`) en même temps.

**Flow obligatoire à chaque fois qu'une clé `t()` est ajoutée ou modifiée dans le code :**

1. Identifier le namespace utilisé (`t('namespace:ma.cle', ...)`)
2. Ajouter la clé dans `public/locales/fr/{namespace}.json`
3. Ajouter la clé dans `public/locales/en/{namespace}.json`
4. Vérifier que la `defaultValue` dans le code correspond à la valeur FR du fichier JSON (elles doivent être identiques)
5. Si le namespace est nouveau, l'ajouter dans `src/i18n.ts` (voir §5.4)
6. Supprimer les clés devenues inutilisées (clés mortes) dans les deux fichiers JSON
7. Lancer le script d'audit depuis le dossier `front/` et corriger tous les problèmes remontés :

> **Action utilisateur requise**
>
> ```bash
> node check/translation.mjs
> ```

### 5.3 Utiliser les traductions

Utiliser le namespace unique quand le composant n'accède qu'à ses propres clés. Ajouter `'common'` dans l'array dès que le composant appelle `t('common:...')` (message d'erreur 500, libellés partagés, etc.) — c'est le cas de la quasi-totalité des formulaires qui ont un `onError`.

```tsx
import { useTranslation } from 'react-i18next';

// Namespace unique — pas d'accès à 'common'
const { t } = useTranslation('{namespace}');

// Avec 'common' — dès que le composant utilise t('common:...')
const { t } = useTranslation(['{namespace}', 'common']);

// Utilisation — toujours préfixer avec le namespace
t('{namespace}:form.field.label', { defaultValue: '...' })
t('common:error.500', { defaultValue: '...' })
```

### 5.4 Déclarer le namespace dans i18n

Si c'est un **nouveau namespace**, l'ajouter dans la liste `ns` de `src/i18n.ts` :

```typescript
ns: [
  'header', 'footer', 'login', ...,
  '{nouveau-namespace}', // ← ajouter ici
],
```

Voir aussi : [Traductions](./06-traductions.md)

---

## Étape 6 — Ajouter la navigation (si nouvelle page)

Si la nouvelle page doit apparaître dans le header :

### 6.1 Ajouter le chemin dans les constantes

Dans `src/constants/constants.ts` :

```typescript
export const APP_PATHS = {
  // ... chemins existants
  MA_FEATURE: '/ma-feature',
};
```

### 6.2 Ajouter dans la navigation

Dans `src/constants/header/nav.tsx`, ajouter l'entrée dans le tableau approprié selon le type de page :

| Tableau | Usage |
|---------|-------|
| `usersConnectedPages` | Pages accessibles aux utilisateurs connectés (profil, déconnexion, etc.) |
| `usersNotConnectedPages` | Pages réservées aux non-connectés (login, register, forgot-password) |
| `publicPages` | Pages accessibles à tous (sample, 404, etc.) |

```tsx
import { monIcon } from '@iconify-icons/solar/...';

// Dans usersConnectedPages, usersNotConnectedPages, ou publicPages
{
  title: 'nav.connected-pages.ma-feature', // clé i18n (namespace: 'header')
  path: APP_PATHS.MA_FEATURE,
  needToBeConnected: false,
  needToBeDisconnected: false,
  inLeftSide: true,
  icon: <Icon icon={monIcon} width={24} height={24} />,
  defaultValue: 'Ma Feature',
}
```

### 6.3 Ajouter la clé i18n de navigation

Les titres des entrées de navigation sont traduits via `useTranslation('header')` dans `LeftSideHeader`. Ajouter la clé dans `public/locales/fr/header.json` et `public/locales/en/header.json` :

```json
{
  "nav": {
    "connected-pages": {
      "ma-feature": "Ma Feature"
    }
  }
}
```

---

## Checklist de validation

### 1. Analyse
- [ ] Feature résumée (comportement, qui l'utilise, endpoints appelés)
- [ ] Pages / sections / formulaires identifiés
- [ ] Hooks Orval disponibles identifiés (ou regénération nécessaire)
- [ ] Code existant des sections impactées lu avant d'écrire

### 2. Génération Orval
- [ ] Orval regénéré si l'API a changé (`npm run gen:win` / `npm run gen:linux`)
- [ ] Types et hooks attendus présents dans `src/api/generated/`
- [ ] Prop `required` de chaque input vérifiée après regénération (`!zSchema.shape.field.safeParse('').success`)

### 3. Pages et routing
- [ ] Page placée dans le bon groupe de routes (`(logged)`, `(unlogged)`, ou racine)
- [ ] Structure en 3 couches respectée (`page.tsx` → `*-view.tsx` → `*-form.tsx`)
- [ ] `generateMetadata` ou `generateNoIndexMetadata` utilisé selon l'indexation
- [ ] `canonical` correct dans la metadata

### 4. Implémentation
- [ ] Schema Zod Orval utilisé comme source de vérité (`featureControllerActionBody`)
- [ ] `useZodI18n(methods)` présent dans chaque formulaire
- [ ] `useAppRouter` utilisé à la place de `useRouter`
- [ ] Inputs RHF utilisés (pas d'input HTML natif dans les formulaires)
- [ ] `LoadingButton` avec `loading={isSubmitting || isPending}`
- [ ] Erreurs de validation mappées sur les champs (`setError` par violation)
- [ ] Erreur globale affichée via `Alert` si pas de violations (`errors.root`)
- [ ] `parseAxiosError<TypeErreur>()` utilisé pour extraire les erreurs
- [ ] `useAuth()` pour accéder à l'utilisateur connecté
- [ ] `useToast()` pour les feedbacks utilisateur
- [ ] `cn()` pour les classes CSS conditionnelles
- [ ] Formulaire de connexion : `FLAG_SESSION` + `fetchAndSetUser` dans `onSuccess`

### 5. Traductions
- [ ] Namespace créé ou existant identifié
- [ ] Fichiers JSON créés dans `fr` ET `en`
- [ ] Namespace ajouté dans `src/i18n.ts` (si nouveau)
- [ ] Chaque clé `t()` présente dans `fr/{namespace}.json` ET `en/{namespace}.json`
- [ ] `defaultValue` dans le code identique à la valeur `fr/{namespace}.json`
- [ ] Clés devenues inutilisées supprimées des deux fichiers JSON
- [ ] `node check/translation.mjs` lancé depuis `front/` — résultat : 0 problème
- [ ] Aucune chaîne de caractères codée en dur dans les composants
- [ ] Clés i18n de navigation ajoutées dans `header.json` (si nouvelle page dans le header)

### 6. Navigation et constantes
- [ ] Chemin ajouté dans `APP_PATHS` (si nouvelle page)
- [ ] Entrée ajoutée dans `nav.tsx` dans le bon tableau (`usersConnectedPages`, `usersNotConnectedPages` ou `publicPages`)
- [ ] Si la feature ajoute un **nouvel endpoint API** qui ne doit pas déclencher le refresh/retry automatique (login, logout, reset-password…) : d'abord ajouter le chemin dans `API_PATHS` (ex: `API_PATHS.MON_DOMAINE.ACTION`), puis l'ajouter dans `ROUTES_WITHOUT_RETRY` via `API_PATHS.XXX` dans `constants.ts`

### 7. SEO — sitemap & robots (uniquement si nouvelle page)
> Les metadata (`generateMetadata` / `generateNoIndexMetadata`) sont obligatoires sur chaque `page.tsx` — elles sont vérifiées à l'étape 3.

- [ ] Page privée : ajouter le chemin dans `exclude` et `robotsTxtOptions.policies[0].disallow` de `next-sitemap.config.js` (ex: pour `/ma-feature`, ajouter `'/ma-feature*'` dans `exclude` et `'/ma-feature'` dans `disallow`)
- [ ] Page publique indexée : rien à faire (inclusion automatique dans le sitemap)
- [ ] OG image fournie si page publique indexée

---

## Scripts utilitaires

### `check/translation.mjs` — Audit des traductions

Script Node.js à exécuter depuis le dossier `front/` :

```bash
node check/translation.mjs
```

**Ce qu'il vérifie :**

| Problème détecté | Signification | Action |
|---|---|---|
| ❌ Namespace introuvable | Le namespace utilisé dans `t()` n'a pas de fichier JSON dans `locales/fr/` | Créer le fichier ou corriger le namespace |
| ❌ Clé absente du FR | La clé est utilisée dans le code mais absente de `locales/fr/{ns}.json` | Ajouter la clé dans le fichier FR |
| ❌ Clé absente du EN | La clé est présente en FR mais absente de `locales/en/{ns}.json` | Ajouter la clé dans le fichier EN |
| ⚠️ Pas de `defaultValue` | Un appel `t()` n'a pas de `defaultValue` | Ajouter `{ defaultValue: '...' }` |
| ⚠️ `defaultValue` ≠ valeur FR | La `defaultValue` dans le code diffère du texte dans le JSON FR | Synchroniser les deux |
| ⚠️ Clé inutilisée (code mort) | Une clé est dans le JSON mais jamais appelée dans le code | Supprimer la clé du JSON |
| ℹ️ Apostrophe U+0027 vs U+2019 | Différence de type d'apostrophe entre code et JSON | Harmoniser (préférer U+2019 `'` dans les JSONs) |

**Périmètre :** parcourt récursivement tous les fichiers `.ts` et `.tsx` de `src/`. Le namespace `zod` est exclu (ses clés sont gérées par le système Zod i18n).

**Résultat attendu après toute modification de traductions :**

```
  Total problèmes à corriger         : 0
```

---

[← Retour au sommaire](./SUMMARY.md)
