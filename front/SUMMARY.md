# Documentation Frontend - Sommaire

[← Retour à la documentation principale](../README.md)

## Navigation

1. [Light and Dark mode](./01-theme.md)
2. [Tous les composants](./02-components.md)
3. [Tailwind](./03-tailwind.md)
4. [Shadcn](./04-shadcn.md)
5. [Sample](./05-sample.md)
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
18. [Guide IA — Contribuer au Frontend](./18-agent-ia.md)

## Résumés

### Light and Dark mode
Système de thème dark/light avec `next-themes`, variables CSS et persistance localStorage. Le projet utilise Shadcn/ui pour les composants et Tailwind CSS v4 pour le styling. Détaille la structure des fichiers CSS (palette, extends, toast), la configuration des variables CSS, la gestion du thème avec `next-themes` et la persistance.

### Tous les composants
Présente tous les composants UI disponibles dans le projet. Le projet utilise Shadcn/ui comme base pour les composants UI. Détaille la structure (ui, form/inputs, translation, loaders), liste les composants Shadcn installés et personnalisés, et les wrappers React Hook Form.

### Tailwind
Présente la configuration de Tailwind CSS v4. Avec Tailwind v4, la configuration se fait principalement dans les fichiers CSS via la directive `@theme inline`. Détaille la configuration PostCSS, l'architecture Tailwind v4, les fichiers de configuration et l'utilisation.

### Shadcn
Présente l'utilisation de Shadcn/ui pour les composants UI de base, tous personnalisés avec le thème du projet. Détaille la configuration dans `components.json`, les composants utilisés, les modifications apportées pour utiliser les variables CSS du thème et la personnalisation.

### Sample
Présente la page Sample (`/sample`) qui permet de visualiser l'UI/UX du starter kit, personnaliser le thème et tester les interactions. Détaille la structure (Thème Builder, onglet Formulaires, onglet Composants), les fonctionnalités et l'utilisation.

### Traductions
Présente le système d'internationalisation (i18n) avec i18next. L'i18n s'appuie sur i18next avec un backend chaîné (cache localStorage + chargement HTTP). La langue est choisie via un sélecteur visible et propagée aux appels API (`Accept-Language`). Les validations Zod sont automatiquement traduites grâce au hook `useZodI18n`.

### Pages présentes de base
Présente la structure des pages de l'application utilisant le système de routage App Router de Next.js 13+. Détaille l'organisation des routes (logged, unlogged, reset-password), les guards de protection, et les pages disponibles.

### Layout de base
Présente le système de layouts imbriqués basé sur l'App Router de Next.js. Détaille le layout racine (Space_Grotesk, theme sanitize script), la structure Header/Main/Footer, et le header responsif : `HeaderDesktop`, `HeaderMobile`, `HeaderDrawer` et le composant partagé `LeftSideHeader`. Les entrées de navigation sont définies dans `src/constants/header/nav.tsx`. Les providers sont documentés dans [Contexts](./13-contexts.md).

### Génération
Présente l'utilisation d'Orval pour générer automatiquement des types TypeScript, des hooks React Query et des schémas Zod à partir du schéma OpenAPI de l'API. Détaille la configuration Orval, le processus de génération, et l'intégration avec React Query et Zod.

### Formulaires
Explique comment utiliser les formulaires avec React Hook Form et Zod, en utilisant des composants de formulaire prêts à l'emploi. Détaille la structure et les composants, React Hook Form, les inputs personnalisés, la validation avec Zod, la gestion des erreurs et les patterns.

### Récupération et envoi de données
Explique comment gérer les appels API et la récupération de données avec React Query et les hooks générés par Orval. Détaille les hooks générés, React Query, le mutator Axios, la gestion des erreurs, le retry automatique et la gestion des 401.

### Gestion de la session
Présente le système de gestion de session avec AuthContext, Guards, QueryClient et le flag de session. La session est stateless (cookies HTTP-only côté backend). Détaille l'AuthContext, les guards, le QueryClient et le rafraîchissement automatique du token.

### Contexts
Présente le composant `Providers` et les différents contextes React utilisés pour gérer l'état global : AuthContext (authentification), ToastContext (notifications), ThemeSampleContext (personnalisation du thème), ReactQueryDevtoolsClient (outils de développement). Détaille l'ordre d'imbrication des providers, la structure de page globale, et chaque contexte.

### Hooks
Présente les hooks personnalisés développés pour l'application : `useAppRouter` (navigation avec barre de progression), `useIsScreenBelowBreakpoint` (détection de breakpoints), `useZodI18n` (mise à jour des messages d'erreur lors du changement de langue). Détaille chaque hook et son utilisation.

### Utils
Présente les fonctions utilitaires disponibles : manipulation de données (`getAvatar`, `hexToRgba`, `normalizeDate`), gestion des erreurs (`parseAxiosError`, `makeValidationErrorResponse`), SEO (`getPageMetadata`, `generateLayoutMetadata`, `SITE_CONFIG`, `getCanonicalUrl`). Détaille chaque utilitaire et son utilisation.

### Config
Présente les différentes configurations du projet frontend : `next.config.ts` (optimisations performance et sécurité), `tsconfig.json` (configuration TypeScript), `tailwind.config.ts` (configuration Tailwind), `orval.config.ts` (génération API), scripts npm, variables d'environnement et constantes.

### Guide SEO & Métadonnées
Guide pratique pour la gestion des métadonnées et URLs canoniques. Explique le système centralisé (`SITE_CONFIG`, `PAGES_METADATA`, `getPageMetadata`), liste les clés par page, et donne les bonnes pratiques pour les titres, descriptions et Open Graph.

### Guide IA — Contribuer au Frontend
Guide destiné aux agents IA pour créer, modifier ou étendre des features du frontend. Définit un workflow structuré en 6 étapes : analyse & découpage, régénération Orval, structure de la page (3 couches), implémentation des composants (formulaires, appels API, auth, toasts, navigation, responsive), traductions, et navigation. Inclut une checklist de validation complète.

---

[← Retour à la documentation principale](../README.md)
