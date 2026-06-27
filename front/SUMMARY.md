# Documentation Frontend - Sommaire

[← Retour à la documentation principale](../README.md)

## Navigation

1. [CLI](./00-cli.md)
2. [Light and Dark mode](./01-theme.md)
3. [Tous les composants](./02-components.md)
4. [Tailwind](./03-tailwind.md)
5. [Shadcn](./04-shadcn.md)
6. [Traductions](./06-traductions.md)
7. [Pages présentes de base](./07-pages.md)
8. [Layout de base](./08-layout.md)
9. [Génération](./09-generation.md)
10. [Formulaires](./10-formulaires.md)
11. [Récupération et envoi de données](./11-data.md)
12. [Gestion de la session](./12-session.md)
13. [Contexts](./13-contexts.md)
14. [Hooks](./14-hooks.md)
15. [Utils](./15-utils.md)
16. [Config](./16-config.md)
17. [Guide SEO & Métadonnées](./17-seo-metadata.md)
18. [Guide IA : Contribuer au Frontend](./18-agent-ia.md)

## Résumés

### CLI
Présente le CLI officiel `fdn` : installation et commandes pour initialiser, builder et synchroniser le frontend (`fdn init front`, `fdn build front`, `fdn sync`). Point d'entrée recommandé pour démarrer ou mettre à jour le projet.

### Light and Dark mode
Système de thème dark/light avec `next-themes`, variables CSS oklch et persistance localStorage. Le projet utilise Shadcn/ui pour les composants et Tailwind CSS v4 pour le styling. Détaille la structure des fichiers CSS (`theme.css`, `sonner.css`), la palette oklch `--slate-*`, les tokens sémantiques, et la configuration `next-themes`.

### Tous les composants
Présente tous les composants UI disponibles dans le projet. Le projet utilise Shadcn/ui comme base. Détaille la structure (`ui`, `background`, `form/inputs`, `loaders`, `modals`), liste les 37 composants UI installés et personnalisés (Shadcn + custom), les wrappers React Hook Form, et le BackgroundGrid (Magic UI). Référence le Storybook (`npm run storybook`, port 6006) comme outil d'exploration interactive.

### Tailwind
Présente la configuration de Tailwind CSS v4. Le thème et les couleurs se configurent dans les fichiers CSS via `@theme inline` : le `tailwind.config.ts` est minimal (uniquement les chemins de contenu). Détaille la configuration PostCSS, l'architecture v4 et l'utilisation des variables de couleur.

### Shadcn
Présente l'utilisation de Shadcn/ui pour les composants UI de base, tous personnalisés avec le thème du projet. 37 composants installés. Référence le Storybook pour explorer les composants visuellement.

### Traductions
Présente le système d'internationalisation (i18n) avec i18next. L'i18n s'appuie sur i18next avec un backend chaîné (cache localStorage + chargement HTTP). La langue est choisie via un sélecteur visible et propagée aux appels API (`Accept-Language`). Les validations Zod sont automatiquement traduites grâce au hook `useZodI18n`.

### Pages présentes de base
Présente la structure des pages de l'application. Routes organisées en groupes : `(logged)` (AuthGuard), `(unlogged)` (UnloggedGuard), `(public)` (sans guard). Pages : home, login, register, forgot-password, new-password, verify-account, profile, private, contact, not-found. Toutes les pages auth/public utilisent `AuthPageLayout` pour le responsive mobile/desktop. Routes centralisées dans `APP_PATHS`.

### Layout de base
Présente le système de layouts imbriqués. Détaille `AuthPageLayout` (`src/layout/`) : layout partagé pour les pages auth/public avec gestion automatique mobile/desktop via `useIsScreenBelowBreakpoint`. Décrit le pattern mobile/desktop avec sous-composants, le header responsif (`HeaderDesktop`/`HeaderMobile` via hook), les constantes de navigation dans `src/constants/header/nav.tsx`, et les constantes de layout (`header: 56px`).

### Génération
Présente l'utilisation d'Orval pour générer automatiquement des types TypeScript, des hooks React Query et des schémas Zod à partir du schéma OpenAPI de l'API.

### Formulaires
Explique comment utiliser les formulaires avec React Hook Form et Zod, en utilisant des composants de formulaire prêts à l'emploi. Détaille la structure et les composants, React Hook Form, les inputs personnalisés, la validation avec Zod, la gestion des erreurs et les patterns.

### Récupération et envoi de données
Explique comment gérer les appels API et la récupération de données avec React Query et les hooks générés par Orval. Détaille les hooks générés, React Query, le mutator Axios, la gestion des erreurs, le retry automatique et la gestion des 401.

### Gestion de la session
Présente le système de gestion de session avec AuthContext, Guards et QueryClient. La session est stateless (cookies HTTP-only côté backend). Détaille l'AuthContext, les quatre guards (AuthGuard, UnloggedGuard, ForgotPasswordGuard, VerifyAccountGuard), le QueryClient et le rafraîchissement automatique du token.

### Contexts
Présente le composant `Providers` et les différents contextes React utilisés pour gérer l'état global : AuthContext (authentification), ThemeSampleContext (thème), ReactQueryDevtoolsClient (outils de développement). Détaille l'ordre d'imbrication des providers et chaque contexte.

### Hooks
Présente les hooks personnalisés : `useAppRouter` (navigation avec barre de progression), `useIsScreenBelowBreakpoint` (détection de breakpoints : utilisé pour le responsive mobile/desktop dans les vues), `useZodI18n` (messages d'erreur lors du changement de langue), `useFileOpen` et `useFileDownload` (récupération de fichiers protégés via AXIOS_INSTANCE avec refresh token automatique).

### Utils
Présente les fonctions utilitaires disponibles : manipulation de données (`hexToRgba`, `normalizeDate`), gestion des erreurs (`handleServerErrors`, `displayError`, `mapFieldsErrors`, `parseAxiosError`, `makeValidationErrorResponse`), SEO (`getPageMetadata`, `generateLayoutMetadata`, `SITE_CONFIG`).

### Config
Présente les différentes configurations du projet frontend : `next.config.ts`, `tsconfig.json`, `orval.config.ts`, scripts npm, variables d'environnement et constantes.

### Guide SEO & Métadonnées
Guide pratique pour la gestion des métadonnées et URLs canoniques. Explique le système centralisé (`SITE_CONFIG`, `PAGES_METADATA`, `getPageMetadata`), liste les clés par page, et donne les bonnes pratiques.

### Guide IA : Contribuer au Frontend
Guide destiné aux agents IA pour créer, modifier ou étendre des features du frontend. Définit un workflow structuré : analyse & découpage, régénération Orval, structure de la page (3 couches), implémentation, traductions, navigation. Inclut une checklist de validation complète.

---

[← Retour à la documentation principale](../README.md)
