# Shadcn

Le projet utilise **Shadcn/ui** pour les composants UI de base, tous personnalisés avec le thème du projet.

## Configuration

**Fichier :** `components.json`

- **`style: "new-york"`** - Style de base (les styles ont été personnalisés)
- **`cssVariables: true`** - Utilise les variables CSS (compatibles avec le système de thème)
- **`aliases`** - Chemins d'import (`@/components`, `@/lib`, etc.)

## Composants installés

**Dossier :** `src/components/ui/`

37 composants UI installés et personnalisés (Shadcn + custom). **Voir [Tous les composants](./02-components.md)** pour la liste complète et l'utilisation.

Les composants ont été modifiés pour :
- Utiliser les variables CSS du thème (light/dark automatique)
- S'adapter aux standards visuels du projet

Pour explorer tous les composants et leurs variantes de façon interactive, lancer le **Storybook** :

```bash
npm run storybook
# → http://localhost:6006
```

## Ajouter un composant Shadcn

```bash
npx shadcn@latest add [nom-du-composant]
```

Le composant sera installé dans `src/components/ui/`. Penser à l'adapter aux variables CSS du thème après installation.
