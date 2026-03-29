# Faker

[← Retour au sommaire](./SUMMARY.md)

Ce document explique le système de génération de données de test avec Faker.js dans l'API.

## Vue d'ensemble

Le système Faker permet de générer automatiquement des données réalistes pour peupler la base de données en développement et pour les tests. Il utilise la bibliothèque **@faker-js/faker** pour créer des utilisateurs, des profils et d'autres données fictives.

### Avantages

- 🚀 **Développement rapide** : Créer rapidement un jeu de données pour tester l'application
- 🧪 **Tests réalistes** : Données variées et cohérentes pour les tests E2E
- 🎲 **Diversité** : Génération aléatoire de données variées (noms, emails, dates, etc.)

## Structure

```
src/faker/
├── module/
│   └── faker.module.ts      # Module NestJS Faker
├── service/
│   ├── faker.service.ts     # Service de génération de données
│   └── scripts/
│       └── seed.ts          # Script d'exécution du seed
```

## Installation

Le package Faker est déjà installé dans les dépendances de développement :

```json
{
  "devDependencies": {
    "@faker-js/faker": "^10.3.0"
  }
}
```

## Commande de seed

Pour générer des données de test :

```bash
npm run seed
```

Cette commande exécute le script `src/faker/service/scripts/seed.ts` avec TypeScript et les alias de chemins configurés.

## FakerModule

Le module Faker encapsule le service de génération de données.

**Fichier** : `src/faker/module/faker.module.ts`

```typescript
import { Module } from '@nestjs/common';
import { FakerService } from '../service/faker.service';
import { PrismaService } from 'src/domains/common/services/prisma.service';

@Module({
  imports: [],
  providers: [FakerService, PrismaService],
  exports: [FakerService],
})
export class FakerModule {}
```

### Caractéristiques

- Fournit le `FakerService`
- Injecte `PrismaService` pour l'accès à la base de données
- Exportable pour utilisation dans d'autres modules

## FakerService

Le service contient la logique de génération de données fictives.

**Fichier** : `src/faker/service/faker.service.ts`

### Structure de base

```typescript
import { Injectable } from '@nestjs/common';
import { faker } from '@faker-js/faker';
import { PrismaService } from 'src/domains/common/services/prisma.service';
import * as bcrypt from 'bcrypt';

@Injectable()
export class FakerService {
  constructor(private readonly prisma: PrismaService) {}

  // Méthodes de génération
}
```

## Script de seed

Le script `seed.ts` orchestre la génération de données.

**Fichier** : `src/faker/service/scripts/seed.ts`

### Structure cible (non encore implémentée)

> ⚠️ **Attention** : Le code ci-dessous décrit le comportement **cible** du script de seed. Les méthodes `clearDatabase()`, `generateAdminUser()` et `generateUsers()` **n'existent pas encore** dans `FakerService` — le code est commenté. Voir la section "État actuel du projet" ci-dessous.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from 'src/domains/app/module/app.module';
import { FakerService } from '../faker.service';

async function bootstrap() {
  console.log('🌱 Démarrage du seed...\n');
  
  const app = await NestFactory.createApplicationContext(AppModule);
  const fakerService = app.get(FakerService);

  try {
    // 1. Nettoyer la base de données
    console.log('🗑️  Nettoyage de la base de données...');
    await fakerService.clearDatabase();
    console.log('✅ Base de données nettoyée\n');

    // 2. Créer un administrateur
    console.log('👤 Création de l\'administrateur...');
    const admin = await fakerService.generateAdminUser();
    console.log(`✅ Admin créé: ${admin.email}\n`);

    // 3. Créer des utilisateurs de test
    console.log('👥 Création de 10 utilisateurs...');
    const users = await fakerService.generateUsers(10);
    console.log(`✅ ${users.length} utilisateurs créés\n`);

    console.log('🎉 Seed terminé avec succès!');
    console.log('\n📋 Credentials de test:');
    console.log('  Email: admin@example.com');
    console.log('  Password: Admin123!');
    
  } catch (error) {
    console.error('❌ Erreur lors du seed:', error);
    process.exit(1);
  } finally {
    await app.close();
  }
}

bootstrap();
```

### Exécution

```bash
npm run seed
```

### Sortie attendue

```
🌱 Démarrage du seed...

🗑️  Nettoyage de la base de données...
✅ Base de données nettoyée

👤 Création de l'administrateur...
✅ Admin créé: admin@example.com

👥 Création de 10 utilisateurs...
✅ 10 utilisateurs créés

🎉 Seed terminé avec succès!

📋 Credentials de test:
  Email: admin@example.com
  Password: Admin123!
```

## Commandes utiles

```bash
# Générer les données de seed
npm run seed

# Réinitialiser la base et re-seed
npx prisma migrate reset
npm run seed

# Voir les données générées (Prisma Studio)
npx prisma studio
```

## Ressources

- **Documentation Faker.js** : https://fakerjs.dev/
- **API Faker.js** : https://fakerjs.dev/api/
- **Locales disponibles** : https://fakerjs.dev/guide/localization.html
- **Prisma transactions** : https://www.prisma.io/docs/concepts/components/prisma-client/transactions

## État actuel du projet

✅ **Le système Faker est implémenté** : Le module, le service et le script de seed sont en place et fonctionnels.

**État actuel du script** : Le fichier `seed.ts` livré exécute uniquement un bootstrap minimal (création du contexte NestJS puis « Seed completed ») et n’appelle pas `FakerService`. La « Structure complète » décrite plus haut correspond au comportement cible une fois le code décommenté.

⚠️ **Note** : Par défaut, le code dans `faker.service.ts` et `seed.ts` est commenté pour éviter la génération automatique de données lors du développement. Pour utiliser le système de seed :

1. Décommenter le code dans `faker.service.ts` (méthodes de génération)
2. Décommenter le code dans `seed.ts` (script d'exécution)
3. Exécuter `npm run seed`

Cette documentation fournit un guide complet pour utiliser le système de seed avec Faker.js.

---

[← Retour au sommaire](./SUMMARY.md)
