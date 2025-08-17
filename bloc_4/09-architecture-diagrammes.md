# Architecture et Diagrammes Benevoclic

## 🏗️ Vue d'ensemble de l'architecture

L'architecture Benevoclic repose sur une **stack moderne** avec séparation claire des responsabilités, monitoring complet et haute disponibilité.

## 📊 Diagramme d'architecture globale

```mermaid
graph TB
    %% Frontend
    subgraph "Frontend - Nuxt.js"
        WEB[🌐 benevoclic-web<br/>Nuxt.js 3.16.0<br/>Vue 3.5.18<br/>Port: 5482]
        WEB_STORE[📦 Pinia Store]
        WEB_COMP[🧩 Components]
        WEB_PAGES[📄 Pages]
    end

    %% Backend
    subgraph "Backend - NestJS"
        API[🚀 benevoclic-api<br/>NestJS 11.0.7<br/>Port: 3000]
        API_AUTH[🔐 Auth Service]
        API_EVENTS[📅 Events Service]
        API_USERS[👥 Users Service]
        API_FILES[📁 Files Service]
    end

    %% Database
    subgraph "Base de données"
        MONGODB[(🗄️ MongoDB<br/>6.13.0<br/>Atlas/Production<br/>Local/Dev)]
    end

    %% External Services
    subgraph "Services externes"
        FIREBASE[🔥 Firebase Auth]
        AWS_S3[☁️ AWS S3<br/>Stockage fichiers]
        GOOGLE_MAPS[🗺️ Google Maps API]
    end

    %% Monitoring Stack
    subgraph "Monitoring & Alerting"
        PROMETHEUS[📊 Prometheus<br/>Port: 9090]
        GRAFANA[📈 Grafana<br/>Port: 3001]
        ALERTMANAGER[🚨 AlertManager<br/>Port: 9093]
        NODE_EXPORTER[📋 Node Exporter<br/>Port: 9100]
    end

    %% Process Manager
    subgraph "Process Management"
        PM2_API[⚙️ PM2 API]
        PM2_WEB[⚙️ PM2 Web]
    end

    %% CI/CD
    subgraph "CI/CD & DevOps"
        GITHUB[🐙 GitHub]
        DEPENDABOT[🤖 Dependabot]
        ACTIONS[⚡ GitHub Actions]
        DOCKER[🐳 Docker]
    end

    %% Connections
    WEB --> API
    API --> MONGODB
    API --> FIREBASE
    API --> AWS_S3
    API --> GOOGLE_MAPS
    
    API --> PROMETHEUS
    WEB --> PROMETHEUS
    PROMETHEUS --> GRAFANA
    PROMETHEUS --> ALERTMANAGER
    NODE_EXPORTER --> PROMETHEUS
    
    PM2_API --> API
    PM2_WEB --> WEB
    
    GITHUB --> ACTIONS
    DEPENDABOT --> GITHUB
    ACTIONS --> DOCKER
    
    %% Styling
    classDef frontend fill:#e1f5fe
    classDef backend fill:#f3e5f5
    classDef database fill:#e8f5e8
    classDef external fill:#fff3e0
    classDef monitoring fill:#fce4ec
    classDef process fill:#f1f8e9
    classDef cicd fill:#e0f2f1
    
    class WEB,WEB_STORE,WEB_COMP,WEB_PAGES frontend
    class API,API_AUTH,API_EVENTS,API_USERS,API_FILES backend
    class MONGODB database
    class FIREBASE,AWS_S3,GOOGLE_MAPS external
    class PROMETHEUS,GRAFANA,ALERTMANAGER,NODE_EXPORTER monitoring
    class PM2_API,PM2_WEB process
    class GITHUB,DEPENDABOT,ACTIONS,DOCKER cicd
```

## 🔄 Diagramme de séquence - Flux d'authentification

```mermaid
sequenceDiagram
    participant U as 👤 Utilisateur
    participant W as 🌐 Frontend (Nuxt.js)
    participant A as 🚀 API (NestJS)
    participant F as 🔥 Firebase
    participant M as 🗄️ MongoDB
    participant P as 📊 Prometheus
    participant G as 📈 Grafana

    Note over U,G: Flux d'authentification complet

    U->>W: 1. Accès à la page de connexion
    W->>W: 2. Initialisation Firebase
    W->>F: 3. Authentification Google
    F-->>W: 4. Token JWT
    
    W->>A: 5. POST /api/user/login
    Note right of A: Validation du token Firebase
    A->>M: 6. Recherche utilisateur dans collection users
    M-->>A: 7. Données utilisateur
    
    A->>A: 8. Création session JWT
    A->>M: 9. Sauvegarde session
    A-->>W: 10. Session + User data
    
    W->>W: 11. Stockage Pinia
    W->>W: 12. Redirection dashboard
    
    Note over P,G: Monitoring automatique
    A->>P: 13. Métriques auth via /metrics
    P->>G: 14. Visualisation temps réel
```

## 📈 Diagramme de séquence - Création d'annonce

```mermaid
sequenceDiagram
    participant U as 👤 Association
    participant W as 🌐 Frontend
    participant A as 🚀 API
    participant M as 🗄️ MongoDB
    participant S as ☁️ AWS S3
    participant P as 📊 Prometheus
    participant AM as 🚨 AlertManager

    Note over U,AM: Création d'une annonce d'événement

    U->>W: 1. Remplir formulaire annonce
    W->>W: 2. Validation côté client
    W->>A: 3. POST /api/announcements/createAnnouncement
    
    Note right of A: Validation + Authentification
    A->>M: 4. Vérification permissions dans collection users
    M-->>A: 5. Confirmation association
    
    U->>W: 6. Upload image annonce
    W->>A: 7. POST /api/announcements/upload-image
    A->>S: 8. Stockage fichier AWS S3
    S-->>A: 9. URL fichier
    
    A->>M: 10. Création annonce dans collection announcements
    M-->>A: 11. Annonce créée avec ID
    A-->>W: 12. Confirmation + ID
    
    W->>W: 13. Mise à jour interface
    W-->>U: 14. Annonce visible
    
    Note over P,AM: Monitoring et alertes
    A->>P: 15. Métriques création via /metrics
    P->>AM: 16. Vérification seuils
    alt Seuil dépassé
        AM->>AM: 17. Envoi alerte Discord
    end
```

## 🔍 Diagramme de séquence - Monitoring et alertes

```mermaid
sequenceDiagram
    participant P as 📊 Prometheus
    participant API as 🚀 API NestJS
    participant WEB as 🌐 Frontend Nuxt
    participant NE as 📋 Node Exporter
    participant AM as 🚨 AlertManager
    participant D as 💬 Discord
    participant G as 📈 Grafana

    Note over P,G: Cycle de monitoring continu

    loop Toutes les 15s
        P->>API: Scrape métriques API
        API-->>P: Métriques performance
        P->>WEB: Scrape métriques Frontend
        WEB-->>P: Métriques Lighthouse
        P->>NE: Scrape métriques système
        NE-->>P: CPU, RAM, Disque
    end

    P->>P: Évaluation des règles d'alerte
    
    alt Seuil critique dépassé
        P->>AM: Alerte critique
        AM->>D: Notification immédiate
        AM->>G: Mise à jour dashboard
    else Seuil warning dépassé
        P->>AM: Alerte warning
        AM->>D: Notification différée
    end

    G->>P: Requête métriques
    P-->>G: Données temps réel
    G->>G: Mise à jour dashboards
```

## 🏢 Architecture détaillée par couches

### Couche Présentation (Frontend)
```
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND - NUXT.JS                       │
├─────────────────────────────────────────────────────────────┤
│  🌐 Pages (Vue Router)                                      │
│  ├── / (Accueil)                                            │
│  ├── /auth/login (Connexion)                                │
│  ├── /association/* (Espace association)                    │
│  └── /volunteer/* (Espace bénévole)                         │
│                                                             │
│  🧩 Components (Vue 3)                                      │
│  ├── Header, Footer, Navigation                             │
│  ├── Forms (Inscription, Événements)                        │
│  ├── Maps (Maplibre GL)                                     │
│  └── Modals (Détails, Confirmation)                         │
│                                                             │
│  📦 Stores (Pinia)                                          │
│  ├── User Store (Authentification)                          │
│  ├── Events Store (Gestion événements)                      │
│  └── UI Store (État interface)                              │
│                                                             │
│  🔧 Composables (Logique métier)                            │
│  ├── useAuth (Authentification)                             │
│  ├── useEvents (Gestion événements)                         │
│  └── useGeolocation (Géolocalisation)                       │
└─────────────────────────────────────────────────────────────┘
```

### Couche Application (Backend)
```
┌─────────────────────────────────────────────────────────────┐
│                    BACKEND - NESTJS                         │
├─────────────────────────────────────────────────────────────┤
│  🚀 Controllers (Endpoints API)                             │
│  ├── AuthController (/api/auth/*)                           │
│  ├── UsersController (/api/users/*)                         │
│  ├── EventsController (/api/events/*)                       │
│  ├── AssociationsController (/api/associations/*)           │
│  └── FilesController (/api/files/*)                         │
│                                                             │
│  🔧 Services (Logique métier)                               │
│  ├── AuthService (Firebase + JWT)                           │
│  ├── UsersService (CRUD utilisateurs)                       │
│  ├── EventsService (Gestion événements)                     │
│  ├── FileService (AWS S3)                                   │
│  └── NotificationService (Emails + Push)                    │
│                                                             │
│  🗄️ Repositories (Accès données)                            │
│  ├── UserRepository (MongoDB)                               │
│  ├── EventRepository (MongoDB)                              │
│  ├── AssociationRepository (MongoDB)                        │
│  └── ParticipationRepository (MongoDB)                      │
│                                                             │
│  🛡️ Guards & Interceptors                                   │
│  ├── AuthGuard (Protection routes)                          │
│  ├── RoleGuard (Permissions)                                │
│  └── LoggingInterceptor (Monitoring)                        │
└─────────────────────────────────────────────────────────────┘
```

### Couche Infrastructure (Monitoring)
```
┌─────────────────────────────────────────────────────────────┐
│                  MONITORING STACK                           │
├─────────────────────────────────────────────────────────────┤
│  📊 Prometheus (Collecte métriques)                         │
│  ├── Métriques API (Temps réponse, erreurs)                 │
│  ├── Métriques Frontend (Lighthouse scores)                 │
│  ├── Métriques Système (CPU, RAM, Disque)                   │
│  └── Métriques Business (Utilisateurs, événements)          │
│                                                             │
│  📈 Grafana (Visualisation)                                 │
│  ├── Dashboard Principal (Vue d'ensemble)                   │
│  ├── Dashboard Technique (Détails)                          │
│  ├── Dashboard Business (Métriques utilisateurs)            │
│  └── Dashboard Sécurité (Tentatives d'accès)                │
│                                                             │
│  🚨 AlertManager (Gestion alertes)                          │
│  ├── Règles d'alerte (Seuils critiques/warning)             │
│  ├── Notifications Discord (Temps réel)                     │
│  ├── Notifications Email (Rapports)                         │
│  └── Escalade automatique (Critique)                        │
│                                                             │
│  ⚙️ PM2 (Process Manager)                                   │
│  ├── Monitoring processus (API + Web)                       │
│  ├── Logs structurés (JSON)                                 │
│  ├── Restart automatique (Crash)                            │
│  └── Métriques performance (CPU, RAM)                       │
└─────────────────────────────────────────────────────────────┘
```

## 🔄 Flux de données et communication

### Communication inter-services
```
┌─────────────┐    HTTP/REST    ┌─────────────┐
│   Frontend  │◄──────────────►│    API      │
│  (Nuxt.js)  │                 │  (NestJS)   │
└─────────────┘                 └─────────────┘
       │                              │
       │ WebSocket                    │ MongoDB
       │ (Real-time)                  │ Driver
       ▼                              ▼
┌─────────────┐                 ┌─────────────┐
│   Socket.IO │                 │   MongoDB   │
│  (Events)   │                 │  (Atlas)    │
└─────────────┘                 └─────────────┘
```

### Monitoring et observabilité
```
┌─────────────┐    Métriques    ┌─────────────┐
│   Prometheus│◄──────────────►│    API      │
│             │                 │  (NestJS)   │
└─────────────┘                 └─────────────┘
       │                              │
       │ Alerting                     │ Logs
       ▼                              ▼
┌─────────────┐                 ┌─────────────┐
│AlertManager │                 │   PM2       │
│             │                 │  (Logs)     │
└─────────────┘                 └─────────────┘
       │                              │
       │ Notifications                │ Monitoring
       ▼                              ▼
┌─────────────┐                 ┌─────────────┐
│   Discord   │                 │   Grafana   │
│  (Webhook)  │                 │ (Dashboard) │
└─────────────┘                 └─────────────┘
```

## 🎯 Points clés de l'architecture

### ✅ **Avantages de cette architecture**

1. **Séparation des responsabilités** : Frontend, Backend, Base de données, Monitoring
2. **Scalabilité** : Chaque service peut être mis à l'échelle indépendamment
3. **Observabilité** : Monitoring complet avec Prometheus + Grafana
4. **Haute disponibilité** : PM2 + AlertManager + Redémarrage automatique
5. **Sécurité** : Firebase Auth + JWT + Guards NestJS
6. **Performance** : Optimisations MongoDB

### 🔧 **Technologies utilisées**

- **Frontend** : Nuxt.js 3, Vue 3, Pinia, Tailwind CSS
- **Backend** : NestJS 11, TypeScript, MongoDB
- **Monitoring** : Prometheus, Grafana, AlertManager, PM2
- **Infrastructure** : Docker, AWS S3, Firebase
- **CI/CD** : GitHub Actions, Dependabot

### 📊 **Métriques de performance**

- **Temps de réponse API** : < 200ms (P95)
- **Disponibilité** : 99.9% (SLA)
- **Lighthouse Score** : > 80 (Performance)
- **Couverture de tests** : > 80%
- **Temps de résolution incidents** : < 5 minutes

## 🗄️ Schéma de base de données MongoDB

### Vue d'ensemble des collections

```mermaid
erDiagram
    USERS {
        String userId PK
        String email UK
        String role "VOLUNTEER|ASSOCIATION|ADMIN"
        Boolean isOnline
        Boolean disabled
        Boolean isVerified
        Boolean isCompleted
        Object location
        Object imageProfile
        String avatarFileKey
        String lastConnection
        String createdAt
        Date updatedAt
    }

    VOLUNTEERS {
        String volunteerId PK
        String bio
        String city
        String postalCode
        String country
        String birthDate
        String firstName
        String lastName
        String phone
        Object locationVolunteer
    }

    ASSOCIATIONS {
        String associationId PK
        String associationName
        String phone
        String bio
        String city
        String type
        String postalCode
        String country
        Object locationAssociation
        Array volunteers
        Array volunteersWaiting
    }

    ANNOUNCEMENTS {
        String id PK
        String description
        String datePublication
        String dateEvent
        String hoursEvent
        String nameEvent
        Array tags
        String associationId FK
        String associationName
        String associationLogo
        String announcementImage
        Object addressAnnouncement
        Object locationAnnouncement
        Array participants
        Number nbParticipants
        Number maxParticipants
        String status
        Boolean isHidden
        Number nbVolunteers
        Number maxVolunteers
        Array volunteers
        Array volunteersWaiting
    }

    FAVORITES {
        String volunteerId FK
        String announcementId FK
    }

    SUPPORT {
        String id PK
        String type
        String category
        String description
        String userId
        String userEmail
        String announcementId
        String status
        String priority
        Date createdAt
        Date updatedAt
        String adminNotes
        String userAgent
        String pageUrl
        String browserInfo
        String deviceInfo
        String screenshotUrl
    }

    ADMIN {
        String adminId PK
        String email
        String role
        Boolean isApproved
        Date createdAt
        Date updatedAt
    }

    USERS ||--o{ VOLUNTEERS : "has_profile"
    USERS ||--o{ ASSOCIATIONS : "has_profile"
    USERS ||--o{ ADMIN : "has_profile"
    ASSOCIATIONS ||--o{ ANNOUNCEMENTS : "creates"
    VOLUNTEERS ||--o{ FAVORITES : "has"
    ANNOUNCEMENTS ||--o{ FAVORITES : "is_favorited"
    USERS ||--o{ SUPPORT : "creates"
    ANNOUNCEMENTS ||--o{ SUPPORT : "reported_in"
```

### Détail des collections

#### Collection `users`
```javascript
{
  userId: String,                   // ID unique utilisateur (Firebase UID)
  email: String,                    // Email unique
  role: String,                     // "VOLUNTEER" | "ASSOCIATION" | "ADMIN"
  isOnline: Boolean,                // Statut en ligne
  disabled: Boolean,                // Compte désactivé
  isVerified: Boolean,              // Email vérifié
  isCompleted?: Boolean,            // Profil complété
  location?: {
    address: String,                // Adresse complète
    city: String,                   // Ville
    postalCode: String,             // Code postal
    country: String                 // Pays
  },
  imageProfile?: {
    data: String,                   // Données image (base64)
    contentType: String,            // Type MIME
    uploadedAt: Date                // Date upload
  },
  avatarFileKey?: String,           // Clé fichier avatar
  lastConnection: String,           // Dernière connexion
  createdAt: String,                // Date création
  updatedAt: Date                   // Date modification
}
```

#### Collection `volunteers`
```javascript
{
  volunteerId: String,              // ID unique bénévole (Firebase UID)
  bio?: String,                     // Biographie
  city?: String,                    // Ville
  postalCode?: String,              // Code postal
  country?: String,                 // Pays
  birthDate?: String,               // Date de naissance
  firstName: String,                // Prénom
  lastName: String,                 // Nom
  phone?: String,                   // Téléphone
  locationVolunteer?: {
    type: "Point",                  // Type GeoJSON
    coordinates: [Number, Number]   // [longitude, latitude]
  }
}
```

#### Collection `associations`
```javascript
{
  associationId: String,            // ID unique association (Firebase UID)
  associationName: String,          // Nom de l'association
  phone: String,                    // Téléphone
  bio: String,                      // Description
  city: String,                     // Ville
  type: String,                     // Type d'association
  postalCode: String,               // Code postal
  country: String,                  // Pays
  locationAssociation?: {
    type: "Point",                  // Type GeoJSON
    coordinates: [Number, Number]   // [longitude, latitude]
  },
  volunteers?: [{
    volunteerId: String,            // ID bénévole
    volunteerName: String,          // Nom bénévole
    dateAdded?: String              // Date ajout
  }],
  volunteersWaiting?: [{
    volunteerId: String,            // ID bénévole en attente
    volunteerName: String,          // Nom bénévole
    dateAdded?: String              // Date demande
  }]
}
```

#### Collection `announcements`
```javascript
{
  id?: String,                      // ID unique annonce
  description: String,              // Description de l'événement
  datePublication: String,          // Date de publication
  dateEvent: String,                // Date de l'événement
  hoursEvent: String,               // Heure de l'événement
  nameEvent: String,                // Nom de l'événement
  tags?: [String],                  // Tags/catégories
  associationId: String,            // ID association créatrice
  associationName: String,          // Nom association
  associationLogo?: String,         // Logo association
  announcementImage?: String,       // Image événement
  addressAnnouncement?: {
    address: String,                // Adresse complète
    city: String,                   // Ville
    postalCode: String,             // Code postal
    country: String                 // Pays
  },
  locationAnnouncement?: {
    type: "Point",                  // Type GeoJSON
    coordinates: [Number, Number]   // [longitude, latitude]
  },
  participants?: [{
    id: String,                     // ID participant
    name: String,                   // Nom participant
    isPresent?: Boolean,            // Présent à l'événement
    dateAdded?: String              // Date inscription
  }],
  nbParticipants?: Number,          // Nombre participants actuels
  maxParticipants: Number,          // Nombre max participants
  status: String,                   // Statut annonce
  isHidden?: Boolean,               // Annonce masquée
  nbVolunteers?: Number,            // Nombre bénévoles actuels
  maxVolunteers: Number,            // Nombre max bénévoles
  volunteers?: [{
    id: String,                     // ID bénévole
    name: String,                   // Nom bénévole
    isPresent?: Boolean,            // Présent à l'événement
    dateAdded?: String              // Date inscription
  }],
  volunteersWaiting?: [{
    id: String,                     // ID bénévole en attente
    name: String,                   // Nom bénévole
    isPresent?: Boolean,            // Présent à l'événement
    dateAdded?: String              // Date demande
  }]
}
```

#### Collection `favorites`
```javascript
{
  volunteerId: String,              // ID bénévole
  announcementId: String            // ID annonce
}
```

#### Collection `support`
```javascript
{
  id?: String,                      // ID unique rapport
  type: String,                     // Type de rapport
  category: String,                 // Catégorie spécifique
  description: String,              // Description du problème
  userId?: String,                  // ID utilisateur rapporteur
  userEmail?: String,               // Email utilisateur
  announcementId?: String,          // ID annonce concernée
  status: String,                   // Statut du rapport
  priority: String,                 // Priorité
  createdAt: Date,                  // Date création
  updatedAt: Date,                  // Date modification
  adminNotes?: String,              // Notes administrateur
  userAgent?: String,               // User agent navigateur
  pageUrl?: String,                 // URL page concernée
  browserInfo?: String,             // Informations navigateur
  deviceInfo?: String,              // Informations appareil
  screenshotUrl?: String            // URL capture d'écran
}
```

#### Collection `admin`
```javascript
{
  adminId: String,                  // ID unique administrateur
  email: String,                    // Email admin
  role: String,                     // Rôle administrateur
  isApproved: Boolean,              // Statut approbation
  createdAt: Date,                  // Date création
  updatedAt: Date                   // Date modification
}
```

### Agrégations MongoDB courantes

#### Statistiques des annonces par association
```javascript
db.announcements.aggregate([
  {
    $group: {
      _id: "$associationId",
      totalAnnouncements: { $sum: 1 },
      activeAnnouncements: {
        $sum: { $cond: [{ $eq: ["$status", "ACTIVE"] }, 1, 0] }
      },
      avgParticipants: { $avg: "$maxParticipants" },
      avgVolunteers: { $avg: "$maxVolunteers" }
    }
  },
  {
    $lookup: {
      from: "associations",
      localField: "_id",
      foreignField: "associationId",
      as: "association"
    }
  }
])
```

#### Annonces proches d'un utilisateur
```javascript
db.announcements.find({
  locationAnnouncement: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [longitude, latitude]
      },
      $maxDistance: 10000 // 10km
    }
  },
  status: "ACTIVE",
  dateEvent: { $gte: new Date().toISOString() }
})
```

#### Participants d'une annonce avec détails
```javascript
db.announcements.aggregate([
  {
    $match: { id: "announcement_id" }
  },
  {
    $unwind: "$participants"
  },
  {
    $lookup: {
      from: "volunteers",
      localField: "participants.id",
      foreignField: "volunteerId",
      as: "volunteer"
    }
  },
  {
    $unwind: "$volunteer"
  },
  {
    $group: {
      _id: "$participants.isPresent",
      count: { $sum: 1 },
      volunteers: { $push: "$volunteer" }
    }
  }
])
```

#### Favoris d'un bénévole avec détails des annonces
```javascript
db.favorites.aggregate([
  {
    $match: { volunteerId: "volunteer_id" }
  },
  {
    $lookup: {
      from: "announcements",
      localField: "announcementId",
      foreignField: "id",
      as: "announcement"
    }
  },
  {
    $unwind: "$announcement"
  },
  {
    $match: { "announcement.status": "ACTIVE" }
  },
  {
    $project: {
      _id: 0,
      announcementId: 1,
      nameEvent: "$announcement.nameEvent",
      dateEvent: "$announcement.dateEvent",
      associationName: "$announcement.associationName"
    }
  }
])
```

#### Rapports de support par catégorie
```javascript
db.support.aggregate([
  {
    $group: {
      _id: "$category",
      count: { $sum: 1 },
      avgPriority: { $avg: { $toInt: "$priority" } }
    }
  },
  {
    $sort: { count: -1 }
  }
])
```

Cette architecture garantit une **plateforme robuste, scalable et maintenable** pour l'écosystème Benevoclic ! 🚀
