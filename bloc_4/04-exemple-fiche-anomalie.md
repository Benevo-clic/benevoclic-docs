# Exemple de Fiche d'Anomalie - Benevoclic

## 🐛 ANOM-2024-07-15-001 : Dégradation des Performances API

### 📋 Informations Générales

| Champ | Valeur |
|-------|--------|
| **ID Anomalie** | ANOM-2024-07-15-001 |
| **Date de détection** | 15/07/2024 14:30:00 |
| **Détecté par** | Prometheus AlertManager |
| **Sévérité** | Critical |
| **Catégorie** | Performance |
| **Statut** | Résolu |
| **Environnement** | Production |

### 📝 Description

#### Résumé
L'API Benevoclic présente une dégradation significative des performances avec un temps de réponse moyen de 2.5 secondes (vs 145ms normal), causant des timeouts pour les utilisateurs.

#### Impact Utilisateur
- **Utilisateurs affectés** : 150 utilisateurs actifs
- **Fonctionnalités impactées** : Recherche d'événements, inscription aux événements
- **Perte de service** : Partielle (timeouts fréquents)
- **Impact business** : Élevé (frustration utilisateur, abandon)

#### Symptômes Observés
- Temps de réponse API > 2 secondes
- Timeouts sur les requêtes de recherche
- Taux d'erreur 5xx à 8.5%
- CPU du serveur à 95%
- Mémoire utilisée à 87%

### 🔧 Informations Techniques

#### Environnement
- **Application** : Backend NestJS
- **Version** : 11.0.7
- **Serveur** : OVH VPS (4 cores, 8GB RAM)
- **Base de données** : MongoDB Atlas

#### Logs d'Erreur
```bash
# Logs PM2 - 15/07/2024 14:30-15:00
2024-07-15 14:30:15 ERROR [benevoclic-api] Database query timeout: 5000ms exceeded
2024-07-15 14:30:18 ERROR [benevoclic-api] Slow query detected: /api/events/search took 3200ms
2024-07-15 14:30:22 ERROR [benevoclic-api] Memory usage critical: 87% (6.96GB/8GB)
2024-07-15 14:30:25 ERROR [benevoclic-api] Connection pool exhausted: 100/100 connections used
```

#### Métriques au Moment de l'Incident
```yaml
# Métriques Prometheus - 14:30:00
system_metrics:
  cpu_usage: "95%"
  memory_usage: "87%"
  disk_usage: "45%"
  load_average: "3.8"

api_metrics:
  response_time_p95: "2500ms"
  response_time_p99: "4500ms"
  error_rate_5xx: "8.5%"
  active_connections: "100/100"
  database_queries_per_sec: "150"

business_metrics:
  active_users: "150"
  failed_requests: "12.5%"
  search_timeouts: "25%"
```

### 🛠️ Actions Entreprises

#### Actions Immédiates (14:30-14:45)
```bash
# 14:31 - Redémarrage du service
pm2 restart benevoclic-api

# 14:32 - Vérification du statut
pm2 status benevoclic-api
curl http://localhost:3000/health

# 14:35 - Augmentation des ressources
# Modification ecosystem.config.js
max_memory_restart: '2G'  # Augmenté de 1G à 2G

# 14:40 - Redémarrage avec nouvelles config
pm2 reload benevoclic-api
```

#### Actions de Résolution (14:45-15:30)
```bash
# 14:45 - Analyse des requêtes lentes
# MongoDB profiler activé
db.setProfilingLevel(2, { slowms: 1000 })

# 14:50 - Identification du problème
# Requête de recherche sans index sur le champ 'location'

# 15:00 - Création d'index manquant
db.events.createIndex({ "location.city": 1, "date": 1 })

# 15:15 - Optimisation de la requête
# Ajout de projection pour limiter les champs retournés

# 15:30 - Tests de validation
npm run test:integration
```

### ✅ Résolution

#### Correctif Appliqué
1. **Index manquant** : Création d'un index composite sur `location.city` et `date`
2. **Optimisation requête** : Ajout de projection pour limiter les champs retournés
3. **Configuration mémoire** : Augmentation de la limite mémoire PM2
4. **Monitoring renforcé** : Ajout d'alertes sur les requêtes lentes

#### Code du Correctif
```typescript
// Avant - Requête non optimisée
const events = await this.eventModel.find({
  'location.city': city,
  date: { $gte: new Date() }
}).exec();

// Après - Requête optimisée avec projection
const events = await this.eventModel.find({
  'location.city': city,
  date: { $gte: new Date() }
}, {
  title: 1,
  description: 1,
  date: 1,
  location: 1,
  maxVolunteers: 1,
  currentVolunteers: 1
}).exec();
```

#### Tests de Validation
- ✅ **Test de performance** : Temps de réponse < 200ms
- ✅ **Test de charge** : 1000 requêtes simultanées
- ✅ **Test d'intégration** : Tous les endpoints fonctionnels
- ✅ **Validation utilisateur** : Recherche fluide

### 📊 Analyse Post-Mortem

#### Cause Racine
**Index manquant sur la base de données** : La requête de recherche d'événements par ville et date effectuait un scan complet de la collection `events` au lieu d'utiliser un index optimisé.

#### Facteurs Contributifs
1. **Absence de monitoring des requêtes** : Pas d'alerte sur les requêtes lentes
2. **Tests de charge insuffisants** : Pas de test avec charge réelle
3. **Documentation manquante** : Index requis non documentés
4. **Monitoring mémoire insuffisant** : Pas d'alerte sur l'utilisation mémoire

#### Actions Préventives
```yaml
# Actions préventives mises en place
preventive_actions:
  - action: "Monitoring des requêtes MongoDB"
    responsible: "DevOps"
    deadline: "2024-07-20"
    status: "En cours"
  
  - action: "Tests de charge automatisés"
    responsible: "Lead Developer"
    deadline: "2024-07-25"
    status: "Planifié"
  
  - action: "Documentation des index requis"
    responsible: "Developer"
    deadline: "2024-07-18"
    status: "Terminé"
  
  - action: "Alertes mémoire renforcées"
    responsible: "DevOps"
    deadline: "2024-07-16"
    status: "Terminé"
```

### 📈 Métriques de Suivi

#### Temps de Résolution
| Phase | Durée | Responsable |
|-------|-------|-------------|
| **Détection → Décision** | 2 minutes | AlertManager |
| **Décision → Stabilisation** | 15 minutes | DevOps |
| **Stabilisation → Résolution** | 45 minutes | Developer |
| **Total** | 62 minutes | Équipe |

#### Coût Estimé
- **Temps perdu** : 2.5 heures (équipe technique)
- **Impact business** : 150 utilisateurs frustrés
- **Coût technique** : 62 minutes d'indisponibilité partielle

#### Amélioration Post-Résolution
| Métrique | Avant | Après | Amélioration |
|----------|-------|-------|--------------|
| **Temps de réponse** | 2500ms | 145ms | 94% |
| **Taux d'erreur** | 8.5% | 0.03% | 99.6% |
| **CPU moyen** | 95% | 35% | 63% |
| **Mémoire moyenne** | 87% | 45% | 48% |

### 📝 Notes et Commentaires

#### Leçons Apprises
1. **Importance des index** : Toujours valider les index requis pour les requêtes fréquentes
2. **Monitoring proactif** : Détecter les problèmes avant qu'ils n'affectent les utilisateurs
3. **Tests de charge** : Essentiels pour valider les performances en conditions réelles
4. **Documentation** : Cruciale pour éviter les régressions

#### Améliorations Futures
- [ ] Implémentation d'un cache Redis pour les requêtes fréquentes
- [ ] Migration vers MongoDB Atlas pour une meilleure scalabilité
- [ ] Mise en place d'un APM (Application Performance Monitoring)
- [ ] Automatisation des tests de performance

Cette fiche d'anomalie démontre la **capacité à détecter, analyser et résoudre** efficacement les problèmes de performance, garantissant la **continuité opérationnelle** de l'application Benevoclic.


