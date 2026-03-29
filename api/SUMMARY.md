# Documentation API - Sommaire

[← Retour à la documentation principale](../README.md)

## Navigation

1. [Configuration](./01-config.md)
2. [Database](./02-database.md)
3. [Diagrams](./03-diagrams.md)
4. [Faker](./04-faker.md)
5. [Filters](./05-filters.md)
6. [Responses](./06-responses.md)
7. [Gestion de la session](./07-session.md)
8. [Guards](./08-guards.md)
9. [Media](./09-media.md)
10. [Prisma](./10-prisma.md)
11. [Structure domain](./11-structure-domain.md)
12. [Routes](./12-routes.md)
13. [Swagger](./13-swagger.md)
14. [Tests](./14-tests.md)
15. [Traductions](./15-traductions.md)
16. [Guide IA — Contribuer à l'API](./16-agent-ia.md)

## Résumés

### Configuration
Présente les différentes configurations du projet API : variables d'environnement (DB, JWT, SMTP, i18n), configuration NestJS (nest-cli.json, tsconfig.json), Swagger, cookies, validation des données, fichiers statiques et constantes de l'application.

### Database
Présente la structure de la base de données PostgreSQL gérée via Prisma ORM. Détaille les modèles (User, Media, UserMedia, RefreshToken, RevokedToken, ResetPasswordToken, LoggerMail), leurs relations, contraintes et index. Inclut les commandes Prisma et les aspects sécurité.

### Diagrams
Présente les diagrammes de flux (flowcharts) créés avec PlantUML qui documentent chaque endpoint de l'API. Chaque diagramme visualise les étapes de traitement, validations, branchements et codes HTTP. Les diagrammes sont organisés par domaine fonctionnel et doivent rester synchronisés avec le code.

### Faker
Explique le système de génération de données de test avec Faker.js. Le système est implémenté et permet de générer automatiquement des données réalistes (utilisateurs, profils) pour peupler la base de données en développement et pour les tests. Par défaut, le code est commenté et peut être activé en décommentant les fichiers concernés. Structure, installation et utilisation du système de seed.

### Filters
Explique le système de filtres d'exception, notamment le `ValidationExceptionFilter` qui intercepte les erreurs de validation. Les filtres standardisent les réponses d'erreur, internationalisent les messages via i18n, détaillent les violations et facilitent le débogage côté frontend.

### Responses
Explique comment l'API standardise ses réponses HTTP avec `ResponseUtil`. L'API retourne toujours des réponses JSON structurées avec un code HTTP approprié (succès, erreur, validation). Les différents cas d'usage et formats de réponses sont détaillés.

### Gestion de la session
Explique le système complet de gestion de session : authentification JWT avec access token (15min, cookie HTTP-only) et refresh token (24h, stocké en DB). Détaille la révocation des tokens, le flag `needToReconnect`, le multi-appareils, la traçabilité et les mesures de sécurité (XSS, CSRF, HTTPS).

### Guards
Explique le système de guards (gardes), notamment le `AuthGuard` qui protège les routes et vérifie l'authentification. Le guard est global par défaut et peut être contourné avec le décorateur `@Public()`. Détaille le flux de vérification et la gestion des tokens.

### Media
Présente le système de gestion des médias (images, fichiers) : upload avec validation (`ParseFilePipe`), redimensionnement automatique via Sharp, stockage séparé (public/private), gestion des métadonnées en DB et sécurité (path traversal protection, vérifications d'autorisation).

### Prisma
Présente l'utilisation de Prisma ORM avec PostgreSQL. Configuration spéciale (client généré personnalisé dans `generated/prisma`), structure du schéma, énumérations, modèles, relations, utilisation dans les services, types générés, migrations et bonnes pratiques.

### Structure domain
Présente l'architecture par domaines (Domain-Driven Design) organisée dans `src/domains/`. Chaque domaine encapsule contrôleurs, services, DTOs, décorateurs, guards et modules. Détaille les domaines existants, les patterns et conventions, et comment créer un nouveau domaine.

### Routes
Présente toutes les routes disponibles dans l'API, organisées par catégorie selon leur niveau d'authentification requis (publiques, protégées). Le comportement détaillé de chaque route est documenté dans les diagrammes. Détaille les fonctionnalités et utilisations de chaque endpoint.

### Swagger
Présente l'utilisation de Swagger/OpenAPI pour générer automatiquement une documentation interactive de l'API. Configuration, décorateurs Swagger, documentation des DTOs, authentification, upload de fichiers, internationalisation et génération automatique de clients frontend.

### Tests
Présente l'architecture et l'organisation des tests e2e avec Jest et Supertest. Le projet utilise une base réutilisable (`initTestApp`, `cleanupTestApp`) pour simplifier la création et la maintenance. Détaille la structure, configuration, stratégie de nettoyage et processus de développement (diagrammes → tests → route).

### Traductions
Présente le système d'internationalisation (i18n) avec `nestjs-i18n`. Détection automatique de la langue via le header `Accept-Language`, organisation des fichiers JSON par domaine, gestion des paramètres dynamiques (`{constraints.0}`, `{enumValues}`), utilisation de `i18nValidationMessage()` et intégration avec les filtres.

### Guide IA — Contribuer à l'API
Guide destiné aux agents IA pour créer, modifier ou étendre des features de l'API. Définit un workflow structuré en 5 étapes : analyse & découpage, diagramme PlantUML, modifications Prisma, implémentation dans les domaines (décorateurs, DTOs, traductions, controllers/services), et tests. Inclut une checklist de validation complète pour s'assurer du bon fonctionnement (Swagger, tests, routes, diagrammes).

---

[← Retour à la documentation principale](../README.md)
