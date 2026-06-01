# Structure domain

[← Retour au sommaire](./SUMMARY.md)

Ce document présente l'architecture et l'organisation des domaines dans l'API. L'application suit une architecture modulaire basée sur des domaines métier, où chaque domaine encapsule toutes les fonctionnalités liées à une entité ou un concept spécifique.

## Vue d'ensemble de l'architecture

L'API utilise une **architecture par domaines** (Domain-Driven Design) organisée dans le dossier `src/domains/`. Chaque domaine représente une fonctionnalité métier complète et autonome.

### Avantages de cette architecture

- **Séparation des responsabilités** : Chaque domaine gère ses propres fonctionnalités
- **Maintenabilité** : Code organisé et facile à maintenir
- **Évolutivité** : Ajout de nouveaux domaines sans impact sur l'existant
- **Réutilisabilité** : Services et composants réutilisables entre domaines
- **Testabilité** : Tests isolés par domaine

## Structure générale d'un domaine

Chaque domaine suit une structure standardisée avec les dossiers suivants :

```
domaine/
├── controller/          # Contrôleurs REST
├── service/            # Logique métier
├── dto/               # Data Transfer Objects
├── decorators/        # Décorateurs personnalisés
├── guard/            # Guards d'authentification/autorisation
├── module/           # Module NestJS
├── interface/        # Interfaces TypeScript
├── enums/           # Énumérations
├── types/           # Types personnalisés
└── utils/           # Utilitaires spécifiques au domaine
```

## Domaines existants

### 1. **common** - Services partagés
**Rôle** : Services et utilitaires utilisés par tous les domaines

**Structure** :
```
common/
├── constants/        # Constantes globales
├── decorators/       # Décorateurs partagés (@Public)
├── dto/             # DTOs de réponse standardisés
├── services/        # Services globaux (PrismaService, UtilsService)
├── utils/           # Utilitaires communs
└── module/          # Module global
```

**Services et utilitaires principaux** :
- `PrismaService` : Client Prisma pour la base de données
- `UtilsService` : Utilitaires généraux
- `ResponseUtil` : Gestion des réponses API (`utils/response/response.util.ts`)

### 2. **auth** - Authentification
**Rôle** : Gestion de l'authentification JWT et des sessions

**Structure** :
```
auth/
├── controller/       # AuthController
├── service/         # AuthService
├── guard/          # AuthGuard (global)
├── decorators/     # Décorateurs Swagger
├── dto/           # DTOs d'authentification
├── interface/     # JwtPayload
└── module/        # AuthModule
```

**Fonctionnalités** :
- Login/logout utilisateur
- Gestion des tokens JWT
- Validation des sessions
- Refresh des tokens

### 3. **user** - Gestion des utilisateurs
**Rôle** : CRUD des utilisateurs et gestion des profils

**Structure** :
```
user/
├── controller/       # UserController
├── service/         # UserService
├── decorators/     # Décorateurs Swagger
├── dto/           # DTOs utilisateur
├── types/         # Types personnalisés
└── module/        # UserModule
```

**Fonctionnalités** :
- Inscription des utilisateurs
- Mise à jour du profil
- Gestion des images de profil
- Changement de mot de passe

### 4. **reset-password** - Réinitialisation de mot de passe
**Rôle** : Processus de réinitialisation de mot de passe

**Structure** :
```
reset-password/
├── controller/       # ResetPasswordController
├── service/         # ResetPasswordService
├── decorators/     # Décorateurs Swagger
├── dto/           # DTOs de réinitialisation
└── module/        # ResetPasswordModule
```

**Fonctionnalités** :
- Demande de réinitialisation
- Validation des tokens
- Mise à jour du mot de passe

### 5. **media** - Gestion des médias
**Rôle** : Upload et gestion des fichiers

**Structure** :
```
media/
├── controller/       # MediaController
├── service/         # MediaService
├── decorators/     # Décorateurs Swagger
├── dto/           # DTOs de médias
├── enums/         # Types de médias
└── module/        # MediaModule
```

**Fonctionnalités** :
- Upload de fichiers
- Validation des types de fichiers
- Gestion des médias publics/privés

### 6. **access-token** - Gestion des access tokens
**Rôle** : Validation et gestion des access tokens JWT

### 7. **refresh-token** - Gestion des refresh tokens
**Rôle** : Création et validation des refresh tokens

### 8. **revoked-token** - Gestion des tokens révoqués
**Rôle** : Blacklist des tokens révoqués

### 9. **media-relation** - Relations médias-utilisateurs
**Rôle** : Liaison entre utilisateurs et médias

### 10. **mail** - Service d'envoi d'emails
**Rôle** : Envoi d'emails (notifications, réinitialisation)

## Patterns et conventions

### 1. **Module** - Configuration NestJS

Chaque domaine expose un module NestJS qui configure :
- **Imports** : Modules requis
- **Controllers** : Contrôleurs du domaine
- **Providers** : Services du domaine
- **Exports** : Services exportés pour d'autres domaines

**Exemple** :
```typescript
@Module({
  imports: [MediaModule, AccessTokenModule],
  controllers: [UserController],
  providers: [UserService],
  exports: [UserService],
})
export class UserModule {}
```

### 2. **Controller** - Endpoints REST

Les contrôleurs gèrent les endpoints HTTP et utilisent :
- **Décorateurs Swagger** : Documentation automatique
- **DTOs** : Validation des données d'entrée
- **Guards** : Protection des routes
- **Services** : Logique métier déléguée

**Exemple** :
```typescript
@ApiTags('user')
@Controller('user')
export class UserController {
  constructor(
    private readonly userService: UserService,
    private readonly i18n: I18nService,
  ) {}

  @RegisterEndpoint('register')
  async register(@Body() userRegisterDto: UserRegisterBodyDTO) {
    // Logique du contrôleur
  }
}
```

### 3. **Service** - Logique métier

Les services contiennent la logique métier et :
- **Injection de dépendances** : PrismaService, autres services
- **Validation** : Vérification des données
- **Gestion d'erreurs** : Exceptions appropriées
- **Internationalisation** : Messages traduits

**Exemple** :
```typescript
@Injectable()
export class UserService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly i18n: I18nService,
  ) {}

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: { email },
    });
  }
}
```

### 4. **DTO** - Data Transfer Objects

Les DTOs définissent la structure des données avec :
- **Validation** : Décorateurs class-validator
- **Documentation** : Décorateurs Swagger
- **Internationalisation** : Messages d'erreur traduits

**Exemple** :
```typescript
export class UserRegisterBodyDTO {
  @ApiProperty()
  @IsString({ message: 'user.register.validation.first-name.is-string' })
  @IsNotEmpty({ message: 'user.register.validation.first-name.is-not-empty' })
  @MinLength(4, { message: i18nValidationMessage('user.register.validation.first-name.min-length') })
  firstName: string;

  @ApiProperty()
  @IsEmail({}, { message: 'user.register.validation.email.is-email' })
  @IsNotEmpty({ message: 'user.register.validation.email.is-not-empty' })
  email: string;
}
```

### 5. **Decorators** - Décorateurs personnalisés

Les décorateurs personnalisés combinent :
- **Méthodes HTTP** : @Post, @Get, etc.
- **Documentation Swagger** : @ApiOperation, @ApiResponse (voir [Swagger](./13-swagger.md))
- **Sécurité** : @Public, @Throttle
- **Validation** : Pipes personnalisés

**Exemple** :
```typescript
export function RegisterEndpoint(path: string) {
  return applyDecorators(
    Post(path),
    Public(),
    ApiOperation({
      summary: 'Register a user',
      description: 'Register a user',
    }),
    ApiBody({
      description: 'User information',
      type: UserRegisterBodyDTO,
    }),
    ApiResponse({
      status: 201,
      description: 'User created',
      type: UserRegisterResponseDTO,
    }),
  );
}
```

### 6. **Guards** - Protection des routes

Les guards implémentent :
- **Authentification** : Vérification des tokens
- **Autorisation** : Vérification des permissions
- **Métadonnées** : Utilisation des décorateurs

**Exemple** :
```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (isPublic) {
      return true;
    }
    
    // Vérification du token...
    return true;
  }
}
```

## Création d'un nouveau domaine

### 1. Structure des dossiers

```bash
mkdir -p src/domains/nouveau-domaine/{controller,service,dto,decorators,module}
```

### 2. Module NestJS

```typescript
// src/domains/nouveau-domaine/module/nouveau-domaine.module.ts
import { Module } from '@nestjs/common';
import { NouveauDomaineController } from '../controller/nouveau-domaine.controller';
import { NouveauDomaineService } from '../service/nouveau-domaine.service';

@Module({
  controllers: [NouveauDomaineController],
  providers: [NouveauDomaineService],
  exports: [NouveauDomaineService],
})
export class NouveauDomaineModule {}
```

### 3. Service

```typescript
// src/domains/nouveau-domaine/service/nouveau-domaine.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from 'src/domains/common/services/prisma.service';
import { I18nService } from 'nestjs-i18n';

@Injectable()
export class NouveauDomaineService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly i18n: I18nService,
  ) {}

  async create(data: CreateNouveauDomaineDTO) {
    // Logique métier
  }
}
```

### 4. Contrôleur

```typescript
// src/domains/nouveau-domaine/controller/nouveau-domaine.controller.ts
import { Controller, Post, Body } from '@nestjs/common';
import { ApiTags } from '@nestjs/swagger';
import { NouveauDomaineService } from '../service/nouveau-domaine.service';
import { CreateNouveauDomaineDTO } from '../dto/create-nouveau-domaine.dto';

@ApiTags('nouveau-domaine')
@Controller('nouveau-domaine')
export class NouveauDomaineController {
  constructor(private readonly service: NouveauDomaineService) {}

  @Post()
  async create(@Body() data: CreateNouveauDomaineDTO) {
    return this.service.create(data);
  }
}
```

### 5. DTOs

```typescript
// src/domains/nouveau-domaine/dto/create-nouveau-domaine.dto.ts
import { ApiProperty } from '@nestjs/swagger';
import { IsString, IsNotEmpty } from 'class-validator';

export class CreateNouveauDomaineDTO {
  @ApiProperty()
  @IsString({ message: 'nouveau-domaine.validation.name.isString' })
  @IsNotEmpty({ message: 'nouveau-domaine.validation.name.isNotEmpty' })
  name: string;
}
```

### 6. Décorateurs Swagger

```typescript
// src/domains/nouveau-domaine/decorators/nouveau-domaine-create.decorator.ts
import { applyDecorators, Post } from '@nestjs/common';
import { ApiOperation, ApiResponse } from '@nestjs/swagger';

export function CreateEndpoint(path: string) {
  return applyDecorators(
    Post(path),
    ApiOperation({
      summary: 'Create new item',
      description: 'Create a new item',
    }),
    ApiResponse({
      status: 201,
      description: 'Item created successfully',
    }),
  );
}
```

Cette architecture modulaire permet une maintenance facile, une évolution progressive et une séparation claire des responsabilités dans l'API.

---

[← Retour au sommaire](./SUMMARY.md)
