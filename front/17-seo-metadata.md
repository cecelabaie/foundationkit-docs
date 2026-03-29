# Guide de gestion des métadonnées et URLs canoniques

Ce guide explique comment utiliser le système de métadonnées mis en place pour améliorer le SEO du site.

## Utilitaire de métadonnées (`src/utils/metadata.ts`)

Quatre fonctions exportées :

- **`generateLayoutMetadata()`** — pour le layout racine uniquement, sans URL canonique. Retourne aussi `manifest: '/site.webmanifest'` pour le support PWA.
- **`generateMetadata(params)`** — pour les pages publiques indexées
- **`generateNoIndexMetadata(params)`** — pour les pages privées non indexées (appelle `generateMetadata` avec `noIndex: true`)
- **`getCanonicalUrl(path)`** — convertit un chemin relatif en URL absolue

Le format du titre généré est automatiquement `${title} | Mon projet`.

L'image Open Graph par défaut est `/web-app-manifest-512x512.png`.

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

| Page | Chemin | Fonction utilisée | Indexée |
|------|--------|-------------------|---------|
| Accueil | `/` | `generateMetadata` | ✓ |
| Login | `/login` | `generateNoIndexMetadata` | ✗ |
| Register | `/register` | `generateNoIndexMetadata` | ✗ |
| Forgot Password | `/forgot-password` | `generateNoIndexMetadata` | ✗ |
| New Password | `/forgot-password/new-password` | `generateNoIndexMetadata` | ✗ |
| Sample | `/sample` | `generateNoIndexMetadata` | ✗ |
| Logout | `/logout` | `generateNoIndexMetadata` | ✗ |
| Profile | `/profile` | `generateNoIndexMetadata` | ✗ |
| Update Password | `/profile/update-password` | `generateNoIndexMetadata` | ✗ |
| Private | `/private` | `generateNoIndexMetadata` | ✗ |
| Not Found | — | aucune | — |

## Comment ajouter des métadonnées à une nouvelle page

### Page publique indexée

```typescript
// ⚠️ Utiliser un alias pour éviter le conflit avec generateMetadata de Next.js
import { generateMetadata as generateMeta } from '@/utils/metadata';

export const metadata = generateMeta({
  title: 'Titre de ma page',          // Suffixé automatiquement en "Titre | Mon projet"
  description: 'Description SEO',
  canonical: '/chemin-de-ma-page',    // OBLIGATOIRE
  ogTitle: 'Titre pour les réseaux sociaux',    // Optionnel
  ogDescription: 'Description pour le partage', // Optionnel
  ogImage: '/images/og-image.jpg',              // Optionnel (défaut: /web-app-manifest-512x512.png)
});
```

### Page privée non indexée

```typescript
import { generateNoIndexMetadata } from '@/utils/metadata';

export const metadata = generateNoIndexMetadata({
  title: 'Mon compte',
  description: 'Espace personnel utilisateur',
  canonical: '/mon-compte',
});
```

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
