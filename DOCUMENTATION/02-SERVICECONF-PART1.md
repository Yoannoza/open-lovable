# 📋 SERVICECONF.JSON - CONFIGURATION COMPLÈTE

**Documentation exhaustive du fichier de configuration principal d'un e-service.**

---

## 🎯 RÔLE DU FICHIER

Le fichier `PSxxxxx.serviceconf.json` est le **cœur** de votre e-service. Il définit:

- ✅ L'identité du service (ID, version, visibilité)
- ✅ Le workflow complet (états et transitions)
- ✅ Les permissions (qui peut faire quoi)
- ✅ Les intégrations externes (UXP, paiement)
- ✅ Les génér actions de documents (PDF)
- ✅ Les notifications (email, SMS)

**Sans ce fichier, votre e-service n'existe pas.**

---

## 📁 EMPLACEMENT

```
PSxxxxx/
└── x.y/
    └── PSxxxxx.serviceconf.json    ← ICI
```

**Règles strictes:**
- ✅ Nom DOIT être: `{serviceId}.serviceconf.json`
- ✅ Extension DOIT être: `.json` (pas `.jsonc`, pas `.txt`)
- ✅ Encodage DOIT être: UTF-8
- ✅ Format DOIT être: JSON valide (pas de commentaires `//`)

---

## 🏗️ STRUCTURE RACINE

### Exemple minimal fonctionnel

```json
{
  "serviceId": "PS00565",
  "serviceVersion": "0.1",
  "public": true,
  "stages": {
    "start": { ... },
    "citizen-input": { ... },
    "REQUESTED": { ... },
    "APPROVED": { ... }
  }
}
```

### Exemple complet avec options

```json
{
  "serviceId": "PS00565",
  "serviceVersion": "0.1",
  "public": true,
  "backOffice": {
    "customColumnOfficial": "data.company.name",
    "customColumnTitleOfficial": {
      "en": "COMPANY NAME",
      "fr": "NOM COMMERCIAL"
    }
  },
  "stages": { ... }
}
```

---

## 🔧 PROPRIÉTÉS RACINE

### 1. `serviceId` (STRING - OBLIGATOIRE)

**Description:** Identifiant unique du service dans CatIS.

**Format:** `PSxxxxx` où `xxxxx` = 5 chiffres

**Exemples valides:**
```json
"serviceId": "PS00565"
"serviceId": "PS01259"
"serviceId": "PS00906"
```

**Exemples invalides:**
```json
"serviceId": "ps00565"     ❌ (minuscules)
"serviceId": "PS565"       ❌ (moins de 5 chiffres)
"serviceId": "PS-00565"    ❌ (tiret)
"serviceId": "SERVICE001"  ❌ (format incorrect)
```

**⚠️ RÈGLES CRITIQUES:**
1. DOIT correspondre au nom du dossier racine
2. DOIT être enregistré dans CatIS
3. NE PEUT PAS être modifié après création
4. Doit être unique dans tout le portail

**Exemple complet:**
```
Dossier: PS00565/
Fichier: PS00565/0.1/PS00565.serviceconf.json
Contenu: {
  "serviceId": "PS00565",    ← DOIT correspondre!
  ...
}
```

---

### 2. `serviceVersion` (STRING - OBLIGATOIRE)

**Description:** Version du service.

**Format:** `x.y` (version sémantique simplifiée)

**Exemples valides:**
```json
"serviceVersion": "0.1"
"serviceVersion": "0.2"
"serviceVersion": "1.0"
"serviceVersion": "2.3"
```

**Exemples invalides:**
```json
"serviceVersion": "v0.1"      ❌ (préfixe v)
"serviceVersion": "0.1.0"     ❌ (3 chiffres)
"serviceVersion": "1"         ❌ (manque .y)
"serviceVersion": 0.1         ❌ (nombre au lieu de string)
```

**⚠️ RÈGLES:**
1. DOIT correspondre au nom du dossier de version
2. Format `"x.y"` en STRING (pas de nombre)
3. Incrémenter pour nouvelles versions

**Exemple de versioning:**
```
PS00565/
├── 0.1/
│   └── PS00565.serviceconf.json
│       → "serviceVersion": "0.1"
├── 0.2/
│   └── PS00565.serviceconf.json
│       → "serviceVersion": "0.2"
└── 1.0/
    └── PS00565.serviceconf.json
        → "serviceVersion": "1.0"
```

---

### 3. `public` (BOOLEAN - OBLIGATOIRE)

**Description:** Définit si le service est accessible sans authentification.

**Valeurs possibles:**
- `true` : Service public, démarre sans login
- `false` : Authentification requise pour démarrer

**Exemples:**

#### Service public (citoyen lambda)
```json
{
  "serviceId": "PS00565",
  "public": true,
  "stages": {
    "start": {
      "permissions": {
        "type": "meta-array",
        "actor": "CITIZEN",
        "metaPathToArray": ["citizen"],
        "public": true    ← Cohérent avec public:true racine
      }
    }
  }
}
```

#### Service réservé (authentification requise)
```json
{
  "serviceId": "PS01259",
  "public": false,
  "stages": {
    "start": {
      "permissions": {
        "type": "meta-array",
        "actor": "CITIZEN",
        "metaPathToArray": ["citizen"],
        "public": false    ← Cohérent avec public:false racine
      }
    }
  }
}
```

**⚠️ INTERACTION AVEC `start.permissions.public`:**

| `public` racine | `start.permissions.public` | Résultat |
|-----------------|---------------------------|----------|
| `true` | `true` | ✅ Accessible sans login |
| `true` | `false` | ⚠️ Login requis pour CONTINUER après start |
| `false` | `true` | ⚠️ Incohérent (éviter) |
| `false` | `false` | ✅ Login requis dès le départ |

**Recommandation:** Garder cohérence entre les deux.

---

### 4. `backOffice` (OBJECT - OPTIONNEL)

**Description:** Configuration de l'affichage dans le back-office (interface agents).

**Propriétés:**
- `customColumnOfficial` (string) : Chemin vers donnée à afficher en colonne
- `customColumnTitleOfficial` (object) : Titre traduit de la colonne

**Exemple complet:**
```json
{
  "backOffice": {
    "customColumnOfficial": "data.company.name",
    "customColumnTitleOfficial": {
      "en": "COMPANY NAME",
      "fr": "NOM COMMERCIAL"
    }
  }
}
```

**Résultat dans le back-office:**

| Ref | Citizen | Status | **NOM COMMERCIAL** | Date |
|-----|---------|--------|--------------------|------|
| 001 | John Doe | REQUESTED | **Entreprise ABC** | 2025-10-24 |
| 002 | Jane Smith | APPROVED | **Société XYZ** | 2025-10-23 |

**Chemins possibles:**
```json
"customColumnOfficial": "data.person.first_name"        // Prénom
"customColumnOfficial": "data.company.name"             // Nom entreprise
"customColumnOfficial": "data.document.type"            // Type document
"customColumnOfficial": "metaData.request_category"     // Catégorie
```

**⚠️ Si le chemin n'existe pas:**
- La colonne affiche "(vide)" ou "-"
- Pas d'erreur générée

**Omission:**
Si `backOffice` est omis, seules les colonnes par défaut sont affichées:
- Référence
- Citoyen
- Statut
- Date création
- Date dernière modification

---

## 🎭 STAGES - CŒUR DU WORKFLOW

### Vue d'ensemble

```json
{
  "stages": {
    "start": { ... },                    // 1 seul obligatoire
    "citizen-input": { ... },            // Stages intermédiaires
    "REQUESTED": { ... },                // Main stages
    "official-review": { ... },
    "gen-certificate": { ... },
    "APPROVED": { ... },                 // Final stage
    "REJECTED": { ... }                  // Final stage
  }
}
```

**Types de stages:**
- **Obligatoires:** `start` (1 seul)
- **Main stages:** États majeurs (REQUESTED, APPROVED, REJECTED...)
- **Intermediate stages:** Actions entre états (ui, uxp, email, pdf...)

### Convention de nommage

| Type | Convention | Exemples |
|------|-----------|----------|
| `start` | `"start"` (fixe) | `start` |
| Main stages | MAJUSCULES | `REQUESTED`, `APPROVED`, `REJECTED` |
| Intermediate | minuscules-tirets | `citizen-input`, `gen-certificate` |
| Error stages | ERROR-description | `ERROR-uxp`, `ERROR-connection` |

---

## 🚀 STAGE TYPE: START

**Rôle:** Point de départ OBLIGATOIRE de tout service.

**Caractéristiques:**
- 1 SEUL par service
- Crée automatiquement `metaData.citizen = [NPI]` au démarrage
- Définit qui peut continuer le workflow

### Structure complète

```json
"start": {
  "type": "start",
  "shortTitle": {
    "en": "STARTED",
    "fr": "DÉBUT"
  },
  "title": {
    "en": "Application started",
    "fr": "Demande démarrée"
  },
  "pathTitle": {
    "en": "Continue application creation",
    "fr": "Poursuivre la demande"
  },
  "permissions": {
    "type": "meta-array",
    "actor": "CITIZEN",
    "metaPathToArray": ["citizen"],
    "public": true
  },
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input"
  }
}
```

### Propriétés détaillées

#### `type` (STRING - OBLIGATOIRE)
**Valeur:** `"start"` (fixe, immuable)

#### `shortTitle` (OBJECT - OBLIGATOIRE)
**Description:** Titre court affiché dans la liste AVANT le premier main stage.

**Format:**
```json
"shortTitle": {
  "en": "STARTED",    // Anglais (1-2 mots max)
  "fr": "DÉBUT"       // Français (1-2 mots max)
}
```

**Recommandations:**
- ✅ Court (1-2 mots)
- ✅ MAJUSCULES pour cohérence avec main stages
- ❌ Éviter phrases longues

#### `title` (OBJECT - OBLIGATOIRE)
**Description:** Description complète affichée dans la timeline.

```json
"title": {
  "en": "Application has been started",
  "fr": "La demande a été démarrée"
}
```

#### `pathTitle` (OBJECT - OBLIGATOIRE)
**Description:** Titre du bouton pour continuer.

```json
"pathTitle": {
  "en": "Continue application creation",
  "fr": "Poursuivre la création de la demande"
}
```

**Affiché comme:** Bouton "Continue application creation" dans l'interface.

#### `permissions` (OBJECT - OBLIGATOIRE)

Voir section dédiée **PERMISSIONS** ci-dessous.

#### `transitions` (OBJECT - OBLIGATOIRE)

Pour `start`, toujours type `"single"`:

```json
"transitions": {
  "type": "single",
  "nextStage": "citizen-input"    // ID du stage suivant
}
```

**Pas d'exception mappable sur `start`.**

---

## 🎯 STAGE TYPE: MAIN

**Rôle:** Représente un état majeur du workflow (REQUESTED, APPROVED, REJECTED...).

**Caractéristiques:**
- Affiché dans la liste des demandes (colonne Status)
- Apparaît dans la timeline de la demande
- Peut avoir plusieurs chemins sortants (transitions array)
- Au moins 1 DOIT avoir flag `"final"` pour terminer le workflow

### Structure complète

```json
"REQUESTED": {
  "type": "main",
  "shortTitle": {
    "en": "REQUESTED",
    "fr": "DEMANDÉE"
  },
  "title": {
    "en": "Certificate has been requested",
    "fr": "Le certificat a été demandé"
  },
  "flags": [],
  "transitions": [
    {
      "id": "review",
      "title": {
        "en": "Review the application",
        "fr": "Examiner la demande"
      },
      "nextStage": "official-review",
      "resultMainStages": ["APPROVED", "REJECTED"],
      "permissions": {
        "type": "hardcoded-array",
        "actor": "OFFICIAL",
        "array": ["ReviewGroup", "ManagerGroup"]
      }
    }
  ]
}
```

### Propriétés détaillées

#### `type` (STRING - OBLIGATOIRE)
**Valeur:** `"main"`

#### `shortTitle` (OBJECT - OBLIGATOIRE)
**Description:** Titre court affiché dans la colonne Status.

**Format:**
```json
"shortTitle": {
  "en": "REQUESTED",
  "fr": "DEMANDÉE"
}
```

**Recommandations:**
- ✅ 1-2 mots MAXIMUM
- ✅ MAJUSCULES (convention)
- ✅ Clair et descriptif
- ❌ Éviter phrases complètes

**Exemples bons:**
- `"REQUESTED"` / `"DEMANDÉE"`
- `"APPROVED"` / `"APPROUVÉE"`
- `"IN REVIEW"` / `"EN RÉVISION"`

**Exemples mauvais:**
- `"The application has been requested"` ❌ (trop long)
- `"requested"` ❌ (minuscules)

#### `title` (OBJECT - OBLIGATOIRE)
**Description:** Description complète dans la timeline.

```json
"title": {
  "en": "The certificate has been requested and is awaiting review",
  "fr": "Le certificat a été demandé et est en attente de révision"
}
```

#### `flags` (ARRAY - OPTIONNEL)

**Valeurs possibles:**
- `"final"` : Marque la demande comme terminée (plus de transitions possibles)
- `"accept"` : Affichage vert (succès)
- `"reject"` : Affichage rouge (échec)

**Combinaisons:**

| Flags | Signification | Affichage |
|-------|---------------|-----------|
| `[]` | État intermédiaire | Neutre |
| `["final"]` | Terminé (neutre) | Gris |
| `["final", "accept"]` | Terminé avec succès | ✅ Vert |
| `["final", "reject"]` | Terminé avec échec | ❌ Rouge |

**Exemples:**

```json
// État intermédiaire (pas final)
"REQUESTED": {
  "type": "main",
  "flags": [],
  "transitions": [...]    // A des transitions
}

// Succès final
"APPROVED": {
  "type": "main",
  "flags": ["final", "accept"],
  "transitions": []       // Aucune transition
}

// Échec final
"REJECTED": {
  "type": "main",
  "flags": ["final", "reject"],
  "transitions": []
}

// Terminé neutre
"CANCELLED": {
  "type": "main",
  "flags": ["final"],
  "transitions": []
}
```

**⚠️ RÈGLE CRITIQUE:**
Au moins 1 main stage DOIT avoir `"final"` dans flags, sinon:
- ❌ Demande ne peut jamais se terminer
- ❌ Reste "en cours" indéfiniment
- ❌ Erreur workflow

#### `transitions` (ARRAY - OBLIGATOIRE)

**Pour main stages:** Array de chemins (paths).

Chaque chemin = objet avec:

```json
{
  "id": "review",                          // ID unique du chemin
  "title": {                               // Titre du bouton/action
    "en": "Review the application",
    "fr": "Examiner la demande"
  },
  "nextStage": "official-review",          // Premier stage du chemin
  "resultMainStages": ["APPROVED", "REJECTED"],  // Main stages de sortie possibles
  "permissions": { ... }                   // Qui peut démarrer ce chemin
}
```

**Propriétés du chemin:**

##### `id` (STRING - OBLIGATOIRE)
**Description:** Identifiant unique du chemin.

**Règles:**
- ✅ Unique dans TOUT le serviceconf.json
- ✅ kebab-case recommandé
- ❌ Pas de duplication entre différents main stages

**Exemples valides:**
```json
"id": "review"
"id": "citizen-edit"
"id": "cancel-request"
"id": "submit-changes"
```

**Exemples invalides:**
```json
"id": "review"     // Dans REQUESTED
"id": "review"     // Dans SUBMITTED ❌ Duplication!
```

##### `title` (OBJECT - OBLIGATOIRE)
**Description:** Titre du bouton/action affiché à l'utilisateur.

```json
"title": {
  "en": "Review the application",
  "fr": "Examiner la demande"
}
```

**Recommandations:**
- ✅ Verbe d'action (Review, Submit, Cancel, Approve...)
- ✅ Clair sur ce qui va se passer
- ❌ Éviter noms génériques ("Continue", "Next"...)

##### `nextStage` (STRING - OBLIGATOIRE)
**Description:** ID du premier intermediate stage de ce chemin.

```json
"nextStage": "official-review"    // ID d'un stage UI, UXP, etc.
```

**⚠️ DOIT pointer vers:**
- Un stage intermediate (ui, uxp, email, etc.) OU
- Un autre main stage (dans cas rares)

**Ne PEUT PAS pointer vers:**
- ❌ `"start"` (interdit)
- ❌ Stage inexistant (erreur)

##### `resultMainStages` (ARRAY - OBLIGATOIRE)
**Description:** Liste des main stages finaux possibles de ce chemin.

**Rôle:** Informatif (aide à comprendre le workflow, pas exécuté).

```json
"resultMainStages": ["APPROVED", "REJECTED", "CHANGES-REQUESTED"]
```

**Exemple de workflow complet:**

```json
"REQUESTED": {
  "transitions": [
    {
      "id": "review",
      "nextStage": "official-review",
      "resultMainStages": ["APPROVED", "REJECTED"],  // Possibilités
      ...
    }
  ]
}

// Ensuite:
"official-review": {
  "type": "ui",
  "transitions": {
    "type": "map-by-meta",
    "map": {
      "approve": "gen-certificate",  // → APPROVED
      "reject": "REJECTED"           // → REJECTED
    }
  }
}
```

##### `permissions` (OBJECT - OBLIGATOIRE)

Voir section dédiée **PERMISSIONS** ci-dessous.

### Exemples de main stages

#### Main stage avec 1 chemin
```json
"REQUESTED": {
  "type": "main",
  "shortTitle": {"en": "REQUESTED", "fr": "DEMANDÉE"},
  "title": {"en": "Application requested", "fr": "Demande soumise"},
  "transitions": [
    {
      "id": "review",
      "title": {"en": "Review", "fr": "Examiner"},
      "nextStage": "official-review",
      "resultMainStages": ["APPROVED", "REJECTED"],
      "permissions": {
        "type": "hardcoded-array",
        "actor": "OFFICIAL",
        "array": ["ReviewGroup"]
      }
    }
  ]
}
```

#### Main stage avec 2 chemins
```json
"SUBMITTED": {
  "type": "main",
  "shortTitle": {"en": "SUBMITTED", "fr": "SOUMISE"},
  "title": {"en": "Application submitted", "fr": "Demande soumise"},
  "transitions": [
    // Chemin 1: Citoyen modifie
    {
      "id": "citizen-edit",
      "title": {"en": "Edit application", "fr": "Modifier"},
      "nextStage": "citizen-edit-form",
      "resultMainStages": ["SUBMITTED"],
      "permissions": {
        "type": "meta-array",
        "actor": "CITIZEN",
        "metaPathToArray": ["citizen"]
      }
    },
    // Chemin 2: Officiel révise
    {
      "id": "official-review-path",
      "title": {"en": "Review", "fr": "Examiner"},
      "nextStage": "official-review",
      "resultMainStages": ["APPROVED", "REJECTED"],
      "permissions": {
        "type": "hardcoded-array",
        "actor": "OFFICIAL",
        "array": ["ReviewGroup"]
      }
    }
  ]
}
```

#### Main stage final (aucune transition)
```json
"APPROVED": {
  "type": "main",
  "shortTitle": {"en": "APPROVED", "fr": "APPROUVÉE"},
  "title": {"en": "Application approved", "fr": "Demande approuvée"},
  "flags": ["final", "accept"],
  "transitions": []    // Vide = aucune action possible
}
```

---

## 🔒 PERMISSIONS

**Rôle:** Contrôle qui peut accéder/continuer un stage ou chemin.

**3 types disponibles:**
1. `meta-array` : Tableau de NPIs dans metaData
2. `hardcoded-array` : Liste fixe de groupes/NPIs
3. `public` : Accessible à tous

### Type 1: meta-array

**Usage:** Permissions basées sur données dynamiques (citoyens propriétaires).

**Structure complète:**
```json
"permissions": {
  "type": "meta-array",
  "actor": "CITIZEN",
  "metaPathToArray": ["citizen"],
  "public": true
}
```

**Propriétés:**

#### `type`
**Valeur:** `"meta-array"`

#### `actor`
**Valeurs possibles:**
- `"CITIZEN"` : Citoyens (utilisateurs portail)
- `"OFFICIAL"` : Agents/officiels (back-office)

#### `metaPathToArray`
**Description:** Chemin vers tableau de NPIs dans `metaData`.

**Format:** Array de strings (notation chemin)

**Exemples:**
```json
"metaPathToArray": ["citizen"]              // → metaData.citizen
"metaPathToArray": ["owners", "list"]       // → metaData.owners.list
"metaPathToArray": ["authorized", "npis"]   // → metaData.authorized.npis
```

**Fonctionnement:**
1. Lit le tableau au chemin spécifié
2. Vérifie si le NPI de l'utilisateur actuel est dans le tableau
3. Si oui: accès autorisé
4. Si non: accès refusé

**Exemple complet:**

```json
// Configuration
"permissions": {
  "type": "meta-array",
  "actor": "CITIZEN",
  "metaPathToArray": ["citizen"]
}

// Au runtime
metaData.citizen = ["NPI123456789"]

// Utilisateur avec NPI123456789: ✅ Autorisé
// Autres utilisateurs: ❌ Refusé
```

#### `public`
**Valeurs:**
- `true` : Accessible sans authentification
- `false` : Authentification requise

**Utilisation:**
```json
// Service public (citoyen lambda peut démarrer)
"permissions": {
  "type": "meta-array",
  "actor": "CITIZEN",
  "metaPathToArray": ["citizen"],
  "public": true
}

// Service privé (login requis)
"permissions": {
  "type": "meta-array",
  "actor": "CITIZEN",
  "metaPathToArray": ["citizen"],
  "public": false
}
```

---

### Type 2: hardcoded-array

**Usage:** Permissions fixes (groupes d'agents spécifiques).

**Structure complète:**
```json
"permissions": {
  "type": "hardcoded-array",
  "actor": "OFFICIAL",
  "array": ["ReviewGroup", "ManagerGroup", "AdminGroup"]
}
```

**Propriétés:**

#### `type`
**Valeur:** `"hardcoded-array"`

#### `actor`
**Valeurs:**
- `"OFFICIAL"` : Agents (utilisation courante)
- `"CITIZEN"` : Citoyens (rare)

#### `array`
**Description:** Liste fixe de groupes ou NPIs autorisés.

**Format:** Array de strings

**Exemples:**
```json
// Groupes d'agents
"array": ["ReviewGroup", "ManagerGroup"]

// NPIs spécifiques
"array": ["NPI123456789", "NPI987654321"]

// Mixte
"array": ["AdminGroup", "NPI111222333"]
```

**⚠️ Noms de groupes:**
- Définis dans la configuration du portail
- Gérés par les administrateurs
- Sensibles à la casse

**Exemple d'utilisation:**

```json
// Main stage: seuls reviewers et managers peuvent examiner
"REQUESTED": {
  "transitions": [
    {
      "id": "review",
      "permissions": {
        "type": "hardcoded-array",
        "actor": "OFFICIAL",
        "array": ["ReviewGroup", "ManagerGroup"]
      }
    }
  ]
}
```

---

### Type 3: public

**Usage:** Accessible à tous sans restriction (RARE).

**Structure:**
```json
"permissions": {
  "type": "public"
}
```

**⚠️ ATTENTION:**
- Vraiment TOUT LE MONDE peut accéder
- Pas de vérification NPI
- Pas de vérification groupe
- **Utiliser avec EXTRÊME précaution!**

**Cas d'usage valide:**
```json
// Stage de consultation publique (rare)
"view-public-info": {
  "type": "ui",
  "permissions": {
    "type": "public"
  }
}
```

**⚠️ NE PAS utiliser pour:**
- Start stage (utiliser meta-array)
- Stages de modification de données
- Stages sensibles

---

## ⚙️ TRANSITIONS

**Rôle:** Définit le prochain stage après le stage actuel.

**2 types:**
1. `single` : Destination unique
2. `map-by-meta` : Branchement conditionnel

### Type 1: single

**Usage:** Toujours la même destination (cas le plus courant).

**Structure complète:**
```json
"transitions": {
  "type": "single",
  "nextStage": "next-stage-id",
  "onStageException": {
    "EXCEPTION_TYPE": "error-stage-id"
  }
}
```

**Propriétés:**

#### `type`
**Valeur:** `"single"`

#### `nextStage`
**Description:** ID du prochain stage.

**Format:** String (ID existant dans `stages`)

**Exemples:**
```json
"nextStage": "citizen-input"
"nextStage": "REQUESTED"
"nextStage": "gen-certificate"
```

**⚠️ RÈGLES:**
- DOIT pointer vers stage existant
- NE PEUT PAS pointer vers `"start"`
- Sensible à la casse

#### `onStageException` (OPTIONNEL)

**Description:** Mapping des exceptions vers stages d'erreur.

**Format:** Object `{ "EXCEPTION_TYPE": "error-stage-id" }`

**Exemple:**
```json
"transitions": {
  "type": "single",
  "nextStage": "REQUESTED",
  "onStageException": {
    "REQUIRED_FIELDS_MISSING": "citizen-input",
    "FIELDS_NOT_ALLOWED": "citizen-input"
  }
}
```

**Exceptions par type de stage:**

Voir sections dédiées à chaque stage type pour liste complète.

**⚠️ Si exception non mappée:**
- Erreur générique affichée
- Utilisateur bloqué
- **Recommandation:** TOUJOURS mapper les exceptions critiques!

---

### Type 2: map-by-meta

**Usage:** Branchement selon valeur dans metaData.

**Structure complète:**
```json
"transitions": {
  "type": "map-by-meta",
  "metaPathToKey": ["official", "choice"],
  "map": {
    "approve": "gen-certificate",
    "reject": "REJECTED",
    "request-changes": "alert-changes"
  },
  "onStageException": {
    "REQUIRED_FIELDS_MISSING": "official-review"
  }
}
```

**Propriétés:**

#### `type`
**Valeur:** `"map-by-meta"`

#### `metaPathToKey`
**Description:** Chemin vers la valeur de branchement dans metaData.

**Format:** Array de strings

**Exemples:**
```json
"metaPathToKey": ["official", "choice"]       // → metaData.official.choice
"metaPathToKey": ["userType"]                 // → metaData.userType
"metaPathToKey": ["category", "type"]         // → metaData.category.type
```

#### `map`
**Description:** Mapping valeur → destination.

**Format:** Object `{ "valeur": "stage-id" }`

**Exemple:**
```json
"map": {
  "approve": "gen-certificate",
  "reject": "REJECTED",
  "request-changes": "alert-changes-requested"
}
```

**Fonctionnement:**
1. Lit `metaData.official.choice`
2. Si valeur = `"approve"` → va vers `gen-certificate`
3. Si valeur = `"reject"` → va vers `REJECTED`
4. Si valeur = `"request-changes"` → va vers `alert-changes-requested`
5. Si valeur autre ou absente → ERREUR!

**⚠️ RÈGLES CRITIQUES:**
1. La valeur dans metaData DOIT correspondre à une clé du `map`
2. Si valeur non trouvée: erreur et blocage
3. **Toujours définir TOUTES les valeurs possibles**

**Exemple workflow complet:**

```json
// Stage UI: agent choisit
"official-review": {
  "type": "ui",
  "uiConfiguration": "official-review.json",
  "transitions": {
    "type": "map-by-meta",
    "metaPathToKey": ["official", "choice"],
    "map": {
      "approve": "gen-certificate",
      "reject": "REJECTED"
    }
  }
}

// Fichier ui/official-review.json:
{
  "form": [
    {
      "id": "decision",
      "type": "STATIC_RADIO_LIST",
      "path": "metaData.official.choice",  // ← Définit la valeur
      "options": [
        {"value": "approve", "label": {"en": "Approve"}},
        {"value": "reject", "label": {"en": "Reject"}}
      ]
    }
  ]
}
```

**⚠️ Pièges courants:**

```json
// ❌ ERREUR: Valeur manquante
"map": {
  "approve": "APPROVED",
  "reject": "REJECTED"
  // Mais UI permet aussi "pending" → ERREUR si choisi!
}

// ✅ CORRECT: Toutes les valeurs
"map": {
  "approve": "APPROVED",
  "reject": "REJECTED",
  "pending": "PENDING"
}
```

#### `onStageException` (OPTIONNEL)

Identique à `single` type.

---

## 🎨 STAGE TYPE: UI

**Rôle:** Affiche un formulaire de saisie utilisateur.

**Voir fichier dédié:** [03-UI-CONFIGURATION.md](03-UI-CONFIGURATION.md)

**Structure minimale:**
```json
"citizen-input": {
  "type": "ui",
  "uiConfiguration": "citizen-input.json",
  "transitions": {
    "type": "single",
    "nextStage": "REQUESTED",
    "onStageException": {
      "REQUIRED_FIELDS_MISSING": "citizen-input",
      "FIELDS_NOT_ALLOWED": "citizen-input"
    }
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"ui"`

### `uiConfiguration`
**Description:** Nom du fichier de configuration dans `ui/`.

**Format:** String (nom de fichier)

**Exemples:**
```json
"uiConfiguration": "citizen-input.json"
"uiConfiguration": "official-review.json"
"uiConfiguration": "company-form.json"
```

**⚠️ RÈGLES:**
- Fichier DOIT exister dans `ui/`
- Extension `.json` obligatoire
- Sensible à la casse

### `transitions`

Type `single` ou `map-by-meta`.

**Exceptions mappables:**
- `REQUIRED_FIELDS_MISSING` : Champs requis manquants
- `FIELDS_NOT_ALLOWED` : Champs non autorisés envoyés

**Recommandation forte:**
```json
"onStageException": {
  "REQUIRED_FIELDS_MISSING": "citizen-input",  // Retour au formulaire
  "FIELDS_NOT_ALLOWED": "citizen-input"        // Retour au formulaire
}
```

---

## 💳 STAGE TYPE: PAY

**Rôle:** Requiert un paiement via KKiaPay/FedaPay.

**Structure complète:**
```json
"payment-stage": {
  "type": "pay",
  "amount": "5000",
  "publicKey": "<PUBLIC_KEY>",
  "privateKey": "<PRIVATE_KEY>",
  "secret": "<SECRET>",
  "sandbox": true,
  "transitions": {
    "type": "single",
    "nextStage": "gen-receipt"
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"pay"`

### `amount` OU `dynamicAmount` (1 OBLIGATOIRE)

**⚠️ MUTUELLEMENT EXCLUSIF: Exactement 1 des 2!**

#### Option 1: Montant fixe
```json
"amount": "5000"    // String, montant en FCFA
```

**Exemples:**
```json
"amount": "5000"    // 5000 FCFA
"amount": "10000"   // 10000 FCFA
"amount": "500"     // 500 FCFA
```

#### Option 2: Montant dynamique
```json
"dynamicAmount": "data.calculated_fee"    // Chemin vers montant
```

**Exemples:**
```json
"dynamicAmount": "data.fee"
"dynamicAmount": "data.payment.amount"
"dynamicAmount": "metaData.calculated_total"
```

**⚠️ RÈGLES:**
- La valeur au chemin DOIT être un nombre ou string numérique
- Si absent ou invalide: erreur paiement

**Exemple avec calcul préalable:**
```json
// Stage avant: calcule le montant
"calculate-fee": {
  "type": "hardcoded-data",
  "writeData": {
    "data": {
      "calculated_fee": "7500"  // Calculé dynamiquement
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "payment"
  }
},

// Stage paiement
"payment": {
  "type": "pay",
  "dynamicAmount": "data.calculated_fee",  // Utilise la valeur calculée
  ...
}
```

### `publicKey`, `privateKey`, `secret` (STRINGS - OBLIGATOIRES)

**Description:** Clés d'API KKiaPay.

**Format:** Strings fournis par KKiaPay.

```json
"publicKey": "pk_test_xxxxxxxxxxxxxxxx",
"privateKey": "sk_test_xxxxxxxxxxxxxxxx",
"secret": "tsec_test_xxxxxxxxxxxxxxxx"
```

**⚠️ SÉCURITÉ:**
- ❌ NE JAMAIS commiter les vraies clés dans Git
- ✅ Utiliser variables d'environnement en production
- ✅ Clés de test (`pk_test_`, `sk_test_`) OK pour dev

### `sandbox` (BOOLEAN - OBLIGATOIRE)

**Valeurs:**
- `true` : Mode test (aucun vrai paiement)
- `false` : Mode production (vrais paiements)

```json
"sandbox": true     // Dev/Test
"sandbox": false    // Production
```

**⚠️ ATTENTION:**
- TOUJOURS `true` en développement
- Passer à `false` SEULEMENT en production
- En mode test: transactions simulées, pas de débit réel

### `transitions`

**Type:** Toujours `single`

**⚠️ PAS D'EXCEPTION MAPPABLE!**

```json
"transitions": {
  "type": "single",
  "nextStage": "gen-receipt"
  // Aucun onStageException possible
}
```

**Fonctionnement:**
- Si paiement réussit: continue vers `nextStage`
- Si paiement échoue: utilisateur BLOQUÉ jusqu'à paiement réussi
- Pas de bypass possible

---

*[Continuera avec les autres stage types...]*

**Fichier trop long, sera scindé. Suite dans la partie 2.**

---

**Dernière mise à jour:** 24 octobre 2025
**Voir aussi:**
- [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md)
- [03-UI-CONFIGURATION.md](03-UI-CONFIGURATION.md)
- [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md)
