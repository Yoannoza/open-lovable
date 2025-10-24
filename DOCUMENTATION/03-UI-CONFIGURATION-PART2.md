# 🎨 UI CONFIGURATION - PARTIE 2 - TYPES AVANCÉS & VALIDATIONS

**Suite de la documentation exhaustive des fichiers UI.**

**⚠️ Lire d'abord:** [03-UI-CONFIGURATION-PART1.md](03-UI-CONFIGURATION-PART1.md)

---

## 📻 Type: RADIO_LIST (Dynamique)

**Usage:** Liste de choix (radio buttons) avec options chargées depuis data/metaData.

**Cas d'usage:**
- Liste de villes (depuis API)
- Catégories dynamiques
- Options variables

**Propriétés spécifiques:**
- `optionsPath` : Chemin vers le tableau d'options
- `optionLabelPath` : Chemin vers le label dans chaque option
- `optionValuePath` : Chemin vers la valeur dans chaque option
- `defaultValue` : Valeur par défaut (string)

**Exemple complet:**
```json
{
  "id": "city",
  "type": "RADIO_LIST",
  "label": {
    "en": "Select your city",
    "fr": "Sélectionnez votre ville"
  },
  "path": "data.person.city",
  "optionsPath": "metaData.available_cities",
  "optionLabelPath": "name",
  "optionValuePath": "code",
  "required": true
}
```

**Structure des données attendues:**

```json
// Dans metaData.available_cities (chargé par stage précédent)
metaData: {
  "available_cities": [
    {
      "code": "CTN",
      "name": "Cotonou"
    },
    {
      "code": "PNV",
      "name": "Porto-Novo"
    },
    {
      "code": "PAR",
      "name": "Parakou"
    }
  ]
}
```

**Affichage généré:**
```
Select your city:
◯ Cotonou
◯ Porto-Novo
◯ Parakou
```

**Valeur stockée:**
```json
// Si utilisateur choisit "Cotonou"
"data": {
  "person": {
    "city": "CTN"    // optionValuePath
  }
}
```

**⚠️ RÈGLES:**
1. `optionsPath` DOIT pointer vers un ARRAY
2. Chaque élément du tableau DOIT avoir les propriétés spécifiées
3. Si tableau vide: champ affiché mais aucune option
4. Si chemin invalide: erreur générée

**Exemple avec workflow:**

```json
// Stage 1: Charge les villes (uxp-rest)
"fetch-cities": {
  "type": "uxp-rest",
  "uxpConfiguration": "get-cities.json",
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input"
  }
}

// Fichier uxp/get-cities.json
{
  "method": "GET",
  "url": "https://api.example.com/cities",
  "responseMapping": [
    {
      "source": "/cities",
      "target": "metaData.available_cities"
    }
  ]
}

// Stage 2: Formulaire avec RADIO_LIST
"citizen-input": {
  "type": "ui",
  "uiConfiguration": "citizen-input.json",
  ...
}

// Fichier ui/citizen-input.json
{
  "form": [
    {
      "id": "city",
      "type": "RADIO_LIST",
      "optionsPath": "metaData.available_cities",
      "optionLabelPath": "name",
      "optionValuePath": "code",
      ...
    }
  ]
}
```

---

## 🎯 Type: STATIC_RADIO_LIST (Statique)

**Usage:** Liste de choix fixe (radio buttons) avec options codées en dur.

**Cas d'usage:**
- Oui/Non
- Homme/Femme
- Choix de décision (approuver/rejeter)
- Options fixes connues à l'avance

**Propriétés spécifiques:**
- `options` : Array d'options fixes
- `defaultValue` : Valeur par défaut (string)

**Exemple complet:**
```json
{
  "id": "gender",
  "type": "STATIC_RADIO_LIST",
  "label": {
    "en": "Gender",
    "fr": "Sexe"
  },
  "path": "data.person.gender",
  "options": [
    {
      "value": "M",
      "label": {
        "en": "Male",
        "fr": "Homme"
      }
    },
    {
      "value": "F",
      "label": {
        "en": "Female",
        "fr": "Femme"
      }
    },
    {
      "value": "O",
      "label": {
        "en": "Other",
        "fr": "Autre"
      }
    }
  ],
  "required": true
}
```

**Structure option:**
```json
{
  "value": "M",        // Valeur stockée (STRING)
  "label": {           // Label affiché (OBJECT traduit)
    "en": "Male",
    "fr": "Homme"
  }
}
```

**Affichage:**
```
Gender:
◯ Male
◯ Female
◯ Other
```

**Valeur stockée:**
```json
// Si utilisateur choisit "Male"
"data": {
  "person": {
    "gender": "M"
  }
}
```

**Exemple: Décision officiel**
```json
{
  "id": "decision",
  "type": "STATIC_RADIO_LIST",
  "label": {
    "en": "Your decision",
    "fr": "Votre décision"
  },
  "path": "metaData.official.choice",
  "options": [
    {
      "value": "approve",
      "label": {
        "en": "Approve the application",
        "fr": "Approuver la demande"
      }
    },
    {
      "value": "reject",
      "label": {
        "en": "Reject the application",
        "fr": "Rejeter la demande"
      }
    },
    {
      "value": "request-changes",
      "label": {
        "en": "Request changes",
        "fr": "Demander des modifications"
      }
    }
  ],
  "required": true
}
```

**Utilisation avec map-by-meta:**

```json
// Dans serviceconf.json
"official-review": {
  "type": "ui",
  "uiConfiguration": "official-review.json",
  "transitions": {
    "type": "map-by-meta",
    "metaPathToKey": ["official", "choice"],
    "map": {
      "approve": "gen-certificate",
      "reject": "REJECTED",
      "request-changes": "alert-changes"
    }
  }
}
```

**⚠️ RÈGLES:**
1. `options` DOIT être un array non vide
2. Chaque option DOIT avoir `value` (string) et `label` (object)
3. `value` DOIT être unique dans le tableau
4. `defaultValue` (si présent) DOIT correspondre à un `value` existant

**Exemple Oui/Non simple:**
```json
{
  "id": "has_employees",
  "type": "STATIC_RADIO_LIST",
  "label": {
    "en": "Do you have employees?",
    "fr": "Avez-vous des employés ?"
  },
  "path": "data.company.has_employees",
  "options": [
    {
      "value": "yes",
      "label": {
        "en": "Yes",
        "fr": "Oui"
      }
    },
    {
      "value": "no",
      "label": {
        "en": "No",
        "fr": "Non"
      }
    }
  ],
  "defaultValue": "no",
  "required": true
}
```

---

## 🔁 Type: REPEATABLE (Section répétable)

**Usage:** Section de champs répétable (tableau dynamique).

**Cas d'usage:**
- Liste d'employés
- Produits multiples
- Membres de famille
- Documents multiples

**Propriétés spécifiques:**
- `fields` : Array de champs (comme `form`)
- `minItems` : Nombre minimum d'entrées (optionnel)
- `maxItems` : Nombre maximum d'entrées (optionnel)
- `addButtonLabel` : Label du bouton "Ajouter" (object traduit)
- `removeButtonLabel` : Label du bouton "Supprimer" (object traduit)

**⚠️ PATH SPÉCIAL:** Pointe vers un ARRAY, champs internes utilisent index automatique.

**Exemple complet:**
```json
{
  "id": "employees",
  "type": "REPEATABLE",
  "label": {
    "en": "Employees",
    "fr": "Employés"
  },
  "path": "data.company.employees",
  "minItems": 1,
  "maxItems": 10,
  "addButtonLabel": {
    "en": "Add employee",
    "fr": "Ajouter un employé"
  },
  "removeButtonLabel": {
    "en": "Remove",
    "fr": "Supprimer"
  },
  "fields": [
    {
      "id": "first_name",
      "type": "TEXT",
      "label": {
        "en": "First name",
        "fr": "Prénom"
      },
      "path": "first_name",    // ⚠️ Relatif à l'index
      "required": true
    },
    {
      "id": "last_name",
      "type": "TEXT",
      "label": {
        "en": "Last name",
        "fr": "Nom"
      },
      "path": "last_name",
      "required": true
    },
    {
      "id": "position",
      "type": "TEXT",
      "label": {
        "en": "Position",
        "fr": "Poste"
      },
      "path": "position",
      "required": false
    }
  ]
}
```

**Affichage interface:**
```
Employees:

[Employee 1]
  First name: [John        ]
  Last name:  [DOE         ]
  Position:   [Manager     ]
  [Remove]

[Employee 2]
  First name: [Jane        ]
  Last name:  [SMITH       ]
  Position:   [Developer   ]
  [Remove]

[+ Add employee]
```

**Valeur stockée:**
```json
"data": {
  "company": {
    "employees": [
      {
        "first_name": "John",
        "last_name": "DOE",
        "position": "Manager"
      },
      {
        "first_name": "Jane",
        "last_name": "SMITH",
        "position": "Developer"
      }
    ]
  }
}
```

**⚠️ CHEMINS DANS `fields`:**

**MAUVAIS (absolu):**
```json
"fields": [
  {
    "path": "data.company.employees.0.first_name"    // ❌ Ne marche pas!
  }
]
```

**BON (relatif):**
```json
"fields": [
  {
    "path": "first_name"    // ✅ Relatif à employees[index]
  }
]
```

**Le système transforme automatiquement:**
```
"path": "first_name"  →  data.company.employees.0.first_name
"path": "first_name"  →  data.company.employees.1.first_name
"path": "first_name"  →  data.company.employees.2.first_name
```

**Propriétés `minItems` et `maxItems`:**

```json
// Minimum 1, maximum 5
"minItems": 1,
"maxItems": 5

// Au moins 2 employés requis
"minItems": 2,
"maxItems": 100

// Aucune limite (défaut)
// (minItems: 0, maxItems: illimité)
```

**Affichage boutons:**
- **Add button:** Visible si `items.length < maxItems`
- **Remove button:** Visible si `items.length > minItems`

**Exemple avec FILE_UPLOAD répétable:**
```json
{
  "id": "documents",
  "type": "REPEATABLE",
  "label": {
    "en": "Supporting documents",
    "fr": "Documents justificatifs"
  },
  "path": "data.files.documents",
  "minItems": 1,
  "maxItems": 5,
  "addButtonLabel": {
    "en": "Add document",
    "fr": "Ajouter un document"
  },
  "fields": [
    {
      "id": "doc_type",
      "type": "STATIC_RADIO_LIST",
      "label": {
        "en": "Document type",
        "fr": "Type de document"
      },
      "path": "type",
      "options": [
        {"value": "ID", "label": {"en": "ID Card", "fr": "Carte d'identité"}},
        {"value": "PROOF", "label": {"en": "Proof of address", "fr": "Justificatif de domicile"}}
      ],
      "required": true
    },
    {
      "id": "doc_file",
      "type": "FILE_UPLOAD",
      "label": {
        "en": "File",
        "fr": "Fichier"
      },
      "path": "file",
      "accept": ".pdf,.png,.jpg",
      "required": true
    }
  ]
}
```

**Résultat stocké:**
```json
"data": {
  "files": {
    "documents": [
      {
        "type": "ID",
        "file": "file-id-111"
      },
      {
        "type": "PROOF",
        "file": "file-id-222"
      }
    ]
  }
}
```

**⚠️ VALIDATIONS sur REPEATABLE:**
- `required: true` sur REPEATABLE → Au moins `minItems` entrées
- `required: true` sur champs internes → Validé pour CHAQUE entrée

**Exemple REPEATABLE imbriqué (rare mais possible):**
```json
{
  "id": "companies",
  "type": "REPEATABLE",
  "path": "data.companies",
  "fields": [
    {
      "id": "company_name",
      "type": "TEXT",
      "path": "name"
    },
    {
      "id": "employees",
      "type": "REPEATABLE",
      "path": "employees",    // Relatif à companies[i]
      "fields": [
        {
          "id": "emp_name",
          "type": "TEXT",
          "path": "name"    // Relatif à companies[i].employees[j]
        }
      ]
    }
  ]
}
```

**Résultat:**
```json
"data": {
  "companies": [
    {
      "name": "Company A",
      "employees": [
        {"name": "John"},
        {"name": "Jane"}
      ]
    },
    {
      "name": "Company B",
      "employees": [
        {"name": "Bob"}
      ]
    }
  ]
}
```

---

## 📝 Type: MARKDOWN_DESCRIPTION

**Usage:** Texte informatif non éditable (instructions, avertissements).

**Cas d'usage:**
- Instructions de formulaire
- Avertissements
- Explications
- Informations légales

**Propriétés spécifiques:**
- `content` : Texte Markdown (object traduit)

**⚠️ PAS DE `path`:** Ce champ n'enregistre rien.

**Exemple complet:**
```json
{
  "id": "instructions",
  "type": "MARKDOWN_DESCRIPTION",
  "content": {
    "en": "## Important Information\n\nPlease ensure all information is accurate. **False information** may result in rejection.\n\n- Have your ID card ready\n- Prepare proof of address\n- Check all fields before submitting",
    "fr": "## Informations importantes\n\nAssurez-vous que toutes les informations sont exactes. **Les fausses informations** peuvent entraîner un rejet.\n\n- Ayez votre carte d'identité prête\n- Préparez un justificatif de domicile\n- Vérifiez tous les champs avant de soumettre"
  }
}
```

**Affichage (rendu Markdown):**
```
┌─────────────────────────────────────────┐
│ Important Information                   │
│                                         │
│ Please ensure all information is        │
│ accurate. False information may result  │
│ in rejection.                           │
│                                         │
│ • Have your ID card ready               │
│ • Prepare proof of address              │
│ • Check all fields before submitting    │
└─────────────────────────────────────────┘
```

**Markdown supporté:**
- Titres (`#`, `##`, `###`)
- Gras (`**texte**`)
- Italique (`*texte*`)
- Listes (`-` ou `*`)
- Liens (`[texte](url)`)
- Code (`` `code` ``)

**Exemples d'usage:**

**Instructions début de formulaire:**
```json
{
  "form": [
    {
      "id": "intro",
      "type": "MARKDOWN_DESCRIPTION",
      "content": {
        "en": "### Certificate Request\n\nFill out this form to request your certificate. Processing time is **5-7 business days**.",
        "fr": "### Demande de certificat\n\nRemplissez ce formulaire pour demander votre certificat. Délai de traitement : **5-7 jours ouvrables**."
      }
    },
    // ... autres champs
  ]
}
```

**Avertissement légal:**
```json
{
  "id": "legal_warning",
  "type": "MARKDOWN_DESCRIPTION",
  "content": {
    "en": "⚠️ **Warning**: Providing false information is a criminal offense punishable by law.",
    "fr": "⚠️ **Avertissement** : Fournir de fausses informations est un délit pénal puni par la loi."
  }
}
```

**Section separator:**
```json
{
  "id": "section_personal",
  "type": "MARKDOWN_DESCRIPTION",
  "content": {
    "en": "---\n## Personal Information",
    "fr": "---\n## Informations personnelles"
  }
}
```

---

## 🚀 Type: SUBMIT

**Usage:** Bouton de soumission du formulaire.

**Propriétés spécifiques:**
- `submitLabel` : Label du bouton (object traduit)

**⚠️ PAS DE `label` ni `path`**

**Exemple complet:**
```json
{
  "id": "submit_button",
  "type": "SUBMIT",
  "submitLabel": {
    "en": "Submit application",
    "fr": "Soumettre la demande"
  }
}
```

**Affichage:**
```
┌─────────────────────────┐
│  Submit application     │
└─────────────────────────┘
```

**⚠️ POSITION:**
Généralement en **dernière position** du formulaire:

```json
{
  "form": [
    { ... },  // Champs
    { ... },
    { ... },
    {
      "id": "submit",
      "type": "SUBMIT",
      "submitLabel": {
        "en": "Submit",
        "fr": "Soumettre"
      }
    }  // En dernier
  ]
}
```

**⚠️ SI OMIS:**
- Bouton "Submit" générique affiché automatiquement
- Label par défaut: "Submit" / "Soumettre"

**Personnalisations:**
```json
// Approuver
"submitLabel": {
  "en": "Approve application",
  "fr": "Approuver la demande"
}

// Enregistrer
"submitLabel": {
  "en": "Save changes",
  "fr": "Enregistrer les modifications"
}

// Continuer
"submitLabel": {
  "en": "Continue",
  "fr": "Continuer"
}
```

---

## ✅ VALIDATIONS EXHAUSTIVES

**Propriété `validations`:** Array d'objets de validation.

**Structure générale:**
```json
"validations": [
  {
    "type": "VALIDATION_TYPE",
    "value": "...",         // Optionnel selon type
    "message": {
      "en": "Error message",
      "fr": "Message d'erreur"
    }
  }
]
```

### Validation: MIN_LENGTH

**Type de champ:** TEXT, TEXT_AREA, TEXT_EDITOR

**Description:** Longueur minimale de caractères.

```json
{
  "type": "MIN_LENGTH",
  "value": 5,
  "message": {
    "en": "Minimum 5 characters required",
    "fr": "Minimum 5 caractères requis"
  }
}
```

**Exemple complet:**
```json
{
  "id": "description",
  "type": "TEXT_AREA",
  "label": {"en": "Description", "fr": "Description"},
  "path": "data.description",
  "validations": [
    {
      "type": "MIN_LENGTH",
      "value": 20,
      "message": {
        "en": "Description must be at least 20 characters",
        "fr": "La description doit faire au moins 20 caractères"
      }
    }
  ]
}
```

---

### Validation: MAX_LENGTH

**Type de champ:** TEXT, TEXT_AREA, TEXT_EDITOR

**Description:** Longueur maximale de caractères.

```json
{
  "type": "MAX_LENGTH",
  "value": 100,
  "message": {
    "en": "Maximum 100 characters allowed",
    "fr": "Maximum 100 caractères autorisés"
  }
}
```

**Combinaison MIN + MAX:**
```json
"validations": [
  {
    "type": "MIN_LENGTH",
    "value": 10,
    "message": {"en": "Minimum 10 characters", "fr": "Minimum 10 caractères"}
  },
  {
    "type": "MAX_LENGTH",
    "value": 500,
    "message": {"en": "Maximum 500 characters", "fr": "Maximum 500 caractères"}
  }
]
```

---

### Validation: REGEX

**Type de champ:** TEXT

**Description:** Pattern d'expression régulière.

```json
{
  "type": "REGEX",
  "value": "^[A-Z]{2}[0-9]{4}$",
  "message": {
    "en": "Format must be: 2 letters + 4 digits (e.g. AB1234)",
    "fr": "Format requis : 2 lettres + 4 chiffres (ex. AB1234)"
  }
}
```

**Exemples courants:**

**Code postal Bénin:**
```json
{
  "type": "REGEX",
  "value": "^[0-9]{5}$",
  "message": {
    "en": "Postal code must be 5 digits",
    "fr": "Le code postal doit contenir 5 chiffres"
  }
}
```

**Plaque d'immatriculation:**
```json
{
  "type": "REGEX",
  "value": "^[A-Z]{2}-[0-9]{4}-[A-Z]{2}$",
  "message": {
    "en": "Format: XX-1234-XX",
    "fr": "Format : XX-1234-XX"
  }
}
```

**Lettres uniquement:**
```json
{
  "type": "REGEX",
  "value": "^[a-zA-ZÀ-ÿ ]+$",
  "message": {
    "en": "Letters only",
    "fr": "Lettres uniquement"
  }
}
```

**⚠️ ÉCHAPPEMENT:**
- Échapper les backslashes: `\\d` au lieu de `\d`
- JSON valide: `"^\\d{3}-\\d{4}$"`

---

### Validation: EMAIL

**Type de champ:** TEXT

**Description:** Format email valide.

```json
{
  "type": "EMAIL",
  "message": {
    "en": "Please enter a valid email address",
    "fr": "Veuillez entrer une adresse email valide"
  }
}
```

**⚠️ PAS de `value`**

**Exemple complet:**
```json
{
  "id": "email",
  "type": "TEXT",
  "label": {"en": "Email", "fr": "Email"},
  "path": "data.person.email",
  "placeholder": {"en": "john@example.com", "fr": "jean@exemple.fr"},
  "required": true,
  "validations": [
    {
      "type": "EMAIL",
      "message": {
        "en": "Invalid email format",
        "fr": "Format d'email invalide"
      }
    }
  ]
}
```

---

### Validation: PHONE

**Type de champ:** TEXT

**Description:** Format numéro de téléphone.

```json
{
  "type": "PHONE",
  "value": "BJ",    // Code pays (optionnel)
  "message": {
    "en": "Please enter a valid Benin phone number",
    "fr": "Veuillez entrer un numéro de téléphone béninois valide"
  }
}
```

**Formats acceptés (Bénin):**
- `97123456` (8 chiffres)
- `+22997123456` (international)
- `00229 97 12 34 56` (avec espaces)

**Exemple:**
```json
{
  "id": "phone",
  "type": "TEXT",
  "label": {"en": "Phone", "fr": "Téléphone"},
  "path": "data.person.phone",
  "placeholder": {"en": "+229 97 12 34 56", "fr": "+229 97 12 34 56"},
  "validations": [
    {
      "type": "PHONE",
      "value": "BJ",
      "message": {
        "en": "Invalid phone number format",
        "fr": "Format de numéro invalide"
      }
    }
  ]
}
```

---

### Validation: MIN_VALUE

**Type de champ:** NUMBER

**Description:** Valeur numérique minimale.

```json
{
  "type": "MIN_VALUE",
  "value": 18,
  "message": {
    "en": "Minimum value is 18",
    "fr": "La valeur minimale est 18"
  }
}
```

**Exemple (âge):**
```json
{
  "id": "age",
  "type": "NUMBER",
  "label": {"en": "Age", "fr": "Âge"},
  "path": "data.person.age",
  "validations": [
    {
      "type": "MIN_VALUE",
      "value": 18,
      "message": {
        "en": "You must be at least 18 years old",
        "fr": "Vous devez avoir au moins 18 ans"
      }
    }
  ]
}
```

---

### Validation: MAX_VALUE

**Type de champ:** NUMBER

**Description:** Valeur numérique maximale.

```json
{
  "type": "MAX_VALUE",
  "value": 120,
  "message": {
    "en": "Maximum value is 120",
    "fr": "La valeur maximale est 120"
  }
}
```

**Combinaison MIN + MAX:**
```json
"validations": [
  {
    "type": "MIN_VALUE",
    "value": 18,
    "message": {"en": "Minimum age: 18", "fr": "Âge minimum : 18"}
  },
  {
    "type": "MAX_VALUE",
    "value": 65,
    "message": {"en": "Maximum age: 65", "fr": "Âge maximum : 65"}
  }
]
```

---

### Validation: INTEGER_ONLY

**Type de champ:** NUMBER

**Description:** Nombre entier uniquement (pas de décimales).

```json
{
  "type": "INTEGER_ONLY",
  "message": {
    "en": "Only whole numbers are allowed",
    "fr": "Seuls les nombres entiers sont autorisés"
  }
}
```

**⚠️ PAS de `value`**

---

### Validation: MIN_DATE

**Type de champ:** DATE

**Description:** Date minimale.

```json
{
  "type": "MIN_DATE",
  "value": "2000-01-01",
  "message": {
    "en": "Date must be after January 1, 2000",
    "fr": "La date doit être après le 1er janvier 2000"
  }
}
```

**Format `value`:** ISO 8601 `YYYY-MM-DD`

---

### Validation: MAX_DATE

**Type de champ:** DATE

**Description:** Date maximale.

```json
{
  "type": "MAX_DATE",
  "value": "2007-12-31",
  "message": {
    "en": "You must be at least 18 years old",
    "fr": "Vous devez avoir au moins 18 ans"
  }
}
```

**Exemple date de naissance:**
```json
{
  "id": "birth_date",
  "type": "DATE",
  "label": {"en": "Date of birth", "fr": "Date de naissance"},
  "path": "data.person.birth_date",
  "validations": [
    {
      "type": "MAX_DATE",
      "value": "2007-10-24",    // 18 ans avant aujourd'hui
      "message": {
        "en": "You must be at least 18 years old",
        "fr": "Vous devez avoir au moins 18 ans"
      }
    }
  ]
}
```

---

### Validation: MUST_BE_TRUE

**Type de champ:** BOOLEAN

**Description:** Case doit être cochée.

```json
{
  "type": "MUST_BE_TRUE",
  "message": {
    "en": "You must accept the terms to continue",
    "fr": "Vous devez accepter les conditions pour continuer"
  }
}
```

**⚠️ PAS de `value`**

**Exemple conditions:**
```json
{
  "id": "accept_terms",
  "type": "BOOLEAN",
  "label": {
    "en": "I accept the terms and conditions",
    "fr": "J'accepte les termes et conditions"
  },
  "path": "data.acceptance.terms",
  "required": true,
  "validations": [
    {
      "type": "MUST_BE_TRUE",
      "message": {
        "en": "You must accept the terms",
        "fr": "Vous devez accepter les conditions"
      }
    }
  ]
}
```

---

### Validation: FILE_TYPE

**Type de champ:** FILE_UPLOAD

**Description:** Types de fichiers acceptés.

```json
{
  "type": "FILE_TYPE",
  "value": ["pdf", "png", "jpg", "jpeg"],
  "message": {
    "en": "Only PDF and image files are accepted",
    "fr": "Seuls les fichiers PDF et images sont acceptés"
  }
}
```

**Format `value`:** Array d'extensions (sans point)

**Exemples:**
```json
// PDF uniquement
"value": ["pdf"]

// Images
"value": ["png", "jpg", "jpeg", "gif"]

// Documents
"value": ["pdf", "doc", "docx"]
```

---

### Validation: FILE_SIZE

**Type de champ:** FILE_UPLOAD

**Description:** Taille maximale fichier (en MB).

```json
{
  "type": "FILE_SIZE",
  "value": 5,
  "message": {
    "en": "File size must not exceed 5MB",
    "fr": "La taille du fichier ne doit pas dépasser 5Mo"
  }
}
```

**Exemple complet FILE_UPLOAD:**
```json
{
  "id": "id_card",
  "type": "FILE_UPLOAD",
  "label": {"en": "ID Card", "fr": "Carte d'identité"},
  "path": "data.files.id_card",
  "accept": ".pdf,.png,.jpg",
  "required": true,
  "validations": [
    {
      "type": "FILE_TYPE",
      "value": ["pdf", "png", "jpg", "jpeg"],
      "message": {
        "en": "Only PDF, PNG, and JPEG files are allowed",
        "fr": "Seuls les fichiers PDF, PNG et JPEG sont autorisés"
      }
    },
    {
      "type": "FILE_SIZE",
      "value": 5,
      "message": {
        "en": "File size cannot exceed 5MB",
        "fr": "La taille ne peut pas dépasser 5Mo"
      }
    }
  ]
}
```

---

## 🎯 CONDITIONS D'AFFICHAGE (displayConditions)

**Rôle:** Afficher/cacher un champ selon conditions.

**Format:** Array d'objets condition (ET logique)

**Structure:**
```json
"displayConditions": [
  {
    "field": "autre_champ_id",
    "operator": "EQUALS",
    "value": "valeur"
  }
]
```

### Opérateurs disponibles

| Opérateur | Description | Types applicables |
|-----------|-------------|-------------------|
| `EQUALS` | Égal à | Tous |
| `NOT_EQUALS` | Différent de | Tous |
| `GREATER_THAN` | Plus grand que | NUMBER, DATE |
| `LESS_THAN` | Plus petit que | NUMBER, DATE |
| `CONTAINS` | Contient (texte) | TEXT |
| `IS_EMPTY` | Vide | Tous |
| `IS_NOT_EMPTY` | Non vide | Tous |

### Exemple: EQUALS

**Afficher champ si choix spécifique:**

```json
{
  "form": [
    {
      "id": "has_employees",
      "type": "STATIC_RADIO_LIST",
      "label": {"en": "Do you have employees?", "fr": "Avez-vous des employés ?"},
      "path": "data.company.has_employees",
      "options": [
        {"value": "yes", "label": {"en": "Yes", "fr": "Oui"}},
        {"value": "no", "label": {"en": "No", "fr": "Non"}}
      ]
    },
    {
      "id": "employee_count",
      "type": "NUMBER",
      "label": {"en": "Number of employees", "fr": "Nombre d'employés"},
      "path": "data.company.employee_count",
      "displayConditions": [
        {
          "field": "has_employees",
          "operator": "EQUALS",
          "value": "yes"
        }
      ]
    }
  ]
}
```

**Fonctionnement:**
- Si utilisateur choisit "Yes" → `employee_count` apparaît
- Si utilisateur choisit "No" → `employee_count` caché

---

### Exemple: NOT_EQUALS

```json
{
  "id": "rejection_reason",
  "type": "TEXT_AREA",
  "label": {"en": "Rejection reason", "fr": "Motif de rejet"},
  "path": "metaData.rejection_reason",
  "displayConditions": [
    {
      "field": "decision",
      "operator": "NOT_EQUALS",
      "value": "approve"
    }
  ]
}
```

**Fonctionnement:**
- Affiché si `decision` ≠ "approve"
- Caché si `decision` = "approve"

---

### Exemple: GREATER_THAN

```json
{
  "id": "senior_discount",
  "type": "BOOLEAN",
  "label": {"en": "Apply senior discount", "fr": "Appliquer réduction senior"},
  "path": "data.apply_senior_discount",
  "displayConditions": [
    {
      "field": "age",
      "operator": "GREATER_THAN",
      "value": 60
    }
  ]
}
```

---

### Exemple: IS_NOT_EMPTY

```json
{
  "id": "company_details",
  "type": "TEXT_AREA",
  "label": {"en": "Company details", "fr": "Détails entreprise"},
  "path": "data.company.details",
  "displayConditions": [
    {
      "field": "company_name",
      "operator": "IS_NOT_EMPTY"
    }
  ]
}
```

---

### Conditions multiples (ET logique)

**Toutes les conditions DOIVENT être vraies:**

```json
{
  "id": "special_field",
  "type": "TEXT",
  "label": {"en": "Special info", "fr": "Info spéciale"},
  "path": "data.special_info",
  "displayConditions": [
    {
      "field": "category",
      "operator": "EQUALS",
      "value": "commercial"
    },
    {
      "field": "employee_count",
      "operator": "GREATER_THAN",
      "value": 10
    },
    {
      "field": "has_license",
      "operator": "EQUALS",
      "value": "yes"
    }
  ]
}
```

**Affiché SEULEMENT si:**
- `category` = "commercial" **ET**
- `employee_count` > 10 **ET**
- `has_license` = "yes"

---

### ⚠️ PIÈGES COURANTS

**1. Référence à champ inexistant:**
```json
"displayConditions": [
  {
    "field": "non_existent_field",    // ❌ Erreur!
    "operator": "EQUALS",
    "value": "yes"
  }
]
```

**2. Type de valeur incorrect:**
```json
// Champ NUMBER
"displayConditions": [
  {
    "field": "age",
    "operator": "EQUALS",
    "value": "25"    // ❌ String au lieu de number!
  }
]

// ✅ CORRECT:
"value": 25
```

**3. Ordre des champs:**
```json
{
  "form": [
    {
      "id": "field_b",
      "displayConditions": [
        {"field": "field_a", ...}    // ❌ field_a pas encore défini!
      ]
    },
    {
      "id": "field_a",
      ...
    }
  ]
}

// ✅ CORRECT: field_a AVANT field_b
{
  "form": [
    {
      "id": "field_a",
      ...
    },
    {
      "id": "field_b",
      "displayConditions": [
        {"field": "field_a", ...}    // ✅ field_a existe
      ]
    }
  ]
}
```

---

## 🎨 EXEMPLE COMPLET DE FORMULAIRE

**Formulaire complet avec TOUS les types de champs:**

```json
{
  "form": [
    {
      "id": "intro",
      "type": "MARKDOWN_DESCRIPTION",
      "content": {
        "en": "## Business Certificate Request\n\nPlease fill out all required fields accurately.",
        "fr": "## Demande de certificat d'entreprise\n\nVeuillez remplir tous les champs requis avec précision."
      }
    },
    {
      "id": "company_name",
      "type": "TEXT",
      "label": {"en": "Company name", "fr": "Nom de l'entreprise"},
      "path": "data.company.name",
      "required": true,
      "validations": [
        {
          "type": "MIN_LENGTH",
          "value": 3,
          "message": {"en": "Minimum 3 characters", "fr": "Minimum 3 caractères"}
        }
      ]
    },
    {
      "id": "registration_date",
      "type": "DATE",
      "label": {"en": "Registration date", "fr": "Date d'enregistrement"},
      "path": "data.company.registration_date",
      "required": true
    },
    {
      "id": "has_employees",
      "type": "STATIC_RADIO_LIST",
      "label": {"en": "Do you have employees?", "fr": "Avez-vous des employés ?"},
      "path": "data.company.has_employees",
      "options": [
        {"value": "yes", "label": {"en": "Yes", "fr": "Oui"}},
        {"value": "no", "label": {"en": "No", "fr": "Non"}}
      ],
      "required": true
    },
    {
      "id": "employees",
      "type": "REPEATABLE",
      "label": {"en": "Employees", "fr": "Employés"},
      "path": "data.company.employees",
      "minItems": 1,
      "maxItems": 10,
      "addButtonLabel": {"en": "Add employee", "fr": "Ajouter employé"},
      "displayConditions": [
        {
          "field": "has_employees",
          "operator": "EQUALS",
          "value": "yes"
        }
      ],
      "fields": [
        {
          "id": "emp_name",
          "type": "TEXT",
          "label": {"en": "Name", "fr": "Nom"},
          "path": "name",
          "required": true
        },
        {
          "id": "emp_position",
          "type": "TEXT",
          "label": {"en": "Position", "fr": "Poste"},
          "path": "position"
        }
      ]
    },
    {
      "id": "registration_doc",
      "type": "FILE_UPLOAD",
      "label": {"en": "Registration document", "fr": "Document d'enregistrement"},
      "path": "data.files.registration",
      "accept": ".pdf",
      "required": true,
      "validations": [
        {
          "type": "FILE_TYPE",
          "value": ["pdf"],
          "message": {"en": "Only PDF", "fr": "PDF uniquement"}
        },
        {
          "type": "FILE_SIZE",
          "value": 5,
          "message": {"en": "Max 5MB", "fr": "Max 5Mo"}
        }
      ]
    },
    {
      "id": "accept_terms",
      "type": "BOOLEAN",
      "label": {"en": "I accept terms", "fr": "J'accepte les conditions"},
      "path": "data.acceptance.terms",
      "required": true,
      "validations": [
        {
          "type": "MUST_BE_TRUE",
          "message": {"en": "Required", "fr": "Requis"}
        }
      ]
    },
    {
      "id": "submit",
      "type": "SUBMIT",
      "submitLabel": {"en": "Submit request", "fr": "Soumettre la demande"}
    }
  ]
}
```

---

**Dernière mise à jour:** 24 octobre 2025
**Voir aussi:**
- [03-UI-CONFIGURATION-PART1.md](03-UI-CONFIGURATION-PART1.md)
- [02-SERVICECONF-PART1.md](02-SERVICECONF-PART1.md)
- [02-SERVICECONF-PART2.md](02-SERVICECONF-PART2.md)
