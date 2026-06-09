# MODULES — Documentation technique SOURCE

> Ce document décrit l'architecture et les responsabilités de chaque module de l'écosystème SOURCE.  
> Il ne contient pas de code source.

---

## Portail SOURCE `v0.6`

**Rôle :** Point d'entrée unique de l'écosystème. Gère l'authentification, le shell visuel et le routing entre modules.

**Responsabilités :**
- Login par email ou nom d'utilisateur + mot de passe
- 2FA TOTP (Google Authenticator) pour les comptes sub_admin
- Sidebar de navigation adaptée au rôle de l'utilisateur connecté
- Chargement des modules dans un `<iframe>` isolé
- Communication bidirectionnelle avec les modules via `postMessage`
- Toggle Direction / Titulaire (pour les comptes sub_admin qui cumulent les deux rôles)
- Gestion de l'année scolaire active
- Système de badges sidebar (signalements, conflits…)
- Affichage des toasts de notification

**Rôles gérés :** `sub_admin`, `direction`, `titulaire`, `specialiste`, `educateur`, `ale`, `autre`

---

## SAGA Online `v1.31`

**Acronyme :** Suivi et Accompagnement Global des Apprenants  
**Accès :** titulaire (édition complète), spécialiste (vue filtrée par classe assignée), sub_admin (mode direction), direction (lecture)

**Responsabilités :**
- Gestion des bulletins trimestriels par classe et par élève
- Fiches élèves individuelles : informations générales, santé et besoins, entretiens avec les parents, DAccE (Dossier d'Accompagnement de l'Élève), observations libres
- Grilles d'observation et notation
- Parcours scolaire longitudinal de chaque élève (de la maternelle à la fin du primaire), stocké comme historique cumulatif en base de données
- Ajout d'élève en cours d'année avec workflow de validation (délai de 5 jours ouvrables, notification à la direction)
- Module NL intégré pour le cours de néerlandais (spécialiste NL)
- Auto-logout après 45 minutes d'inactivité
- Fonctionnement en mode lecture pour les années scolaires passées

**Règles métier fondamentales :**
- Jamais de suppression définitive d'un élève (au minimum : statut `preinscrit`)
- DDN (date de naissance) en lecture seule si déjà présente en base de données
- Chaque fiche est liée à son auteur (`_author`) pour les classes à plusieurs intervenants

**Fichiers :**
```
modules/saga-online/
├── index.html
├── saga-online-core.js    ← Auth, initialisation, parcours scolaire, ajout élève
├── saga-online-ui.js      ← Store local, bulletins, notes, fiches, aperçu impression
├── saga-online-charts.js  ← Graphiques (Chart.js)
├── saga-online-nl.js      ← Module NL complet
└── saga-online.css
```

---

## SAGA Direction `v1.3`

**Accès :** direction, sub_admin

**Responsabilités :**
- Vue d'ensemble des classes et de leurs élèves
- Gestion de la liste des élèves (statuts, informations)
- Wizard "Nouvelle Année" :
  - Import d'un fichier xlsx (colonnes : Nom, Prénom, Classe, DDN, NISS)
  - Matching des élèves existants par NISS, puis DDN+Nom, puis Nom exact
  - Détection automatique des titulaires depuis le nom de classe
  - Écriture automatique du parcours scolaire (`school_students.history`) pour chaque élève
  - Création des classes, inscriptions et affectations pour la nouvelle année
- Menu Signalements :
  - Onglet Arrivées : élèves signalés en cours d'année par les titulaires (statut `signale_arrivee`)
  - Onglet Départs : élèves signalés partis par les titulaires (statut `signale_parti`)
  - Validation ou annulation de chaque signalement
- Enrichissement des données élèves (DDN, NISS) par import

**Statuts élèves gérés :**
| Statut | Signification |
|---|---|
| `active` | Élève actif dans l'école |
| `preinscrit` | Pré-inscrit ou arrivée annulée |
| `attente` | En attente d'inscription |
| `signale_arrivee` | Signalé par un titulaire, en attente de validation direction |
| `signale_parti` | Signalé parti par un titulaire, en attente de confirmation |
| `parti` | Départ confirmé (provisoire) |
| `parti_definitif` | Fin de scolarité dans l'établissement |

**Fichiers :**
```
modules/saga-direction/
├── index.html
├── saga-direction.js
└── saga-direction.css
```

---

## ATLAS `v6.6`

**Accès :** direction, sub_admin (édition complète) · titulaire, spécialiste (lecture seule — vue de son propre horaire)

**Responsabilités :**
- Gestion complète des horaires de tout le personnel de l'école
- Périodes horaires configurables (plages, jours)
- Interventions : qui enseigne quoi, quand, dans quelle classe
- Surveillances : attribution des surveillance par lieu et période
- Événements ponctuels (réunions, conférences, activités)
- Gestion des conflits d'horaire avec distinction conflits réels / conflits voulus
- Charges horaires du personnel (temps plein, mi-temps, personnalisé)
- Notes de cellules sur la grille
- Outil "Nouvelle Année" : création de l'année suivante avec héritage sélectif du personnel et remapping des données
- Calendrier scolaire institutionnel
- Pré-sélection automatique de l'horaire personnel en mode lecture seule

**Rôles ATLAS :** `mat` (maternelle), `prim` (primaire), `spec` (spécialiste), `educ` (éducateur), `ale`, `autre`

**Fichiers :**
```
modules/atlas/
├── index.html
├── atlas.js
└── atlas.css
```

---

## source-users `v1.1`

**Accès :** sub_admin uniquement

**Responsabilités :**
- Liste complète du personnel de l'établissement
- Création de nouveaux membres (avec ou sans compte SOURCE)
- Modification : civilité, prénom/nom, display_name pour ATLAS, rôles, couleur d'affichage, charge horaire
- Reset de mot de passe sécurisé (via Edge Function — la clé `service_role` n'est jamais exposée côté client)
- Onglet "Classes SAGA" pour les spécialistes : affectation par niveau scolaire (crée/supprime les entrées `class_teachers`)
- Filtres : par rôle, par statut de compte, recherche textuelle

**Fichiers :**
```
modules/source-users/
├── index.html
├── source-users.js
└── source-users.css
```

---

## Infrastructure commune

### source-core.js
Module JavaScript partagé chargé dans chaque sous-module. Fournit :
- Authentification et gestion de session (`SESSION_CORE.session`)
- `supaFetch()` : wrapper fetch Supabase avec gestion JWT et erreurs
- `notifyParent()` : envoi de messages `postMessage` vers le portail SOURCE
- `sourceConfirm()` : remplacement de `confirm()` (bloqué dans les iframes)
- Constantes partagées (`SCHOOL_ID`, `SCHOOL_YR`, URL Supabase)

### Schéma de base de données (Supabase)
Tables principales : `users`, `school_users`, `schools`, `classes`, `class_teachers`, `class_students`, `students`, `school_students`, `bulletin_data`, `student_records`, `atlas_data`, `atlas_staff_year`

Sécurité : Row Level Security (RLS) sur toutes les tables sensibles · Fonctions dans un schéma `private` non accessible aux requêtes anonymes · Edge Function `admin-user` pour les opérations nécessitant la clé `service_role`

### Communication postMessage
Protocole de communication entre le portail SOURCE (parent) et les modules (iframes) :

| Direction | Type | Rôle |
|---|---|---|
| SOURCE → Module | `session_ready` | Déclenche l'initialisation du module |
| SOURCE → Module | `navigate` | Navigue vers une page interne |
| SOURCE → Module | `change_year` | Change l'année scolaire active |
| Module → SOURCE | `module_ready` | Signale que le module est prêt |
| Module → SOURCE | `nav_page` | Synchronise la sidebar |
| Module → SOURCE | `update_badge` | Met à jour un badge de la sidebar |
| Module → SOURCE | `sync_state` | Met à jour l'indicateur de synchronisation |
| Module → SOURCE | `toast` | Affiche une notification |
| Module → SOURCE | `logout` | Déclenche la déconnexion |
