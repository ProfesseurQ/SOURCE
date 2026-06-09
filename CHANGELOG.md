# CHANGELOG — SOURCE Ecosystem

Toutes les versions notables de SOURCE sont documentées ici.  
Format : `[version] — description` · Date approximative indiquée quand connue.

---

## [v0.6.53] — Juin 2026 *(version active)*

### SAGA Online v1.31
- Parcours scolaire rearchitecturé : lecture directe depuis `school_students.history` (JSONB cumulatif), une seule requête, accessible à tous les rôles sans contournement RLS
- Ajout élève en cours d'année : bouton "+ Ajouter", statut `signale_arrivee`, délai 5 jours ouvrables avant passage en `active`, annulation → `preinscrit` (jamais suppression définitive)
- DDN en lecture seule si `birth_date` existe en base de données
- Civilité, prénom et nom du professeur peuplés automatiquement depuis la session SOURCE (champs masqués dans l'interface)
- `getProf()` avec fallback session si le store est vide

### SAGA Direction v1.3
- Menu Signalements : remplace l'ancien menu "Départs signalés", deux onglets (Arrivées / Départs)
- Arrivées : confirmation → `active`, annulation → `preinscrit` + retrait `class_students`
- Wizard Nouvelle Année : écriture automatique de `school_students.history` pour chaque élève importé
- Gestion des titulaires inconnus (non bloquante, sélecteur manuel)
- `sdNavigate('classes')` après création (corrige bug redirection)

### ATLAS v6.6
- Conflits voulus : bouton "✓ Voulu" sur chaque conflit, stockage dans `atlas_data.conflits_valides`, badge excluant les conflits validés
- Badge conflits persisté en `localStorage` et restauré au login SOURCE
- Charges horaires : RPC `get_atlas_staff` enrichie avec `charge`, sauvegardée dans `school_users`
- Toast via `notifyParent` (correction bug affichage dans iframe)

### source-users v1.1
- Onglet "Classes SAGA" pour les spécialistes : cases à cocher par niveau, crée/supprime `class_teachers`

### Infrastructure
- Edge Function `admin-user` : `reset_password`, `update_email`, `create_user` via `service_role` sécurisée
- Correction désynchronisation emails : 5 comptes `auth.users` / `users` resynchronisés via SQL
- `_badgeCounts` déclaré globalement, restauré après `buildNav`
- Nouveau statut élève : `signale_arrivee` ajouté à la contrainte `school_students_status_check`
- Rétro-peuplement `school_students.history` pour toutes les années existantes

---

## [v0.6.38] — Mai–Juin 2026

### ATLAS v6.6
- Outil Nouvelle Année : remapping des interventions, surveillances, affectations et notes de cellules par UUID enseignant (format JSON v4)
- Sélecteur d'héritage de personnel par staffId (UUID) au lieu du niveau
- Badge de conflits persisté entre sessions
- Validation des "conflits voulus" avec collection dédiée dans `atlas_data`

### SAGA Direction v1.2
- Wizard Nouvelle Année avec barre de progression
- Support co-titulaires (split `/`, `&`, `+` dans le nom de classe)
- Gestion gracieuse des titulaires inconnus (non bloquante)
- Toast de confirmation après création de l'année

### SAGA Online v1.30
- `enrichirElevesSupabaseId()` : associe `_supabaseId`, DDN, NISS et statut `signaleArrivee` depuis Supabase
- Garantie niveau scolaire : requête `classes` si `curY().config.niveau` absent
- `verifierArrivees()` au chargement : passage automatique `signale_arrivee` → `active`

### Sécurité
- `get_my_global_role` / `get_my_school_role` migrées dans le schéma `private` avec wrappers `public` (REVOKE anon)
- `private.get_atlas_staff` : RPC privée avec wrapper public authentifié

---

## [v0.3 – v0.5] — Avril–Mai 2026

### Architecture
- Migration complète de l'écosystème SOURCE depuis les fichiers monolithiques vers une **architecture SPA multi-fichiers avec iframes**
- Portail `index.html` : login, shell, routing, sidebar, topbar, gestion des années
- Communication bidirectionnelle via `postMessage` (type `session_ready`, `navigate`, `module_ready`, `nav_page`, `update_badge`, `sync_state`, `toast`, `logout`)
- `SOURCE_CORE` : module partagé (auth, `supaFetch`, session, `notifyParent`, constantes)

### Corrections majeures de l'intégration
- `confirm()` bloqué dans les iframes → remplacé par `sourceConfirm()` avec `_confirmCallback`
- `window.location.href` dans les modules → remplacé par `notifyParent`
- RLS Supabase manquante sur `school_students` (UPDATE)
- Double topbar dans SAGA Direction
- Mode lecture seule ATLAS pour les titulaires
- Panel "Mon compte" bloquant la navigation vers ATLAS

### Migration ATLAS → Supabase
- Toutes les données ATLAS migrées de `localStorage` vers `atlas_data` (Supabase)
- Collections : `inters`, `survs`, `surv_affect`, `plages`, `jours`, `poncts`, `cell_notes`, `calendrier`, `cal_evts`, `type_survs`, `lieux_surv`, `grille_pres`
- Partage des données entre utilisateurs rendu possible

---

## [v0.1 – v0.2] — Début 2026

### Origine du projet
- **SAGA** : développé initialement comme outil de bulletins standalone → évolué vers un outil de suivi pédagogique longitudinal complet (maternelle → fin du primaire)
- **ATLAS** : développé initialement comme outil de gestion d'horaires standalone
- Première publication individuelle des deux outils sur GitHub (mai 2026) : repos `ProfesseurQ/SAGA` et `ProfesseurQ/ATLAS`

### Fonctionnalités initiales SAGA
- Bulletins trimestriels par classe
- Fiches élèves individuelles (santé, besoins, DAccE, entretiens parents)
- Grilles d'observation
- Parcours scolaire de la maternelle (M1, M2, M3) au primaire (P1–P6)
- Bloc licence copyright inséré (droit belge du 30 juin 1994)

### Fonctionnalités initiales ATLAS
- Horaires par enseignant
- Gestion des périodes, surveillances, interventions
- Personnel par rôle
- Bloc licence copyright inséré

---

## Roadmap

→ Voir [ROADMAP.md](./ROADMAP.md)
