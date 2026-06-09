# SOURCE — Suivi, Organisation et Unification des Ressources et Coordination des Établissements

> Écosystème numérique de gestion scolaire conçu pour l'enseignement fondamental en Belgique (Fédération Wallonie-Bruxelles)

**Auteur :** Quentin Elacalle — Enseignant, École Fondamentale La Source, Bruxelles  
**Contact :** elacalleq@gmail.com  
**Version active :** v0.6.53  
**Déployé sur :** [source.ecolelasource.be](https://source.ecolelasource.be)  
**Première publication :** mai 2026  
**© 2026 Quentin Elacalle — Tous droits réservés** · [Voir la licence](./LICENSE)

---

## Présentation

SOURCE est un écosystème logiciel complet de gestion scolaire développé de A à Z par un enseignant du fondamental pour répondre aux besoins réels du terrain en Fédération Wallonie-Bruxelles. Il regroupe en une plateforme unifiée le suivi pédagogique des élèves, la gestion des bulletins, les horaires du personnel, l'administration scolaire et — à terme — la collaboration entre enseignants et le journal de classe.

Le projet est né du constat qu'aucun outil existant ne répondait simultanément aux contraintes du système éducatif belge francophone : grille d'observation longitudinale de la maternelle au primaire, gestion des niveaux P1 à P6, parcours scolaire cumulatif, multi-rôles (direction, titulaire, spécialiste), et conformité RGPD avec données de mineurs.

SOURCE est utilisé en production à l'École Fondamentale La Source (Bruxelles) depuis 2026.

---

## Architecture générale

```
SOURCE (portail unifié)
├── Authentification centralisée (email / username + 2FA TOTP)
├── Shell iframe + routing postMessage
└── Modules
    ├── SAGA Online        ← Bulletins, fiches élèves, notes, parcours scolaire
    ├── SAGA Direction     ← Gestion école, nouvelle année, signalements
    ├── ATLAS              ← Horaires, personnel, conflits, surveillance
    ├── source-users       ← Gestion du personnel et des comptes
    ├── AGORA              ← Groupes de travail collaboratifs [en développement]
    └── TEMPO              ← Journal de classe enseignant [planifié]
```

**Stack technique :**
- Frontend : HTML / CSS / JavaScript vanilla, architecture multi-fichiers SPA
- Backend : [Supabase](https://supabase.com) (PostgreSQL + RLS + Edge Functions)
- Hébergement : PlanetHoster
- Communication inter-modules : `postMessage` (iframe ↔ portail)
- Auth : email/password + TOTP (Google Authenticator) pour les administrateurs

---

## Modules

### SAGA Online `v1.31` — Suivi pédagogique enseignant
Outil de suivi pédagogique longitudinal de l'élève, de la maternelle à la fin du primaire.

- Gestion des bulletins trimestriels par classe
- Fiches élèves individuelles (santé, besoins, entretiens parents, DAccE)
- Grilles d'observation et notes
- Parcours scolaire cumulatif (historique année par année)
- Ajout d'élève en cours d'année avec workflow de validation
- Accès multi-rôles : titulaire (édition), spécialiste (vue filtrée), direction (lecture)
- Module NL (cours de néerlandais) intégré

**Déployé également en standalone :** [saga.ecolelasource.be](https://saga.ecolelasource.be)  
**Repo antérieur (standalone) :** [ProfesseurQ/SAGA](https://github.com/ProfesseurQ/SAGA)

---

### SAGA Direction `v1.3` — Administration scolaire
Interface de gestion pour la direction et les administrateurs.

- Vue d'ensemble des classes et des élèves
- Wizard "Nouvelle Année" : import xlsx, matching NISS/DDN, détection titulaires, écriture automatique du parcours scolaire
- Menu Signalements : arrivées et départs signalés par les titulaires, validation/annulation par la direction
- Gestion des statuts élèves (actif, pré-inscrit, parti, parti définitif…)
- Règle absolue : jamais de suppression définitive d'un élève

---

### ATLAS `v6.6` — Horaires et personnel
Gestion complète des horaires et du personnel de l'école.

- Horaires par enseignant avec gestion des périodes, surveillances et interventions
- Gestion des conflits d'horaire avec validation des "conflits voulus"
- Personnel par rôle (maternelle, primaire, spécialiste, éducateur, ALE…)
- Charges horaires (temps plein, mi-temps, personnalisé)
- Outil "Nouvelle Année" avec remapping des données par UUID enseignant
- Accès lecture seule pour titulaires et spécialistes

**Repo antérieur (standalone) :** [ProfesseurQ/ATLAS](https://github.com/ProfesseurQ/ATLAS)

---

### source-users `v1.1` — Gestion du personnel
Interface centralisée de gestion du personnel pour l'administrateur.

- Liste, création et modification des membres du personnel
- Gestion des rôles SOURCE et ATLAS, couleurs, charges
- Reset de mot de passe sécurisé via Edge Function (sans exposition de clé service)
- Attribution des classes SAGA aux spécialistes par niveau

---

### AGORA — Groupes de travail collaboratifs *(en développement)*
Module de collaboration entre enseignants autour de groupes de travail thématiques.

- Création et gestion de groupes de travail scolaires
- Partage et édition collaborative de documents (Quill)
- Kanban de tâches (Sortable.js)
- Versionning manuel par export/réimport
- Architecture et schéma SQL finalisés — implémentation en cours

---

### TEMPO — Journal de classe *(planifié)*
Journal de classe numérique pour l'enseignant titulaire.

- Enregistrement quotidien des activités, présences et événements de classe
- Lien avec le calendrier scolaire institutionnel (ATLAS)
- Référentiel des *attendus* du programme par année et par matière

*Dépendances bloquantes identifiées : exposition structurée du calendrier scolaire depuis ATLAS ; base de données des attendus officiels FWB.*

---

## Sécurité et RGPD

SOURCE traite des données personnelles d'élèves mineurs. Les mesures techniques mises en place incluent :

- **Row Level Security (RLS)** Supabase sur toutes les tables sensibles
- **Schéma privé** pour les fonctions RPC à accès restreint (pas d'accès anonyme)
- **Edge Function sécurisée** pour toutes les opérations sur `auth.users` (service_role non exposé côté client)
- **2FA TOTP** obligatoire pour les comptes administrateurs
- **Pas de suppression définitive** des données élèves
- **HTTPS** exclusif, hébergement EU (PlanetHoster, Supabase EU West — Irlande)

---

## Historique et roadmap

- [CHANGELOG](./CHANGELOG.md) — Historique des versions
- [ROADMAP](./ROADMAP.md) — Fonctionnalités planifiées
- [MODULES](./MODULES.md) — Documentation détaillée par module

---

## Contexte éducatif

SOURCE est conçu pour le système éducatif de la **Fédération Wallonie-Bruxelles** (enseignement fondamental, cycles P1–P6 + maternelle M1–M3). Il tient compte des spécificités locales : grilles d'évaluation, parcours scolaire belge, cours de langue régionale (NL), structure des bulletins, rôles institutionnels (direction, titulaire, spécialiste, ALE…), et exigences RGPD pour les établissements scolaires.

---

## Licence

Ce projet est publié à titre de **preuve d'antériorité et de documentation publique**. Il n'est pas open source. Toute reproduction, utilisation ou adaptation sans autorisation écrite préalable est interdite.

→ [Lire la licence complète](./LICENSE)

Pour toute demande : **elacalleq@gmail.com**
