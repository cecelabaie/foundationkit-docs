# Pages

Ce document présente la structure des pages de l'application et explique leur fonctionnement.

## Structure des routes

L'application utilise le système de routage basé sur les dossiers de Next.js 13+ (App Router). La structure est organisée comme suit :

```
src/app/
  ├── (logged)/             # Pages accessibles uniquement aux utilisateurs connectés
  │   ├── layout.tsx        # Layout avec AuthGuard
  │   ├── logout/           # Page de déconnexion
  │   ├── private/          # Page privée (exemple)
  │   └── profile/          # Profil utilisateur
  │       └── update-password/  # Mise à jour du mot de passe
  │
  ├── (unlogged)/           # Pages accessibles uniquement aux utilisateurs non connectés
  │   ├── layout.tsx        # Layout avec UnloggedGuard
  │   ├── login/            # Page de connexion
  │   ├── register/         # Page d'inscription
  │   └── (reset-password)/ # Groupe de pages pour la réinitialisation du mot de passe
  │       └── forgot-password/
  │           ├── page.tsx              # Demande de réinitialisation
  │           └── new-password/         # Création d'un nouveau mot de passe
  │               ├── layout.tsx        # Layout avec ForgotPasswordGuard
  │               └── page.tsx          # Création d'un nouveau mot de passe
  │
  ├── layout.tsx            # Layout racine
  ├── not-found.tsx         # Page 404
  ├── page.tsx              # Page d'accueil
  └── sample/               # Page d'exemple (démo des composants)
```

## Système de protection des routes

### Guards

L'application utilise trois types de guards pour protéger les routes :

1. **AuthGuard** (`src/guards/authGuard.tsx`)
   - Protège les routes nécessitant une authentification
   - Vérifie la validité du token à chaque changement de route
   - Redirige vers la page de connexion si l'utilisateur n'est pas connecté
   - Supporte la vérification des rôles (ex: `requiredRoles={['user']}`)
   - Crée le paramètre `returnTo` pour rediriger après connexion

2. **UnloggedGuard** (`src/guards/unloggedGuard.tsx`)
   - Protège les routes accessibles uniquement aux utilisateurs non connectés
   - Redirige vers la page d'accueil si l'utilisateur est déjà connecté

3. **ForgotPasswordGuard** (`src/guards/forgotPasswordGuard.tsx`)
   - Protège la page de création de nouveau mot de passe
   - Vérifie la validité du token de réinitialisation
   - Redirige vers la page de demande de réinitialisation si le token est invalide

### Groupes de routes

Les routes sont organisées en groupes pour appliquer des layouts et des guards communs :

- **`(logged)`** : Routes nécessitant une authentification
- **`(unlogged)`** : Routes accessibles uniquement aux utilisateurs non connectés
- **`(reset-password)`** : Sous-groupe pour la réinitialisation du mot de passe

## Pages principales

### Page d'accueil (`/page.tsx`)

- Point d'entrée principal de l'application
- Accessible à tous les utilisateurs
- Utilise `HomeView` pour le rendu du contenu
- Inclut des métadonnées SEO optimisées

### Authentification

- **Login** (`/(unlogged)/login/page.tsx`)
  - Page de connexion avec formulaire d'authentification
  - Redirige vers la page demandée après connexion (via `returnTo`)

- **Register** (`/(unlogged)/register/page.tsx`)
  - Page d'inscription pour créer un nouveau compte
  - Contient un formulaire avec validation

- **Logout** (`/(logged)/logout/page.tsx`)
  - Page de déconnexion
  - Utilise `getPageMetadata('logout')` pour éviter l'indexation par les moteurs de recherche

### Profil utilisateur

- **Profile** (`/(logged)/profile/page.tsx`)
  - Affiche et permet de modifier les informations du profil
  - Accessible uniquement aux utilisateurs connectés

- **Update Password** (`/(logged)/profile/update-password/page.tsx`)
  - Permet à l'utilisateur de changer son mot de passe
  - Accessible uniquement aux utilisateurs connectés

### Réinitialisation du mot de passe

- **Forgot Password** (`/(unlogged)/(reset-password)/forgot-password/page.tsx`)
  - Formulaire pour demander une réinitialisation du mot de passe
  - Envoie un email avec un lien contenant un token

- **New Password** (`/(unlogged)/(reset-password)/forgot-password/new-password/page.tsx`)
  - Formulaire pour créer un nouveau mot de passe
  - Protégé par `ForgotPasswordGuard` qui vérifie la validité du token
  - Accessible via un lien envoyé par email

### Pages spéciales

- **Sample** (`/sample/page.tsx`)
  - Page de démonstration des composants et du thème
  - Utile pour le développement et les tests

- **Not Found** (`/not-found.tsx`)
  - Page d'erreur 404 personnalisée
  - Affiche une illustration et un bouton pour retourner à l'accueil

## Métadonnées et SEO

Les pages utilisent des utilitaires pour générer des métadonnées optimisées pour le SEO :

- `getPageMetadata('clé')` : Récupère les métadonnées centralisées pour la page — **obligatoire dans chaque `page.tsx`**, vérifié par une règle ESLint bloquante
- `generateLayoutMetadata` : Génère les métadonnées de base pour le layout racine (layout uniquement)

Toutes les metadata de pages sont définies dans `PAGES_METADATA` dans `src/utils/metadata.ts`. Chaque page contient :
- Titre
- Description
- URL canonique
- Métadonnées Open Graph (titre, description, image)

## Architecture des composants

Chaque page suit une architecture en couches :

1. **Page** (`page.tsx`) : Composant léger qui définit les métadonnées et importe la vue
2. **View** (`*-view.tsx`) : Contient la structure et le layout de la page (Card, illustration, etc.)
3. **Form** (`*-form.tsx`) : Optionnel, composant de formulaire séparé contenant la logique du formulaire (utilisé pour les pages avec formulaires)
4. **Components** : Composants réutilisables utilisés dans les vues

**Exemple avec la page de login :**
- `page.tsx` : Définit les métadonnées et importe `LoginView`
- `login-view.tsx` : Structure de la page (Card, layout, illustration)
- `login-form.tsx` : Logique du formulaire de connexion (React Hook Form, validation, soumission)