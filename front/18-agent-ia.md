# Guide IA : Contribuer au Frontend

[← Retour au sommaire](./SUMMARY.md)

Ce document est destiné aux agents IA travaillant sur le FoundationKit. Il définit un workflow structuré et reproductible pour créer, modifier ou étendre des features du frontend tout en respectant la philosophie du projet : **architecture par sections, génération Orval, formulaires typés, traductions systématiques**.

Suivre ce guide dans l'ordre garantit une implémentation cohérente avec le reste du projet.

> **Features touchant l'API :** Si la feature modifie ou ajoute des endpoints, consulter aussi le [Guide IA API](../api/16-agent-ia.md) pour garantir la cohérence backend/frontend.

> **Important : code généré automatiquement :** Le dossier `src/api/generated/` est entièrement géré par Orval. **Ne jamais modifier ces fichiers manuellement** : ils sont écrasés à chaque regénération. Tout le code frontend (hooks, types, schemas Zod) découle du spec OpenAPI de l'API.

---

## CLI FoundationKit

Le projet s'utilise via le CLI `fdn`. En tant qu'agent IA, tu n'exécutes pas ces commandes toi-même : tu **demandes à l'utilisateur** de les lancer aux moments appropriés du workflow.

| Commande | Quand l'utiliser |
|----------|-----------------|
| `fdn login` | Première utilisation : connecter le compte FoundationKit |
| `fdn create` | Créer un nouveau projet (télécharge le starter kit) |
| `fdn init front` | Première installation : dépendances + sync Orval + build |
| `fdn sync` | Après chaque changement de l'API (nouveaux endpoints, DTOs modifiés) |
| `fdn build front` | Chaque déploiement en production |
| `fdn license` | Vérifier le statut de la licence |
| `fdn logout` | Déconnecter le compte courant |

> Les détails de chaque commande sont dans [CLI](./00-cli.md).

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
| 02 | [Components](./02-components.md) | Liste des 37 composants UI + 16 inputs RHF, Storybook |
| 03 | [Tailwind](./03-tailwind.md) | Tailwind v4, `@theme inline`, classes custom |
| 04 | [Shadcn](./04-shadcn.md) | Installation, personnalisation, conventions |
| 06 | [Traductions](./06-traductions.md) | i18next, namespaces, Zod i18n, `useZodI18n` |
| 07 | [Pages](./07-pages.md) | App Router, guards, structure page/view/form |
| 08 | [Layout](./08-layout.md) | Layout hiérarchique, header, navigation, footer |
| 09 | [Generation](./09-generation.md) | Orval, hooks générés, types, Zod schemas |
| 10 | [Formulaires](./10-formulaires.md) | React Hook Form, Zod, inputs RHF, erreurs |
| 11 | [Data](./11-data.md) | React Query, hooks Orval, gestion des erreurs |
| 12 | [Session](./12-session.md) | AuthContext, guards, refresh token, callbacks QueryClient |
| 13 | [Contexts](./13-contexts.md) | AuthContext, ThemeSampleContext, providers |
| 14 | [Hooks](./14-hooks.md) | useAppRouter, useZodI18n, useIsScreenBelowBreakpoint, useFileOpen, useFileDownload |
| 15 | [Utils](./15-utils.md) | handleServerErrors, displayError, mapFieldsErrors, parseAxiosError, cn(), metadata |
| 16 | [Config](./16-config.md) | Variables d'env, constantes, scripts, next.config |
| 17 | [SEO](./17-seo-metadata.md) | getPageMetadata, SITE_CONFIG, PAGES_METADATA, sitemap, og |

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

## Étape 1 : Analyse & découpage de la feature

### 1.1 Résumer la feature

Avant toute implémentation, répondre à ces questions :
- **Que fait cette feature ?** (comportement attendu côté utilisateur)
- **Qui l'utilise ?** (connecté, déconnecté, public)
- **Est-ce qu'elle appelle l'API ?** (quels endpoints)
- **Est-ce qu'elle modifie l'état global ?** (AuthContext)

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
| SEO | `getPageMetadata('clé')` | Obligatoire : clé à ajouter dans `PAGES_METADATA` (`src/utils/metadata.ts`) |

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

## Étape 2 : Régénération Orval (si nécessaire)

### 2.1 Quand régénérer

Régénérer dès que :
- Un nouvel endpoint a été ajouté à l'API
- Un DTO existant a été modifié (champs, types, validations)
- Un champ a changé de statut requis/optionnel dans un DTO
- Un nouveau domaine API a été créé

### 2.2 Configuration Orval

Orval est configuré via `orval.config.ts` à la racine du frontend. La variable d'environnement `SWAGGER_JSON` (dans `.env`, `.env.local` ou `.env.production`) pointe vers le spec OpenAPI de l'API avec le token d'accès (ex: `{API_URL}/api-json?token={SWAGGER_TOKEN}`). `SWAGGER_TOKEN` doit également être défini : une erreur est levée si absent.

### 2.3 Lancer la génération

> **Action utilisateur requise**
>
> ```bash
> fdn sync
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
        └── {domain}.ts      ← schemas Zod (AuthControllerLoginBody, UserControllerUpdateBody...)
```

Le nom du domaine correspond au tag `@ApiTags()` du controller côté API. Exemple : controller tagué `auth` → `useAuthControllerLogin` dans `src/api/generated/auth/auth.ts`.

**Règle absolue :** ne jamais modifier ces fichiers. Toute correction passe par l'API puis une regénération.

Voir aussi : [Generation](./09-generation.md)

---

## Étape 3 : Structure de la page

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

**Couche 1 : `page.tsx` :** fine, uniquement la metadata et l'import de la view.

```tsx
import { getPageMetadata } from '@/utils/metadata';
import FeatureView from '@/sections/feature/view/feature-view';

export const metadata = getPageMetadata('feature');

export default function FeaturePage() {
  return <FeatureView />;
}
```

**Couche 2 : `*-view.tsx` :** structure visuelle (Card, titre, sous-titre, mise en page). Pour les pages avec illustration (login, register, profile, etc.), s'inspirer des composants existants dans `src/assets/illustrations/`.

> **`'use client'` obligatoire** sur tous les fichiers `*-view.tsx` et `*-form.tsx` : ils utilisent des hooks React (useTranslation, useIsScreenBelowBreakpoint, etc.).

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

**Couche 3 : `*-form.tsx` :** logique React Hook Form, appel API, gestion des erreurs. Si la page n'a pas de formulaire (ex : page de lecture seule), la view contient directement le contenu : pas de fichier `*-form.tsx`.

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

**Pages publiques :** dans `src/app/(public)/`.

**Page protégée par un guard spécial (ex : ForgotPasswordGuard) :** placer dans le groupe et layout approprié. La page `/forgot-password/new-password` utilise `ForgotPasswordGuard` via le layout `(unlogged)/(reset-password)/forgot-password/new-password/layout.tsx` : s'en inspirer pour des pages similaires (ex : validation de token par email).

### 3.3 Metadata SEO

Toutes les metadata de pages sont centralisées dans `PAGES_METADATA` dans `src/utils/metadata.ts`.

**Étape 1** : Ajouter la clé dans `PAGES_METADATA` :

```typescript
// Page publique indexée
maPage: generateMetadata({
  title: 'Mon titre',
  description: 'Description',
  canonical: '/ma-page',
  ogImage: '/images/og.jpg', // optionnel
}),

// Page privée / auth
maPage: generateNoIndexMetadata({
  title: 'Mon titre',
  description: 'Description',
  canonical: '/ma-page',
}),
```

**Étape 2** : Appeler `getPageMetadata` dans la page :

```tsx
import { getPageMetadata } from '@/utils/metadata';

export const metadata = getPageMetadata('maPage');
```

> Une règle ESLint bloquante vérifie que chaque `page.tsx` appelle `getPageMetadata`. Le build échoue si absent. Le root layout utilise `generateLayoutMetadata()` directement (exception).

> **Si la page est accessible via le header**, ajouter le chemin dans `APP_PATHS` (`src/constants/constants.ts`) et utiliser cette constante dans `nav.tsx` : ne jamais écrire de chemin en dur.

Voir aussi : [Pages](./07-pages.md), [SEO](./17-seo-metadata.md)

---

## Étape 4 : Implémentation des composants

### 4a. Formulaires

**Pattern complet d'un formulaire :**

```tsx
'use client';

import { AlertCircleIcon } from 'lucide-react';
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { useTranslation } from 'react-i18next';
import { toast } from 'sonner';
import { z } from 'zod';

// 1. Schema Zod généré par Orval (préféré) ou défini manuellement
import { FeatureControllerActionBody } from '@/api/generated/zod/{domain}/{domain}';
// 2. Hook React Query généré par Orval
import { useFeatureControllerAction } from '@/api/generated/{domain}/{domain}';
// 3. Composants form
import { Form } from '@/components/form/form';
import { RhfTextInput, RhfPasswordInput } from '@/components/form/inputs';
import { Button } from '@/components/ui/button';
import { Alert, AlertTitle } from '@/components/ui/alert';
// 4. Hooks
import { useZodI18n } from '@/hooks/useZodI18n';
import { useAppRouter } from '@/hooks/useAppRouter';
import { handleServerErrors } from '@/utils/errors';
import { APP_PATHS } from '@/constants/constants';

const zSchema = FeatureControllerActionBody;
type FormInputs = z.infer<typeof zSchema>;

export default function FeatureForm() {
  const { t } = useTranslation('{namespace}');
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
          toast.success(t('{namespace}:form.success', { defaultValue: '...' }));
          router.push(APP_PATHS.HOME);
        },
        onError: (error) => {
          handleServerErrors(error, setError);
        },
      }
    );
  });

  return (
    <Form methods={methods} onSubmit={onSubmit}>
      {/* Erreur globale : affichée automatiquement par handleServerErrors sur errors.root */}
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

      <Button type="submit" loading={isSubmitting || isPending}>
        {t('form.submit')}
      </Button>
    </Form>
  );
}
```

**Règles importantes :**
- Toujours utiliser le schema Zod généré par Orval (`FeatureControllerActionBody`) : il est en sync avec les validations de l'API
- `useZodI18n(methods)` est **obligatoire** dans chaque formulaire avec Zod
- Pour la prop `required` sur les inputs, la dériver du schéma Zod plutôt que de la coder en dur : `required={!zSchema.shape.field.safeParse('').success}`. Ce pattern est fiable pour les champs `string` ; pour les champs d'un autre type (boolean, number…), `safeParse('')` échoue systématiquement : utiliser la valeur vide du type concerné (ex: `safeParse(false)` pour un boolean).
- Le message affiché en `onSuccess` vient d'une clé de traduction frontend, pas de `response.message`
- Gestion du 401 dans `onError` : utiliser `handleServerErrors(error, setError)` — le 401 est silencieux par défaut car l'interceptor Axios de `httpClient.ts` gère refresh + retry automatiquement avant que React Query voie l'erreur. Pour les routes dans `ROUTES_WITHOUT_RETRY` (login, reset-password…), un 401 est une vraie erreur métier : passer `{ includeUnauthorizedAlert: true }`.
- Utiliser `mutate` (avec callbacks) pour les formulaires, `mutateAsync` uniquement si un `await` est nécessaire
- Utiliser `useAppRouter` à la place de `useRouter` (barre de progression)

**Cas particulier : formulaire de connexion :**

Après un login réussi, appeler `refetchMe()` pour recharger les données utilisateur depuis l'API :

```tsx
import { useAuth } from '@/contexts/AuthContext';

const { refetchMe } = useAuth();

mutate(
  { data: input },
  {
    onSuccess: () => {
      refetchMe();
    },
    onError: (error) => {
      // LOGIN est dans ROUTES_WITHOUT_RETRY : 401 = mauvais identifiants
      handleServerErrors(error, setError, { includeUnauthorizedAlert: true });
    },
  }
);
```

### 4b. Inputs disponibles

| Composant | Cas d'usage |
|-----------|------------|
| `RhfTextInput` | Texte libre |
| `RhfEmailInput` | Email (avec icône `@` intégrée) |
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
| `RhfMultiSelectInput` | Sélection multiple avec recherche et "Tous" |
| `RhfDynamicSelectInput` | Select avec création à la volée via modale |

Tous partagent les props : `control`, `name`, `label`, `placeholder`, `required`, `disabled`, `className`.

**`RhfFileInput` : props spécifiques :**

| Prop | Type | Description |
|------|------|-------------|
| `preview` | `string` | URL de prévisualisation de l'image actuelle |
| `defaultImage` | `string` | Image affichée si aucune valeur |
| `previewWidth` | `number` | Largeur de la prévisualisation en px |
| `previewHeight` | `number` | Hauteur de la prévisualisation en px |
| `onDelete` | `() => void` | Callback déclenché au clic sur "Supprimer" |

Le fichier sélectionné est passé comme objet `File` directement à la mutation : la conversion en `FormData` est gérée en interne par le mutator Orval.

### 4c. Appels API sans formulaire

Pour les appels API hors formulaire (ex: bouton d'action, chargement de données) :

```tsx
'use client';

// Mutation (POST/PATCH/DELETE)
import { useFeatureControllerAction } from '@/api/generated/{domain}/{domain}';
// Query (GET)
import { useFeatureControllerGetData } from '@/api/generated/{domain}/{domain}';
import { useTranslation } from 'react-i18next';
import { displayError } from '@/utils/errors';
import { toast } from 'sonner';

const { t } = useTranslation('{namespace}');
const { mutate, isPending } = useFeatureControllerAction();

const handleAction = () => {
  mutate(
    { data: payload },
    {
      onSuccess: () => { toast.success(t('{namespace}:form.success', { defaultValue: '...' })); },
      onError: (error) => {
        displayError(error); // silencieux sur 401 (QueryClient gère le refresh + retry)
      },
    }
  );
};

const { data, isLoading, isError } = useFeatureControllerGetData();
```

### 4d. Accéder à l'utilisateur connecté

Dans n'importe quel composant client, l'utilisateur est disponible via `useAuth()` :

```tsx
'use client';

import { useAuth } from '@/contexts/AuthContext';

export default function MyComponent() {
  const { user, isLoadingUser } = useAuth();

  if (isLoadingUser) return <PageLoader />;
  if (!user) return null;

  return <p>Bonjour {user.firstName}</p>;
}
```

`user` est de type `UserProfileDTO` (généré par Orval). Champs disponibles : `id`, `firstName`, `lastName`, `email`, `username`, `role`, `gender`, `dateOfBirth`, `notificationEmail`, `profilePicture`, etc.

**Toutes les valeurs retournées par `useAuth()` :**

| Valeur | Type | Usage |
|--------|------|-------|
| `user` | `UserProfileDTO \| undefined` | Données de l'utilisateur connecté, `undefined` si non connecté |
| `isLoadingUser` | `boolean` | Vrai pendant le premier chargement du profil |
| `refetchMe` | `() => Promise<...>` | Recharge les données utilisateur depuis l'API : à appeler après un login ou une mise à jour du profil |

**`onSettled` : même comportement succès et erreur :**

Quand l'action doit produire le même effet qu'elle réussisse ou échoue (ex : logout) :

```tsx
mutate(undefined, {
  onSettled: () => {
    queryClient.setQueryData(getUserControllerGetProfileQueryKey(), null);
  },
});
```

**Validation côté client avant l'appel API :**

Pour valider des champs que Zod ne peut pas vérifier (ex : confirmation de mot de passe), construire les violations manuellement avec `makeValidationErrorResponse` puis passer à `mapFieldsErrors` :

```tsx
import { makeValidationErrorResponse, mapFieldsErrors } from '@/utils/errors';

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
import { toast } from 'sonner';

toast.success('Opération réussie');
toast.error('Une erreur est survenue');
toast.warning('Attention');
toast.info('Information');
```

### 4f. Navigation

**Toujours utiliser `useAppRouter`** à la place de `useRouter` : il intègre la barre de progression. **Utiliser `APP_PATHS`** pour tous les chemins (y compris les liens `<a href>` et `Link`) : éviter les chemins en dur comme `href="/forgot-password"`.

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

## Étape 5 : Traductions

### 5.1 Choisir le namespace

Chaque page/feature a son propre namespace, correspondant à un fichier JSON dans `public/locales/`.

| Namespace existant | Usage |
|-------------------|-------|
| `login` | Page de connexion |
| `logout` | Namespace de déconnexion |
| `register` | Page d'inscription |
| `profile` | Page profil |
| `update-password` | Changement de mot de passe |
| `forgot-password` | Page demande de réinitialisation |
| `forgot-password-guard` | Messages du guard de réinitialisation |
| `forgot-password-new-password` | Page nouveau mot de passe |
| `home` | Page d'accueil |
| `verify-account` | Page de vérification de compte (email) |
| `private` | Page protégée de démonstration |
| `contact` | Page de contact |
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

Utiliser le namespace unique quand le composant n'accède qu'à ses propres clés. Ajouter `'common'` dans l'array dès que le composant appelle `t('common:...')` (message d'erreur 500, libellés partagés, etc.) : c'est le cas de la quasi-totalité des formulaires qui ont un `onError`.

```tsx
import { useTranslation } from 'react-i18next';

// Namespace unique : pas d'accès à 'common'
const { t } = useTranslation('{namespace}');

// Avec 'common' : dès que le composant utilise t('common:...')
const { t } = useTranslation(['{namespace}', 'common']);

// Utilisation : toujours préfixer avec le namespace
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

## Étape 6 : Ajouter la navigation (si nouvelle page)

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
| `publicPages` | Pages accessibles à tous (contact, 404, etc.) |

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
- [ ] Orval regénéré si l'API a changé (`fdn sync`)
- [ ] Types et hooks attendus présents dans `src/api/generated/`
- [ ] Prop `required` de chaque input vérifiée après regénération (`!zSchema.shape.field.safeParse('').success`)

### 3. Pages et routing
- [ ] Page placée dans le bon groupe de routes (`(logged)`, `(unlogged)`, `(public)`, ou racine)
- [ ] Structure en 3 couches respectée (`page.tsx` → `*-view.tsx` → `*-form.tsx`)
- [ ] Clé ajoutée dans `PAGES_METADATA` (`src/utils/metadata.ts`) avec `generateMetadata` ou `generateNoIndexMetadata` selon l'indexation
- [ ] `getPageMetadata('clé')` exporté depuis `page.tsx`
- [ ] `canonical` correct dans la metadata

### 4. Implémentation
- [ ] Schema Zod Orval utilisé comme source de vérité (`FeatureControllerActionBody`)
- [ ] `useZodI18n(methods)` présent dans chaque formulaire
- [ ] `useAppRouter` utilisé à la place de `useRouter`
- [ ] Inputs RHF utilisés (pas d'input HTML natif dans les formulaires)
- [ ] `Button` avec `loading={isSubmitting || isPending}`
- [ ] `handleServerErrors(error, setError)` dans `onError` des formulaires
- [ ] `displayError(error)` dans `onError` hors formulaire
- [ ] Erreur globale affichée via `Alert` sur `errors.root`
- [ ] `useAuth()` pour accéder à l'utilisateur connecté (`user`, `isLoadingUser`, `refetchMe`)
- [ ] `toast.success / toast.error` (depuis `'sonner'`) pour les feedbacks utilisateur
- [ ] `cn()` pour les classes CSS conditionnelles
- [ ] Formulaire de connexion : `refetchMe()` dans `onSuccess` + `includeUnauthorizedAlert: true` dans `onError`

### 5. Traductions
- [ ] Namespace créé ou existant identifié
- [ ] Fichiers JSON créés dans `fr` ET `en`
- [ ] Namespace ajouté dans `src/i18n.ts` (si nouveau)
- [ ] Chaque clé `t()` présente dans `fr/{namespace}.json` ET `en/{namespace}.json`
- [ ] `defaultValue` dans le code identique à la valeur `fr/{namespace}.json`
- [ ] Clés devenues inutilisées supprimées des deux fichiers JSON
- [ ] `node check/translation.mjs` lancé depuis `front/` : résultat : 0 problème
- [ ] Aucune chaîne de caractères codée en dur dans les composants
- [ ] Clés i18n de navigation ajoutées dans `header.json` (si nouvelle page dans le header)

### 6. Navigation et constantes
- [ ] Chemin ajouté dans `APP_PATHS` (si nouvelle page)
- [ ] Entrée ajoutée dans `nav.tsx` dans le bon tableau (`usersConnectedPages`, `usersNotConnectedPages` ou `publicPages`)
- [ ] Si la feature ajoute un **nouvel endpoint API** qui ne doit pas déclencher le refresh/retry automatique (login, logout, reset-password…) : d'abord ajouter le chemin dans `API_PATHS` (ex: `API_PATHS.MON_DOMAINE.ACTION`), puis l'ajouter dans `ROUTES_WITHOUT_RETRY` via `API_PATHS.XXX` dans `constants.ts`

### 7. SEO : sitemap & robots (uniquement si nouvelle page)
> `getPageMetadata('clé')` est obligatoire sur chaque `page.tsx` : vérifié par une règle ESLint bloquante (build échoue si absent).

- [ ] Page privée : ajouter le chemin dans `exclude` et `robotsTxtOptions.policies[0].disallow` de `next-sitemap.config.js` (ex: pour `/ma-feature`, ajouter `'/ma-feature*'` dans `exclude` et `'/ma-feature'` dans `disallow`)
- [ ] Page publique indexée : rien à faire (inclusion automatique dans le sitemap)
- [ ] OG image fournie si page publique indexée

---

## Scripts utilitaires

### `check/translation.mjs` : Audit des traductions

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
