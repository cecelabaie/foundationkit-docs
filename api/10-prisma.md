# Prisma

[← Retour au sommaire](./SUMMARY.md)

Ce document présente l'utilisation de Prisma ORM dans le projet API. Le projet utilise une configuration standard de Prisma avec PostgreSQL et quelques pratiques spécifiques.

## Vue d'ensemble

Le projet utilise **Prisma** comme ORM pour interagir avec la base de données PostgreSQL. L'utilisation est relativement basique mais suit les bonnes pratiques de NestJS.

### Configuration

- **Base de données** : PostgreSQL
- **Version Prisma** : ^7.5.0
- **Client généré** : `generated/prisma` (dossier personnalisé)
- **Migrations** : Gérées automatiquement
- **Configuration CLI** : `prisma.config.ts` — voir [Database → Configuration Prisma](./02-database.md#configuration-prisma)

## Configuration spéciale

### Client généré personnalisé

Le projet utilise un dossier personnalisé pour le client Prisma généré :

```prisma
generator client {
  provider = "prisma-client"
  output   = "../generated/prisma"
}
```

**Avantages** :
- Évite les conflits avec `node_modules`
- Gestion centralisée du client généré
- Meilleur contrôle sur les versions

### Service Prisma personnalisé

Le projet utilise un service Prisma personnalisé qui étend `PrismaClient` :

```typescript
// src/domains/common/services/prisma.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from 'generated/prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }
}
```

**Caractéristiques** :
- Implémente `OnModuleInit` pour la connexion automatique
- Injectable dans tous les modules NestJS
- Gestion automatique de la connexion à la base

## Structure du schéma

### Énumérations

```prisma
enum Gender {
  male
  female
  other
}

enum Role {
  admin
  user
}
```

### Modèles principaux

Le schéma contient 7 modèles principaux :

1. **User** : Utilisateurs avec authentification
2. **Media** : Fichiers uploadés (images, documents)
3. **UserMedia** : Liaison utilisateur-média
4. **RefreshToken** : Tokens de rafraîchissement (token aléatoire, stocké hashé)
5. **RevokedToken** : Tokens révoqués
6. **ResetPasswordToken** : Tokens de réinitialisation
7. **LoggerMail** : Logs des emails envoyés

### Relations

- **User ↔ UserMedia** : 1:1 (image de profil)
- **User ↔ RefreshToken** : 1:N
- **User ↔ RevokedToken** : 1:N
- **User ↔ ResetPasswordToken** : 1:N
- **Media ↔ UserMedia** : 1:N (CASCADE on delete)

## Utilisation dans les services

### Injection du service

```typescript
import { PrismaService } from 'src/domains/common/services/prisma.service';

@Injectable()
export class UserService {
  constructor(
    private readonly prisma: PrismaService,
  ) {}
}
```

### Exemples d'utilisation

```typescript
// Recherche par email
async findByEmail(email: string): Promise<User | null> {
  return this.prisma.user.findUnique({
    where: { email },
  });
}

// Création avec relations
async createUser(userData: UserRegisterBodyDTO): Promise<User> {
  return this.prisma.user.create({
    data: {
      firstName: userData.firstName,
      lastName: userData.lastName,
      username: userData.username,
      email: userData.email,
      dateOfBirth: userData.dateOfBirth,
      gender: userData.gender,
      password: hashedPassword,
      role: ['user'],
    },
  });
}

// Récupération avec include
async findByIdWithProfile(id: string): Promise<User | null> {
  return this.prisma.user.findUnique({
    where: { id },
    include: {
      profilePicture: {
        include: { media: true },
      },
    },
  });
}
```

## Types générés

### Import des types

```typescript
import { User, Role, Gender } from 'generated/prisma/client';
```

Les types sont importés depuis le dossier `generated/prisma` et utilisés dans :
- **DTOs** : Pour typer les données d'entrée
- **Services** : Pour typer les retours de méthodes
- **Contrôleurs** : Pour typer les réponses

### Exemple d'utilisation des types

```typescript
// DTO utilisant les types Prisma
export class UserUpdateBodyDTO {
  @IsOptional()
  username?: string;

  @IsOptional()
  dateOfBirth?: string;

  @IsOptional()
  @IsEnum(Gender)
  gender?: Gender;

  @IsOptional()
  notificationEmail?: boolean;

  @IsOptional()
  deleteProfilePicture?: boolean;

  @IsOptional()
  profilePicture?: Express.Multer.File;
}
```

## Migrations

### Structure des migrations

```
api/prisma/migrations/
├── 20250803111422_init/
│   └── migration.sql
├── 20250820183535_date_format/
│   └── migration.sql
├── 20251130203122_add_cascade_delete/
│   └── migration.sql
├── 20260219144619_unused_role/
│   └── migration.sql
└── migration_lock.toml
```

### Commandes de migration

```bash
# Créer une nouvelle migration
npx prisma migrate dev --name nom_de_la_migration

# Appliquer les migrations en production
npx prisma migrate deploy

# Réinitialiser la base (⚠️ supprime toutes les données)
npx prisma migrate reset
```

## Bonnes pratiques utilisées

### 1. Mapping des noms de tables

```prisma
model User {
  // ...
  @@map("users")
}

model RevokedToken {
  // ...
  @@map("revoked_tokens")
}
```

### 2. Mapping des colonnes

```prisma
model RevokedToken {
  revokedAt      DateTime @map("revoked_at")
  expirationDate DateTime @map("expiration_date")
}
```

### 3. Contraintes et index

```prisma
model User {
  id       String @id @unique @default(uuid())
  username String @unique
  email    String @unique
  // ...
}
```

### 4. Valeurs par défaut

```prisma
model User {
  role              Role[]     @default([])
  isVerified        Boolean    @default(false)
  notificationEmail Boolean    @default(false)
  createdAt         DateTime   @default(now())
  updatedAt         DateTime   @updatedAt
}
```

### 5. Suppression en cascade

```prisma
model UserMedia {
  media   Media  @relation(fields: [mediaId], references: [id], onDelete: Cascade)
}
```

## Création d'une nouvelle table

### 1. Définir le modèle dans le schéma

```prisma
model Product {
  id          String   @id @default(uuid())
  name        String
  description String?
  price       Decimal  @db.Decimal(10, 2)
  categoryId  String
  category    Category @relation(fields: [categoryId], references: [id])
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@map("products")
}
```

### 2. Créer la migration

```bash
npx prisma migrate dev --name add_products_table
```

### 3. Générer le client

```bash
npx prisma generate
```

### 4. Utiliser dans un service

```typescript
@Injectable()
export class ProductService {
  constructor(private readonly prisma: PrismaService) {}

  async createProduct(data: CreateProductDTO): Promise<Product> {
    return this.prisma.product.create({
      data: {
        name: data.name,
        description: data.description,
        price: data.price,
        categoryId: data.categoryId,
      },
    });
  }

  async findProducts(): Promise<Product[]> {
    return this.prisma.product.findMany({
      include: {
        category: true,
      },
    });
  }
}
```

## Commandes utiles

### Développement

```bash
# Générer le client Prisma
npx prisma generate

# Ouvrir Prisma Studio (interface graphique)
npx prisma studio

# Voir le statut de la base
npx prisma db status

# Formater le schéma
npx prisma format
```

### Production

```bash
# Appliquer les migrations
npx prisma migrate deploy

# Générer le client en production
npx prisma generate
```

### Debugging

```bash
# Voir les requêtes SQL générées
npx prisma db execute --stdin < query.sql

# Valider le schéma
npx prisma validate
```

---

[← Retour au sommaire](./SUMMARY.md)
