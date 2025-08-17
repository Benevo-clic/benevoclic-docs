# Cahier de Recettes - Benevoclic

## Vue d'ensemble

Ce cahier de recettes présente les scénarios de tests fonctionnels pour valider le bon fonctionnement de l'application Benevoclic. Il couvre toutes les fonctionnalités principales et les cas d'usage critiques.

## 1. Tests d'Authentification

### 1.1 Inscription Bénévole

#### Scénario : Inscription d'un nouveau bénévole
# Scénario de Test - Inscription Bénévole

**Prérequis :** Aucun compte existant avec l'email *test@benevolat.fr*

---

## Étape 1 : Création du compte

**Actions :**
1. Accéder à la page d'accueil
2. Cliquer sur "S'inscrire"
3. Sélectionner "Bénévole"
4. Remplir le formulaire :
   - Email : *test@benevolat.fr*
   - Mot de passe : *MotDePasse123!*
   - Confirmation : *MotDePasse123!*
5. Accepter les conditions d'utilisation
6. Cliquer sur "Créer mon compte"

**Résultats attendus :**
- ✅ Message de confirmation d'inscription
- ✅ Email de vérification envoyé
- ✅ Redirection vers la page d'enregistrement du **bénévole**

---

## Étape 2 : Compléter le profil

**Actions :**
1. Remplir le formulaire :
   - Nom : "Dupont"
   - Prénom : "Marie"
   - Téléphone : "0123456789"
   - Adresse : "123 Rue de la Paix, 75001 Paris"
   - Bio : Une description du bénévole
2. Cliquer sur "Terminer"

**Résultats attendus :**
- ✅ Compte créé en base de données
- ✅ Redirection vers la page d’upload de la photo de **profil**

---

## Étape 3 : Ajouter une photo de profil

**Actions :**
1. Enregistrer une image :
   - Uploader une image depuis votre galerie
2. Cliquer sur "Terminer"
3. Ou cliquer sur "Ignorer"

**Résultats attendus :**
- ✅ Compte mis à jour en base de données
- ✅ Redirection vers la page d'accueil

---

## Validation

- [ ] Email reçu dans la boîte de réception
- [ ] Compte visible dans l'interface admin

### 1.2 Inscription Association

#### Scénario : Inscription d'une nouvelle association
**Prérequis :** Aucun compte existant avec le SIRET `12345678901234`

---

## Étape 1 : Création du compte

**Actions :**
1. Accéder à la page d'inscription
2. Sélectionner **"Association"**
3. Remplir le formulaire initial :
   - **SIRET** : `12345678901234`
   - **Email** : `contact@croixrouge-paris.fr`
   - **Mot de passe** : `Association123!`
   - **Confirmation mot de passe** : `Association123!`
4. Accepter les conditions d’utilisation
5. Cliquer sur **"Créer mon compte"**

**Résultats attendus :**
- ✅ Message confirmant la création du compte
- ✅ Email de vérification envoyé à l’adresse renseignée
- ✅ Redirection vers la **page de vérification de l’email**

---

## Étape 2 : Vérification de l’email

**Actions :**
1. Cliquer sur le lien de vérification reçu par email
2. Être redirigé vers la **page d’enregistrement des informations de l’association**

---

## Étape 3 : Enregistrement des informations de l’association

**Actions :**
1. Remplir le formulaire :
   - **Nom de l'association** : "Croix-Rouge Paris"
   - **Adresse** : "456 Avenue de la République, 75011 Paris"
   - **Téléphone** : "0123456789"
   - **Description** : "Association humanitaire locale"
2. Cliquer sur **"Valider"**

**Résultats attendus :**
- ✅ Données de l’association enregistrées en base
- ✅ Statut du compte : *En attente de validation*

---

## Étape 4 : Upload du logo

**Actions :**
1. Uploader un fichier PNG (< 2MB) depuis son ordinateur ou sa galerie
2. Cliquer sur **"Terminer"** ou sur **"Ignorer"**

**Résultats attendus :**
- ✅ Logo enregistré et associé à l’association (si fourni)
- ✅ Redirection vers le **dashboard de l'association**

---

## Validation

- [ ] Email de confirmation reçu
- [ ] Association visible dans l’interface admin
- [ ] Logo présent dans le profil (si uploadé)

### 1.3 Connexion Utilisateur

#### Scénario : Connexion avec des identifiants valides
**Prérequis :** Compte bénévole existant et vérifié

**Actions :**
1. Accéder à la page de connexion
2. Saisir l'email : "marie.dupont@email.com"
3. Saisir le mot de passe : "MotDePasse123!"
4. Cliquer sur "Se connecter"

**Résultats attendus :**
- ✅ Connexion réussie
- ✅ Redirection vers le dashboard bénévole
- ✅ Token JWT généré et stocké
- ✅ Session active

### 1.4 Récupération de Mot de Passe

#### Scénario : Demande de réinitialisation de mot de passe
**Prérequis :** Compte existant avec email valide

**Actions :**
1. Accéder à la page de connexion
2. Cliquer sur "Mot de passe oublié ?"
3. Saisir l'email : "marie.dupont@email.com"
4. Cliquer sur "Envoyer le lien"

**Résultats attendus :**
- ✅ Message de confirmation
- ✅ Email avec lien de réinitialisation envoyé
- ✅ Lien valide pendant 24h

## 2. Tests de Gestion des Événements

### 2.1 Création d'Événement

#### Scénario : Création d'un événement par une association
**Prérequis :** Compte association connecté et validé

**Actions :**
1. Se connecter en tant qu'association
2. Accéder au dashboard association
3. Cliquer sur "Créer un événement"
4. Remplir le formulaire :
   - Nom de l'événement : "Distribution alimentaire"
   - Description : "Distribution de repas aux personnes démunies"
   - Date de l'événement : "20/08/2025"
   - Heure : "14:00"
   - Lieu : "Centre social, 123 Rue de la Paix, 75001 Paris"
   - Nombre de bénévoles max : "10"
   - Nombre de participants max : "10"
   - Tags : alimentaire
   - Statut : ACTIVE
5. Cliquer sur "continuer"
6. Ajouter une image (optionnel)
7. Cliquer sur "Terminer"
8. Cliquer sur "Ignorer"


**Résultats attendus :**
- ✅ Événement créé et publié
- ✅ Événement visible dans la recherche
- ✅ Notification envoyée aux bénévoles intéressés
- ✅ Événement ajouté au calendrier

### 2.2 Modification d'Événement

#### Scénario : Modification d'un événement existant
**Prérequis :** Événement créé par l'association connectée

**Actions :**
1. Accéder à la liste des événements de l'association
2. Cliquer sur "Modifier" pour l'événement "Distribution alimentaire"
3. Modifier la description : "Distribution de repas chauds et colis alimentaires"
4. Modifier la date : "20/09/2025"
5. Cliquer sur "Enregistrer les modifications"

**Résultats attendus :**
- ✅ Modifications sauvegardées
- ✅ Notification aux bénévoles inscrits
- ✅ Historique des modifications conservé


## 3. Tests de Recherche et Inscription

### 3.1 Recherche d'Événements

#### Scénario : Recherche d'événements par localisation
**Prérequis :** Événements existants dans la base de données

**Actions :**
1. Accéder à la page de recherche
2. Saisir la ville : "Paris"
3. Sélectionner un rayon : "10 km"
4. Cliquer sur "Rechercher"

**Résultats attendus :**
- ✅ Liste d'événements affichée
- ✅ Événements triés par proximité
- ✅ Carte interactive avec les événements
- ✅ Filtres supplémentaires disponibles

### 3.2 Recherche Avancée

#### Scénario : Recherche avec filtres multiples
**Prérequis :** Événements de différentes catégories

**Actions :**
1. Accéder à la recherche avancée
2. Sélectionner la catégorie : "Environnement"
3. Choisir la date : "20/09/2025" à "20/11/2025"
4. Activer le filtre "statut"
5. Cliquer sur "Rechercher"

**Résultats attendus :**
- ✅ Événements filtrés selon les critères
- ✅ Nombre de résultats affiché
- ✅ Possibilité de sauvegarder la recherche

### 3.3 Inscription à un Événement

#### Scénario : Inscription d'un bénévole à un événement
**Prérequis :** Bénévole connecté, événement disponible

**Actions :**
1. Rechercher un événement disponible
2. Cliquer sur "Voir les détails"
3. Vérifier les informations de l'événement
4. Cliquer sur "S'inscrire"
5. Confirmer l'inscription

**Résultats attendus :**
- ✅ Inscription confirmée
- ✅ Événement ajouté au calendrier personnel
- ✅ Notification envoyée à l'association

### 3.4 Désinscription d'un Événement

#### Scénario : Désinscription d'un bénévole
**Prérequis :** Bénévole inscrit à un événement

**Actions :**
1. Accéder à "Mes événements"
2. Trouver l'événement inscrit
3. Cliquer sur "Se désinscrire"
4. Confirmer la désinscription

**Résultats attendus :**
- ✅ Désinscription confirmée
- ✅ Événement retiré du calendrier
- ✅ Place libérée pour d'autres bénévoles

## 4. Tests de Gestion des Profils

### 4.1 Modification du Profil Bénévole

#### Scénario : Mise à jour des informations personnelles
**Prérequis :** Compte bénévole connecté

**Actions :**
1. Accéder au profil personnel
2. Cliquer sur "Modifier le profil"
3. Modifier le téléphone : "0987654321"
4. Ajouter des compétences : "Premiers secours, Langue des signes"
5. Modifier la bio : "Passionnée par l'aide aux autres"
6. Sauvegarder les modifications

**Résultats attendus :**
- ✅ Profil mis à jour
- ✅ Modifications visibles immédiatement
- ✅ Données sauvegardées en base

## 5. Tests de Dashboard et Analytics

### 5.1 Dashboard Bénévole

#### Scénario : Consultation des statistiques personnelles
**Prérequis :** Bénévole avec historique d'activité

**Actions :**
1. Accéder au dashboard bénévole
2. Consulter les statistiques
3. Vérifier les badges obtenus
4. Consulter l'historique des événements

**Résultats attendus :**
- ✅ Statistiques correctes affichées
- ✅ Historique complet visible
- ✅ Graphiques interactifs fonctionnels

### 5.2 Dashboard Association

#### Scénario : Consultation des métriques de l'association
**Prérequis :** Association avec événements créés

**Actions :**
1. Accéder au dashboard association
2. Consulter les statistiques globales
3. Vérifier la liste des bénévoles actifs
4. Consulter les événements en cours

**Résultats attendus :**
- ✅ Métriques précises affichées
- ✅ Liste des bénévoles à jour
- ✅ Événements correctement listés
- ✅ Graphiques de tendances fonctionnels

## 6. Tests de Sécurité

### 6.1 Protection CSRF

#### Scénario : Tentative d'attaque CSRF
**Prérequis :** Session utilisateur active

**Actions :**
1. Ouvrir la console du navigateur
2. Exécuter une requête POST sans token CSRF
3. Vérifier la réponse

**Résultats attendus :**
- ❌ Requête rejetée
- ✅ Erreur 403 retournée
- ✅ Log de sécurité généré

### 7.2 Validation des Entrées

#### Scénario : Injection de script malveillant
**Prérequis :** Formulaire de création d'événement

**Actions :**
1. Saisir dans le titre : `<script>alert('XSS')</script>`
2. Soumettre le formulaire
3. Vérifier l'affichage

**Résultats attendus :**
- ✅ Script non exécuté
- ✅ Contenu affiché comme texte
- ✅ Validation côté client et serveur

### 7.3 Gestion des Sessions

#### Scénario : Expiration de session
**Prérequis :** Session utilisateur active

**Actions :**
1. Attendre l'expiration du token (24h)
2. Tenter d'accéder à une page protégée
3. Vérifier la redirection

**Résultats attendus :**
- ✅ Redirection vers la page de connexion
- ✅ Message d'expiration affiché
- ✅ Session correctement invalidée

## 8. Tests de Performance

### 8.1 Temps de Chargement

#### Scénario : Test de performance de la page d'accueil
**Prérequis :** Connexion internet normale

**Actions :**
1. Mesurer le temps de chargement initial
2. Vérifier le First Contentful Paint
3. Mesurer le Largest Contentful Paint

**Résultats attendus :**
- ✅ Temps de chargement < 3 secondes
- ✅ FCP < 1.5 secondes
- ✅ LCP < 2.5 secondes

### 8.2 Responsive Design

#### Scénario : Test sur différents appareils
**Prérequis :** Accès à différents écrans

**Actions :**
1. Tester sur desktop (1920x1080)
2. Tester sur tablette (768x1024)
3. Tester sur mobile (375x667)

**Résultats attendus :**
- ✅ Interface adaptée à chaque taille
- ✅ Navigation fonctionnelle
- ✅ Contenu lisible

## 9. Tests d'Accessibilité

### 9.1 Navigation Clavier

#### Scénario : Navigation complète au clavier
**Prérequis :** Aucun

**Actions :**
1. Naviguer sur le site uniquement avec Tab
2. Utiliser les touches fléchées
3. Activer les éléments avec Entrée/Espace

**Résultats attendus :**
- ✅ Tous les éléments accessibles
- ✅ Ordre de tabulation logique
- ✅ Indicateurs de focus visibles

### 9.2 Lecteurs d'Écran

#### Scénario : Test avec lecteur d'écran
**Prérequis :** Lecteur d'écran activé

**Actions :**
1. Naviguer sur le site avec le lecteur
2. Vérifier les labels des formulaires
3. Tester les messages d'erreur

**Résultats attendus :**
- ✅ Contenu correctement lu
- ✅ Labels appropriés
- ✅ Messages d'erreur clairs

## 10. Tests de Récupération

### 10.1 Récupération après Erreur

#### Scénario : Test de résilience de l'application
**Prérequis :** Application en fonctionnement

**Actions :**
1. Simuler une erreur de base de données
2. Vérifier l'affichage de la page d'erreur
3. Tester la récupération automatique

**Résultats attendus :**
- ✅ Page d'erreur appropriée affichée
- ✅ Récupération automatique
- ✅ Données non perdues

### 10.2 Sauvegarde et Restauration

#### Scénario : Test de sauvegarde
**Prérequis :** Données de test

**Actions :**
1. Effectuer une sauvegarde manuelle
2. Supprimer des données de test
3. Restaurer depuis la sauvegarde

**Résultats attendus :**
- ✅ Sauvegarde créée avec succès
- ✅ Données restaurées correctement
- ✅ Intégrité des données préservée

## 11. Tests d'Intégration

### 11.1 Intégration API

#### Scénario : Test complet du flux API
**Prérequis :** API accessible

**Actions :**
1. Créer un utilisateur via API
2. Authentifier l'utilisateur
3. Créer un événement
4. Récupérer la liste des événements
5. Supprimer les données de test

**Résultats attendus :**
- ✅ Toutes les opérations réussies
- ✅ Réponses API conformes
- ✅ Gestion d'erreurs appropriée

### 11.2 Intégration Base de Données

#### Scénario : Test des opérations CRUD
**Prérequis :** Base de données accessible

**Actions :**
1. Créer un enregistrement
2. Lire l'enregistrement
3. Modifier l'enregistrement
4. Supprimer l'enregistrement

**Résultats attendus :**
- ✅ Opérations réussies
- ✅ Intégrité des données
- ✅ Performance acceptable

## 12. Tests de Compatibilité

### 12.1 Compatibilité Navigateurs

#### Scénario : Test sur différents navigateurs
**Prérequis :** Accès à différents navigateurs

**Actions :**
1. Tester sur Chrome (dernière version)
2. Tester sur Firefox (dernière version)
3. Tester sur Safari (dernière version)
4. Tester sur Edge (dernière version)

**Résultats attendus :**
- ✅ Fonctionnement identique
- ✅ Aucune régression visuelle
- ✅ Performance similaire

### 12.2 Compatibilité Mobile

#### Scénario : Test sur différents appareils mobiles
**Prérequis :** Accès à différents appareils

**Actions :**
1. Tester sur iOS (iPhone, iPad)
2. Tester sur Android (téléphone, tablette)
3. Vérifier les fonctionnalités tactiles

**Résultats attendus :**
- ✅ Interface adaptée
- ✅ Gestes tactiles fonctionnels
- ✅ Performance optimale

## Conclusion

Ce cahier de recettes couvre l'ensemble des fonctionnalités critiques de l'application Benevoclic. Chaque test doit être exécuté régulièrement pour garantir la qualité et la fiabilité du système.

### Critères de Validation

- ✅ **Tous les tests passent** : 100% de réussite
- ✅ **Performance** : Temps de réponse < 3 secondes
- ✅ **Sécurité** : Aucune vulnérabilité détectée
- ✅ **Accessibilité** : Conformité WCAG 2.1 AA
- ✅ **Compatibilité** : Fonctionnement sur tous les navigateurs

### Fréquence des Tests

- **Tests critiques** : Avant chaque déploiement
- **Tests complets** : Hebdomadaire
- **Tests de performance** : Mensuel
- **Tests de sécurité** : Trimestriel
