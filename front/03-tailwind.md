# Tailwind

Le projet utilise **Tailwind CSS v4**.

## Configuration

**Fichiers principaux :**

- `postcss.config.mjs` - Configure le plugin PostCSS pour Tailwind v4
- `src/assets/styles/tailwind-color.css` - Configuration moderne avec `@theme inline`
- `tailwind.config.ts` - Configuration minimale pour les chemins de contenu (moins utilisé avec v4)

### Architecture Tailwind v4

Avec Tailwind CSS v4, la configuration se fait principalement dans les fichiers CSS via la directive `@theme inline` :

**Fichier:** `src/assets/styles/tailwind-color.css`

```css
@import 'tailwindcss';

@theme inline {
  --radius: 0.5rem;
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-secondary: var(--secondary);
  /* ... autres variables de couleur */
}
```

**Important :** Le préfixe `--color-` est obligatoire pour les couleurs dans Tailwind v4. Les variables CSS sans ce préfixe ne seront pas disponibles comme classes utilitaires.

### PostCSS Configuration

**Fichier:** `postcss.config.mjs`

```js
const config = {
  plugins: ['@tailwindcss/postcss'],
};

export default config;
```

Cette configuration utilise le nouveau plugin PostCSS de Tailwind v4 qui remplace l'ancien système.

### Tailwind Config (optionnel avec v4)

**Fichier:** `tailwind.config.ts`

```ts
const config: Config = {
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

Avec Tailwind v4, ce fichier est principalement utilisé pour définir les chemins de contenu. Les personnalisations de thème se font dans les fichiers CSS.

## Système de thème

Le thème (couleurs, variables, modes clair/sombre) est entièrement géré via les variables CSS. **Voir [Light and Dark mode](./01-theme.md)** pour toute la configuration du thème.

## Utilisation

Les classes Tailwind sont disponibles avec les variables définies dans `tailwind-color.css` :

```tsx
<div className="bg-primary text-foreground">Contenu</div>
<div className="bg-secondary text-accent">Autre contenu</div>
```

Les variables CSS sont automatiquement converties en classes utilitaires grâce au préfixe `--color-`.
