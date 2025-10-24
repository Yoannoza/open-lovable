# 🎨 UI CONFIGURATION - FORMULAIRES COMPLETS

**Documentation exhaustive des fichiers de configuration UI (formulaires).**

---

## 🎯 RÔLE DES FICHIERS UI

Les fichiers dans `ui/` définissent les **formulaires de saisie** affichés aux utilisateurs (citoyens ou agents).

**Ils contrôlent:**
- ✅ Quels champs afficher
- ✅ Types de champs (texte, date, fichier, radio...)
- ✅ Validations (requis, format, longueur...)
- ✅ Conditions d'affichage (si X alors afficher Y)
- ✅ Sections répétables (tableaux)
- ✅ Où sauvegarder les données (data vs metaData)

---

## 📁 EMPLACEMENT

```
PSxxxxx/
└── x.y/
    └── ui/
        ├── citizen-input.json       ← Formulaire citoyen
        ├── official-review.json     ← Formulaire agent
        └── company-form.json        ← Autre formulaire
```

**Règles strictes:**
- ✅ Extension `.json` OBLIGATOIRE
- ✅ Encodage UTF-8
- ✅ JSON valide (pas de commentaires)
- ✅ Nom libre (référencé dans serviceconf.json)

---

## 🏗️ STRUCTURE RACINE

### Exemple minimal

```json
{
  "form": [
    {
      "id": "first_name",
      "type": "TEXT",
      "label": {
        "en": "First name",
        "fr": "Prénom"
      },
      "path": "data.person.first_name"
    }
  ]
}
```

### Exemple complet avec toutes options

```json
{
  "form": [
    {
      "id": "first_name",
      "type": "TEXT",
      "label": {
        "en": "First name",
        "fr": "Prénom"
      },
      "helpText": {
        "en": "Enter your legal first name",
        "fr": "Entrez votre prénom légal"
      },
      "placeholder": {
        "en": "John",
        "fr": "Jean"
      },
      "path": "data.person.first_name",
      "defaultValue": "",
      "required": true,
      "requiredMessage": {
        "en": "First name is required",
        "fr": "Le prénom est requis"
      },
      "validations": [
        {
          "type": "MIN_LENGTH",
          "value": 2,
          "message": {
            "en": "Minimum 2 characters",
            "fr": "Minimum 2 caractères"
          }
        }
      ],
      "displayConditions": []
    }
  ]
}
```

---

## 📋 PROPRIÉTÉ `form` (ARRAY - OBLIGATOIRE)

**Description:** Liste de tous les champs du formulaire.

**Format:** Array d'objets (champs)

**Ordre:** Les champs sont affichés dans l'ordre du tableau.

```json
{
  "form": [
    { ... },  // Champ 1 (affiché en premier)
    { ... },  // Champ 2
    { ... }   // Champ 3
  ]
}
```

---

## 🔧 PROPRIÉTÉS COMMUNES À TOUS LES CHAMPS

Chaque champ (objet dans `form`) a des propriétés communes:

### 1. `id` (STRING - OBLIGATOIRE)

**Description:** Identifiant unique du champ dans le formulaire.

**Format:** String (kebab-case ou snake_case recommandé)

**Exemples valides:**
```json
"id": "first_name"
"id": "company-name"
"id": "birth_date"
"id": "file_upload_1"
```

**Exemples invalides:**
```json
"id": "first name"     ❌ (espace)
"id": "1_name"         ❌ (commence par chiffre)
"id": ""               ❌ (vide)
```

**⚠️ RÈGLES CRITIQUES:**
1. DOIT être unique dans le fichier UI
2. Sensible à la casse (`firstName` ≠ `firstname`)
3. Utilisé pour références dans `displayConditions`

---

### 2. `type` (STRING - OBLIGATOIRE)

**Description:** Type de champ à afficher.

**13 types disponibles:**

| Type | Description | Exemple |
|------|-------------|---------|
| `TEXT` | Champ texte court | Nom, prénom |
| `TEXT_AREA` | Texte multiligne | Adresse, commentaire |
| `TEXT_EDITOR` | Éditeur riche (Markdown) | Description détaillée |
| `NUMBER` | Champ numérique | Âge, montant |
| `DATE` | Sélecteur de date | Date de naissance |
| `BOOLEAN` | Case à cocher | J'accepte les conditions |
| `FILE_UPLOAD` | Upload fichier | Pièce d'identité |
| `RADIO_LIST` | Boutons radio (dynamique) | Liste de villes |
| `STATIC_RADIO_LIST` | Boutons radio (fixe) | Oui/Non, M/F |
| `MARKDOWN_DESCRIPTION` | Texte informatif (non éditable) | Instructions |
| `REPEATABLE` | Section répétable | Liste d'employés |
| `SUBMIT` | Bouton de soumission | Soumettre |

**Voir sections dédiées ci-dessous pour détails par type.**

---

### 3. `label` (OBJECT - OBLIGATOIRE sauf SUBMIT)

**Description:** Étiquette affichée au-dessus du champ.

**Format:** Object avec langues

```json
"label": {
  "en": "First name",
  "fr": "Prénom"
}
```

**Recommandations:**
- ✅ Court et descriptif
- ✅ Majuscule au début
- ❌ Éviter ponctuation finale

**Exemples bons:**
```json
"label": {"en": "First name", "fr": "Prénom"}
"label": {"en": "Company name", "fr": "Nom de l'entreprise"}
"label": {"en": "Upload ID card", "fr": "Téléverser la carte d'identité"}
```

**Exemples mauvais:**
```json
"label": {"en": "first name", "fr": "prénom"}           ❌ (minuscules)
"label": {"en": "Enter your first name here:", "fr": ...}  ❌ (trop long)
```

---

### 4. `path` (STRING - OBLIGATOIRE sauf MARKDOWN_DESCRIPTION et SUBMIT)

**Description:** Chemin où sauvegarder la valeur dans `data` ou `metaData`.

**Format:** String (dot-notation)

**Exemples:**
```json
// Dans data
"path": "data.person.first_name"
"path": "data.company.name"
"path": "data.files.id_card"

// Dans metaData
"path": "metaData.category"
"path": "metaData.official.choice"
"path": "metaData.calculated_total"
```

**⚠️ RÈGLES CRITIQUES:**

#### Règle 1: DOIT commencer par `data.` OU `metaData.`
```json
✅ "path": "data.person.name"
✅ "path": "metaData.category"
❌ "path": "person.name"          // Manque "data."
❌ "path": "files.certificate"    // Manque "data."
```

#### Règle 2: FILE_UPLOAD DOIT pointer vers `data.`
```json
✅ "type": "FILE_UPLOAD", "path": "data.files.id_card"
❌ "type": "FILE_UPLOAD", "path": "metaData.file"    // Erreur!
```

#### Règle 3: Chemins uniques (sauf exception)
```json
// ❌ ERREUR: 2 champs → même path
{
  "id": "first_name_en",
  "path": "data.name"
},
{
  "id": "first_name_fr",
  "path": "data.name"    // ❌ Écrasera la première valeur!
}

// ✅ CORRECT: Chemins différents
{
  "id": "first_name",
  "path": "data.person.first_name"
},
{
  "id": "last_name",
  "path": "data.person.last_name"
}
```

#### Règle 4: Shallow merge warning
```json
// État initial
data = {
  "person": {
    "name": "John",
    "age": 30
  }
}

// Champ avec path
"path": "data.person"    // ❌ DANGER!

// Si utilisateur envoie seulement:
{
  "person": {
    "city": "Cotonou"
  }
}

// Résultat (SHALLOW MERGE):
data = {
  "person": {
    "city": "Cotonou"    // ❌ name et age perdus!
  }
}

// ✅ SOLUTION: Toujours paths profonds
"path": "data.person.name"    // ✅
"path": "data.person.age"     // ✅
"path": "data.person.city"    // ✅
```

---

### 5. `helpText` (OBJECT - OPTIONNEL)

**Description:** Texte d'aide affiché sous le champ.

**Format:** Object avec langues

```json
"helpText": {
  "en": "Enter your legal first name as it appears on your ID",
  "fr": "Entrez votre prénom légal tel qu'il apparaît sur votre pièce d'identité"
}
```

**Affichage:** Texte gris/petit sous le champ.

**Recommandations:**
- ✅ Clarifications utiles
- ✅ Exemples de format
- ✅ Instructions spécifiques
- ❌ Répétition du label

**Exemples utiles:**
```json
"helpText": {
  "en": "Format: DD/MM/YYYY",
  "fr": "Format: JJ/MM/AAAA"
}

"helpText": {
  "en": "Accepted formats: PDF, PNG, JPEG (max 5MB)",
  "fr": "Formats acceptés: PDF, PNG, JPEG (max 5Mo)"
}
```

---

### 6. `placeholder` (OBJECT - OPTIONNEL)

**Description:** Texte indicatif affiché DANS le champ vide.

**Format:** Object avec langues

**Applicables uniquement à:**
- `TEXT`
- `TEXT_AREA`
- `NUMBER`

```json
"placeholder": {
  "en": "John",
  "fr": "Jean"
}
```

**Affichage:** Texte gris clair dans le champ, disparaît quand l'utilisateur tape.

**Recommandations:**
- ✅ Exemples courts
- ✅ Formats attendus
- ❌ Instructions longues (utiliser helpText)

**Exemples:**
```json
// Pour champ texte
"placeholder": {
  "en": "Acme Corporation",
  "fr": "Société Acme"
}

// Pour champ nombre
"placeholder": {
  "en": "25",
  "fr": "25"
}

// Pour email
"placeholder": {
  "en": "john.doe@example.com",
  "fr": "jean.dupont@exemple.fr"
}
```

---

### 7. `defaultValue` (STRING/NUMBER/BOOLEAN - OPTIONNEL)

**Description:** Valeur pré-remplie par défaut.

**Format:** Dépend du type de champ

**Exemples:**

#### Pour TEXT
```json
"type": "TEXT",
"defaultValue": "Cotonou"
```

#### Pour NUMBER
```json
"type": "NUMBER",
"defaultValue": 0
```

#### Pour BOOLEAN
```json
"type": "BOOLEAN",
"defaultValue": false
```

#### Pour STATIC_RADIO_LIST
```json
"type": "STATIC_RADIO_LIST",
"defaultValue": "option1"    // DOIT correspondre à une option.value
```

**⚠️ RÈGLES:**
1. Type DOIT correspondre au type de champ
2. Pour RADIO_LIST: DOIT être une valeur valide des options
3. Pour DATE: format ISO `YYYY-MM-DD`

**⚠️ NE PAS utiliser pour:**
- Données dynamiques (utiliser stages `hardcoded-data` ou `person-data`)
- Valeurs calculées

---

### 8. `required` (BOOLEAN - OPTIONNEL)

**Description:** Indique si le champ est obligatoire.

**Valeurs:**
- `true` : Champ obligatoire (validation côté serveur)
- `false` : Champ optionnel (défaut si omis)

```json
"required": true
```

**⚠️ COMPORTEMENT:**
- Si `true` et champ vide → Exception `REQUIRED_FIELDS_MISSING`
- Doit être mappée dans serviceconf.json:

```json
// Dans serviceconf.json
"transitions": {
  "type": "single",
  "nextStage": "REQUESTED",
  "onStageException": {
    "REQUIRED_FIELDS_MISSING": "citizen-input"  // Retour au formulaire
  }
}
```

**Affichage:** Astérisque rouge `*` à côté du label.

---

### 9. `requiredMessage` (OBJECT - OPTIONNEL)

**Description:** Message d'erreur personnalisé si champ requis manquant.

**Format:** Object avec langues

```json
"required": true,
"requiredMessage": {
  "en": "First name is required",
  "fr": "Le prénom est requis"
}
```

**⚠️ Si omis:**
- Message générique: "This field is required" / "Ce champ est requis"

**Recommandations:**
- ✅ Messages clairs et spécifiques
- ✅ Répéter le nom du champ pour contexte

---

### 10. `validations` (ARRAY - OPTIONNEL)

**Description:** Liste de validations supplémentaires.

**Format:** Array d'objets de validation

**Voir section dédiée VALIDATIONS ci-dessous.**

---

### 11. `displayConditions` (ARRAY - OPTIONNEL)

**Description:** Conditions pour afficher/cacher le champ.

**Format:** Array d'objets de condition

**Voir section dédiée CONDITIONS D'AFFICHAGE ci-dessous.**

---

## 🎨 TYPES DE CHAMPS DÉTAILLÉS

### Type: TEXT

**Usage:** Champ texte court (une ligne).

**Cas d'usage:**
- Nom, prénom
- Email
- Numéro de téléphone
- Adresse courte

**Propriétés spécifiques:**
- `placeholder` : Texte indicatif
- `defaultValue` : Valeur par défaut (string)

**Validations applicables:**
- `MIN_LENGTH`
- `MAX_LENGTH`
- `REGEX`
- `EMAIL`
- `PHONE`

**Exemple complet:**
```json
{
  "id": "first_name",
  "type": "TEXT",
  "label": {
    "en": "First name",
    "fr": "Prénom"
  },
  "helpText": {
    "en": "Your legal first name",
    "fr": "Votre prénom légal"
  },
  "placeholder": {
    "en": "John",
    "fr": "Jean"
  },
  "path": "data.person.first_name",
  "defaultValue": "",
  "required": true,
  "requiredMessage": {
    "en": "First name is required",
    "fr": "Le prénom est requis"
  },
  "validations": [
    {
      "type": "MIN_LENGTH",
      "value": 2,
      "message": {
        "en": "Minimum 2 characters",
        "fr": "Minimum 2 caractères"
      }
    },
    {
      "type": "MAX_LENGTH",
      "value": 50,
      "message": {
        "en": "Maximum 50 characters",
        "fr": "Maximum 50 caractères"
      }
    }
  ]
}
```

---

### Type: TEXT_AREA

**Usage:** Texte multiligne (plusieurs lignes).

**Cas d'usage:**
- Adresse complète
- Commentaires
- Description
- Justification

**Propriétés spécifiques:**
- `placeholder` : Texte indicatif
- `defaultValue` : Valeur par défaut (string)
- `rows` : Nombre de lignes visibles (optionnel)

**Validations applicables:**
- `MIN_LENGTH`
- `MAX_LENGTH`

**Exemple complet:**
```json
{
  "id": "address",
  "type": "TEXT_AREA",
  "label": {
    "en": "Full address",
    "fr": "Adresse complète"
  },
  "helpText": {
    "en": "Include street, city, and postal code",
    "fr": "Incluez rue, ville et code postal"
  },
  "placeholder": {
    "en": "123 Main Street\nCotonou\n01 BP 1234",
    "fr": "123 Rue Principale\nCotonou\n01 BP 1234"
  },
  "path": "data.person.address",
  "required": true,
  "validations": [
    {
      "type": "MIN_LENGTH",
      "value": 10,
      "message": {
        "en": "Address is too short",
        "fr": "Adresse trop courte"
      }
    }
  ]
}
```

**Affichage:** Zone de texte extensible avec scrollbar si nécessaire.

---

### Type: TEXT_EDITOR

**Usage:** Éditeur de texte riche (Markdown/HTML).

**Cas d'usage:**
- Descriptions détaillées
- Rapports
- Contenu formaté

**Propriétés spécifiques:**
- `defaultValue` : Contenu Markdown par défaut

**Validations applicables:**
- `MIN_LENGTH`
- `MAX_LENGTH`

**Exemple complet:**
```json
{
  "id": "project_description",
  "type": "TEXT_EDITOR",
  "label": {
    "en": "Project description",
    "fr": "Description du projet"
  },
  "helpText": {
    "en": "Use Markdown for formatting (bold, lists, links...)",
    "fr": "Utilisez Markdown pour la mise en forme (gras, listes, liens...)"
  },
  "path": "data.project.description",
  "required": true,
  "validations": [
    {
      "type": "MIN_LENGTH",
      "value": 100,
      "message": {
        "en": "Description must be at least 100 characters",
        "fr": "La description doit faire au moins 100 caractères"
      }
    }
  ]
}
```

**Fonctionnalités éditeur:**
- Gras, italique, souligné
- Listes (puces, numéros)
- Liens
- Titres (H1-H6)
- Code
- Aperçu temps réel

**Sortie:** Markdown ou HTML (selon config portail).

---

### Type: NUMBER

**Usage:** Champ numérique.

**Cas d'usage:**
- Âge
- Montant
- Quantité
- Nombre d'employés

**Propriétés spécifiques:**
- `placeholder` : Exemple (string)
- `defaultValue` : Valeur par défaut (number)
- `min` : Valeur minimale (number, optionnel)
- `max` : Valeur maximale (number, optionnel)
- `step` : Incrément (number, optionnel, défaut: 1)

**Validations applicables:**
- `MIN_VALUE`
- `MAX_VALUE`
- `INTEGER_ONLY`

**Exemple complet:**
```json
{
  "id": "age",
  "type": "NUMBER",
  "label": {
    "en": "Age",
    "fr": "Âge"
  },
  "placeholder": {
    "en": "25",
    "fr": "25"
  },
  "path": "data.person.age",
  "min": 18,
  "max": 120,
  "step": 1,
  "required": true,
  "validations": [
    {
      "type": "MIN_VALUE",
      "value": 18,
      "message": {
        "en": "You must be at least 18 years old",
        "fr": "Vous devez avoir au moins 18 ans"
      }
    },
    {
      "type": "INTEGER_ONLY",
      "message": {
        "en": "Age must be a whole number",
        "fr": "L'âge doit être un nombre entier"
      }
    }
  ]
}
```

**⚠️ Contrôles:**
- Boutons +/- (si `step` défini)
- Validation HTML5 (`min`, `max`)
- Validation serveur (validations array)

---

### Type: DATE

**Usage:** Sélecteur de date.

**Cas d'usage:**
- Date de naissance
- Date de début/fin
- Échéance

**Propriétés spécifiques:**
- `defaultValue` : Date par défaut (format ISO `YYYY-MM-DD`)
- `min` : Date minimale (ISO, optionnel)
- `max` : Date maximale (ISO, optionnel)

**Validations applicables:**
- `MIN_DATE`
- `MAX_DATE`
- `AFTER_DATE` (après une autre date)
- `BEFORE_DATE` (avant une autre date)

**Exemple complet:**
```json
{
  "id": "birth_date",
  "type": "DATE",
  "label": {
    "en": "Date of birth",
    "fr": "Date de naissance"
  },
  "helpText": {
    "en": "You must be at least 18 years old",
    "fr": "Vous devez avoir au moins 18 ans"
  },
  "path": "data.person.birth_date",
  "max": "2007-01-01",
  "required": true,
  "validations": [
    {
      "type": "MAX_DATE",
      "value": "2007-01-01",
      "message": {
        "en": "You must be at least 18 years old",
        "fr": "Vous devez avoir au moins 18 ans"
      }
    }
  ]
}
```

**Format attendu:**
- **Entrée utilisateur:** DD/MM/YYYY (selon locale)
- **Stockage:** ISO 8601 `YYYY-MM-DD`
- **Affichage:** Selon langue de l'interface

**Dates dynamiques:**
```json
// Aujourd'hui
"max": "${_i18n.current_date}"

// Il y a 18 ans (calculé côté serveur)
"max": "${_date.subtract_years(18)}"
```

---

### Type: BOOLEAN

**Usage:** Case à cocher (checkbox).

**Cas d'usage:**
- Acceptation conditions
- Options oui/non
- Flags

**Propriétés spécifiques:**
- `defaultValue` : `true` ou `false`

**Validations applicables:**
- `MUST_BE_TRUE` (pour conditions obligatoires)

**Exemple complet:**
```json
{
  "id": "accept_terms",
  "type": "BOOLEAN",
  "label": {
    "en": "I accept the terms and conditions",
    "fr": "J'accepte les termes et conditions"
  },
  "path": "data.acceptance.terms",
  "defaultValue": false,
  "required": true,
  "validations": [
    {
      "type": "MUST_BE_TRUE",
      "message": {
        "en": "You must accept the terms to continue",
        "fr": "Vous devez accepter les conditions pour continuer"
      }
    }
  ]
}
```

**Affichage:**
```
☐ I accept the terms and conditions
```

**Valeur stockée:**
- Coché: `true`
- Décoché: `false`

---

### Type: FILE_UPLOAD

**Usage:** Upload de fichier.

**Cas d'usage:**
- Pièce d'identité
- Justificatif
- Photo
- Document PDF

**Propriétés spécifiques:**
- `accept` : Types MIME acceptés (optionnel)
- `maxSize` : Taille max en MB (optionnel)
- `multiple` : Autoriser plusieurs fichiers (boolean, optionnel)

**⚠️ PATH OBLIGATOIRE:** DOIT commencer par `data.` (jamais `metaData.`)

**Validations applicables:**
- `FILE_TYPE`
- `FILE_SIZE`

**Exemple complet:**
```json
{
  "id": "id_card",
  "type": "FILE_UPLOAD",
  "label": {
    "en": "ID card (front and back)",
    "fr": "Carte d'identité (recto-verso)"
  },
  "helpText": {
    "en": "Accepted formats: PDF, PNG, JPEG (max 5MB)",
    "fr": "Formats acceptés: PDF, PNG, JPEG (max 5Mo)"
  },
  "path": "data.files.id_card",
  "accept": ".pdf,.png,.jpg,.jpeg",
  "maxSize": 5,
  "required": true,
  "validations": [
    {
      "type": "FILE_TYPE",
      "value": ["pdf", "png", "jpg", "jpeg"],
      "message": {
        "en": "Only PDF, PNG, and JPEG files are accepted",
        "fr": "Seuls les fichiers PDF, PNG et JPEG sont acceptés"
      }
    },
    {
      "type": "FILE_SIZE",
      "value": 5,
      "message": {
        "en": "File size must not exceed 5MB",
        "fr": "La taille du fichier ne doit pas dépasser 5Mo"
      }
    }
  ]
}
```

**Propriété `accept` (types MIME):**
```json
// PDF uniquement
"accept": ".pdf"

// Images uniquement
"accept": ".png,.jpg,.jpeg"

// Documents
"accept": ".pdf,.doc,.docx"

// Tous
"accept": "*"    // ⚠️ Non recommandé!
```

**Propriété `multiple`:**
```json
// Un seul fichier (défaut)
"multiple": false

// Plusieurs fichiers
"multiple": true
```

**Valeur stockée:**
```json
// Un fichier
"data": {
  "files": {
    "id_card": "file-id-12345"
  }
}

// Plusieurs fichiers (avec multiple:true)
"data": {
  "files": {
    "documents": ["file-id-111", "file-id-222", "file-id-333"]
  }
}
```

---

*[La suite dans la prochaine partie avec les types RADIO_LIST, STATIC_RADIO_LIST, REPEATABLE, SUBMIT, MARKDOWN_DESCRIPTION, validations complètes, et conditions d'affichage...]*

**Dernière mise à jour:** 24 octobre 2025
**Voir aussi:**
- [03-UI-CONFIGURATION-PART2.md](03-UI-CONFIGURATION-PART2.md) (suite)
- [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md)
- [02-SERVICECONF-PART1.md](02-SERVICECONF-PART1.md)
