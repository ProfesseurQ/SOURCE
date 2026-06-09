# ROADMAP — SOURCE Ecosystem

État au : **juin 2026** · Version active : **v0.6.53**

---

## En développement actif

### AGORA — Groupes de travail collaboratifs
Module de travail collaboratif entre enseignants d'une même école ou d'un même Pouvoir Organisateur.

**Concept :**
- Création et gestion de groupes de travail thématiques (conseil de classe, CPMS, projets pédagogiques…)
- Édition collaborative de documents avec historique de versions
- Kanban de tâches partagé
- Un seul groupe par utilisateur actif (contrainte de focus)

**État :** Schéma SQL (6 tables + RLS) finalisé. Choix techniques arrêtés (Quill pour les documents, Sortable.js pour le kanban, versionning par export/réimport). Implémentation du shell en cours.

**Technologies choisies :** Quill (CDN), Sortable.js (CDN), Supabase RLS

---

## Planifié

### TEMPO — Journal de classe numérique
Journal de classe numérique pour l'enseignant titulaire, intégré à l'écosystème SOURCE.

**Concept :**
- Enregistrement quotidien structuré : activités, présences, événements de classe
- Lien avec le calendrier scolaire institutionnel (périodes, congés, conférences pédagogiques)
- Référentiel des *attendus* officiels du programme FWB par année et par matière
- Connexion avec la fiche élève SAGA pour la continuité pédagogique

**Dépendances bloquantes (documentées) :**
1. ATLAS ne propose pas encore d'exposition structurée du calendrier institutionnel (congés, périodes, conférences pédagogiques) dans un format consommable par TEMPO
2. Absence d'une base de données des *attendus* officiels FWB par niveau et par matière

Ces deux dépendances sont identifiées et documentées dans le backlog. TEMPO reprendra dès qu'elles seront levées.

---

### SAGA — Module Maternelle `v1.32`
Extension de SAGA Online pour les enseignantes de maternelle.

**Concept :**
- Bulletin unique avec évaluation par appréciation (TB / B / AB / I ou équivalent)
- 3 colonnes de dates (Décembre / Avril / Juin)
- Domaines évalués :
  - Graphisme
  - Motricité
  - Pré-lecture
  - Mathématiques
  - Écoute
  - Langage
  - Comportement
- Toggle Maternelle / Primaire dans l'interface
- Outils partagés avec le primaire : fiches élèves, archives, page de garde

*Ce module est développé en consultation directe avec les enseignantes de maternelle de l'école. La structure des appréciations et le contenu exact des domaines feront l'objet d'une validation pédagogique avant implémentation.*

---

## Backlog priorisé

### Priorité haute
- **source-users** : droits différenciés direction (peut voir/modifier infos RH, désactiver) vs sub_admin (tous droits, y compris création et reset MDP)
- **source-users** : bouton "Activer compte SOURCE" pour les membres sans compte (`has_account: false → true`)
- **Sécurité** : déplacer `set_class_type_doc` dans le schéma privé Supabase (conformité RGPD)
- **Edge Function** : rate limiting, log d'audit, vérification école dans `create_user`, effacement `temp_password` après premier login

### Priorité moyenne
- **SAGA Direction** : nettoyage du code "Enseignants" redondant avec source-users
- **SAGA Online** : correction bug retour accueil pour le spécialiste NL (timing postMessage)
- **Co-titulariat primaire** : `bulletin_data` lié à `class_id` pour les classes à deux titulaires
- **ATLAS > Calendrier → SAGA Périodes** : à creuser avec la direction (même découpage mat/prim ?)

### Optimisations techniques
- **Wizard Nouvelle Année** : bulk insert (~1000 requêtes séquentielles → 5 batches). Gain estimé : 50s → 2s. Risque élevé — à tester avec données réelles avant déploiement.
- **saga-direction.js** : ~370 lignes de code mort (section Import Bulk, lignes 4202–4535)
- **saga-online-ui.js** : fonction `buildPrintZone` définie deux fois (3793 morte, 3827 active)
- **saga-online-core.js** : 2 `console.log('DEBUG...')` à retirer

### Backlog général
- Export PDF : bulletins SAGA + horaires ATLAS
- UX mobile / tablette Android (responsive)
- Accessibilité : 125 warnings a11y dans ATLAS
- Suppression dépendance `localStorage` dans SAGA Online (v1.33)
- Règle de nettoyage élèves par âge (>16–20 ans — règle à définir)
- Badge arrivées côté titulaire dans SAGA Direction
- Menus EPS et autres spécialistes dans SAGA Online

---

## Horizon Phase 2 (long terme)

- **Portail parents** : authentification forte via ItsMe (OIDC)
- **Intégration Registre National** : CSAM / données NISS officielles (SOURCE v2+)
- **Multi-école** : déploiement à l'échelle d'un Pouvoir Organisateur (4–5 écoles)
- **AGORA inter-écoles** : groupes de travail partagés entre établissements du PO
