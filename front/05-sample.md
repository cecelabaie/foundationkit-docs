# Sample

La page Sample (accessible via l'URL `/sample`) est une page qui permet aux développeurs de :

- **Visualiser l'UI/UX** du starter kit en action avec tous ses composants
- **Personnaliser le thème** selon les besoins spécifiques du projet
- **Tester les interactions** entre les différents composants et états

Cette page sert à la fois de vitrine des possibilités du kit et d'outil de développement.

## Structure de la page Sample

Elle est organisée en trois sections principales :

1. **Thème Builder** - Panneau latéral pour personnaliser les couleurs du thème
2. **Onglet Formulaires** - Démonstration des composants de formulaire
3. **Onglet Composants** - Présentation des éléments d'interface utilisateur

```tsx
// Structure simplifiée
<ThemeSampleProvider>
  {/* Panneau de personnalisation des couleurs */}
  <ThemeColorSample /> {/* Version desktop */}
  <ThemeColorSampleMobile /> {/* Version mobile */}
  
  {/* Onglets de démonstration */}
  <Tabs defaultValue="forms">
    <TabsList>
      <TabsTrigger value="forms">Formulaires</TabsTrigger>
      <TabsTrigger value="components">Composants</TabsTrigger>
    </TabsList>
    
    <TabsContent value="forms">
      <TabFormSample />
    </TabsContent>
    
    <TabsContent value="components">
      <TabComponentSample />
    </TabsContent>
  </Tabs>
</ThemeSampleProvider>
```

## Galerie des composants

La page Sample présente tous les composants du starter kit, organisés par catégories :

### Composants de formulaire

L'onglet Formulaires présente une collection complète d'inputs optimisés pour React Hook Form :

- **Champs textuels** - Text, Email, Password, TextArea
- **Sélecteurs** - Select, Combobox, Radio, Checkbox, Switch
- **Pickers** - Date, Calendar, ColorPicker
- **Upload** - Sélection et prévisualisation de fichiers

Un panneau de configuration permet de tester facilement les états obligatoires ou désactivés, et un affichage en temps réel montre les valeurs saisies.

### Composants d'interface

L'onglet Composants UI démontre les éléments visuels essentiels :

- **Boutons** - Tous les variants et états (avec animations de chargement)
- **Alertes et notifications** - Messages contextuels avec code couleur
- **Menus de navigation** - Différents styles d'interaction

> Pour avoir plus d'informations sur les composants, voir [Tous les composants](./02-components.md).

## Thème Builder

Le Thème Builder est un outil permettant d'adapter l'apparence visuelle du starter kit à vos besoins.

### ThemeSampleContext

La page utilise un contexte `ThemeSampleContext` qui :
- Gère les couleurs pour cette page sans affecter le reste de l'application
- Stocke les valeurs par défaut des modes clair et sombre
- Applique les changements en temps réel
- Permet de basculer entre les modes

### Fonctionnalités principales

- **Modification des couleurs** - Sélecteurs pour chaque élément du thème
- **Prévisualisation instantanée** - Les changements s'appliquent en temps réel
- **Support des modes clair et sombre** - Personnalisation pour chaque mode
- **Adaptation responsive** - Versions desktop et mobile disponibles
- **Opacité pour boutons link** - Particularité de style pour le fond du bouton / lien. Il a une opacité par défaut de 0.2 ajustable. Le bouton est utilisé dans le header et dans le calendrier.

### Récupération des valeurs pour votre projet

Pour appliquer les couleurs personnalisées à votre projet :

1. Cliquez sur le bouton "Get Colors" dans le panneau du Thème Builder
2. Dans le drawer qui s'ouvre, deux sections sont disponibles :
   - "Values for palette.css" - Format CSS pour le fichier de palette
   - "Values for ThemeSampleContext" - Format JavaScript pour les valeurs par défaut
3. **Important : Les deux formats doivent être copiés et appliqués**
4. Pour une application complète :
   - Copiez les valeurs CSS dans `palette.css` pour le thème global de l'application
   - Copiez les valeurs JavaScript dans `ThemeSampleContext.tsx` pour les valeurs par défaut de la page Sample

> Pour plus de détails sur la structure du système de thème, consultez [Light and Dark mode](./01-theme.md).