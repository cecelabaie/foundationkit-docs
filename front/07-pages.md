# Pages

Ce document présente la structure des pages de l'application et explique leur fonctionnement.

## Structure des routes

L'application utilise le système de routage App Router de Next.js. La structure est organisée comme suit :

```
src/app/
  ├── (logged)/             # Pages accessibles uniquement aux utilisateurs connectés
  │   ├── layout.tsx        # Layout avec AuthGuard
  │   ├── private/          # Page privée (exemple de zone authentifiée)
  │   └── profile/          # Profil utilisateur (tabs : compte, mot de passe, sécurité)
  │
  ├── (unlogged)/           # Pages accessibles uniquement aux utilisateurs non connectés
  │   ├── layout.tsx        # Layout avec UnloggedGuard
  │   ├── login/            # Page de connexion
  │   ├── register/         # Page d'inscription
  │   ├── verify-account/   # Vérification de compte
  │   └── (reset-password)/ # Groupe pour la réinitialisation du mot de passe
  │       └── forgot-password/
  │           ├── page.tsx              # Demande de réinitialisation
  │           └── new-password/         # Création d'un nouveau mot de passe
  │               ├── layout.tsx        # Layout avec ForgotPasswordGuard
  │               └── page.tsx
  │
  ├── (public)/             # Pages publiques accessibles à tous
  │   ├── layout.tsx        # Layout public (sans guard)
  │   ├── page.tsx          # Page d'accueil (/)
  │   └── contact/          # Page de contact
  │
  ├── layout.tsx            # Layout racine
  └── not-found.tsx         # Page 404 personnalisée
```

## Système de protection des routes

### Guards

L'application utilise trois guards pour protéger les routes :

1. **AuthGuard** (`src/guards/authGuard.tsx`)
   - Protège les routes nécessitant une authentification
   - Vérifie la validité du token à chaque changement de route
   - Redirige vers `/login?returnTo=...` si l'utilisateur n'est pas connecté
   - Supporte la vérification des rôles (`requiredRoles`)

2. **UnloggedGuard** (`src/guards/unloggedGuard.tsx`)
   - Protège les routes accessibles uniquement aux utilisateurs non connectés
   - Redirige vers l'accueil si l'utilisateur est déjà connecté

3. **ForgotPasswordGuard** (`src/guards/forgotPasswordGuard.tsx`)
   - Protège la page de nouveau mot de passe
   - Vérifie la validité du token de réinitialisation via l'API
   - Redirige vers `/forgot-password` si le token est invalide ou expiré

### Groupes de routes

- **`(logged)`** : Routes nécessitant une authentification (AuthGuard)
- **`(unlogged)`** : Routes pour utilisateurs non connectés (UnloggedGuard)
- **`(public)`** : Routes publiques sans guard
- **`(reset-password)`** : Sous-groupe avec ForgotPasswordGuard

## Pages principales

### Page d'accueil (`/`)

Point d'entrée de l'application, accessible à tous. Affiche un hero avec illustration, badge, titre, et une grille bento de liens vers toutes les pages du starter kit.

Structure responsive via sous-composants :
- `HomeViewMobile` : stack vertical (texte → illustration → cards), affiché sous `lg`
- `HomeViewDesktop` : même stack mais avec illustration plus grande et grille asymétrique `5fr 4fr 3fr`
- Le hook `useIsScreenBelowBreakpoint('lg')` choisit lequel rendre (un seul dans le DOM)

### Authentification

- **Login** (`/login`) : Connexion avec redirection `returnTo` après succès
- **Register** (`/register`) : Inscription avec validation
- **Forgot Password** (`/forgot-password`) : Demande de réinitialisation par email
- **New Password** (`/forgot-password/new-password?token=...`) : Création d'un nouveau mot de passe (protégé par ForgotPasswordGuard)
- **Verify Account** (`/verify-account`) : Vérification du compte après inscription

Toutes ces pages utilisent `AuthPageLayout` (`src/layout/auth-page-layout.tsx`) qui gère automatiquement la mise en page mobile/desktop via `useIsScreenBelowBreakpoint('md')`.

### Zone connectée

- **Profile** (`/profile`) : 3 tabs (verticaux desktop, horizontaux mobile) : **Compte** (infos profil), **Mot de passe** (`?tab=password`), **Sécurité** (`?tab=security` : suppression de compte)
- **Private** (`/private`) : Exemple de page accessible uniquement aux utilisateurs authentifiés

### Pages publiques

- **Contact** (`/contact`) : Formulaire/informations de contact
- **Not Found** (`/not-found`) : Page 404 avec illustration et retour à l'accueil

## Métadonnées et SEO

Les pages utilisent des utilitaires centralisés :

- `getPageMetadata('clé')` : Métadonnées de la page : **obligatoire dans chaque `page.tsx`**, vérifié par une règle ESLint bloquante
- `generateLayoutMetadata` : Métadonnées de base pour le layout racine

Toutes les métadonnées sont définies dans `PAGES_METADATA` dans `src/utils/metadata.ts`. Chaque page contient titre, description, URL canonique et Open Graph.

## Routes : constantes

Toutes les routes de l'application sont centralisées dans `APP_PATHS` (`src/constants/constants.ts`). Ne jamais écrire de chemin de route en dur dans le code, toujours passer par cette constante.

```tsx
import { APP_PATHS } from '@/constants/constants';

<Link href={APP_PATHS.LOGIN}>Connexion</Link>
router.push(APP_PATHS.HOME);
```

## Architecture des composants

Chaque page suit une architecture en couches :

1. **Page** (`page.tsx`) : Composant léger, définit les métadonnées et importe la vue
2. **View** (`*-view.tsx`) : Structure et layout de la page
3. **Form** (`*-form.tsx`) : Optionnel, logique du formulaire (React Hook Form, validation, soumission)
4. **Components** : Composants réutilisables

**Exemple avec la page de login :**
- `page.tsx` : Métadonnées + import `LoginView`
- `login-view.tsx` : Passe illustration, titre, subtitle à `AuthPageLayout`
- `login-form.tsx` : Logique du formulaire (RHF, Zod, appel API, toast, redirect)
