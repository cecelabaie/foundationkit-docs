# Tailwind

Le projet utilise **Tailwind CSS v4**.

## Configuration

**Fichiers principaux :**

- `postcss.config.mjs` - Configure le plugin PostCSS pour Tailwind v4
- `tailwind.config.ts` - Minimal : déclare uniquement les chemins de contenu pour le scan des classes
- `src/assets/styles/theme.css` - Configuration des couleurs et tokens sémantiques via `@theme inline`

### Architecture Tailwind v4

Avec Tailwind CSS v4, le thème et les couleurs se configurent dans les fichiers CSS via la directive `@theme inline`. Le `tailwind.config.ts` n'est utilisé que pour définir les chemins de contenu.

**Fichier:** `src/assets/styles/theme.css`

```css
@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-muted: var(--muted);
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

## Système de thème

Le thème (couleurs, variables, modes clair/sombre) est entièrement géré via les variables CSS. **Voir [Light and Dark mode](./01-theme.md)** pour toute la configuration du thème.

## Utilisation

Les classes Tailwind sont disponibles avec les variables définies dans `theme.css` :

```tsx
<div className="bg-primary text-foreground">Contenu</div>
<div className="bg-secondary text-accent">Autre contenu</div>
```

Les variables CSS sont automatiquement converties en classes utilitaires grâce au préfixe `--color-`.

## Explorer visuellement les classes du thème

Le **Storybook** présente tous les composants rendus avec le thème actif, ce qui permet de vérifier l'apparence des couleurs et variables en contexte réel :

```bash
npm run storybook
# → http://localhost:6006
```
