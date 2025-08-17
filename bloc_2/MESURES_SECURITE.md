# Mesures de Sécurité - BeneVoclic

## 📋 Vue d'ensemble

Les mesures de sécurité de BeneVoclic assurent la protection des données, l'authentification robuste et la prévention des vulnérabilités courantes. Ces mesures s'appliquent à l'API NestJS v11.0.7 et l'application web Nuxt.js v3.16.0.

### Objectifs de sécurité

- ✅ **Protection des données** : Chiffrement et sécurisation des informations sensibles
- ✅ **Authentification robuste** : Système d'authentification multi-facteurs
- ✅ **Autorisation granulaire** : Contrôle d'accès basé sur les rôles
- ✅ **Prévention des attaques** : Protection contre XSS, CSRF, injection
- ✅ **Audit et monitoring** : Surveillance continue des activités
- ✅ **Conformité** : Respect des réglementations RGPD et standards de sécurité

## 🔐 Authentification et autorisation

### Firebase Authentication (Implémenté)

#### Configuration Firebase
Le projet utilise Firebase Authentication pour la gestion sécurisée des utilisateurs. La configuration est centralisée dans `firebase.config.ts` avec les variables d'environnement appropriées.

#### Guard d'authentification NestJS
L'API utilise un guard d'authentification personnalisé (`AuthGuard`) qui :
- Vérifie les tokens Firebase JWT
- Valide les rôles utilisateur (volunteer, association, admin)
- Gère les routes publiques et protégées
- Implémente le contrôle d'accès basé sur les rôles (RBAC)

#### Commandes de sécurité
```bash
# Vérifier les vulnérabilités des dépendances
npm audit

# Vérifier les vulnérabilités avec correction automatique
npm audit fix

# Linter avec règles de sécurité
npm run lint:security

# Vérifier les imports de sécurité
npm run check:lodash
```

**Explication :** Ces commandes permettent de maintenir la sécurité du code en détectant les vulnérabilités et en appliquant les bonnes pratiques de sécurité.

### Firebase Authentication
```typescript
// firebase.config.ts
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: process.env.FIREBASE_API_KEY,
  authDomain: process.env.FIREBASE_AUTH_DOMAIN,
  projectId: process.env.FIREBASE_PROJECT_ID,
  storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.FIREBASE_APP_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
```

#### Service d'authentification
```typescript
// services/auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    @InjectModel(User.name) private userModel: Model<User>,
    private jwtService: JwtService,
    private logger: Logger,
  ) {}

  async validateUser(email: string, password: string): Promise<any> {
    try {
      const user = await this.userModel.findOne({ email }).exec();
      
      if (user && await this.verifyPassword(password, user.password)) {
        const { password, ...result } = user.toObject();
        return result;
      }
      
      return null;
    } catch (error) {
      this.logger.error(`Error validating user: ${error.message}`);
      throw error;
    }
  }

  async login(user: any) {
    const payload = { 
      email: user.email, 
      sub: user._id, 
      role: user.role 
    };
    
    return {
      access_token: this.jwtService.sign(payload),
      user: {
        id: user._id,
        email: user.email,
        firstName: user.firstName,
        lastName: user.lastName,
        role: user.role,
      },
    };
  }

  private async hashPassword(password: string): Promise<string> {
    const saltRounds = 12; // Augmenté pour plus de sécurité
    return bcrypt.hash(password, saltRounds);
  }

  private async verifyPassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }
}
```

### JWT (JSON Web Tokens)

#### Configuration JWT
```typescript
// jwt.config.ts
import { JwtModule } from '@nestjs/jwt';

@Module({
  imports: [
    JwtModule.register({
      secret: process.env.JWT_SECRET,
      signOptions: { 
        expiresIn: '1h',
        issuer: 'benevoclic',
        audience: 'benevoclic-users',
      },
    }),
  ],
})
export class AuthModule {}
```

#### Middleware JWT
```typescript
// middleware/jwt.middleware.ts
import { Injectable, NestMiddleware, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class JwtMiddleware implements NestMiddleware {
  constructor(private jwtService: JwtService) {}

  use(req: Request, res: Response, next: NextFunction) {
    const token = this.extractTokenFromHeader(req);
    
    if (!token) {
      throw new UnauthorizedException('Token manquant');
    }

    try {
      const payload = this.jwtService.verify(token);
      req['user'] = payload;
      next();
    } catch (error) {
      throw new UnauthorizedException('Token invalide');
    }
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}
```

### Contrôle d'accès basé sur les rôles (RBAC)

#### Décorateur de rôles
```typescript
// decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);
```

#### Guard d'autorisation
```typescript
// guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../decorators/roles.decorator';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.role === role);
  }
}
```

#### Utilisation dans les contrôleurs
```typescript
// controllers/user.controller.ts
@Controller('users')
export class UserController {
  @Get(':id')
  @UseGuards(AuthGuard, RolesGuard)
  @Roles('admin', 'association')
  async findOne(@Param('id') id: string): Promise<User> {
    return this.userService.findById(id);
  }

  @Delete(':id')
  @UseGuards(AuthGuard, RolesGuard)
  @Roles('admin')
  async remove(@Param('id') id: string): Promise<void> {
    return this.userService.remove(id);
  }
}
```

## 🛡️ Protection contre les attaques

### Validation des entrées

#### DTOs avec validation
```typescript
// dto/create-user.dto.ts
import { IsEmail, IsString, MinLength, MaxLength, IsEnum, Matches } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  @Matches(/^[a-zA-ZÀ-ÿ\s]+$/, {
    message: 'Le prénom ne doit contenir que des lettres',
  })
  firstName: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  @Matches(/^[a-zA-ZÀ-ÿ\s]+$/, {
    message: 'Le nom ne doit contenir que des lettres',
  })
  lastName: string;

  @IsEmail({}, { message: 'Format d\'email invalide' })
  email: string;

  @IsString()
  @MinLength(8)
  @MaxLength(128)
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/, {
    message: 'Le mot de passe doit contenir au moins une minuscule, une majuscule, un chiffre et un caractère spécial',
  })
  password: string;

  @IsEnum(['volunteer', 'association', 'admin'])
  role: string;
}
```

#### Pipe de validation global
```typescript
// main.ts
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.useGlobalPipes(new ValidationPipe({
    whitelist: true, // Supprime les propriétés non définies
    forbidNonWhitelisted: true, // Rejette les requêtes avec des propriétés non définies
    transform: true, // Transforme automatiquement les types
    transformOptions: {
      enableImplicitConversion: true,
    },
  }));

  await app.listen(3000);
}
```

### Protection XSS (Cross-Site Scripting)

#### Sanitisation côté client
```typescript
// utils/sanitize.ts
import DOMPurify from 'dompurify';

export const sanitizeInput = (input: string): string => {
  return DOMPurify.sanitize(input, {
    ALLOWED_TAGS: [],
    ALLOWED_ATTR: [],
  });
};

export const sanitizeHtml = (html: string): string => {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: ['class'],
  });
};
```

#### Composant Vue sécurisé
```vue
<!-- components/SafeContent.vue -->
<template>
  <div v-html="sanitizedContent"></div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { sanitizeHtml } from '@/utils/sanitize'

interface Props {
  content: string
}

const props = defineProps<Props>()

const sanitizedContent = computed(() => {
  return sanitizeHtml(props.content)
})
</script>
```

### Protection CSRF (Cross-Site Request Forgery)

#### Configuration CSRF
```typescript
// csrf.config.ts
import * as csrf from 'csurf';

export const csrfConfig = csrf({
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 3600, // 1 heure
  },
  ignoreMethods: ['GET', 'HEAD', 'OPTIONS'],
});
```

#### Middleware CSRF
```typescript
// middleware/csrf.middleware.ts
import { Injectable, NestMiddleware } from '@nestjs/common';
import * as csrf from 'csurf';

@Injectable()
export class CsrfMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    csrf({
      cookie: {
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'strict',
      },
    })(req, res, next);
  }
}
```

### Protection contre l'injection NoSQL

#### Validation MongoDB
```typescript
// utils/mongo-validator.ts
import { Types } from 'mongoose';

export const validateObjectId = (id: string): boolean => {
  return Types.ObjectId.isValid(id);
};

export const sanitizeMongoQuery = (query: any): any => {
  const sanitized = {};
  
  for (const [key, value] of Object.entries(query)) {
    if (typeof value === 'string') {
      // Éviter les injections d'opérateurs MongoDB
      if (key.startsWith('$')) {
        continue;
      }
      sanitized[key] = value;
    } else if (typeof value === 'object' && value !== null) {
      sanitized[key] = sanitizeMongoQuery(value);
    } else {
      sanitized[key] = value;
    }
  }
  
  return sanitized;
};
```

#### Service sécurisé
```typescript
// services/user.service.ts
@Injectable()
export class UserService {
  async findById(id: string): Promise<User> {
    if (!validateObjectId(id)) {
      throw new BadRequestException('ID invalide');
    }

    const user = await this.userModel.findById(id).exec();
    if (!user) {
      throw new NotFoundException('Utilisateur non trouvé');
    }
    
    return user;
  }

  async searchUsers(query: any): Promise<User[]> {
    const sanitizedQuery = sanitizeMongoQuery(query);
    return this.userModel.find(sanitizedQuery).exec();
  }
}
```

## 🔒 Chiffrement et sécurité des données

### Chiffrement des données sensibles

#### Service de chiffrement
```typescript
// services/encryption.service.ts
import { Injectable } from '@nestjs/common';
import * as crypto from 'crypto';

@Injectable()
export class EncryptionService {
  private readonly algorithm = 'aes-256-gcm';
  private readonly key = Buffer.from(process.env.ENCRYPTION_KEY, 'hex');

  encrypt(text: string): { encryptedData: string; iv: string; authTag: string } {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipher(this.algorithm, this.key);
    cipher.setAAD(Buffer.from('benevoclic', 'utf8'));
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return {
      encryptedData: encrypted,
      iv: iv.toString('hex'),
      authTag: cipher.getAuthTag().toString('hex'),
    };
  }

  decrypt(encryptedData: string, iv: string, authTag: string): string {
    const decipher = crypto.createDecipher(this.algorithm, this.key);
    decipher.setAAD(Buffer.from('benevoclic', 'utf8'));
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));
    
    let decrypted = decipher.update(encryptedData, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}
```

#### Modèle avec données chiffrées
```typescript
// schemas/user.schema.ts
@Schema({ timestamps: true })
export class User {
  @Prop({ required: true, unique: true })
  email: string;

  @Prop({ required: true })
  firstName: string;

  @Prop({ required: true })
  lastName: string;

  @Prop({ required: true, select: false }) // Ne pas inclure par défaut
  password: string;

  @Prop({ type: Object, select: false })
  encryptedData?: {
    phone?: string;
    address?: string;
    personalInfo?: string;
  };

  @Prop({ required: true, enum: ['volunteer', 'association', 'admin'] })
  role: string;

  @Prop({ default: true })
  isActive: boolean;
}
```

### Sécurisation des mots de passe

#### Politique de mots de passe
```typescript
// utils/password-policy.ts
export const validatePassword = (password: string): { isValid: boolean; errors: string[] } => {
  const errors: string[] = [];
  
  if (password.length < 8) {
    errors.push('Le mot de passe doit contenir au moins 8 caractères');
  }
  
  if (!/(?=.*[a-z])/.test(password)) {
    errors.push('Le mot de passe doit contenir au moins une lettre minuscule');
  }
  
  if (!/(?=.*[A-Z])/.test(password)) {
    errors.push('Le mot de passe doit contenir au moins une lettre majuscule');
  }
  
  if (!/(?=.*\d)/.test(password)) {
    errors.push('Le mot de passe doit contenir au moins un chiffre');
  }
  
  if (!/(?=.*[@$!%*?&])/.test(password)) {
    errors.push('Le mot de passe doit contenir au moins un caractère spécial');
  }
  
  return {
    isValid: errors.length === 0,
    errors,
  };
};
```

#### Service de hachage sécurisé
```typescript
// services/password.service.ts
import { Injectable } from '@nestjs/common';
import * as bcrypt from 'bcrypt';

@Injectable()
export class PasswordService {
  private readonly saltRounds = 12;

  async hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, this.saltRounds);
  }

  async verifyPassword(password: string, hash: string): Promise<boolean> {
    return bcrypt.compare(password, hash);
  }

  async needsRehash(hash: string): Promise<boolean> {
    return bcrypt.getRounds(hash) < this.saltRounds;
  }
}
```

## 🚨 Monitoring et audit de sécurité

### Logs de sécurité

#### Service de logging sécurisé
```typescript
// services/security-logger.service.ts
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class SecurityLoggerService extends Logger {
  logSecurityEvent(event: string, details: any, userId?: string) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      event,
      details,
      userId,
      ipAddress: this.getClientIp(),
      userAgent: this.getUserAgent(),
    };

    this.log(`SECURITY: ${JSON.stringify(logEntry)}`);
  }

  logAuthenticationAttempt(email: string, success: boolean, ipAddress?: string) {
    this.logSecurityEvent('AUTH_ATTEMPT', {
      email,
      success,
      ipAddress,
    });
  }

  logAuthorizationFailure(userId: string, resource: string, action: string) {
    this.logSecurityEvent('AUTHZ_FAILURE', {
      userId,
      resource,
      action,
    });
  }

  logDataAccess(userId: string, resource: string, action: string) {
    this.logSecurityEvent('DATA_ACCESS', {
      userId,
      resource,
      action,
    });
  }

  private getClientIp(): string {
    // Implémentation pour récupérer l'IP du client
    return 'unknown';
  }

  private getUserAgent(): string {
    // Implémentation pour récupérer le User-Agent
    return 'unknown';
  }
}
```

### Audit de sécurité automatisé

#### Script d'audit
```javascript
// scripts/security-audit.js
#!/usr/bin/env node

const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

class SecurityAuditor {
  constructor() {
    this.projectRoot = process.cwd();
    this.auditReport = {
      timestamp: new Date().toISOString(),
      project: path.basename(this.projectRoot),
      vulnerabilities: {
        critical: 0,
        high: 0,
        moderate: 0,
        low: 0,
      },
      recommendations: [],
    };
  }

  async runNpmAudit() {
    try {
      console.log('🔍 Running npm audit...');
      const auditOutput = execSync('npm audit --json', { encoding: 'utf8' });
      const auditData = JSON.parse(auditOutput);

      this.auditReport.vulnerabilities = auditData.metadata?.vulnerabilities || {};
      this.auditReport.recommendations = this.extractRecommendations(auditData);

      console.log('✅ npm audit completed');
      return true;
    } catch (error) {
      console.error('❌ npm audit failed:', error.message);
      return false;
    }
  }

  async scanCodeVulnerabilities() {
    const vulnerabilities = [];

    // Scan XSS
    const xssPattern = /innerHTML\s*=/g;
    const files = this.findFiles(['.vue', '.js', '.ts']);

    files.forEach(file => {
      const content = fs.readFileSync(file, 'utf8');
      const matches = content.match(xssPattern);

      if (matches) {
        vulnerabilities.push({
          type: 'XSS',
          file: file,
          line: this.findLineNumber(content, xssPattern),
          severity: 'HIGH',
          description: 'Unsafe use of innerHTML detected',
        });
      }
    });

    return vulnerabilities;
  }

  async scanExposedSecrets() {
    const secrets = [];

    // Patterns de secrets
    const secretPatterns = [
      /api[_-]?key\s*[:=]\s*['"][^'"]+['"]/gi,
      /password\s*[:=]\s*['"][^'"]+['"]/gi,
      /secret\s*[:=]\s*['"][^'"]+['"]/gi,
    ];

    const files = this.findFiles(['.js', '.ts', '.vue', '.env']);

    files.forEach(file => {
      const content = fs.readFileSync(file, 'utf8');

      secretPatterns.forEach(pattern => {
        const matches = content.match(pattern);
        if (matches) {
          secrets.push({
            file: file,
            pattern: pattern.source,
            severity: 'CRITICAL',
            description: 'Potential secret exposure detected',
          });
        }
      });
    });

    return secrets;
  }

  generateReport() {
    const reportPath = path.join(this.projectRoot, 'security-audit-report.json');
    fs.writeFileSync(reportPath, JSON.stringify(this.auditReport, null, 2));

    console.log('📊 Security audit report generated:', reportPath);

    // Afficher le résumé
    const totalVulns = Object.values(this.auditReport.vulnerabilities).reduce((sum, count) => sum + count, 0);
    console.log(`\n🔒 Security Summary:`);
    console.log(`   Critical: ${this.auditReport.vulnerabilities.critical || 0}`);
    console.log(`   High: ${this.auditReport.vulnerabilities.high || 0}`);
    console.log(`   Moderate: ${this.auditReport.vulnerabilities.moderate || 0}`);
    console.log(`   Low: ${this.auditReport.vulnerabilities.low || 0}`);
    console.log(`   Total: ${totalVulns}`);

    return this.auditReport;
  }

  async run() {
    console.log('🚀 Starting security audit...');

    await this.runNpmAudit();
    await this.runSecurityScan();

    const report = this.generateReport();

    // Déterminer le statut
    const hasCritical = (this.auditReport.vulnerabilities.critical || 0) > 0;
    const hasHigh = (this.auditReport.vulnerabilities.high || 0) > 0;

    if (hasCritical || hasHigh) {
      console.log('❌ Security audit failed - Critical/High vulnerabilities found');
      process.exit(1);
    } else {
      console.log('✅ Security audit passed');
      process.exit(0);
    }
  }
}

// Exécuter l'audit
const auditor = new SecurityAuditor();
auditor.run().catch(console.error);
```

### Alertes de sécurité

#### Configuration des alertes
```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'team-benevoclic'

receivers:
  - name: 'team-benevoclic'
    discord_configs:
      - webhook_url: 'VOTRE_WEBHOOK_DISCORD_ICI'
        channel: '#alerts-benevoclic'
        title: '{{ .GroupLabels.alertname }}'
        text: '{{ .CommonAnnotations.summary }}'
        color: '{{ if eq .GroupLabels.severity "critical" }}FF0000{{ else if eq .GroupLabels.severity "warning" }}FFA500{{ else }}00FF00{{ end }}'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname']
```

## 🔧 Configuration de sécurité

### Headers de sécurité

#### Configuration Helmet
```typescript
// security.config.ts
import { NestFactory } from '@nestjs/core';
import * as helmet from 'helmet';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Headers de sécurité
  app.use(helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
        fontSrc: ["'self'", "https://fonts.gstatic.com"],
        imgSrc: ["'self'", "data:", "https:"],
        scriptSrc: ["'self'"],
        connectSrc: ["'self'", "https://api.benevoclic.com"],
        frameSrc: ["'none'"],
        objectSrc: ["'none'"],
        upgradeInsecureRequests: [],
      },
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
      preload: true,
    },
    noSniff: true,
    referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
    frameguard: { action: 'deny' },
    xssFilter: true,
  }));

  await app.listen(3000);
}
```

### Variables d'environnement sécurisées

#### Configuration des variables
```bash
# .env.production
NODE_ENV=production
PORT=3000

# Base de données
MONGODB_URL=mongodb://localhost:27017/benevoclic

# JWT
JWT_SECRET=votre-secret-jwt-tres-long-et-complexe
JWT_EXPIRES_IN=1h

# Firebase
FIREBASE_API_KEY=votre-api-key-firebase
FIREBASE_AUTH_DOMAIN=benevoclic.firebaseapp.com
FIREBASE_PROJECT_ID=benevoclic
FIREBASE_STORAGE_BUCKET=benevoclic.appspot.com
FIREBASE_MESSAGING_SENDER_ID=123456789
FIREBASE_APP_ID=1:123456789:web:abcdef

# AWS S3
AWS_ACCESS_KEY_ID=votre-access-key
AWS_SECRET_ACCESS_KEY=votre-secret-key
AWS_BUCKET_NAME=benevoclic-files
AWS_REGION=eu-west-3

# Chiffrement
ENCRYPTION_KEY=votre-cle-de-chiffrement-32-caracteres

# Monitoring
PROMETHEUS_PORT=9090
GRAFANA_PORT=3001
```

### Validation des variables d'environnement

#### Validateur d'environnement
```typescript
// utils/env-validator.ts
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class EnvValidator {
  private readonly logger = new Logger(EnvValidator.name);

  validateRequiredEnvVars(): void {
    const requiredVars = [
      'MONGODB_URL',
      'JWT_SECRET',
      'FIREBASE_API_KEY',
      'AWS_ACCESS_KEY_ID',
      'AWS_SECRET_ACCESS_KEY',
      'ENCRYPTION_KEY',
    ];

    const missingVars = requiredVars.filter(varName => !process.env[varName]);

    if (missingVars.length > 0) {
      this.logger.error(`Variables d'environnement manquantes: ${missingVars.join(', ')}`);
      throw new Error(`Variables d'environnement manquantes: ${missingVars.join(', ')}`);
    }

    this.logger.log('✅ Toutes les variables d\'environnement requises sont présentes');
  }

  validateJwtSecret(): void {
    const jwtSecret = process.env.JWT_SECRET;
    if (jwtSecret && jwtSecret.length < 32) {
      this.logger.warn('⚠️ Le secret JWT est trop court. Recommandé: au moins 32 caractères');
    }
  }

  validateEncryptionKey(): void {
    const encryptionKey = process.env.ENCRYPTION_KEY;
    if (encryptionKey && encryptionKey.length !== 32) {
      this.logger.error('❌ La clé de chiffrement doit faire exactement 32 caractères');
      throw new Error('Clé de chiffrement invalide');
    }
  }
}
```

## 📊 Métriques de sécurité

### Indicateurs de sécurité

| Métrique | Objectif | Seuil d'alerte |
|----------|----------|----------------|
| **Vulnérabilités critiques** | 0 | > 0 |
| **Vulnérabilités élevées** | 0 | > 0 |
| **Tentatives d'authentification échouées** | < 5% | > 10% |
| **Tentatives d'accès non autorisées** | 0 | > 0 |
| **Temps de réponse d'authentification** | < 500ms | > 1000ms |
| **Taux de mots de passe forts** | > 95% | < 90% |

### Dashboard de sécurité

```yaml
# grafana/dashboards/security.json
{
  "dashboard": {
    "title": "Security Dashboard",
    "panels": [
      {
        "title": "Security Events",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(security_events_total[5m])",
            "legendFormat": "{{event_type}}"
          }
        ]
      },
      {
        "title": "Failed Authentication Attempts",
        "type": "stat",
        "targets": [
          {
            "expr": "rate(auth_failures_total[5m])",
            "legendFormat": "Failed attempts/sec"
          }
        ]
      },
      {
        "title": "Active Sessions",
        "type": "stat",
        "targets": [
          {
            "expr": "active_sessions_total",
            "legendFormat": "Active sessions"
          }
        ]
      }
    ]
  }
}
```

---

*Ces mesures de sécurité garantissent la protection des données et la sécurité de la plateforme BeneVoclic.* 🚀 