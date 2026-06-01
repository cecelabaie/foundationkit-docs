# Responses

[← Retour au sommaire](./SUMMARY.md)

Ce document explique comment l'API standardise ses réponses HTTP et comment utiliser `ResponseUtil`.

> 💡 **Pour comprendre comment les erreurs sont interceptées**, consultez **[filters.md](./05-filters.md)**.

## Concept de base

L'API retourne **toujours** des réponses JSON structurées avec un code HTTP approprié. Cela permet au frontend de :

- ✅ **Savoir immédiatement si c'est un succès ou une erreur** (code HTTP)
- ✅ **Afficher les bons messages** à l'utilisateur
- ✅ **Mapper les erreurs aux champs de formulaire** si besoin
- ✅ **Gérer les erreurs de manière cohérente**

## ResponseUtil

**Fichier** : `src/domains/common/utils/response/response.util.ts`

C'est l'utilitaire central pour créer toutes les réponses de l'API. Il garantit la **cohérence** des réponses.

### Les différents cas d'usage

#### 1. Succès : `return ResponseUtil.success()`

```typescript
// ✅ On RETOURNE la réponse
return ResponseUtil.success(HttpStatus.OK, 'Profil récupéré', user);
```

→ La fonction se termine normalement avec une réponse HTTP 200 ou 201.

#### 2. Erreur de validation : `throw BadRequestException` + `validationError()`

```typescript
// ✅ On LANCE une exception avec violations
throw new BadRequestException(
  ResponseUtil.validationError([
    { field: 'email', message: 'Email invalide' },
    { field: 'password', message: 'Mot de passe trop court' }
  ])
);
```

→ Permet de **mapper les erreurs aux champs du formulaire** grâce au tableau `violations`.

#### 3. Autres erreurs : `throw` une exception NestJS

```typescript
// Utilisateur non trouvé
throw new NotFoundException('Utilisateur non trouvé');

// Token invalide
throw new UnauthorizedException('Token expiré');

// Erreur serveur
throw new InternalServerErrorException('Erreur serveur');
```

→ Pour les erreurs qui ne sont **pas liées à un formulaire** (ressource introuvable, accès refusé, etc.).

## Les codes HTTP

| Code | Nom | Quand l'utiliser | Méthode |
|------|-----|-----------------|---------|
| **200** | OK | Opération réussie (lecture, mise à jour) | `ResponseUtil.success(HttpStatus.OK, ...)` |
| **201** | Created | Ressource créée | `ResponseUtil.success(HttpStatus.CREATED, ...)` |
| **400** | Bad Request | Données invalides | `throw new BadRequestException(...)` |
| **401** | Unauthorized | Non authentifié | `throw new UnauthorizedException(...)` |
| **404** | Not Found | Ressource inexistante | `throw new NotFoundException(...)` |
| **429** | Too Many Requests | Rate limit dépassé | Géré automatiquement |
| **500** | Internal Server Error | Erreur serveur | `throw new InternalServerErrorException(...)` |

## Formats de réponse

### Succès (200, 201)

**Avec données** :
```json
{
  "statusCode": 200,
  "message": "Profil récupéré avec succès",
  "data": {
    "id": "123",
    "email": "user@example.com",
    "username": "johndoe"
  }
}
```

**Sans données** :
```json
{
  "statusCode": 201,
  "message": "Utilisateur créé avec succès"
}
```

### Erreur de validation (400)

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "Bad Request",
  "violations": [
    {
      "field": "email",
      "message": "L'email doit être valide"
    },
    {
      "field": "password",
      "message": "Le mot de passe doit contenir au moins 8 caractères"
    }
  ]
}
```

> **Note** : Le champ `violations` permet au frontend de mapper automatiquement les erreurs aux champs du formulaire.

### Erreur générale (401, 404, 500)

```json
{
  "statusCode": 404,
  "message": "Utilisateur non trouvé",
  "error": "Not Found"
}
```
## Comment ça marche techniquement ?

### Le rôle du code HTTP

Quand tu lances une exception :
```typescript
throw new BadRequestException(...)
```

**Voici le processus** :

1. **Exception lancée** → Le code s'arrête immédiatement
2. **NestJS attrape l'exception** → Il identifie le type (`BadRequestException`)
3. **Filtre approprié activé** → Le `ValidationExceptionFilter` ou le filtre par défaut
4. **Réponse HTTP créée** → Le filtre appelle `response.status(400).json({...})`
5. **Client reçoit la réponse** → Avec le code 400 + le JSON

### Pourquoi c'est important ?

Le **code HTTP** est le signal universel pour dire si une requête a réussi ou échoué :

- **2xx** (200, 201) = Succès → Axios/Fetch retourne la réponse normalement
- **4xx** (400, 401, 404) = Erreur client → Axios/Fetch lance une exception (`catch`)
- **5xx** (500) = Erreur serveur → Axios/Fetch lance une exception (`catch`)

C'est grâce à ça que le frontend peut gérer automatiquement les erreurs :

```typescript
// Côté frontend
try {
  const response = await api.post('/register', data);
  // Succès → on affiche un message
  toast.success(response.data.message);
} catch (error) {
  // Erreur → on affiche les violations
  if (error.response.data.violations) {
    error.response.data.violations.forEach(v => {
      setFieldError(v.field, v.message);
    });
  }
}
```

## Voir aussi

- **[filters.md](./05-filters.md)** - Comment les filtres interceptent et transforment les erreurs
- **[config.md](./01-config.md)** - Configuration de l'API

---

[← Retour au sommaire](./SUMMARY.md)
