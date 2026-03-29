# Layout de base

## Structure générale

L'application utilise un système de layouts imbriqués basé sur le système de routage App Router de Next.js :

1. **Layout racine** (`src/app/layout.tsx`)
   - S'applique à toutes les pages de l'application
   - Définit la structure HTML de base
   - Configure la police, les métadonnées et les scripts
   - Initialise les providers via le composant `Providers`

2. **Layout authentifié** (`src/app/(logged)/layout.tsx`)
   - S'applique aux pages nécessitant une authentification
   - Utilise `AuthGuard` pour protéger les routes

3. **Layout non-authentifié** (`src/app/(unlogged)/layout.tsx`)
   - S'applique aux pages accessibles uniquement aux utilisateurs non connectés
   - Utilise `UnloggedGuard` pour rediriger les utilisateurs connectés

4. **Layout de réinitialisation de mot de passe** (`src/app/(unlogged)/(reset-password)/forgot-password/new-password/layout.tsx`)
   - S'applique à la page de création de nouveau mot de passe
   - Utilise `ForgotPasswordGuard` pour vérifier la validité du token

## Layout racine

Le layout racine (`src/app/layout.tsx`) est le point d'entrée principal de l'application :

```tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fr">
      <head>
        {/* Préchargement du CDN Iconify */}
        <link rel="preconnect" href="https://api.iconify.design" />
        <link rel="dns-prefetch" href="https://api.iconify.design" />
        <Script id="theme-sanitize" strategy="beforeInteractive" {...} />
      </head>
      <body className={`${spaceGrotesk.className} ${spaceGrotesk.variable} antialiased`}>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

Caractéristiques principales :
- **Police personnalisée** : Utilisation de `Space_Grotesk` via le système de polices optimisées de Next.js qui gère automatiquement le préchargement, l'optimisation et le fallback.
- **Script d'initialisation du thème** (`theme-sanitize`) : Script exécuté avec `strategy="beforeInteractive"` qui s'exécute avant l'hydratation React pour éviter le flash de contenu non stylisé (FOUC). Il vérifie que la valeur dans `localStorage.theme` est bien dans `['light', 'dark']` — sinon il force **`dark`** et aligne la classe sur `<html>`. C'est ce mécanisme qui fixe le thème par défaut ; le `ThemeProvider` (`next-themes`) n'utilise pas de prop `defaultTheme`, il lit la même clé `theme` après coup.
- **Optimisation des ressources externes** : Préchargement du CDN Iconify avec `preconnect` et `dns-prefetch` pour améliorer les performances de chargement des icônes.

## Providers

Le composant `Providers` (`src/providers/providers.tsx`) encapsule l'application avec tous les contextes et fournit la structure de page globale (header, footer, barre de progression). **Voir [Contexts](./13-contexts.md)** pour la liste complète des providers, leur ordre d'imbrication et leur configuration.

## Header

Le header est responsif et s'adapte aux tailles d'écran :

### Structure du header

Le composant `HeaderView` (`src/sections/header/view/header-view.tsx`) utilise `useIsScreenBelowBreakpoint` pour déterminer quelle version du header afficher :

```tsx
export default function HeaderView() {
  const isBelowLargeScreen = useIsScreenBelowBreakpoint('lg');

  if (isBelowLargeScreen) {
    return <HeaderMobile />;
  }

  return <HeaderDesktop />;
}
```

### Header Desktop

**Fichier :** `src/sections/header/header-desktop.tsx`

Structure en deux zones :

- **Gauche** : `LeftSideHeader` — lien Accueil + trois menus déroulants (pages non connectées, pages connectées, pages publiques). La liste de pages vient de `src/constants/header/nav.tsx`.
- **Droite** : bouton toggle thème, `LanguageSelector`, lien Connexion (si non connecté) ou avatar utilisateur avec `DropdownMenu` (si connecté). Le dropdown utilisateur contient un lien vers le profil et un bouton de déconnexion.

### Header Mobile

**Fichier :** `src/sections/header/header-mobile.tsx`

Structure en deux zones :

- **Gauche** : `HeaderDrawer` (burger menu qui ouvre un `Drawer` depuis la gauche contenant `LeftSideHeader`) + icône maison vers `/`.
- **Droite** : identique au desktop (toggle thème, `LanguageSelector`, login ou avatar).

### HeaderDrawer

**Fichier :** `src/sections/header/drawer.tsx`

Drawer latéral gauche (max 300px) utilisé uniquement en mobile. Il contient :
- Un header avec titre et description (traduits via les clés `nav.drawer.*`)
- `LeftSideHeader` — même composant que le desktop, ferme le drawer au clic via `onCloseDrawer`
- Un footer avec un lien vers `APP_PATHS.LANDING_PAGE`

### LeftSideHeader (partagé)

**Fichier :** `src/sections/header/left-side-header.tsx`

Composant partagé entre desktop et mobile. Affiche :
- Lien Accueil
- Menu déroulant "pages non connectées" (`usersNotConnectedPages`)
- Menu déroulant "pages connectées" (`usersConnectedPages`)
- Menu déroulant "pages publiques" (`publicPages`)

Les pages de chaque menu sont définies dans `src/constants/header/nav.tsx`. C'est dans ce fichier qu'il faut ajouter ou modifier des entrées de navigation.

### Structure de `src/constants/header/nav.tsx`

Le fichier exporte trois tableaux, correspondant aux trois menus déroulants du header :

| Export | Menu affiché |
|--------|-------------|
| `usersNotConnectedPages` | Pages accessibles sans compte (login, register, forgot-password) |
| `usersConnectedPages` | Pages réservées aux connectés (profil, logout, pages privées) |
| `publicPages` | Pages publiques diverses (sample, 404) |

**Structure d'une entrée :**

```tsx
{
  title: 'nav.connected-pages.profile', // Clé i18n (namespace "common")
  path: APP_PATHS.PROFILE,             // Chemin de la route (depuis constants)
  needToBeConnected: false,            // Réservé pour usage futur (non utilisé actuellement)
  needToBeDisconnected: false,         // Réservé pour usage futur (non utilisé actuellement)
  inLeftSide: true,                    // Afficher dans la nav gauche (toujours true)
  icon: (                              // Composant icône Iconify
    <Icon className="text-inherit" icon={userBroken} width={24} height={24} />
  ),
  defaultValue: 'Profil',             // Texte de fallback si la clé i18n n'est pas chargée
}
```

**Ajouter une entrée de navigation :**

1. Ajouter le chemin dans `src/constants/constants.ts` (objet `APP_PATHS`)
2. Importer l'icône Iconify souhaitée en haut du fichier nav.tsx
3. Ajouter l'objet dans le tableau approprié (`usersConnectedPages`, `usersNotConnectedPages` ou `publicPages`)
4. Ajouter la clé de traduction dans les fichiers JSON i18n correspondants

```tsx
// Exemple : ajouter "Dashboard" dans les pages connectées
import dashboardBroken from '@iconify-icons/solar/chart-square-broken';

export const usersConnectedPages = [
  // ... entrées existantes
  {
    title: 'nav.connected-pages.dashboard',
    path: APP_PATHS.DASHBOARD,
    needToBeConnected: false,
    needToBeDisconnected: false,
    inLeftSide: true,
    icon: <Icon className="text-inherit" icon={dashboardBroken} width={24} height={24} />,
    defaultValue: 'Dashboard',
  },
];
```

## Footer

Le composant `FooterView` (`src/sections/footer/view/footer-view.tsx`) affiche un pied de page simple :

```tsx
export default function FooterView() {
  const { t } = useTranslation('footer');

  return (
    <footer className="relative bg-background z-[2]">
      <div className="bg-primary/10 py-8 md:py-12 px-4 md:px-8 border-t border-border">
        <div className="max-w-[1400px] mx-auto flex justify-between items-center flex-wrap gap-4 md:gap-8">
          <ul className="flex gap-4 md:gap-8 list-none flex-wrap">
            <li><Link prefetch={false} href="/cgv">{t('links.cgv', { defaultValue: 'CGV' })}</Link></li>
            <li><Link prefetch={false} href="/mentions-legales">{t('links.legal-mentions', { defaultValue: 'Mentions légales' })}</Link></li>
            {/* Autres liens */}
          </ul>
          <p className="text-muted text-sm md:text-base">
            {t('copyright', { year: new Date().getFullYear() })}
          </p>
        </div>
      </div>
    </footer>
  );
}
```

## Constantes de layout

Les dimensions du header et du footer sont définies dans `src/constants/constants.ts` :

```tsx
export const layout = {
  header: '73px',
  footer: '116px',
};
```

Ces constantes sont utilisées pour calculer la hauteur du contenu principal, notamment pour les pages qui doivent occuper toute la hauteur de l'écran :

```tsx
<div style={{ minHeight: `calc(100vh - ${layout.header} - ${layout.footer})` }}>
  {/* Contenu */}
</div>
```
