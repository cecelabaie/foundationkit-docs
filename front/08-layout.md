# Layout de base

## Structure générale

L'application utilise un système de layouts imbriqués basé sur l'App Router de Next.js :

1. **Layout racine** (`src/app/layout.tsx`)
   - S'applique à toutes les pages
   - Configure la police, les métadonnées et les scripts
   - Initialise les providers via `Providers`

2. **Layout authentifié** (`src/app/(logged)/layout.tsx`)
   - S'applique aux pages nécessitant une authentification
   - Utilise `AuthGuard`

3. **Layout non-authentifié** (`src/app/(unlogged)/layout.tsx`)
   - S'applique aux pages pour utilisateurs non connectés
   - Utilise `UnloggedGuard`

4. **Layout reset password** (`src/app/(unlogged)/(reset-password)/forgot-password/new-password/layout.tsx`)
   - Utilise `ForgotPasswordGuard` pour vérifier la validité du token

5. **Layout verify account** (`src/app/(unlogged)/(verify-account)/layout.tsx`)
   - Utilise `VerifyAccountGuard` pour vérifier le compte après inscription

## Layout racine

```tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fr">
      <head>
        <Script id="theme-sanitize" strategy="beforeInteractive" {...} />
      </head>
      <body className={`${spaceGrotesk.className} ${spaceGrotesk.variable} antialiased`}>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

- **Police** : `Space_Grotesk` via le système de polices Next.js
- **Script `theme-sanitize`** : exécuté avant l'hydratation React pour éviter le FOUC (flash de thème). Vérifie que `localStorage.theme` vaut `'light'` ou `'dark'`, sinon force `dark`.

## AuthPageLayout

**Fichier :** `src/layout/auth-page-layout.tsx`

Layout partagé utilisé par toutes les pages auth/publiques : login, register, forgot-password, new-password, contact.

Gère automatiquement l'affichage mobile/desktop via `useIsScreenBelowBreakpoint('md')` : un seul sous-composant est rendu dans le DOM à la fois.

```tsx
import { AuthPageLayout } from '@/layout/auth-page-layout';

export default function LoginView() {
  const { t } = useTranslation('login');

  return (
    <AuthPageLayout
      illustration={<LoginIllustration ariaLabel="..." />}
      title={t('form.title-login', { defaultValue: 'Connexion' })}
      subtitle={t('form.subtitle-login', { defaultValue: 'Bienvenue...' })}
    >
      <LoginForm />
    </AuthPageLayout>
  );
}
```

**Props :**

| Prop | Type | Description |
|---|---|---|
| `illustration` | `ReactNode` | SVG illustration affiché dans la colonne gauche (desktop) ou le hero (mobile) |
| `title` | `string` | Titre de la page |
| `subtitle` | `string` | Sous-titre / description |
| `children` | `ReactNode` | Formulaire ou contenu principal |

**Comportement :**
- **Mobile (`< md`)** : hero illustré (`bg-muted`) en haut + formulaire plein écran (`bg-background`) en bas, sans Card
- **Desktop (`≥ md`)** : Card split : illustration à gauche (45%), formulaire à droite

## Pattern mobile/desktop avec le hook

Pour les vues plus complexes (ex: `HomeView`), le même pattern peut être appliqué manuellement en créant deux sous-composants et en utilisant le hook :

```tsx
import { useIsScreenBelowBreakpoint } from '@/hooks/useIsScreenBelowBreakpoint';

function HomeViewMobile() { /* ... */ }
function HomeViewDesktop() { /* ... */ }

export default function HomeView() {
  const isMobile = useIsScreenBelowBreakpoint('lg');

  if (isMobile) return <HomeViewMobile />;
  return <HomeViewDesktop />;
}
```

L'avantage par rapport aux classes `md:hidden` / `hidden md:flex` : un seul composant est dans le DOM, pas deux.

## Providers

Le composant `Providers` (`src/providers/providers.tsx`) encapsule l'application avec tous les contextes et fournit la structure de page globale (header, footer, barre de progression). **Voir [Contexts](./13-contexts.md)** pour la liste complète.

## Header

Le header est responsif et utilise `useIsScreenBelowBreakpoint` pour choisir quelle version rendre :

```tsx
export default function HeaderView() {
  const isBelowLargeScreen = useIsScreenBelowBreakpoint('lg');

  if (isBelowLargeScreen) return <HeaderMobile />;
  return <HeaderDesktop />;
}
```

### Header Desktop

**Fichier :** `src/sections/header/header-desktop.tsx`

- **Gauche** : `LeftSideHeader` : lien Accueil + trois menus déroulants (pages non connectées, pages connectées, pages publiques)
- **Droite** : toggle thème, `LanguageSelector`, lien Connexion ou `UserMenu` (si connecté)

### Header Mobile

**Fichier :** `src/sections/header/header-mobile.tsx`

- **Gauche** : `HeaderDrawer` (burger → `Drawer` avec `LeftSideHeader`) + icône maison
- **Droite** : identique au desktop

Les liens du menu mobile utilisent les mêmes constantes de navigation que le desktop (`usersConnectedPages`, `usersNotConnectedPages`, `publicPages` depuis `src/constants/header/nav.tsx`).

### LeftSideHeader (partagé)

**Fichier :** `src/sections/header/left-side-header.tsx`

Composant partagé desktop/mobile. Affiche les trois menus de navigation définis dans `src/constants/header/nav.tsx`.

### Structure de `src/constants/header/nav.tsx`

| Export | Menu affiché |
|--------|-------------|
| `usersNotConnectedPages` | Login, register, forgot-password |
| `usersConnectedPages` | Profil, pages privées |
| `publicPages` | Contact, 404 |

**Ajouter une entrée de navigation :**

1. Ajouter le chemin dans `APP_PATHS` (`src/constants/constants.ts`)
2. Importer l'icône Iconify souhaitée dans `nav.tsx`
3. Ajouter l'objet dans le tableau approprié
4. Ajouter la clé de traduction dans les fichiers JSON i18n

## Footer

**Fichier :** `src/sections/footer/view/footer-view.tsx`

Pied de page simple avec liens légaux et copyright.

## Constantes de layout

Les dimensions du header et du footer sont définies dans `src/constants/constants.ts` :

```tsx
export const layout = {
  header: '56px',
  footer: '116px',
};
```

Utilisées pour calculer la hauteur du contenu principal :

```tsx
<div style={{ minHeight: `calc(100vh - ${layout.header})` }}>
  {/* Contenu */}
</div>
```
