# Shadcn

Le projet utilise **Shadcn/ui** pour les composants UI de base, tous personnalisés avec le thème du projet.

## Configuration

**Fichier :** `components.json`

- **`style: "new-york"`** - Style de composants utilisé de base (Attention les styles ont été modifié)
- **`cssVariables: true`** - Utilise les variables CSS (compatibles avec le système de thème)
- **`aliases`** - Chemins d'import (`@/components`, `@/lib`, etc.)

## Composants utilisés

**Dossier :** `src/components/ui/`

27 composants Shadcn installés et personnalisés. **Voir [Tous les composants](./02-components.md)** pour la liste complète et l'utilisation.

Les composants ont été modifiés pour :
- Utiliser les variables CSS du thème (light/dark automatique)
- S'adapter aux standards du projet (style)

## Ajouter un composant Shadcn

```bash
npx shadcn@latest add [nom-du-composant]
```

Le composant sera automatiquement :
- Installé dans `src/components/ui/`
- Pensez à override le nouveau composant
