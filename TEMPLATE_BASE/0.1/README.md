# 📋 TEMPLATE EXHAUSTIF - E-SERVICE PORTAIL CITOYEN BÉNINOIS

Ce template contient **TOUS** les exemples possibles pour créer un e-service complet.

---

## 🎯 UTILISATION RAPIDE

### 1. Copier le template
```bash
cp -r TEMPLATE_BASE/ PS<VOTRE_ID>/
cd PS<VOTRE_ID>/0.1/
```

### 2. Modifier le fichier principal
```bash
# Éditer le serviceconf.json (minimal et fonctionnel)
vim PS<VOTRE_ID>.serviceconf.json
```

### 3. Consulter le fichier EXHAUSTIF pour ajouter des stages
```bash
# Référence: voir TEMPLATE_BASE.EXHAUSTIF.jsonc
# Copier/coller les stages dont vous avez besoin
```

### 4. Créer vos templates UI/email/PDF selon besoins
```bash
# Voir sections ci-dessous pour chaque type
```

---

## 📁 STRUCTURE D'UN E-SERVICE

```
PS<ID>/
├── 0.1/                                  # Version du service
│   ├── PS<ID>.serviceconf.json           # ⭐ Configuration principale
│   ├── ui/                               # Formulaires utilisateur
│   │   ├── citizen-input.json
│   │   ├── official-review.json
│   │   └── ...
│   ├── email/                            # Templates email
│   │   ├── citizen-approved.txt
│   │   ├── official-notification.txt
│   │   ├── official-emails.txt           # Liste destinataires
│   │   └── ...
│   ├── notification/                     # Templates SMS
│   │   ├── citizen-approved.txt
│   │   └── ...
│   ├── pdf/                              # Templates PDF
│   │   ├── header-template.html
│   │   ├── footer-template.html
│   │   ├── certificate-template.html
│   │   └── ...
│   └── uxp/                              # Intégrations UXP
│       ├── request-template.json         # REST
│       ├── request-template.xml          # SOAP
│       └── ...
└── 0.2/                                  # Versions ultérieures
```

---

## 🔧 CONFIGURATION PRINCIPALE (serviceconf.json)

### Structure racine

```json
{
  "serviceId": "PS00565",          // ID unique (correspond au dossier)
  "serviceVersion": "0.1",         // Version
  "public": true,                  // Accessible sans authentification?
  "backOffice": { ... },           // Config back-office (optionnel)
  "stages": { ... }                // ⭐ Workflow complet
}
```

### Back-office (optionnel)

```json
"backOffice": {
  "customColumnOfficial": "data.company.name",
  "customColumnTitleOfficial": {
    "en": "COMPANY NAME",
    "fr": "NOM COMMERCIAL"
  }
}
```

Affiche une colonne personnalisée dans la liste des demandes (back-office).

---

## 🚦 TYPES DE STAGES

### 1️⃣ START (OBLIGATOIRE)

**Rôle:** Point de départ du workflow. Définit les permissions initiales.

**Exemple minimal:**
```json
"start": {
  "type": "start",
  "shortTitle": {"en": "STARTED", "fr": "DÉBUT"},
  "title": {"en": "Application started", "fr": "Demande démarrée"},
  "pathTitle": {"en": "Continue", "fr": "Poursuivre"},
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

**Options permissions:**
- `"type": "meta-array"` + `metaPathToArray`: NPIs dans metaData
- `"type": "hardcoded-array"` + `array`: liste fixe de groupes
- `"type": "public"`: accessible à tous

---

### 2️⃣ MAIN STAGE

**Rôle:** État majeur du workflow (REQUESTED, APPROVED, REJECTED...).

**Exemple standard:**
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

**Exemple final (avec flags):**
```json
"APPROVED": {
  "type": "main",
  "shortTitle": {"en": "APPROVED", "fr": "APPROUVÉE"},
  "title": {"en": "Approved", "fr": "Approuvée"},
  "flags": ["final", "accept"],  // ✅ Affichage vert
  "transitions": []
}
```

**Flags disponibles:**
- `"final"`: marque comme terminée (requis pour fin workflow)
- `"accept"`: affichage vert (succès)
- `"reject"`: affichage rouge (échec)

---

### 3️⃣ UI STAGE

**Rôle:** Formulaire de saisie utilisateur.

**Exemple avec gestion d'erreurs:**
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

**Exceptions possibles:**
- `REQUIRED_FIELDS_MISSING`: champs requis manquants → retour au formulaire
- `FIELDS_NOT_ALLOWED`: champs non autorisés → retour au formulaire

**Avec branchement:**
```json
"official-review": {
  "type": "ui",
  "uiConfiguration": "official-review.json",
  "transitions": {
    "type": "map-by-meta",
    "metaPathToKey": ["official", "choice"],
    "map": {
      "approve": "gen-certificate",
      "reject": "REJECTED"
    },
    "onStageException": {
      "REQUIRED_FIELDS_MISSING": "official-review"
    }
  }
}
```

➡️ **Voir section UI CONFIGURATION ci-dessous pour format des fichiers ui/*.json**

---

### 4️⃣ UXP-REST STAGE

**Rôle:** Appel REST/JSON vers service externe.

**Exemple complet:**
```json
"get-company-info": {
  "type": "uxp-rest",
  "templateFile": "get-company.json",
  "responseMapping": {
    "data.company.name": {
      "pointer": "/company/name",
      "required": true
    },
    "data.company.rccm": {
      "pointer": "/company/registration/rccm",
      "required": true
    },
    "data.company.address": {
      "pointer": "/company/address/full",
      "required": false
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "display-company",
    "onStageException": {
      "PROCESSING_ERROR": "ERROR-uxp",
      "MISSING_REQUIRED_NODE": "ERROR-company-not-found",
      "CONNECTION_ERROR": "ERROR-connection"
    }
  }
}
```

**Exceptions:**
- `PROCESSING_ERROR`: erreur inattendue
- `INVALID_CONFIGURATION`: template invalide
- `MISSING_TEMPLATE_VARIABLE`: variable `${}` manquante
- `MISSING_REQUIRED_NODE`: nœud `required:true` absent
- `WEB_APPLICATION_EXCEPTION`: HTTP 4xx/5xx
- `CONNECTION_ERROR`: échec connexion

➡️ **Voir section UXP TEMPLATES ci-dessous**

---

### 5️⃣ UXP STAGE (SOAP/XML)

**Rôle:** Appel SOAP/XML vers service X-Road.

**Exemple:**
```json
"check-eligibility": {
  "type": "uxp",
  "template": {
    "templateFile": "check-eligibility.xml",
    "namespaces": {
      "soapenv": "http://schemas.xmlsoap.org/soap/envelope/",
      "wsdl": "http://x-road.eu/xsd/connector/service/v1"
    },
    "responseMap": {
      "metaData.eligible": {
        "xpath": "/soapenv:Envelope/soapenv:Body/wsdl:checkResponse/ELIGIBLE",
        "required": true
      }
    }
  },
  "transitions": {
    "type": "map-by-meta",
    "metaPathToKey": ["eligible"],
    "map": {
      "true": "continue-process",
      "false": "REJECTED"
    },
    "onStageException": {
      "CONFIG_ERROR": "ERROR-uxp",
      "CONNECTION_ERROR": "ERROR-connection",
      "DATA_ERROR": "ERROR-uxp"
    }
  }
}
```

**Exceptions:**
- `CONFIG_ERROR`: erreur parsing config/template
- `CONNECTION_ERROR`: erreur envoi/réception
- `DATA_ERROR`: nœud requis absent
- `DATA_WARNING`: nœud optionnel absent (non bloquant)

➡️ **Voir section UXP TEMPLATES ci-dessous**

---

### 6️⃣ GEN-DOCUMENT STAGE

**Rôle:** Génération PDF à partir de template HTML.

**Exemple:**
```json
"gen-certificate": {
  "type": "gen-document",
  "template": "certificate.html",
  "fileName": "Certificate_${data.person.last_name}.pdf",
  "transitions": {
    "type": "single",
    "nextStage": "share-certificate",
    "onStageException": {
      "NO_TEMPLATE": "ERROR-pdf",
      "API_EXCEPTION": "ERROR-pdf",
      "CONNECTION_ERROR": "ERROR-pdf"
    }
  }
}
```

**Avec fichiers liés:**
```json
"gen-certificate": {
  "type": "gen-document",
  "template": "certificate.html",
  "fileName": "Certificate.pdf",
  "bindToFile": "identity-documents",  // ⭐ Lie fichiers uploadés
  "transitions": { ... }
}
```

**Exceptions:**
- `NO_TEMPLATE`: template HTML non trouvé
- `READING_FILE_CONTENT_FAILED`: erreur lecture
- `PERSISTING_FILE_CONTENT_FAILED`: PDF trop gros
- `API_EXCEPTION`: erreur API génération
- `CONNECTION_ERROR`: échec connexion

➡️ **Voir section PDF TEMPLATES ci-dessous**

---

### 7️⃣ EMAIL STAGE

**Rôle:** Envoi email automatique.

**3 types de cibles:**

#### Type 1: Chemin vers email
```json
"email-citizen": {
  "type": "email",
  "bodyTemplatePath": "citizen-approved.txt",
  "subjectTemplate": "Demande approuvée",
  "targetType": "path-to-email",
  "targetPath": "data.person.email",
  "transitions": {
    "type": "single",
    "nextStage": "APPROVED",
    "onStageException": {
      "NOTIFICATION_SERVICE_ERROR": "APPROVED"  // Pas bloquant
    }
  }
}
```

#### Type 2: Fichier texte (multiples emails)
```json
"email-officials": {
  "type": "email",
  "bodyTemplatePath": "official-notification.txt",
  "subjectTemplate": "Nouvelle demande",
  "targetType": "email-file",
  "targetsFile": "official-emails.txt",  // ⭐ 1 email/ligne
  "transitions": { ... }
}
```

#### Type 3: Emails hardcodés
```json
"email-admins": {
  "type": "email",
  "bodyTemplatePath": "admin-alert.txt",
  "subjectTemplate": "ALERTE",
  "targetType": "hardcoded-email",
  "targets": ["admin@gouv.bj", "supervisor@gouv.bj"],
  "transitions": { ... }
}
```

**Exceptions:**
- `NO_TARGET_CONFIGURED`: target manquant
- `FAILED_TO_GET_TARGET_NODE`: chemin non trouvé
- `NO_TEMPLATE`: template non trouvé
- `NOTIFICATION_SERVICE_ERROR`: erreur API

➡️ **Voir section EMAIL TEMPLATES ci-dessous**

---

### 8️⃣ NOTIFICATION STAGE (SMS)

**Rôle:** Envoi SMS.

**3 types de cibles:**

#### Type 1: NPI (cherche téléphone dans DB)
```json
"notify-citizen": {
  "type": "notification",
  "template": "citizen-approved.txt",
  "targetType": "path-to-npi",
  "target": "metaData.citizen.0",
  "transitions": {
    "type": "single",
    "nextStage": "APPROVED"
  }
}
```

#### Type 2: Numéro direct
```json
"notify-phone": {
  "type": "notification",
  "template": "reminder.txt",
  "targetType": "path-to-nr",
  "target": "data.contact.phone",  // +229 ajouté auto
  "transitions": { ... }
}
```

#### Type 3: Numéro hardcodé
```json
"notify-hotline": {
  "type": "notification",
  "template": "urgent.txt",
  "targetType": "hardcoded-nr",
  "target": "+22997123456",
  "transitions": { ... }
}
```

**Exceptions:**
- `FAILED_TO_GET_TARGET_NODE`: chemin non trouvé
- `INVALID_PHONE_NUMBER`: numéro invalide
- `NO_VALID_TARGETS`: aucune cible valide
- `NO_TEMPLATE`: template non trouvé
- `NOTIFICATION_SERVICE_ERROR`: erreur API
- `INVALID_CONFIG`: targetType invalide

➡️ **Voir section NOTIFICATION TEMPLATES ci-dessous**

---

### 9️⃣ PERSON-DATA STAGE

**Rôle:** Récupération données citoyen depuis DB (par NPI).

**Exemple complet:**
```json
"get-citizen-data": {
  "type": "person-data",
  "npiSource": "metaData.citizen.0",
  "skipIfNoNpi": false,
  "dataPoints": {
    "first_name": "data.person.first_name",
    "last_name": "data.person.last_name",
    "birth_date": "data.person.birth_date",
    "email": "data.person.email",
    "phone_number": "data.person.phone_number",
    "residence_address": "data.person.residence_address"
  },
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input",
    "onStageException": {
      "CANNOT_GET_NPI": "ERROR-person-data",
      "CANNOT_FIND_PERSON": "ERROR-person-not-found"
    }
  }
}
```

**Champs disponibles:**
- **Identité:** `first_name`, `last_name`, `marital_name`, `sex`
- **Naissance:** `birth_date`, `birth_country_code`, `birth_department`, `birth_town`, `birth_district`, `birth_village`, `birth_place`
- **Résidence:** `residence_department`, `residence_town`, `residence_district`, `residence_village`, `residence_address`
- **Contact:** `email`, `phone_country_code`, `phone_number`
- **Autre:** `nationality`, `mother_npi`, `father_npi`

**Exceptions:**
- `CANNOT_GET_NPI`: NPI non trouvé au chemin
- `CANNOT_FIND_PERSON`: citoyen inexistant

---

### 🔟 PAYMENT STAGE

**Rôle:** Paiement KKiaPay/FedaPay.

**Exemple montant fixe:**
```json
"payment-stage": {
  "type": "pay",
  "amount": "5000",  // FCFA
  "publicKey": "<PUBLIC_KEY>",
  "privateKey": "<PRIVATE_KEY>",
  "secret": "<SECRET>",
  "sandbox": true,  // true=test, false=production
  "transitions": {
    "type": "single",
    "nextStage": "gen-receipt"
  }
}
```

**Exemple montant dynamique:**
```json
"payment-dynamic": {
  "type": "pay",
  "dynamicAmount": "data.calculated_fee",  // Chemin vers montant
  "publicKey": "...",
  "privateKey": "...",
  "secret": "...",
  "sandbox": true,
  "transitions": { ... }
}
```

⚠️ **ATTENTION:**
- Montant (`amount`) et montant dynamique (`dynamicAmount`) sont **mutuellement exclusifs**
- **Aucune exception mappable** → le paiement DOIT réussir pour continuer

---

### 1️⃣1️⃣ HARDCODED-DATA STAGE

**Rôle:** Injection données fixes.

**Exemple:**
```json
"set-approval-flag": {
  "type": "hardcoded-data",
  "writeData": {
    "metaData": {
      "status": "approved",
      "approvedBy": "system"
    },
    "data": {
      "paymentDone": "true"
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "gen-certificate"
  }
}
```

⚠️ **SHALLOW MERGE:** objets imbriqués écrasés entièrement (comme UI stage).

---

### 1️⃣2️⃣ BLACKLIST STAGE

**Rôle:** Suppression de champs.

**Exemple:**
```json
"clean-temp-data": {
  "type": "blacklist",
  "blacklist": [
    "metaData._comment",
    "data.formDraft",
    "data.intermediate.calculation"
  ],
  "transitions": {
    "type": "single",
    "nextStage": "email-official"
  }
}
```

**Usage courant:** nettoyer données temporaires, supprimer commentaires d'un chemin.

**Astuce:** `"blacklist": []` (vide) = aucune suppression, juste routing.

---

### 1️⃣3️⃣ ALERT STAGE

**Rôle:** Afficher message à l'utilisateur.

**Exemple erreur:**
```json
"ERROR-uxp": {
  "type": "alert",
  "alertType": "ERROR",  // ERROR, WARNING, ou SUCCESS
  "message": {
    "en": "An error occurred. Please try again.",
    "fr": "Une erreur s'est produite. Veuillez réessayer."
  },
  "clearMessages": true,  // Efface messages précédents
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input"
  }
}
```

**Types d'alerte:**
- `"ERROR"`: rouge (erreur)
- `"WARNING"`: orange (avertissement)
- `"SUCCESS"`: vert (succès)

---

### 1️⃣4️⃣ SHARE-FILES STAGE

**Rôle:** Partage ZIP de TOUS les fichiers.

**Exemple:**
```json
"share-all-files": {
  "type": "share-files",
  "codePath": "data.files.accessCode",
  "baseUrlPath": "data.files.baseUrl",
  "fileNamePath": "data.files.zipName",
  "zipFileName": "Demande_${data.person.last_name}",
  "transitions": {
    "type": "single",
    "nextStage": "email-official"
  }
}
```

⚠️ **SÉCURITÉ:**
- Lien **PUBLIC** (pas d'authentification)
- Lien **PERMANENT** (aucune expiration)
- Utiliser avec précaution!

**URL générée:**
```
${data.files.baseUrl}/api/portal/shared/files/${data.files.accessCode}
```

---

### 1️⃣5️⃣ SHARE-ONE-PDF STAGE

**Rôle:** Partage d'un PDF unique.

**Exemple:**
```json
"share-certificate": {
  "type": "share-one-pdf",
  "codePath": "data.certificate.shareCode",
  "baseUrlPath": "data.certificate.baseUrl",
  "fileNamePath": "data.certificate.fileName",
  "transitions": {
    "type": "single",
    "nextStage": "email-citizen"
  }
}
```

Même avertissements sécurité que `share-files`.

---

## 🎨 UI CONFIGURATION (ui/*.json)

Les fichiers UI définissent les formulaires utilisateur.

### Structure de base

```json
{
  "form": [
    {
      "id": "field_name",
      "type": "TEXT",
      "required": true,
      "label": {
        "en": "Field label",
        "fr": "Libellé du champ"
      },
      "displayConditions": [...],
      "validations": [...]
    }
  ]
}
```

### Types de champs disponibles

| Type | Description | Exemple |
|------|-------------|---------|
| `TEXT` | Texte court | Nom, prénom |
| `TEXT_AREA` | Texte long | Commentaire |
| `TEXT_EDITOR` | Éditeur riche | Contenu formaté |
| `NUMBER` | Nombre | Âge, montant |
| `BOOLEAN` | Case à cocher | J'accepte |
| `DATE` | Sélecteur date | Date naissance |
| `FILE_UPLOAD` | Upload fichier | Photo, document |
| `RADIO_LIST` | Boutons radio | Choix unique |
| `STATIC_RADIO_LIST` | Radio fixes | Oui/Non |
| `REPEATABLE` | Groupe répétable | Liste enfants |
| `MARKDOWN_DESCRIPTION` | Texte informatif | Instructions |
| `SUBMIT` | Bouton submit | Soumettre |

### Exemple complet (citizen-input.json)

```json
{
  "form": [
    {
      "id": "personal_info_title",
      "type": "MARKDOWN_DESCRIPTION",
      "required": false,
      "label": {
        "en": "## Personal Information",
        "fr": "## Informations personnelles"
      }
    },
    {
      "id": "first_name",
      "type": "TEXT",
      "required": true,
      "label": {
        "en": "First name",
        "fr": "Prénom"
      },
      "path": "data.person.first_name",
      "validations": [
        {
          "type": "MAX_LENGTH",
          "value": 50,
          "errorMessage": {
            "en": "Maximum 50 characters",
            "fr": "Maximum 50 caractères"
          }
        }
      ]
    },
    {
      "id": "last_name",
      "type": "TEXT",
      "required": true,
      "label": {
        "en": "Last name",
        "fr": "Nom"
      },
      "path": "data.person.last_name"
    },
    {
      "id": "birth_date",
      "type": "DATE",
      "required": true,
      "label": {
        "en": "Date of birth",
        "fr": "Date de naissance"
      },
      "path": "data.person.birth_date",
      "validations": [
        {
          "type": "DATE_BEFORE_NOW",
          "errorMessage": {
            "en": "Birth date must be in the past",
            "fr": "La date de naissance doit être passée"
          }
        }
      ]
    },
    {
      "id": "has_children",
      "type": "BOOLEAN",
      "required": false,
      "label": {
        "en": "Do you have children?",
        "fr": "Avez-vous des enfants ?"
      },
      "path": "data.person.has_children"
    },
    {
      "id": "children",
      "type": "REPEATABLE",
      "required": false,
      "label": {
        "en": "Children",
        "fr": "Enfants"
      },
      "path": "data.person.children",
      "displayConditions": [
        {
          "id": "has_children",
          "value": true
        }
      ],
      "form": [
        {
          "id": "child_name",
          "type": "TEXT",
          "required": true,
          "label": {
            "en": "Child name",
            "fr": "Nom de l'enfant"
          },
          "path": "name"
        },
        {
          "id": "child_age",
          "type": "NUMBER",
          "required": true,
          "label": {
            "en": "Age",
            "fr": "Âge"
          },
          "path": "age"
        }
      ]
    },
    {
      "id": "document_type",
      "type": "STATIC_RADIO_LIST",
      "required": true,
      "label": {
        "en": "Document type",
        "fr": "Type de document"
      },
      "path": "data.document.type",
      "options": [
        {
          "label": {
            "en": "Identity Card",
            "fr": "Carte d'identité"
          },
          "value": "ID_CARD"
        },
        {
          "label": {
            "en": "Passport",
            "fr": "Passeport"
          },
          "value": "PASSPORT"
        }
      ]
    },
    {
      "id": "document_file",
      "type": "FILE_UPLOAD",
      "required": true,
      "label": {
        "en": "Upload document",
        "fr": "Téléverser le document"
      },
      "uploadId": "identity-documents",
      "validations": [
        {
          "type": "FILE_TYPE",
          "value": ["pdf", "jpg", "png"],
          "errorMessage": {
            "en": "Only PDF, JPG, PNG allowed",
            "fr": "Seulement PDF, JPG, PNG autorisés"
          }
        },
        {
          "type": "FILE_SIZE_MAX",
          "value": 5242880,
          "errorMessage": {
            "en": "Maximum 5MB",
            "fr": "Maximum 5Mo"
          }
        }
      ]
    },
    {
      "id": "submit",
      "type": "SUBMIT",
      "required": false,
      "label": {
        "en": "Submit application",
        "fr": "Soumettre la demande"
      }
    }
  ]
}
```

### Validations disponibles

| Type | Description | Paramètres |
|------|-------------|------------|
| `MAX_LENGTH` | Longueur max | `value`: nombre |
| `MIN_LENGTH` | Longueur min | `value`: nombre |
| `REGEX` | Expression régulière | `value`: pattern |
| `DATE_BEFORE_NOW` | Date passée | - |
| `DATE_AFTER_NOW` | Date future | - |
| `FILE_TYPE` | Types fichiers | `value`: array extensions |
| `FILE_SIZE_MAX` | Taille max | `value`: octets |
| `MIN_VALUE` | Valeur min (nombre) | `value`: nombre |
| `MAX_VALUE` | Valeur max (nombre) | `value`: nombre |

### Display Conditions

```json
"displayConditions": [
  {
    "id": "has_children",
    "value": true
  }
]
```

Affiche le champ **seulement si** `has_children` est `true`.

**Conditions multiples (AND):**
```json
"displayConditions": [
  {"id": "condition1", "value": true},
  {"id": "condition2", "value": "yes"}
]
```

---

## 📧 EMAIL TEMPLATES (email/*.txt)

Templates texte avec variables `${data.field}` et `${metaData.field}`.

### Exemple: citizen-approved.txt

```
Bonjour ${data.person.first_name} ${data.person.last_name},

Votre demande de certificat a été approuvée.

Numéro de référence: ${metaData.request_id}
Date d'approbation: ${metaData.approval_date}

Vous pouvez télécharger votre certificat en cliquant sur le lien suivant:
${data.certificate.baseUrl}/api/portal/shared/files/${data.certificate.shareCode}

Cordialement,
Le Portail Citoyen
```

### Exemple: official-emails.txt (liste)

```
admin@ministry.bj
reviewer1@ministry.bj
reviewer2@ministry.bj
manager@ministry.bj
```

**Format:** 1 email par ligne.

---

## 📱 NOTIFICATION TEMPLATES (notification/*.txt)

Templates SMS avec variables `${}`.

### Exemple: citizen-approved.txt

```
Demande ${metaData.request_id} approuvée. Téléchargez votre certificat sur ${data.certificate.baseUrl}
```

⚠️ **Limite SMS:** ~160 caractères recommandé.

---

## 📄 PDF TEMPLATES (pdf/*.html)

Templates HTML pour génération PDF.

### Structure recommandée

- `header-template.html`: En-tête (toutes pages)
- `footer-template.html`: Pied de page (toutes pages)
- `certificate-template.html`: Corps du PDF

### Exemple: header-template.html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <style>
        .header {
            text-align: center;
            font-size: 18px;
            font-weight: bold;
            border-bottom: 2px solid #000;
            padding-bottom: 10px;
        }
    </style>
</head>
<body>
    <div class="header">
        RÉPUBLIQUE DU BÉNIN<br>
        Ministère du Développement Numérique
    </div>
</body>
</html>
```

### Exemple: footer-template.html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <style>
        .footer {
            text-align: center;
            font-size: 10px;
            border-top: 1px solid #000;
            padding-top: 10px;
        }
    </style>
</head>
<body>
    <div class="footer">
        Document généré le ${metaData.generation_date}<br>
        Page <span class="page"></span> sur <span class="topage"></span>
    </div>
</body>
</html>
```

**Variables spéciales footer:**
- `<span class="page"></span>`: numéro page actuelle
- `<span class="topage"></span>`: total pages

### Exemple: certificate-template.html

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        .title {
            text-align: center;
            font-size: 24px;
            font-weight: bold;
            margin-bottom: 30px;
        }
        .content {
            font-size: 14px;
            line-height: 1.6;
        }
        .signature {
            margin-top: 50px;
            text-align: right;
        }
    </style>
</head>
<body>
    <div class="title">
        ATTESTATION DE ${data.certificate.type}
    </div>
    
    <div class="content">
        <p>Je soussigné, ${data.official.name}, ${data.official.title},</p>
        
        <p>Atteste que ${data.person.first_name} ${data.person.last_name}, 
        né(e) le ${data.person.birth_date} à ${data.person.birth_place},
        résidant à ${data.person.residence_address},</p>
        
        <p>${data.certificate.attestation_text}</p>
        
        <p>En foi de quoi la présente attestation lui est délivrée 
        pour servir et valoir ce que de droit.</p>
    </div>
    
    <div class="signature">
        <p>Fait à Cotonou, le ${metaData.issue_date}</p>
        <p>${data.official.name}<br>
        ${data.official.title}</p>
    </div>
</body>
</html>
```

### Images dans PDF

**Option 1: Base64**
```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..." alt="Logo">
```

**Option 2: URL**
```html
<img src="https://example.com/logo.png" alt="Logo">
```

---

## 🔗 UXP TEMPLATES

### REST/JSON (uxp/*.json)

**Exemple: get-company.json**

```json
{
  "method": "POST",
  "url": "https://uxp.gouv.bj/api/company/search",
  "headers": {
    "Content-Type": "application/json",
    "X-API-Key": "YOUR_API_KEY"
  },
  "body": {
    "rccm": "${data.company.rccm}",
    "name": "${data.company.name}"
  }
}
```

**Variables disponibles:**
- `${data.field}`: accès à `data`
- `${metaData.field}`: accès à `metaData`

**Réponse mappée via `responseMapping` (voir config serviceconf.json).**

### SOAP/XML (uxp/*.xml)

**Exemple: check-eligibility.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" 
                  xmlns:wsdl="http://x-road.eu/xsd/connector/service/v1">
    <soapenv:Header>
        <wsdl:client>BJ.GOV.PORTAL</wsdl:client>
        <wsdl:service>BJ.GOV.MINISTRY.ELIGIBILITY</wsdl:service>
        <wsdl:userId>${metaData.citizen.0}</wsdl:userId>
    </soapenv:Header>
    <soapenv:Body>
        <wsdl:checkEligibility>
            <NPI>${metaData.citizen.0}</NPI>
            <SERVICE_ID>${serviceId}</SERVICE_ID>
        </wsdl:checkEligibility>
    </soapenv:Body>
</soapenv:Envelope>
```

**Namespaces et XPath définis dans config serviceconf.json.**

---

## 🎯 PATTERNS COURANTS

### Pattern 1: Demande simple citoyen → révision officiel → approbation/rejet

**Workflow:**
```
start → citizen-input → REQUESTED → official-review → gen-certificate → APPROVED
                                                     → REJECTED
```

**serviceconf.json:**
```json
{
  "stages": {
    "start": { ... },
    "citizen-input": {
      "type": "ui",
      "uiConfiguration": "citizen-input.json",
      "transitions": {"type": "single", "nextStage": "REQUESTED"}
    },
    "REQUESTED": {
      "type": "main",
      "transitions": [{
        "id": "review",
        "title": {"en": "Review", "fr": "Examiner"},
        "nextStage": "official-review",
        "resultMainStages": ["APPROVED", "REJECTED"],
        "permissions": {
          "type": "hardcoded-array",
          "actor": "OFFICIAL",
          "array": ["ReviewGroup"]
        }
      }]
    },
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
    },
    "gen-certificate": {
      "type": "gen-document",
      "template": "certificate.html",
      "fileName": "Certificate.pdf",
      "transitions": {"type": "single", "nextStage": "APPROVED"}
    },
    "APPROVED": {
      "type": "main",
      "flags": ["final", "accept"],
      "transitions": []
    },
    "REJECTED": {
      "type": "main",
      "flags": ["final", "reject"],
      "transitions": []
    }
  }
}
```

---

### Pattern 2: Vérification UXP → formulaire → paiement → génération

**Workflow:**
```
start → get-citizen-data → uxp-check → citizen-input → payment → gen-document → APPROVED
                                     → REJECTED (inéligible)
```

---

### Pattern 3: Workflow avec modifications demandées

**Workflow:**
```
start → citizen-input → REQUESTED → official-review → gen-certificate → APPROVED
                                                     → alert-changes → CHANGES-REQUESTED
                                                                      
CHANGES-REQUESTED → citizen-resubmit → REQUESTED
```

---

## ⚠️ PIÈGES COURANTS

### 1. Shallow Merge (UI et hardcoded-data)

❌ **ERREUR:**
```
État initial: data.person = {name: "X", age: 30, city: "Cotonou"}
UI envoie:    data.person = {name: "Y"}
Résultat:     data.person = {name: "Y"}  // age et city PERDUS!
```

✅ **SOLUTION:** Toujours renvoyer TOUS les champs d'un objet dans UI.

---

### 2. Variables manquantes dans templates

❌ **ERREUR:**
```
Template email: "Bonjour ${data.person.firstname}"
Réalité:        data.person.first_name (underscore!)
→ Variable non trouvée
```

✅ **SOLUTION:** Vérifier exactement le nom des champs dans data/metaData.

---

### 3. Boucles infinies avec map-by-meta

❌ **ERREUR:**
```json
"stage-a": {
  "type": "blacklist",
  "blacklist": [],
  "transitions": {
    "type": "map-by-meta",
    "metaPathToKey": ["type"],
    "map": {
      "A": "stage-a",  // 😱 Boucle infinie!
      "B": "stage-b"
    }
  }
}
```

✅ **SOLUTION:** Toujours vérifier qu'aucun branchement ne reboucle sur lui-même.

---

### 4. Permissions trop restrictives

❌ **ERREUR:**
```json
"start": {
  "permissions": {
    "type": "hardcoded-array",
    "actor": "OFFICIAL",
    "array": ["AdminGroup"]
  }
  // Aucun citoyen ne peut démarrer!
}
```

✅ **SOLUTION:** Pour services citoyens, utiliser `"type": "meta-array"` + `"public": true`.

---

### 5. Oubli flag "final"

❌ **ERREUR:**
```json
"APPROVED": {
  "type": "main",
  "transitions": []
  // Pas de flag "final" → demande jamais terminée!
}
```

✅ **SOLUTION:** TOUJOURS ajouter `"flags": ["final"]` aux stages finaux.

---

## 📚 DOCUMENTATION COMPLÈTE

- **Guide développement:** `bj-citizen-portal-service-development.md` (2466 lignes)
- **Spécifications UI:** `bj-citizen-portal-service-ui-spec-complet.md` (1059 lignes)
- **Intégration UXP:** `bj-citizen-portal-uxp-integration.adoc`
- **Génération PDF:** `bj-citizen-portal-pdf-generation.adoc`

---

## ✅ CHECKLIST AVANT DÉPLOIEMENT

- [ ] `serviceId` correspond au nom du dossier
- [ ] 1 seul stage `"type": "start"`
- [ ] Au moins 1 stage final avec `"flags": ["final"]`
- [ ] Tous les `uiConfiguration` référencent des fichiers existants dans `ui/`
- [ ] Tous les templates (email/notification/pdf/uxp) existent
- [ ] Gestion `onStageException` pour stages critiques (uxp, ui, gen-document)
- [ ] Permissions cohérentes (public pour citoyens, hardcoded pour officiels)
- [ ] Variables `${}` correspondent aux vrais chemins data/metaData
- [ ] Aucune boucle infinie dans les transitions
- [ ] Clés paiement correctes (si stage `pay`)
- [ ] Mode sandbox activé pour tests (si stage `pay`)

---

## 🆘 SUPPORT

Pour questions ou problèmes:
1. Consulter la documentation officielle
2. Vérifier les exemples dans services existants (PS00565, PS01259, PS01333...)
3. Contacter l'équipe technique du Portail Citoyen

---

**Bon développement! 🚀**
