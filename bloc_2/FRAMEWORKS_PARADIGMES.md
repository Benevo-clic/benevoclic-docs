# Utilisation de Frameworks et Paradigmes - BeneVoclic

## 📋 Vue d'ensemble

BeneVoclic utilise des frameworks modernes et des paradigmes de développement éprouvés pour assurer la maintenabilité, la performance et la scalabilité. Le projet combine NestJS (backend) et Nuxt.js (frontend) avec des patterns architecturaux solides.

### Stack technologique

| Composant | Framework | Version | Paradigme |
|-----------|-----------|---------|-----------|
| **Backend** | NestJS | 11.0.7 | Architecture modulaire, DI, Decorators |
| **Frontend** | Nuxt.js | 3.16.0 | SSR, Composition API, Auto-imports |
| **Base de données** | MongoDB | 6.13.0 | Document-oriented, NoSQL |
| **État** | Pinia | 0.10.1 | Store pattern, Composition API |
| **Tests** | Jest/Vitest | 29.5.0/1.0.0 | TDD, BDD |
| **Authentification** | Firebase | 11.4.0 | Auth, JWT |
| **Stockage** | AWS S3 | 3.844.0 | Fichiers, images |

## 🏗️ Architecture et paradigmes

### Architecture en couches (Backend)

#### Structure modulaire NestJS
```typescript
// Architecture en couches
src/
├── main.ts                    # Point d'entrée
├── app.module.ts             # Module racine
├── config/                   # Couche configuration
│   ├── database.config.ts
│   ├── auth.config.ts
│   └── app.config.ts
├── modules/                  # Couche modules métier
│   ├── auth/
│   ├── users/
│   ├── events/
│   └── notifications/
├── common/                   # Couche partagée
│   ├── decorators/
│   ├── guards/
│   ├── interceptors/
│   └── filters/
└── test/                     # Couche tests
```

#### Pattern Module
```typescript
// modules/users/user.module.ts
@Module({
  imports: [
    MongooseModule.forFeature([{ name: User.name, schema: UserSchema }]),
    ConfigModule,
  ],
  controllers: [UserController],
  providers: [
    UserService,
    UserRepository,
    PasswordService,
    ValidationService,
  ],
  exports: [UserService, UserRepository],
})
export class UserModule {}
```

### Architecture composant (Frontend)

#### Structure Nuxt.js
```typescript
// Architecture composant
benevoclic-web/
├── app.vue                   # Composant racine
├── nuxt.config.ts           # Configuration
├── pages/                   # Couche pages (routing)
│   ├── index.vue
│   ├── auth/
│   └── volunteer/
├── components/              # Couche composants
│   ├── common/
│   ├── event/
│   └── layout/
├── composables/             # Couche logique métier
│   ├── useAuth.ts
│   ├── useEvents.ts
│   └── useApi.ts
├── stores/                  # Couche état global
│   ├── auth.store.ts
│   └── user.store.ts
└── server/                  # Couche API routes
    └── api/
```

#### Pattern Composant
```vue
<!-- components/event/EventCard.vue -->
<template>
  <article class="event-card" :class="{ 'is-loading': loading }">
    <header class="event-header">
      <h3 class="event-title">{{ event.title }}</h3>
      <EventStatus :status="event.status" />
    </header>
    
    <div class="event-content">
      <p class="event-description">{{ event.description }}</p>
      <EventMetadata :event="event" />
    </div>
    
    <footer class="event-footer">
      <EventActions 
        :event="event"
        @participate="handleParticipate"
        @share="handleShare"
      />
    </footer>
  </article>
</template>

<script setup lang="ts">
// Composition API pattern
interface Props {
  event: Event;
  loading?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  loading: false,
});

const emit = defineEmits<{
  participate: [eventId: string];
  share: [event: Event];
}>();

// Composables pour la logique métier
const { participateInEvent } = useEvents();
const { showNotification } = useNotifications();

const handleParticipate = async () => {
  try {
    await participateInEvent(props.event.id);
    emit('participate', props.event.id);
    showNotification('Participation enregistrée !', 'success');
  } catch (error) {
    showNotification('Erreur lors de la participation', 'error');
  }
};

const handleShare = () => {
  emit('share', props.event);
};
</script>
```

## 🔄 Paradigmes de programmation

### Programmation orientée objet (OOP)

#### Classes et héritage
```typescript
// Base entity class
abstract class BaseEntity {
  _id: string;
  createdAt: Date;
  updatedAt: Date;

  constructor(data: Partial<BaseEntity>) {
    Object.assign(this, data);
  }

  abstract validate(): boolean;
  
  toJSON(): object {
    return {
      id: this._id,
      createdAt: this.createdAt,
      updatedAt: this.updatedAt,
    };
  }
}

// User entity extending base
export class User extends BaseEntity {
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
  isActive: boolean;

  constructor(data: Partial<User>) {
    super(data);
  }

  validate(): boolean {
    return !!this.email && !!this.firstName && !!this.lastName;
  }

  get fullName(): string {
    return `${this.firstName} ${this.lastName}`;
  }

  isAdmin(): boolean {
    return this.role === UserRole.ADMIN;
  }

  canAccess(resource: string): boolean {
    const permissions = this.getPermissions();
    return permissions.includes(resource);
  }

  private getPermissions(): string[] {
    const rolePermissions = {
      [UserRole.ADMIN]: ['*'],
      [UserRole.ASSOCIATION]: ['events:create', 'events:manage', 'users:view'],
      [UserRole.VOLUNTEER]: ['events:view', 'events:participate'],
    };
    return rolePermissions[this.role] || [];
  }
}
```

### Programmation fonctionnelle (FP)

#### Fonctions pures et immutabilité
```typescript
// Utils fonctionnels
export const pipe = <T>(...fns: Array<(arg: T) => T>) => 
  (value: T): T => fns.reduce((acc, fn) => fn(acc), value);

export const compose = <T>(...fns: Array<(arg: T) => T>) => 
  (value: T): T => fns.reduceRight((acc, fn) => fn(acc), value);

// Fonctions pures pour la manipulation des données
export const filterActiveUsers = (users: User[]): User[] =>
  users.filter(user => user.isActive);

export const sortUsersByName = (users: User[]): User[] =>
  [...users].sort((a, b) => a.fullName.localeCompare(b.fullName));

export const groupUsersByRole = (users: User[]): Record<UserRole, User[]> =>
  users.reduce((acc, user) => {
    if (!acc[user.role]) acc[user.role] = [];
    acc[user.role].push(user);
    return acc;
  }, {} as Record<UserRole, User[]>);

// Composition de fonctions
export const getActiveUsersSorted = pipe(
  filterActiveUsers,
  sortUsersByName
);

export const getUsersByRoleSorted = pipe(
  groupUsersByRole,
  (grouped) => Object.fromEntries(
    Object.entries(grouped).map(([role, users]) => [
      role,
      sortUsersByName(users)
    ])
  )
);
```

### Programmation réactive (RP)

#### Observables et streams
```typescript
// Service réactif pour les événements
@Injectable()
export class EventReactiveService {
  private eventSubject = new BehaviorSubject<Event[]>([]);
  public events$ = this.eventSubject.asObservable();

  private userEventsSubject = new BehaviorSubject<UserEvent[]>([]);
  public userEvents$ = this.userEventsSubject.asObservable();

  constructor(
    private eventService: EventService,
    private userService: UserService,
  ) {
    this.initializeEventStream();
  }

  private initializeEventStream(): void {
    // Stream des événements avec filtres réactifs
    this.events$.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(events => this.filterUserRelevantEvents(events))
    ).subscribe(userEvents => {
      this.userEventsSubject.next(userEvents);
    });
  }

  private filterUserRelevantEvents(events: Event[]): Observable<UserEvent[]> {
    return this.userService.getCurrentUser().pipe(
      map(user => events.map(event => ({
        ...event,
        isRelevant: this.isEventRelevantForUser(event, user),
        userParticipation: this.getUserParticipation(event, user),
      })))
    );
  }

  // Méthodes réactives
  getEventsByCategory(category: string): Observable<Event[]> {
    return this.events$.pipe(
      map(events => events.filter(event => event.category === category))
    );
  }

  getUpcomingEvents(): Observable<Event[]> {
    return this.events$.pipe(
      map(events => events.filter(event => 
        new Date(event.date) > new Date()
      )),
      map(events => events.sort((a, b) => 
        new Date(a.date).getTime() - new Date(b.date).getTime()
      ))
    );
  }

  searchEvents(query: string): Observable<Event[]> {
    return this.events$.pipe(
      map(events => events.filter(event =>
        event.title.toLowerCase().includes(query.toLowerCase()) ||
        event.description.toLowerCase().includes(query.toLowerCase())
      ))
    );
  }
}
```

## 🎯 Patterns de conception

### Pattern Repository

#### Interface et implémentation
```typescript
// Interface du repository
export interface IUserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(userData: CreateUserDto): Promise<User>;
  update(id: string, userData: UpdateUserDto): Promise<User | null>;
  delete(id: string): Promise<boolean>;
  findNearby(location: GeoPoint, maxDistance: number): Promise<User[]>;
  findByRole(role: UserRole): Promise<User[]>;
}

// Implémentation MongoDB
@Injectable()
export class UserRepository implements IUserRepository {
  constructor(
    @InjectModel(User.name) private userModel: Model<User>,
    private logger: Logger,
  ) {}

  async findById(id: string): Promise<User | null> {
    try {
      return await this.userModel.findById(id).exec();
    } catch (error) {
      this.logger.error(`Error finding user by id: ${error.message}`);
      throw error;
    }
  }

  async findByEmail(email: string): Promise<User | null> {
    try {
      return await this.userModel.findOne({ email }).exec();
    } catch (error) {
      this.logger.error(`Error finding user by email: ${error.message}`);
      throw error;
    }
  }

  async create(userData: CreateUserDto): Promise<User> {
    try {
      const user = new this.userModel(userData);
      return await user.save();
    } catch (error) {
      this.logger.error(`Error creating user: ${error.message}`);
      throw error;
    }
  }

  async update(id: string, userData: UpdateUserDto): Promise<User | null> {
    try {
      return await this.userModel
        .findByIdAndUpdate(id, userData, { new: true })
        .exec();
    } catch (error) {
      this.logger.error(`Error updating user: ${error.message}`);
      throw error;
    }
  }

  async delete(id: string): Promise<boolean> {
    try {
      const result = await this.userModel.findByIdAndDelete(id).exec();
      return !!result;
    } catch (error) {
      this.logger.error(`Error deleting user: ${error.message}`);
      throw error;
    }
  }

  async findNearby(location: GeoPoint, maxDistance: number): Promise<User[]> {
    try {
      return await this.userModel.find({
        location: {
          $near: {
            $geometry: {
              type: 'Point',
              coordinates: [location.lng, location.lat],
            },
            $maxDistance: maxDistance * 1000,
          },
        },
      }).exec();
    } catch (error) {
      this.logger.error(`Error finding nearby users: ${error.message}`);
      throw error;
    }
  }

  async findByRole(role: UserRole): Promise<User[]> {
    try {
      return await this.userModel.find({ role }).exec();
    } catch (error) {
      this.logger.error(`Error finding users by role: ${error.message}`);
      throw error;
    }
  }
}
```

### Pattern Factory

#### Factory pour les notifications
```typescript
// Factory pour créer différents types de notifications
export abstract class Notification {
  abstract type: string;
  abstract userId: string;
  abstract title: string;
  abstract message: string;
  abstract status: 'pending' | 'sent' | 'failed';
  abstract createdAt: Date;
}

export class EmailNotification extends Notification {
  type = 'email';
  template?: string;
  attachments?: string[];

  constructor(data: Partial<EmailNotification>) {
    super();
    Object.assign(this, data);
  }
}

export class PushNotification extends Notification {
  type = 'push';
  data?: Record<string, any>;
  badge?: number;
  sound?: string;
}

export class SMSNotification extends Notification {
  type = 'sms';
  phone?: string;
  priority?: 'low' | 'normal' | 'high';
}

// Factory pattern
export class NotificationFactory {
  static createEmailNotification(data: {
    userId: string;
    title: string;
    message: string;
    template?: string;
  }): EmailNotification {
    return new EmailNotification({
      ...data,
      status: 'pending',
      createdAt: new Date(),
    });
  }

  static createPushNotification(data: {
    userId: string;
    title: string;
    message: string;
    data?: Record<string, any>;
  }): PushNotification {
    return new PushNotification({
      ...data,
      status: 'pending',
      createdAt: new Date(),
    });
  }

  static createSMSNotification(data: {
    userId: string;
    message: string;
    phone?: string;
  }): SMSNotification {
    return new SMSNotification({
      ...data,
      title: 'BeneVoclic',
      status: 'pending',
      createdAt: new Date(),
    });
  }

  static createFromTemplate(template: string, data: any): Notification {
    const templates = {
      'event-reminder': {
        type: 'email',
        title: 'Rappel d\'événement',
        message: `N'oubliez pas l'événement "${data.eventTitle}" demain !`,
      },
      'welcome': {
        type: 'email',
        title: 'Bienvenue sur BeneVoclic',
        message: 'Merci de vous être inscrit sur notre plateforme !',
      },
      'participation-confirmed': {
        type: 'push',
        title: 'Participation confirmée',
        message: `Votre participation à "${data.eventTitle}" a été confirmée.`,
      },
    };

    const templateData = templates[template];
    if (!templateData) {
      throw new Error(`Template ${template} not found`);
    }

    return this.createNotification({
      ...templateData,
      userId: data.userId,
      ...data,
    });
  }

  private static createNotification(data: any): Notification {
    switch (data.type) {
      case 'email':
        return this.createEmailNotification(data);
      case 'push':
        return this.createPushNotification(data);
      case 'sms':
        return this.createSMSNotification(data);
      default:
        throw new Error(`Unknown notification type: ${data.type}`);
    }
  }
}
```

### Pattern Observer

#### Système d'événements
```typescript
// Event emitter pour les événements métier
@Injectable()
export class EventEmitterService {
  private eventEmitter = new EventEmitter2();

  emit(event: string, data: any): void {
    this.eventEmitter.emit(event, data);
  }

  on(event: string, handler: (data: any) => void): void {
    this.eventEmitter.on(event, handler);
  }

  off(event: string, handler: (data: any) => void): void {
    this.eventEmitter.off(event, handler);
  }
}

// Observers pour les événements métier
@Injectable()
export class EventObserver {
  constructor(
    private eventEmitter: EventEmitterService,
    private notificationService: NotificationService,
    private logger: Logger,
  ) {
    this.initializeObservers();
  }

  private initializeObservers(): void {
    // Observer pour la création d'événements
    this.eventEmitter.on('event.created', this.handleEventCreated.bind(this));
    
    // Observer pour les participations
    this.eventEmitter.on('participation.created', this.handleParticipationCreated.bind(this));
    
    // Observer pour les mises à jour
    this.eventEmitter.on('event.updated', this.handleEventUpdated.bind(this));
  }

  private async handleEventCreated(event: Event): Promise<void> {
    try {
      this.logger.log(`Event created: ${event.id}`);

      // Notifier les utilisateurs intéressés
      await this.notifyInterestedUsers(event);

      // Envoyer une notification à l'association
      await this.notifyAssociation(event);
    } catch (error) {
      this.logger.error(`Error handling event created: ${error.message}`);
    }
  }

  private async handleParticipationCreated(participation: Participation): Promise<void> {
    try {
      this.logger.log(`Participation created: ${participation.id}`);

      // Notifier l'association
      await this.notifyAssociationOfParticipation(participation);

      // Notifier le participant
      await this.notifyParticipant(participation);
    } catch (error) {
      this.logger.error(`Error handling participation created: ${error.message}`);
    }
  }

  private async handleEventUpdated(event: Event): Promise<void> {
    try {
      this.logger.log(`Event updated: ${event.id}`);

      // Notifier les participants
      await this.notifyParticipants(event);
    } catch (error) {
      this.logger.error(`Error handling event updated: ${error.message}`);
    }
  }

  private async notifyInterestedUsers(event: Event): Promise<void> {
    // Logique pour notifier les utilisateurs intéressés
  }

  private async notifyAssociation(event: Event): Promise<void> {
    // Logique pour notifier l'association
  }

  private async notifyParticipants(event: Event): Promise<void> {
    // Logique pour notifier les participants
  }

  private async notifyAssociationOfParticipation(participation: Participation): Promise<void> {
    // Logique pour notifier l'association d'une nouvelle participation
  }

  private async notifyParticipant(participation: Participation): Promise<void> {
    // Logique pour notifier le participant
  }
}
```

## 🔧 Patterns d'injection de dépendances

### Injection de dépendances NestJS

#### Configuration des providers
```typescript
// Configuration des providers avec injection de dépendances
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      load: [databaseConfig, authConfig, appConfig],
    }),
    MongooseModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        uri: configService.get<string>('database.uri'),
        useNewUrlParser: true,
        useUnifiedTopology: true,
      }),
      inject: [ConfigService],
    }),
  ],
  providers: [
    {
      provide: 'DATABASE_CONNECTION',
      useFactory: async (configService: ConfigService) => {
        const uri = configService.get<string>('database.uri');
        return mongoose.connect(uri);
      },
      inject: [ConfigService],
    },
    {
      provide: 'LOGGER',
      useFactory: () => {
        return new Logger('BeneVoclic');
      },
    },
    {
      provide: 'CACHE_MANAGER',
      useFactory: () => {
        return new Map();
      },
    },
  ],
})
export class AppModule {}
```

### Composables Vue.js

#### Injection de dépendances frontend
```typescript
// Composables avec injection de dépendances
export const useApi = () => {
  const { $api } = useNuxtApp();
  return $api;
};

export const useAuth = () => {
  const { $auth } = useNuxtApp();
  return $auth;
};

export const useNotifications = () => {
  const { $notifications } = useNuxtApp();
  return $notifications;
};

// Plugin pour l'injection de dépendances
export default defineNuxtPlugin(() => {
  const config = useRuntimeConfig();
  
  // Injection de l'API
  const api = $fetch.create({
    baseURL: config.public.apiBase,
    headers: {
      'Content-Type': 'application/json',
    },
    onRequest({ request, options }) {
      // Ajout du token d'authentification
      const token = useCookie('auth-token');
      if (token.value) {
        options.headers = {
          ...options.headers,
          Authorization: `Bearer ${token.value}`,
        };
      }
    },
    onResponseError({ response }) {
      // Gestion globale des erreurs
      if (response.status === 401) {
        navigateTo('/auth/login');
      }
    },
  });

  // Injection du service d'authentification
  const auth = {
    user: ref(null),
    isAuthenticated: computed(() => !!auth.user.value),
    
    async login(credentials: LoginCredentials) {
      const response = await api('/auth/login', {
        method: 'POST',
        body: credentials,
      });
      
      auth.user.value = response.user;
      const token = useCookie('auth-token');
      token.value = response.access_token;
    },
    
    async logout() {
      auth.user.value = null;
      const token = useCookie('auth-token');
      token.value = null;
      navigateTo('/auth/login');
    },
    
    async checkAuth() {
      try {
        const response = await api('/auth/me');
        auth.user.value = response.user;
      } catch (error) {
        auth.user.value = null;
      }
    },
  };

  // Injection du service de notifications
  const notifications = {
    show(message: string, type: 'success' | 'error' | 'warning' = 'success') {
      // Implémentation des notifications
    },
  };

  return {
    provide: {
      api,
      auth,
      notifications,
    },
  };
});
```

## 📊 Métriques et performance

### Indicateurs de performance

#### Métriques des frameworks
```typescript
// Métriques de performance
export interface FrameworkMetrics {
  // NestJS metrics
  nestjs: {
    responseTime: number;
    memoryUsage: number;
    activeConnections: number;
    errorRate: number;
  };
  
  // Nuxt.js metrics
  nuxt: {
    pageLoadTime: number;
    bundleSize: number;
    lighthouseScore: number;
    timeToInteractive: number;
  };
  
  // Database metrics
  database: {
    queryTime: number;
    connectionPool: number;
    cacheHitRate: number;
  };
}

// Service de métriques
@Injectable()
export class MetricsService {
  private metrics: FrameworkMetrics = {
    nestjs: {
      responseTime: 0,
      memoryUsage: 0,
      activeConnections: 0,
      errorRate: 0,
    },
    nuxt: {
      pageLoadTime: 0,
      bundleSize: 0,
      lighthouseScore: 0,
      timeToInteractive: 0,
    },
    database: {
      queryTime: 0,
      connectionPool: 0,
      cacheHitRate: 0,
    },
  };

  updateNestJSMetrics(metrics: Partial<FrameworkMetrics['nestjs']>): void {
    this.metrics.nestjs = { ...this.metrics.nestjs, ...metrics };
  }

  updateNuxtMetrics(metrics: Partial<FrameworkMetrics['nuxt']>): void {
    this.metrics.nuxt = { ...this.metrics.nuxt, ...metrics };
  }

  updateDatabaseMetrics(metrics: Partial<FrameworkMetrics['database']>): void {
    this.metrics.database = { ...this.metrics.database, ...metrics };
  }

  getMetrics(): FrameworkMetrics {
    return this.metrics;
  }

  generateReport(): string {
    return `
# Rapport de performance - BeneVoclic

## NestJS (Backend)
- Temps de réponse moyen: ${this.metrics.nestjs.responseTime}ms
- Utilisation mémoire: ${this.metrics.nestjs.memoryUsage}MB
- Connexions actives: ${this.metrics.nestjs.activeConnections}
- Taux d'erreur: ${this.metrics.nestjs.errorRate}%

## Nuxt.js (Frontend)
- Temps de chargement: ${this.metrics.nuxt.pageLoadTime}ms
- Taille du bundle: ${this.metrics.nuxt.bundleSize}KB
- Score Lighthouse: ${this.metrics.nuxt.lighthouseScore}/100
- Time to Interactive: ${this.metrics.nuxt.timeToInteractive}ms

## Base de données
- Temps de requête moyen: ${this.metrics.database.queryTime}ms
- Pool de connexions: ${this.metrics.database.connectionPool}
- Taux de cache: ${this.metrics.database.cacheHitRate}%
    `;
  }
}
```

---

*L'utilisation de frameworks modernes et de paradigmes éprouvés garantit la qualité et la maintenabilité de BeneVoclic.* 🚀 