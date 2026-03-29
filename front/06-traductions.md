# Traductions

Le système de traduction du starter kit utilise une combinaison de bibliothèques pour offrir une solution complète d'internationalisation. Il permet de gérer facilement les textes dans différentes langues et les messages de validation.

## Packages utilisés

Le système de traduction repose sur plusieurs packages :

- **i18next** - Bibliothèque principale d'internationalisation
- **react-i18next** - Intégration de i18next avec React
- **i18next-browser-languagedetector** - Importé mais non utilisé activement (la langue est gérée manuellement via localStorage)
- **i18next-chained-backend** - Permet d'utiliser plusieurs backends en cascade
- **i18next-http-backend** - Chargement des traductions via HTTP
- **i18next-localstorage-backend** - Stockage des traductions dans le localStorage
- `**utils/i18n.zod.ts`** - Implémentation locale de la carte d'erreurs Zod (fork adapté de `zod-i18n-map` pour compatibilité Zod v4, récupéré depuis GitHub)

## Configuration

La configuration du système de traduction se trouve dans le fichier `src/i18n.ts`.

### Namespaces

Les traductions sont organisées en namespaces (fichiers séparés) pour une meilleure organisation :

- **header** - Traductions pour l'en-tête (namespace par défaut)
- **footer** - Traductions pour le pied de page
- **login** - Traductions pour la page de connexion
- **logout** - Traductions pour la page de déconnexion
- **register** - Traductions pour la page d'inscription
- **profile** - Traductions pour la page de profil
- **update-password** - Traductions pour la mise à jour du mot de passe
- **not-found** - Traductions pour la page 404
- **forgot-password** - Traductions pour la récupération de mot de passe
- **forgot-password-guard** - Traductions pour le guard de récupération de mot de passe
- **forgot-password-new-password** - Traductions pour la page de nouveau mot de passe
- **zod** - Traductions pour les messages de validation
- **common** - Traductions communes utilisées dans plusieurs pages
- **form** - Traductions spécifiques aux formulaires
- **sample** - Traductions pour la page d'exemple
- **home** - Traductions pour la page d'accueil
- **verify-account** - Traductions pour la page de vérification de compte
- **private** - Traductions pour la page protégée

L'utilisation de namespaces permet de :

- Charger uniquement les traductions nécessaires pour chaque page
- Organiser logiquement les traductions par fonctionnalité
- Éviter les conflits de clés entre différentes parties de l'application

### Backend chaîné

Le système utilise un backend chaîné pour optimiser les performances :

```typescript
backend: {
  backends: [
    LocalStorageBackend, // Premier: essaie le localStorage
    HttpApi, // Deuxième: si pas trouvé, charge depuis le serveur
  ],
  backendOptions: [
    {
      // Options pour LocalStorageBackend
      expirationTime: 7 * 24 * 60 * 60 * 1000, // 1 semaine
      compression: true,
      defaultVersion: '1.1',
      store: typeof window !== 'undefined' ? window.localStorage : undefined,
    },
    {
      // Options pour HttpApi
      loadPath: '/locales/{{lng}}/{{ns}}.json',
    },
  ],
}
```

#### Comment cela fonctionne dans le projet

Dans ce projet, le système de traduction fonctionne de la façon suivante :

1. **Premier chargement de l'application** :
  - Quand un utilisateur visite l'application pour la première fois, les fichiers de traduction ne sont pas encore dans son localStorage
  - Next.js charge les fichiers JSON depuis le dossier `public/locales/` (par exemple, `public/locales/fr/header.json`)
  - Ces traductions sont ensuite sauvegardées dans le localStorage avec une durée de validité d'une semaine
2. **Visites suivantes** :
  - Lors des visites suivantes, l'application récupère directement les traductions depuis le localStorage
  - Cela évite de recharger les fichiers depuis le serveur, rendant l'application plus rapide
  - Si une traduction n'est pas trouvée dans le localStorage, elle est rechargée depuis le serveur
3. **Mise à jour des traductions** :
  - Si vous modifiez les fichiers de traduction dans `public/locales/`, vous pouvez forcer un rechargement en changeant la valeur de `defaultVersion`
  - Après une semaine, les traductions expirent automatiquement et sont rechargées depuis le serveur
4. **Changement de langue** :
  - Quand l'utilisateur change de langue via le sélecteur, les nouvelles traductions sont chargées selon le même processus
  - Si elles sont déjà dans le localStorage, elles sont utilisées immédiatement
  - Sinon, elles sont chargées depuis le serveur puis mises en cache

Cette approche permet d'avoir une application rapide tout en gardant les traductions à jour.

## Structure des fichiers

Les fichiers de traduction sont stockés dans `public/locales/[langue]/[namespace].json`.

Exemple de structure :

```
public/locales/
├── en/
│   ├── header.json
│   ├── login.json
│   └── ...
└── fr/
    ├── header.json
    ├── login.json
    └── ...
```

Exemple de fichier de traduction (`header.json`) :

```json
{
  "category": "Catégories",
  "login": "Connexion",
  "nav": {
    "login-title": "Connexion",
    "home-title": "Accueil"
  }
}
```

> **⚠️ Exception — `zod.json`** : Contrairement aux autres fichiers de traduction qui utilisent le format **kebab-case**, le fichier `zod.json` utilise le format **snake_case** (`invalid_type`, `too_small`, `not_inclusive`…). Ces clés correspondent directement aux codes d'erreur natifs de Zod v4 (`issue.code`). Ne pas les convertir en kebab-case, la traduction ne fonctionnerait plus.

## Sélecteur de langue

Le composant `LanguageSelector` permet de changer la langue de l'application. La langue est gérée manuellement plutôt que par détection automatique du navigateur.

```tsx
// components/translation/language-selector.tsx
export default function LanguageSelector() {
  // Configuration des langues disponibles
  const langs = [
    {
      value: 'fr',
      label: 'French',
      icon: <Icon icon={flagFr} width={24} height={24} />,
    },
    {
      value: 'en',
      label: 'English',
      icon: <Icon icon={flagUs} width={24} height={24} />,
    },
  ];

  // Initialisation de la langue
  useEffect(() => {
    if (i18n.language !== currentLangValue) {
      const storedLang = localStorage.getItem('i18nextLng');
      
      // Si c'est la première visite, utiliser la langue par défaut (fr)
      if (!storedLang) {
        localStorage.setItem('i18nextLng', currentLangValue);
        i18n.changeLanguage(currentLangValue);
      } 
      // Sinon utiliser la langue stockée si elle est valide
      else if (langs.find((lang) => lang.value === storedLang)) {
        setCurrentLangValue(storedLang);
        i18n.changeLanguage(storedLang);
      }
      
      // Mise à jour de l'en-tête Axios
      AXIOS_INSTANCE.defaults.headers.common['Accept-Language'] = i18n.language;
    }
  }, [currentLangValue, langs]);

  // Changement de langue
  const handleChangeLang = (newLang: string) => {
    if (newLang !== currentLangValue) {
      setCurrentLangValue(newLang);
      i18n.changeLanguage(newLang);
      // Mise à jour de l'en-tête Axios pour les réponses API traduites
      AXIOS_INSTANCE.defaults.headers.common['Accept-Language'] = newLang;
    }
  };
  
  // ...reste du composant
}
```

Le sélecteur :

1. Affiche un menu déroulant avec les drapeaux des langues disponibles
2. Utilise le localStorage pour stocker et récupérer la langue préférée de l'utilisateur
3. Permet de changer manuellement la langue de l'application
4. Met à jour l'en-tête HTTP pour que les réponses API soient dans la bonne langue

> **Note :** Bien que le package `i18next-browser-languagedetector` soit importé, la détection automatique de la langue du navigateur n'est pas activement utilisée. La langue est plutôt gérée manuellement avec une préférence pour le français par défaut.

## Intégration avec Zod

Le système intègre un fichier local `utils/i18n.zod.ts` — fork adapté de `zod-i18n-map` mis à jour pour Zod v4 — pour traduire automatiquement les messages de validation.

```typescript
import { makeZodI18nMap } from './utils/i18n.zod';

// Fonction d'application de la carte d'erreurs
const applyZodErrorMap = () => {
  z.config({
    customError: makeZodI18nMap({
      t: i18n.t,
      ns: 'zod',
    }),
  });
};

// Appliquée au démarrage
applyZodErrorMap();

// Reappliquée automatiquement à chaque changement de langue
i18n.on('languageChanged', applyZodErrorMap);

// L'attribut lang du HTML est aussi mis à jour à chaque changement de langue
i18n.on('languageChanged', (lng) => {
  if (typeof window !== 'undefined') {
    document.documentElement.lang = lng;
  }
});
```

### Traduction des messages de validation

Les messages d'erreur de validation Zod sont automatiquement traduits grâce aux fichiers de traduction spécifiques :

- `public/locales/fr/zod.json`
- `public/locales/en/zod.json`

Ces fichiers contiennent des traductions pour tous les types d'erreurs de validation standard de Zod :

```json
{
  "errors": {
    "invalid_type": "Le type « {{expected}} » est attendu mais « {{received}} » a été reçu",
    "too_small": {
      "string": {
        "inclusive": "Le champ doit contenir au moins {{minimum}} caractère(s)"
      }
    },
    "invalid_string": {
      "email": "{{validation}} non valide"
    }
    // ...
  }
}
```

Lorsqu'une validation échoue (par exemple, un email invalide), Zod utilise ces traductions pour générer un message d'erreur dans la langue actuelle de l'utilisateur. Cela fonctionne automatiquement pour toutes les validations générées par Orval à partir des contraintes de l'API.

### Hook useZodI18n

Force la re-validation des champs lors d'un changement de langue, pour que les messages d'erreur s'affichent dans la bonne langue. Voir [Hooks — useZodI18n](./14-hooks.md#usezodi18n).

## Utilisation dans les composants

### Exemple basique

```tsx
import { useTranslation } from 'react-i18next';

function MyComponent() {
  // Utilisation du namespace par défaut (header)
  const { t } = useTranslation();
  
  // Ou avec un namespace spécifique
  const { t: tLogin } = useTranslation('login');
  
  return <h1>{t('ma.cle.de.traduction')}</h1>;
}
```

### Avec namespace spécifique

```tsx
// Utilisation du namespace 'login'
const { t } = useTranslation('login');

// Récupération de l'objet i18n pour accéder à d'autres fonctionnalités
const { t, i18n } = useTranslation('login');
```

### Avec variables

```tsx
// Traduction avec variables
t('welcome', { name: user.name }); // "Bienvenue, {name}"

// Traduction avec pluralisation
t('items', { count: 5 }); // "5 articles" ou "5 items" selon la langue
```

### Dans les formulaires

```tsx
// Dans un formulaire avec validation Zod
const { t } = useTranslation(['login', 'common']);
useZodI18n(methods); // Suffit pour re-valider les messages d'erreur au changement de langue

// Utilisation dans les labels et placeholders
<RhfTextInput
  control={methods.control}
  name="email"
  label={t('login:form.email.label', { defaultValue: 'Email' })}
  placeholder={t('login:form.email.placeholder', { defaultValue: 'Votre email' })}
  required={!loginSchema.shape.email.safeParse('').success}
/>
```

## Fonction pour les dates

Le système définit une fonction `LanguageToLocale` qui peut être utilisée pour obtenir la locale date-fns correspondant à la langue actuelle :

```typescript
export const LanguageToLocale = (): Locale => {
  const language = i18n.language as Languages;
  switch (language) {
    case 'fr':
      return fr;
    case 'en':
      return enUS;
    default:
      return fr;
  }
};
```

Cette fonction est définie dans le fichier `i18n.ts` mais n'est pas activement utilisée dans le projet. Les composants de date comme `RhfDateInput` et `RhfCalendarInput` utilisent directement la locale française par défaut.

## Traductions des réponses API

Le système de traduction s'étend également aux messages d'erreur renvoyés par l'API :

1. **Envoi de la préférence linguistique** : Lorsque l'utilisateur change de langue, le header HTTP `Accept-Language` est automatiquement mis à jour dans les requêtes Axios :
  ```typescript
   AXIOS_INSTANCE.defaults.headers.common['Accept-Language'] = currentLangValue;
  ```
2. **Traitement côté serveur** : L'API backend utilise ce header pour déterminer la langue à utiliser pour les messages d'erreur.
3. **Réponses localisées** : Les messages d'erreur et les violations de validation sont renvoyés dans la langue demandée :
  ```typescript
   // Exemple de réponse d'erreur de validation
   {
     "statusCode": 400,
     "message": "Erreur de validation",
     "error": "Bad Request",
     "violations": [
       {
         "field": "email",
         "message": "L'adresse e-mail n'est pas valide" // Message traduit
       }
     ]
   }
  ```

Ces messages sont directement utilisables dans l'interface utilisateur sans traduction supplémentaire, car ils sont déjà dans la langue de l'utilisateur.

> **Note :** Pour plus de détails sur la gestion des traductions côté API, consultez la documentation [api/15-traductions.md](../api/15-traductions.md).

## Bonnes pratiques

1. **Organisation des clés** - Utilisez une structure hiérarchique pour vos clés de traduction :
  ```json
   {
     "form": {
       "email": {
         "label": "Adresse e-mail",
         "placeholder": "Entrez votre e-mail",
         "error": "E-mail invalide"
       }
     }
   }
  ```
2. **Évitez les traductions en ligne** - Toujours utiliser des clés de traduction plutôt que du texte en dur
3. **Utilisez le hook useZodI18n** - Ajoutez ce hook dans tous vos formulaires qui utilisent Zod pour la validation. Sans cela, les messages d'erreur ne seront pas traduits automatiquement lors du changement de langue.
4. **Testez les traductions** - Vérifiez régulièrement que toutes les clés sont traduites dans toutes les langues supportées
5. **Utilisez l'API Accept-Language** - Le système configure automatiquement l'en-tête Axios pour que les réponses API soient dans la langue de l'utilisateur

## Script d'audit

### `check/translation.mjs`

Script Node.js qui vérifie la cohérence de toutes les traductions du projet. À lancer depuis le dossier `front/` après toute modification de clés `t()` ou de fichiers JSON :

```bash
node check/translation.mjs
```

**Ce qu'il vérifie :**

| Problème détecté | Signification | Action |
|---|---|---|
| ❌ Namespace introuvable | Le namespace utilisé dans `t()` n'a pas de fichier JSON dans `locales/fr/` | Créer le fichier ou corriger le namespace |
| ❌ Clé absente du FR | La clé est utilisée dans le code mais absente de `locales/fr/{ns}.json` | Ajouter la clé dans le fichier FR |
| ❌ Clé absente du EN | La clé est présente en FR mais absente de `locales/en/{ns}.json` | Ajouter la clé dans le fichier EN |
| ⚠️ Pas de `defaultValue` | Un appel `t()` n'a pas de `defaultValue` | Ajouter `{ defaultValue: '...' }` |
| ⚠️ `defaultValue` ≠ valeur FR | La `defaultValue` dans le code diffère du texte dans le JSON FR | Synchroniser les deux |
| ⚠️ Clé inutilisée (code mort) | Une clé est dans le JSON mais jamais appelée dans le code | Supprimer la clé du JSON |
| ℹ️ Apostrophe U+0027 vs U+2019 | Différence de type d'apostrophe entre code et JSON | Harmoniser (préférer U+2019 `'` dans les JSONs) |

**Périmètre :** parcourt récursivement tous les fichiers `.ts` et `.tsx` de `src/`. Le namespace `zod` est exclu.

**Résultat attendu :**

```
  Total problèmes à corriger         : 0
```

