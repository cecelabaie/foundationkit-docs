# Guide de gestion des métadonnées et URLs canoniques

Ce guide explique comment utiliser le système de métadonnées mis en place pour améliorer le SEO du site.

## Utilitaire de métadonnées (`src/utils/metadata.ts`)

Ce fichier centralise tout le système de métadonnées : configuration du site, metadata de chaque page, et fonctions utilitaires.

### SITE_CONFIG

Contient les valeurs globales du site utilisées dans toutes les metadata :

```typescript
export const SITE_CONFIG = {
  name: 'Weeklog',
  title: 'Weeklog — Le suivi de temps qui ne prend pas de temps',
  description: 'Loggez vos tâches par projet, visualisez vos heures et exportez votre résumé de semaine en un clic.',
  locale: 'fr_FR',
  ogImage: '/web-app-manifest-512x512.png',
  authors: [{ name: 'Weeklog' }],
};
```

### getPageMetadata (à utiliser dans les pages)

**Fonction principale** pour ajouter des métadonnées à une page. Toutes les metadata de pages sont centralisées dans `PAGES_METADATA` dans ce même fichier.

```typescript
export function getPageMetadata(page: string): Metadata
```

Un **ESLint bloquant** vérifie que chaque `page.tsx` sous `src/app/` appelle `getPageMetadata`. Le build échoue si absent.

### Fonctions utilitaires (internes)

- **`generateLayoutMetadata()`** — pour le layout racine uniquement, sans URL canonique. Retourne aussi `manifest: '/site.webmanifest'` pour le support PWA.
- **`generateMetadata(params)`** — construit les métadonnées d'une page (utilisée en interne par `PAGES_METADATA`)
- **`generateNoIndexMetadata(params)`** — appelle `generateMetadata` avec `noIndex: true`
- **`getCanonicalUrl(path)`** — convertit un chemin relatif en URL absolue

Le format du titre généré est automatiquement `${title} | Weeklog`.

L'image Open Graph par défaut est `SITE_CONFIG.ogImage`.

### Favicon et icônes

Les fichiers de favicon sont dans `public/` et servis statiquement :

| Fichier | Usage |
|---------|-------|
| `favicon.ico` | Favicon principal (chargé automatiquement par les navigateurs via `/favicon.ico`) |
| `favicon.svg` | Favicon vectoriel pour navigateurs modernes |
| `favicon-96x96.png` | Favicon PNG 96×96 |
| `apple-touch-icon.png` | Icône pour iOS (chargée automatiquement via `/apple-touch-icon.png`) |
| `web-app-manifest-192x192.png` | Icône PWA 192×192 |
| `web-app-manifest-512x512.png` | Icône PWA 512×512 (aussi image OG par défaut) |

Le fichier `site.webmanifest` (dans `public/`) déclare les icônes PWA et est référencé via `generateLayoutMetadata()`.

> Nous vous conseillons d'utiliser [RealFaviconGenerator.net](https://realfavicongenerator.net) pour la simplicité de génération de l'ensemble des fichiers nécessaires (`.ico`, `.svg`, `.png`, `apple-touch-icon`, manifeste PWA) à partir d'une seule image source.

### État réel par page

| Page | Chemin | Clé `getPageMetadata` | Indexée |
|------|--------|-----------------------|---------|
| Accueil | `/` | `'home'` | ✓ |
| Login | `/login` | `'login'` | ✗ |
| Register | `/register` | `'register'` | ✗ |
| Forgot Password | `/forgot-password` | `'forgotPassword'` | ✗ |
| New Password | `/forgot-password/new-password` | `'newPassword'` | ✗ |
| Logout | `/logout` | `'logout'` | ✗ |
| Profile | `/profile` | `'profile'` | ✗ |
| Update Password | `/profile/update-password` | `'updatePassword'` | ✗ |
| Private | `/private` | `'private'` | ✗ |
| Projects | `/projects` | `'projects'` | ✗ |
| Reports | `/reports` | `'reports'` | ✗ |
| Contact | `/contact` | `'contact'` | ✗ |
| Not Found | — | aucune | — |

## Comment ajouter des métadonnées à une nouvelle page

### 1. Ajouter la clé dans `PAGES_METADATA` (`src/utils/metadata.ts`)

```typescript
// Page publique indexée
home: generateMetadata({
  title: 'Mon titre',
  description: 'Description SEO',
  canonical: '/ma-page',
  ogTitle: 'Titre OG optionnel',
  ogDescription: 'Description OG optionnelle',
  ogImage: '/images/og.jpg', // Optionnel (défaut: SITE_CONFIG.ogImage)
}),

// Page privée non indexée
maPage: generateNoIndexMetadata({
  title: 'Mon titre',
  description: 'Description',
  canonical: '/ma-page',
}),
```

### 2. Appeler `getPageMetadata` dans la page

```typescript
import { getPageMetadata } from '@/utils/metadata';

export const metadata = getPageMetadata('maPage');
```

> Si la clé n'existe pas dans `PAGES_METADATA`, une erreur est levée au runtime. La règle ESLint bloque le build si `getPageMetadata` est absent de la page.

## Bonnes pratiques

### URLs canoniques

- Chaque page a sa propre URL canonique unique
- L'URL canonique correspond exactement au chemin de la page
- Ne pas réutiliser la même URL canonique pour plusieurs pages
- Ne pas omettre l'URL canonique (paramètre requis)

### Indexation

- **Indexer** : Page d'accueil et pages publiques de contenu (`/`)
- **Ne pas indexer** : Pages d'authentification et flux reset password (login, register, forgot-password, new-password), pages privées, outils internes (sample, profile, logout, etc.)

### Titres et descriptions

- **Titres** : Descriptifs, uniques, 50-60 caractères max (sans compter le suffixe `| Mon projet`)
- **Descriptions** : Engageantes, uniques, 150-160 caractères max

### Open Graph

- **Images** : Format 1200x630px recommandé, taille < 8MB
- **Titres** : Peuvent être différents du titre SEO
- **Descriptions** : Adaptées au partage social

## Sitemap et robots.txt

Le package `next-sitemap` génère automatiquement le `sitemap.xml` et le `robots.txt` lors du build, via le script `postbuild` dans `package.json`.

```json
"postbuild": "next-sitemap"
```

Il est exécuté automatiquement après `next build` (via `build:linux` ou `build:win`). La configuration se trouve dans `next-sitemap.config.js` à la racine du projet frontend.

### Configuration actuelle

```js
module.exports = {
  siteUrl: process.env.NEXT_PUBLIC_APP_URL,
  generateRobotsTxt: true,
  generateIndexSitemap: false,

  // Pages exclues du sitemap
  exclude: [
    '/logout*', '/login*', '/register*', '/forgot-password',
    '/profile*', '/api/*', '/admin/*', '/private*', '/sample*',
  ],

  robotsTxtOptions: {
    policies: [
      {
        userAgent: '*',
        allow: '/',
        disallow: [
          '/logout', '/login', '/register', '/forgot-password',
          '/profile', '/api', '/admin', '/private', '/sample',
        ],
      },
    ],
  },
};
```

### Ce que ça génère

- **`sitemap.xml`** — contient uniquement les pages publiques non exclues (actuellement : `/` uniquement). L'URL de base vient de `NEXT_PUBLIC_APP_URL`.
- **`robots.txt`** — autorise `/` et interdit explicitement les chemins privés et outils internes.

> Si tu ajoutes une nouvelle page publique indexée, elle sera automatiquement incluse dans le sitemap à la prochaine build. Si c'est une page privée, ajoute son chemin dans `exclude` et dans `robotsTxtOptions.policies[0].disallow`.

## Vérification

Pour vérifier que les métadonnées sont correctement configurées :

1. **En développement** : Inspecter le `<head>` de la page
2. **Outils SEO** : Utiliser Google Search Console, Lighthouse
3. **Open Graph** : Tester avec Facebook Debugger ou Twitter Card Validator
4. **Sitemap** : Inspecter `public/sitemap.xml` après le build
