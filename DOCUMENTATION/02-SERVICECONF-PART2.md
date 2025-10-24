# 📋 SERVICECONF.JSON - PARTIE 2 - STAGE TYPES AVANCÉS

**Suite de la documentation exhaustive du fichier serviceconf.json.**

**⚠️ Lire d'abord:** [02-SERVICECONF-PART1.md](02-SERVICECONF-PART1.md)

---

## 🌐 STAGE TYPE: UXP (SOAP/XML)

**Rôle:** Appel synchrone vers service externe SOAP/XML.

**Voir aussi:** [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md)

**Structure complète:**
```json
"check-company": {
  "type": "uxp",
  "uxpConfiguration": "get-company.xml",
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input",
    "onStageException": {
      "CONNECTION_FAILED": "ERROR-connection",
      "CALL_FAILED": "ERROR-uxp",
      "INVALID_RESPONSE": "ERROR-invalid-data",
      "MISSING_EXPECTED_FIELDS": "ERROR-missing-fields"
    }
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"uxp"`

### `uxpConfiguration`
**Description:** Nom du fichier XML de configuration dans `uxp/`.

**Format:** String (nom de fichier)

**Exemples:**
```json
"uxpConfiguration": "get-company.xml"
"uxpConfiguration": "check-eligibility.xml"
"uxpConfiguration": "verify-document.xml"
```

**⚠️ RÈGLES:**
- Fichier DOIT exister dans `uxp/`
- Extension `.xml` obligatoire
- Contenu SOAP/XML valide

### `transitions`

**Type:** `single` ou `map-by-meta`

**Exceptions mappables (TOUTES):**

| Exception | Description | Recommandation |
|-----------|-------------|----------------|
| `CONNECTION_FAILED` | Impossible de se connecter au serveur | → Stage erreur avec retry |
| `CALL_FAILED` | Appel SOAP échoué (HTTP 500, timeout...) | → Stage erreur avec retry |
| `INVALID_RESPONSE` | Réponse XML invalide ou malformée | → Stage erreur (contactez admin) |
| `MISSING_EXPECTED_FIELDS` | Champs attendus absents dans réponse | → Stage erreur ou fallback |

**Exemple exhaustif:**
```json
"onStageException": {
  "CONNECTION_FAILED": "ERROR-connection",
  "CALL_FAILED": "ERROR-uxp-failed",
  "INVALID_RESPONSE": "ERROR-invalid-response",
  "MISSING_EXPECTED_FIELDS": "ERROR-missing-data"
}
```

**⚠️ RECOMMANDATION FORTE:**
Toujours mapper AU MOINS `CONNECTION_FAILED` et `CALL_FAILED`:

```json
// Minimum viable
"onStageException": {
  "CONNECTION_FAILED": "ERROR-connection",
  "CALL_FAILED": "ERROR-uxp"
}
```

### Fichier UXP (.xml)

Voir [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md) pour structure complète.

**Structure minimale:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<uxp>
  <endpoint>https://api.example.com/service</endpoint>
  <soapAction>getCompanyInfo</soapAction>
  <namespace prefix="ns">http://example.com/namespace</namespace>
  
  <request>
    <![CDATA[
      <ns:GetCompanyRequest>
        <ns:CompanyId>${data.company.id}</ns:CompanyId>
      </ns:GetCompanyRequest>
    ]]>
  </request>
  
  <responseMapping>
    <field source="//ns:CompanyName" target="data.company.name"/>
    <field source="//ns:Status" target="metaData.company_status"/>
  </responseMapping>
</uxp>
```

**Variables disponibles:**
- `${data.xxx}` : Depuis `data`
- `${metaData.xxx}` : Depuis `metaData`

---

## 🔌 STAGE TYPE: UXP-REST (REST/JSON)

**Rôle:** Appel synchrone vers API REST/JSON.

**Voir aussi:** [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md)

**Structure complète:**
```json
"fetch-company": {
  "type": "uxp-rest",
  "uxpConfiguration": "get-company.json",
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input",
    "onStageException": {
      "CONNECTION_FAILED": "ERROR-connection",
      "CALL_FAILED": "ERROR-api",
      "INVALID_RESPONSE": "ERROR-invalid-json",
      "MISSING_EXPECTED_FIELDS": "ERROR-missing-data"
    }
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"uxp-rest"`

### `uxpConfiguration`
**Description:** Nom du fichier JSON de configuration dans `uxp/`.

**Format:** String (nom de fichier)

**Exemples:**
```json
"uxpConfiguration": "get-company.json"
"uxpConfiguration": "check-status.json"
"uxpConfiguration": "validate-document.json"
```

**⚠️ RÈGLES:**
- Fichier DOIT exister dans `uxp/`
- Extension `.json` obligatoire
- Contenu JSON valide

### `transitions`

**Type:** `single` ou `map-by-meta`

**Exceptions mappables (IDENTIQUES à UXP):**

| Exception | Description |
|-----------|-------------|
| `CONNECTION_FAILED` | Impossible de se connecter |
| `CALL_FAILED` | Appel API échoué (HTTP 4xx/5xx) |
| `INVALID_RESPONSE` | JSON invalide ou malformé |
| `MISSING_EXPECTED_FIELDS` | Champs attendus absents |

**Recommandation identique à UXP:**
```json
"onStageException": {
  "CONNECTION_FAILED": "ERROR-connection",
  "CALL_FAILED": "ERROR-api",
  "INVALID_RESPONSE": "ERROR-invalid-json",
  "MISSING_EXPECTED_FIELDS": "ERROR-missing-data"
}
```

### Fichier UXP-REST (.json)

Voir [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md) pour structure complète.

**Structure minimale:**
```json
{
  "method": "POST",
  "url": "https://api.example.com/companies/${data.company.id}",
  "headers": {
    "Authorization": "Bearer ${env.API_TOKEN}",
    "Content-Type": "application/json"
  },
  "body": {
    "companyId": "${data.company.id}",
    "requestType": "verification"
  },
  "responseMapping": [
    {
      "source": "/company/name",
      "target": "data.company.verified_name"
    },
    {
      "source": "/status",
      "target": "metaData.api_status"
    }
  ]
}
```

**Méthodes HTTP supportées:**
- `GET`
- `POST`
- `PUT`
- `PATCH`
- `DELETE`

**Headers personnalisables:**
```json
"headers": {
  "Authorization": "Bearer ${env.API_TOKEN}",
  "Content-Type": "application/json",
  "Accept": "application/json",
  "X-Custom-Header": "value"
}
```

**Variables disponibles:**
- `${data.xxx}` : Depuis `data`
- `${metaData.xxx}` : Depuis `metaData`
- `${env.XXX}` : Variables d'environnement

---

## 📄 STAGE TYPE: GEN-DOCUMENT (PDF)

**Rôle:** Génère un PDF et l'ajoute aux fichiers de la demande.

**Voir aussi:** [06-PDF-TEMPLATES.md](06-PDF-TEMPLATES.md)

**Structure complète:**
```json
"gen-certificate": {
  "type": "gen-document",
  "templateHTML": "certificate-template.html",
  "pathToNameInData": "data.person.full_name",
  "bindToFile": "certificate",
  "transitions": {
    "type": "single",
    "nextStage": "APPROVED"
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"gen-document"`

### `templateHTML`
**Description:** Nom du fichier HTML template dans `pdf/`.

**Format:** String (nom de fichier)

**Exemples:**
```json
"templateHTML": "certificate-template.html"
"templateHTML": "receipt-template.html"
"templateHTML": "approval-letter.html"
```

**⚠️ RÈGLES:**
- Fichier DOIT exister dans `pdf/`
- Extension `.html` obligatoire
- Contenu HTML valide avec variables `${}`

### `pathToNameInData`
**Description:** Chemin vers nom du demandeur pour nom de fichier.

**Format:** String (chemin dot-notation)

**Exemples:**
```json
"pathToNameInData": "data.person.full_name"
"pathToNameInData": "data.company.name"
"pathToNameInData": "data.applicant.last_name"
```

**⚠️ Si le chemin est vide ou invalide:**
- Nom de fichier par défaut: `document.pdf`

**Résultat fichier:**
```
// Avec pathToNameInData = "data.person.full_name"
// et data.person.full_name = "John DOE"
→ Fichier généré: "John_DOE_certificate.pdf"

// Avec pathToNameInData invalide
→ Fichier généré: "document.pdf"
```

### `bindToFile` (OPTIONNEL)
**Description:** Remplace un fichier uploadé par le PDF généré.

**Format:** String (ID du champ FILE_UPLOAD)

**Utilisation:**

```json
// Dans serviceconf.json
"gen-certificate": {
  "type": "gen-document",
  "templateHTML": "certificate.html",
  "bindToFile": "certificate",  // ← ID du champ
  ...
}

// Dans ui/xxx.json
{
  "form": [
    {
      "id": "certificate",  // ← Même ID
      "type": "FILE_UPLOAD",
      "path": "data.files.certificate"
    }
  ]
}
```

**Fonctionnement:**
1. Citoyen upload un fichier temporaire dans le champ `certificate`
2. Stage `gen-document` génère le PDF
3. PDF généré **remplace** le fichier uploadé
4. `data.files.certificate` contient maintenant le PDF généré

**⚠️ Si `bindToFile` omis:**
- PDF ajouté aux fichiers mais ne remplace rien
- Stocké dans `files/generated/`

### `transitions`

**Type:** `single` ou `map-by-meta`

**⚠️ AUCUNE EXCEPTION MAPPABLE!**

```json
"transitions": {
  "type": "single",
  "nextStage": "APPROVED"
  // Pas de onStageException
}
```

**Si génération échoue:**
- Erreur générique affichée
- Processus bloqué
- **Tester TOUJOURS les templates en dev!**

### Fichier PDF template (.html)

Voir [06-PDF-TEMPLATES.md](06-PDF-TEMPLATES.md) pour structure complète.

**Structure minimale:**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <style>
    body { font-family: Arial; }
    .title { text-align: center; font-size: 24px; }
  </style>
</head>
<body>
  <div class="title">CERTIFICAT D'APPROBATION</div>
  
  <p>Nom: ${data.person.full_name}</p>
  <p>Date: ${metaData.approved_date}</p>
  <p>Référence: ${metaData.request_id}</p>
  
  <p>Ce certificat atteste que la demande a été approuvée.</p>
</body>
</html>
```

**Variables disponibles:**
- `${data.xxx}` : Depuis `data`
- `${metaData.xxx}` : Depuis `metaData`
- `${_i18n.current_date}` : Date actuelle

---

## 📧 STAGE TYPE: EMAIL

**Rôle:** Envoie un email à un ou plusieurs destinataires.

**Voir aussi:** [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md)

**Structure complète avec 3 types de destinataires:**

### Type 1: path-to-email (email dans data)

```json
"email-citizen": {
  "type": "email",
  "templateTxt": "citizen-approved.txt",
  "target": {
    "type": "path-to-email",
    "path": "data.person.email"
  },
  "subject": {
    "en": "Your application has been approved",
    "fr": "Votre demande a été approuvée"
  },
  "transitions": {
    "type": "single",
    "nextStage": "APPROVED"
  }
}
```

### Type 2: email-file (liste d'emails)

```json
"email-officials": {
  "type": "email",
  "templateTxt": "official-notification.txt",
  "target": {
    "type": "email-file",
    "fileName": "official-emails.txt"
  },
  "subject": {
    "en": "New application to review",
    "fr": "Nouvelle demande à examiner"
  },
  "transitions": {
    "type": "single",
    "nextStage": "REQUESTED"
  }
}
```

### Type 3: hardcoded-email (email fixe)

```json
"email-admin": {
  "type": "email",
  "templateTxt": "admin-alert.txt",
  "target": {
    "type": "hardcoded-email",
    "email": "admin@example.com"
  },
  "subject": {
    "en": "Application error occurred",
    "fr": "Erreur survenue dans la demande"
  },
  "transitions": {
    "type": "single",
    "nextStage": "ERROR-admin"
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"email"`

### `templateTxt`
**Description:** Nom du fichier template dans `email/`.

**Format:** String (nom de fichier)

**Exemples:**
```json
"templateTxt": "citizen-approved.txt"
"templateTxt": "official-notification.txt"
"templateTxt": "rejection-notice.txt"
```

**⚠️ RÈGLES:**
- Fichier DOIT exister dans `email/`
- Extension `.txt` obligatoire
- Contenu texte brut avec variables `${}`

### `target`
**Description:** Définit le(s) destinataire(s).

**3 types possibles:**

#### Type 1: `path-to-email`
**Usage:** Email stocké dans data/metaData

```json
"target": {
  "type": "path-to-email",
  "path": "data.person.email"
}
```

**Chemins possibles:**
```json
"path": "data.person.email"
"path": "data.contact.email"
"path": "metaData.recipient_email"
```

**⚠️ Si chemin vide/invalide:**
- Email NON envoyé
- Pas d'erreur générée
- Processus continue

#### Type 2: `email-file`
**Usage:** Liste d'emails dans fichier

```json
"target": {
  "type": "email-file",
  "fileName": "official-emails.txt"
}
```

**Fichier format:**
```
email/official-emails.txt:

admin@example.com
reviewer1@example.com
manager@example.com
```

**⚠️ RÈGLES fichier:**
- 1 email par ligne
- Pas de commentaires
- Lignes vides ignorées
- Emails invalides ignorés

#### Type 3: `hardcoded-email`
**Usage:** Email fixe en dur

```json
"target": {
  "type": "hardcoded-email",
  "email": "admin@example.com"
}
```

**⚠️ Usage recommandé:**
- Notifications admin/système
- Emails de fallback
- Alertes techniques

### `subject`
**Description:** Objet de l'email (traduit).

**Format:** Object avec langues

```json
"subject": {
  "en": "Your application has been approved",
  "fr": "Votre demande a été approuvée"
}
```

**⚠️ Peut contenir des variables:**
```json
"subject": {
  "en": "Application ${metaData.request_id} - Status update",
  "fr": "Demande ${metaData.request_id} - Mise à jour"
}
```

### `transitions`

**Type:** `single` ou `map-by-meta`

**⚠️ AUCUNE EXCEPTION MAPPABLE!**

```json
"transitions": {
  "type": "single",
  "nextStage": "APPROVED"
}
```

**Si envoi échoue:**
- Email non envoyé mais processus continue
- Erreur loggée côté serveur
- **Pas de blocage utilisateur**

### Fichier email template (.txt)

Voir [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md) pour structure complète.

**Structure minimale:**
```
Bonjour ${data.person.first_name},

Votre demande de certificat a été approuvée.

Référence: ${metaData.request_id}
Date d'approbation: ${metaData.approved_date}

Vous pouvez télécharger votre certificat depuis votre espace citoyen.

Cordialement,
Le Service des Citoyens
```

**Variables disponibles:**
- `${data.xxx}` : Depuis `data`
- `${metaData.xxx}` : Depuis `metaData`
- `${_i18n.xxx}` : Traductions

---

## 📱 STAGE TYPE: NOTIFICATION (SMS)

**Rôle:** Envoie un SMS à un destinataire.

**Voir aussi:** [05-NOTIFICATION-TEMPLATES.md](05-NOTIFICATION-TEMPLATES.md)

**Structure complète avec 3 types de destinataires:**

### Type 1: path-to-npi (NPI dans data)

```json
"sms-citizen": {
  "type": "notification",
  "templateTxt": "citizen-approved.txt",
  "target": {
    "type": "path-to-npi",
    "path": "metaData.citizen.0"
  },
  "transitions": {
    "type": "single",
    "nextStage": "APPROVED"
  }
}
```

### Type 2: path-to-nr (numéro dans data)

```json
"sms-contact": {
  "type": "notification",
  "templateTxt": "notification.txt",
  "target": {
    "type": "path-to-nr",
    "path": "data.contact.phone"
  },
  "transitions": {
    "type": "single",
    "nextStage": "NOTIFIED"
  }
}
```

### Type 3: hardcoded-nr (numéro fixe)

```json
"sms-admin": {
  "type": "notification",
  "templateTxt": "admin-alert.txt",
  "target": {
    "type": "hardcoded-nr",
    "nr": "+22997123456"
  },
  "transitions": {
    "type": "single",
    "nextStage": "ALERT-SENT"
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"notification"`

### `templateTxt`
**Description:** Nom du fichier template dans `notification/`.

**Format:** String (nom de fichier)

**Exemples:**
```json
"templateTxt": "citizen-approved.txt"
"templateTxt": "citizen-rejected.txt"
"templateTxt": "alert.txt"
```

**⚠️ RÈGLES:**
- Fichier DOIT exister dans `notification/`
- Extension `.txt` obligatoire
- Contenu texte brut avec variables `${}`
- **Limite 160 caractères recommandée!**

### `target`
**Description:** Définit le destinataire.

**3 types possibles:**

#### Type 1: `path-to-npi`
**Usage:** NPI stocké dans metaData (le plus courant)

```json
"target": {
  "type": "path-to-npi",
  "path": "metaData.citizen.0"
}
```

**Chemins courants:**
```json
"path": "metaData.citizen.0"          // Premier NPI citoyen
"path": "metaData.owners.0"           // Premier propriétaire
"path": "metaData.contact_npi"        // NPI contact
```

**Fonctionnement:**
1. Lit le NPI au chemin spécifié
2. Récupère le numéro de téléphone lié au NPI
3. Envoie le SMS

**⚠️ Si NPI invalide/inexistant:**
- SMS NON envoyé
- Pas d'erreur
- Processus continue

#### Type 2: `path-to-nr`
**Usage:** Numéro de téléphone dans data

```json
"target": {
  "type": "path-to-nr",
  "path": "data.contact.phone"
}
```

**Chemins possibles:**
```json
"path": "data.person.phone"
"path": "data.contact.mobile"
"path": "metaData.emergency_phone"
```

**⚠️ Format numéro:**
- Recommandé: Format international `+229XXXXXXXX`
- Accepté: `97XXXXXX` (numéro local Bénin)

**⚠️ Si numéro invalide:**
- SMS NON envoyé
- Erreur loggée
- Processus continue

#### Type 3: `hardcoded-nr`
**Usage:** Numéro fixe en dur

```json
"target": {
  "type": "hardcoded-nr",
  "nr": "+22997123456"
}
```

**⚠️ Format:**
- DOIT commencer par `+229` (Bénin)
- Ou format international autre pays

**Usage recommandé:**
- Alertes admin
- Notifications système
- Numéros de secours

### `transitions`

**Type:** `single` ou `map-by-meta`

**⚠️ AUCUNE EXCEPTION MAPPABLE!**

```json
"transitions": {
  "type": "single",
  "nextStage": "APPROVED"
}
```

**Si envoi échoue:**
- SMS non envoyé mais processus continue
- Erreur loggée
- **Pas de blocage**

### Fichier notification template (.txt)

Voir [05-NOTIFICATION-TEMPLATES.md](05-NOTIFICATION-TEMPLATES.md) pour structure complète.

**Structure minimale:**
```
Votre demande ${metaData.request_id} a ete approuvee. Telechargez votre certificat sur le portail.
```

**⚠️ CONTRAINTES SMS:**
1. **160 caractères MAX recommandé** (sinon envoi en plusieurs SMS)
2. **Éviter accents** (problèmes encodage)
3. **Pas de retours à la ligne** (ignorés)
4. **Variables courtes**

**Bonnes pratiques:**
```
✅ Demande ${metaData.request_id} approuvee. Certificat disponible.
❌ Bonjour ${data.person.first_name},\nNous avons le plaisir de vous informer que votre demande ${metaData.request_id} a été approuvée...
```

---

## 👤 STAGE TYPE: PERSON-DATA

**Rôle:** Récupère automatiquement les données d'une personne via son NPI.

**Structure complète:**
```json
"fetch-citizen-data": {
  "type": "person-data",
  "readFromNpi": "metaData.citizen.0",
  "writeData": {
    "data": {
      "person": {
        "first_name": "${first_name}",
        "last_name": "${last_name}",
        "birth_date": "${birth_date}",
        "npi": "${npi}"
      }
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input",
    "onStageException": {
      "NPI_NOT_FOUND": "ERROR-npi"
    }
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"person-data"`

### `readFromNpi`
**Description:** Chemin vers le NPI à utiliser.

**Format:** String (chemin dot-notation)

**Exemples:**
```json
"readFromNpi": "metaData.citizen.0"      // Premier citoyen
"readFromNpi": "metaData.owner_npi"      // NPI propriétaire
"readFromNpi": "data.applicant.npi"      // NPI demandeur
```

**⚠️ RÈGLES:**
- Le NPI DOIT exister au chemin spécifié
- Format NPI: 9 chiffres (ex: `123456789`)
- Si absent/invalide: exception `NPI_NOT_FOUND`

### `writeData`
**Description:** Mapping des données récupérées.

**Format:** Object avec chemins de destination

**Variables disponibles (récupérées automatiquement):**
- `${npi}` : Numéro NPI
- `${first_name}` : Prénom
- `${last_name}` : Nom
- `${birth_date}` : Date de naissance
- `${birth_place}` : Lieu de naissance
- `${gender}` : Sexe (M/F)
- `${address}` : Adresse
- `${phone}` : Numéro de téléphone
- `${email}` : Email

**Exemple complet:**
```json
"writeData": {
  "data": {
    "person": {
      "npi": "${npi}",
      "first_name": "${first_name}",
      "last_name": "${last_name}",
      "full_name": "${first_name} ${last_name}",
      "birth_date": "${birth_date}",
      "birth_place": "${birth_place}",
      "gender": "${gender}",
      "contact": {
        "address": "${address}",
        "phone": "${phone}",
        "email": "${email}"
      }
    }
  },
  "metaData": {
    "applicant_npi": "${npi}",
    "applicant_name": "${first_name} ${last_name}"
  }
}
```

**⚠️ SHALLOW MERGE:**
- Écrase les valeurs existantes
- Ne fusionne PAS profondément
- Attention aux conflits!

### `transitions`

**Type:** `single` ou `map-by-meta`

**Exception mappable:**

| Exception | Description | Recommandation |
|-----------|-------------|----------------|
| `NPI_NOT_FOUND` | NPI inexistant ou invalide | → Stage erreur ou formulaire manuel |

**Exemple:**
```json
"onStageException": {
  "NPI_NOT_FOUND": "ERROR-invalid-npi"
}
```

**⚠️ Si exception non mappée:**
- Erreur générique
- Utilisateur bloqué

---

## 🗂️ STAGE TYPE: HARDCODED-DATA

**Rôle:** Écrit des données fixes dans data/metaData.

**Structure complète:**
```json
"set-default-values": {
  "type": "hardcoded-data",
  "writeData": {
    "data": {
      "request_type": "certificate",
      "default_language": "fr",
      "status": "pending"
    },
    "metaData": {
      "submission_date": "${_i18n.current_date}",
      "version": "1.0",
      "source": "citizen-portal"
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input"
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"hardcoded-data"`

### `writeData`
**Description:** Données à écrire.

**Format:** Object avec `data` et/ou `metaData`

**Exemples:**

#### Valeurs fixes
```json
"writeData": {
  "data": {
    "category": "COMMERCIAL",
    "priority": "NORMAL"
  }
}
```

#### Variables dynamiques
```json
"writeData": {
  "metaData": {
    "created_date": "${_i18n.current_date}",
    "created_by": "${metaData.citizen.0}",
    "request_ref": "REQ-${_i18n.current_timestamp}"
  }
}
```

**Variables disponibles:**
- `${data.xxx}` : Valeurs depuis data
- `${metaData.xxx}` : Valeurs depuis metaData
- `${_i18n.current_date}` : Date actuelle (format: DD/MM/YYYY)
- `${_i18n.current_timestamp}` : Timestamp UNIX

**⚠️ SHALLOW MERGE:**
```json
// État initial
data = {
  "person": {
    "name": "John",
    "age": 30
  }
}

// writeData
"writeData": {
  "data": {
    "person": {
      "city": "Cotonou"
    }
  }
}

// Résultat
data = {
  "person": {
    "city": "Cotonou"    // ❌ name et age perdus!
  }
}
```

**✅ Pour ajouter sans écraser:**
```json
"writeData": {
  "data": {
    "person": {
      "name": "${data.person.name}",
      "age": "${data.person.age}",
      "city": "Cotonou"
    }
  }
}
```

### `transitions`

**Type:** `single` ou `map-by-meta`

**⚠️ AUCUNE EXCEPTION MAPPABLE!**

---

## 🚫 STAGE TYPE: BLACKLIST

**Rôle:** Vérifie si un NPI est dans une liste noire.

**Structure complète:**
```json
"check-blacklist": {
  "type": "blacklist",
  "readFromNpi": "metaData.citizen.0",
  "blacklistName": "service-ban-list",
  "transitions": {
    "type": "map-by-meta",
    "metaPathToKey": ["blacklist_check"],
    "map": {
      "blocked": "BLACKLISTED",
      "allowed": "citizen-input"
    }
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"blacklist"`

### `readFromNpi`
**Description:** Chemin vers le NPI à vérifier.

**Format:** String (chemin dot-notation)

```json
"readFromNpi": "metaData.citizen.0"
"readFromNpi": "data.applicant.npi"
```

### `blacklistName`
**Description:** Nom de la liste noire (configurée côté serveur).

**Format:** String

**Exemples:**
```json
"blacklistName": "service-ban-list"
"blacklistName": "fraud-list"
"blacklistName": "restricted-users"
```

**⚠️ Liste DOIT exister côté serveur!**

### Fonctionnement automatique

Le stage écrit automatiquement dans `metaData.blacklist_check`:
- `"blocked"` : NPI trouvé dans la liste
- `"allowed"` : NPI non trouvé

### `transitions`

**Type:** OBLIGATOIREMENT `map-by-meta`

**⚠️ DOIT mapper les 2 valeurs:**

```json
"transitions": {
  "type": "map-by-meta",
  "metaPathToKey": ["blacklist_check"],
  "map": {
    "blocked": "BLACKLISTED",     // NPI banni
    "allowed": "citizen-input"    // NPI autorisé
  }
}
```

**Exemple workflow complet:**
```json
"check-blacklist": {
  "type": "blacklist",
  "readFromNpi": "metaData.citizen.0",
  "blacklistName": "service-ban",
  "transitions": {
    "type": "map-by-meta",
    "metaPathToKey": ["blacklist_check"],
    "map": {
      "blocked": "BLACKLISTED",
      "allowed": "citizen-input"
    }
  }
},

"BLACKLISTED": {
  "type": "main",
  "shortTitle": {"en": "BLOCKED", "fr": "BLOQUÉ"},
  "title": {"en": "Access denied", "fr": "Accès refusé"},
  "flags": ["final", "reject"],
  "transitions": []
}
```

---

## ⚠️ STAGE TYPE: ALERT

**Rôle:** Affiche une alerte/message à l'utilisateur.

**Structure complète:**
```json
"alert-changes-requested": {
  "type": "alert",
  "title": {
    "en": "Changes Requested",
    "fr": "Modifications demandées"
  },
  "message": {
    "en": "The official has requested changes to your application. Please review the comments and resubmit.",
    "fr": "L'agent a demandé des modifications à votre demande. Veuillez consulter les commentaires et resoumettre."
  },
  "transitions": {
    "type": "single",
    "nextStage": "citizen-edit"
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"alert"`

### `title`
**Description:** Titre de l'alerte (traduit).

**Format:** Object avec langues

```json
"title": {
  "en": "Important Notice",
  "fr": "Avis important"
}
```

### `message`
**Description:** Message détaillé (traduit).

**Format:** Object avec langues

```json
"message": {
  "en": "Your application requires additional review. You will be notified when the process is complete.",
  "fr": "Votre demande nécessite un examen supplémentaire. Vous serez notifié lorsque le processus sera terminé."
}
```

**⚠️ Peut contenir des variables:**
```json
"message": {
  "en": "Application ${metaData.request_id}: ${metaData.alert_message}",
  "fr": "Demande ${metaData.request_id}: ${metaData.alert_message}"
}
```

### `transitions`

**Type:** `single` ou `map-by-meta`

**⚠️ AUCUNE EXCEPTION!**

**Affichage:**
- Modal/popup dans l'interface
- Utilisateur doit cliquer "OK" pour continuer
- Continue automatiquement vers `nextStage`

---

## 📎 STAGE TYPE: SHARE-FILES

**Rôle:** Partage TOUS les fichiers uploadés dans un stage.

**Structure complète:**
```json
"share-documents": {
  "type": "share-files",
  "shareWithStage": "citizen-input",
  "transitions": {
    "type": "single",
    "nextStage": "gen-certificate"
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"share-files"`

### `shareWithStage`
**Description:** ID du stage UI contenant les FILE_UPLOAD.

**Format:** String (ID de stage ui)

```json
"shareWithStage": "citizen-input"
"shareWithStage": "upload-documents"
```

**Fonctionnement:**
1. Trouve le stage UI spécifié
2. Récupère TOUS les fichiers uploadés dans ce stage
3. Les rend accessibles/visibles pour téléchargement

**⚠️ Partage TOUT:**
- Tous les champs FILE_UPLOAD du stage
- Impossible de filtrer
- Pour partage sélectif: utiliser `share-one-pdf`

### `transitions`

**Type:** `single` ou `map-by-meta`

**⚠️ AUCUNE EXCEPTION!**

---

## 📄 STAGE TYPE: SHARE-ONE-PDF

**Rôle:** Partage UN fichier PDF spécifique.

**Structure complète:**
```json
"share-certificate": {
  "type": "share-one-pdf",
  "shareWithStage": "gen-certificate",
  "fileName": "certificate",
  "transitions": {
    "type": "single",
    "nextStage": "APPROVED"
  }
}
```

**Propriétés:**

### `type`
**Valeur:** `"share-one-pdf"`

### `shareWithStage`
**Description:** ID du stage qui a généré le PDF.

**Format:** String (ID de stage gen-document)

```json
"shareWithStage": "gen-certificate"
"shareWithStage": "gen-receipt"
```

### `fileName`
**Description:** ID du fichier à partager (bindToFile du gen-document).

**Format:** String

```json
"fileName": "certificate"
"fileName": "receipt"
```

**⚠️ DOIT correspondre au `bindToFile` du stage gen-document:**

```json
// Stage gen-document
"gen-certificate": {
  "type": "gen-document",
  "bindToFile": "certificate",  // ← ID
  ...
}

// Stage share
"share-certificate": {
  "type": "share-one-pdf",
  "shareWithStage": "gen-certificate",
  "fileName": "certificate"      // ← Même ID
}
```

### `transitions`

**Type:** `single` ou `map-by-meta`

**⚠️ AUCUNE EXCEPTION!**

---

## 🔄 RÉSUMÉ DES STAGE TYPES

| Type | Rôle | Fichier associé | Exceptions mappables |
|------|------|-----------------|---------------------|
| `start` | Point de départ | - | ❌ Aucune |
| `main` | État majeur | - | ❌ Aucune |
| `ui` | Formulaire | `ui/*.json` | ✅ 2 (REQUIRED_FIELDS, FIELDS_NOT_ALLOWED) |
| `pay` | Paiement | - | ❌ Aucune |
| `uxp` | SOAP/XML | `uxp/*.xml` | ✅ 4 (CONNECTION, CALL, RESPONSE, FIELDS) |
| `uxp-rest` | REST/JSON | `uxp/*.json` | ✅ 4 (CONNECTION, CALL, RESPONSE, FIELDS) |
| `gen-document` | Génération PDF | `pdf/*.html` | ❌ Aucune |
| `email` | Envoi email | `email/*.txt` | ❌ Aucune |
| `notification` | Envoi SMS | `notification/*.txt` | ❌ Aucune |
| `person-data` | Récup données NPI | - | ✅ 1 (NPI_NOT_FOUND) |
| `hardcoded-data` | Écriture fixe | - | ❌ Aucune |
| `blacklist` | Vérif liste noire | - | ❌ Aucune (mais map-by-meta obligatoire) |
| `alert` | Message utilisateur | - | ❌ Aucune |
| `share-files` | Partage tous fichiers | - | ❌ Aucune |
| `share-one-pdf` | Partage 1 PDF | - | ❌ Aucune |

---

## ✅ CHECKLIST VALIDATION

**Avant de déployer un serviceconf.json:**

### Structure racine
- [ ] `serviceId` correspond au dossier et CatIS
- [ ] `serviceVersion` correspond au dossier de version
- [ ] `public` est cohérent avec `start.permissions.public`
- [ ] `backOffice` (si présent) pointe vers chemins existants

### Stage start
- [ ] Exactement 1 stage `start`
- [ ] `permissions` définies (type meta-array recommandé)
- [ ] `transitions.nextStage` pointe vers stage existant

### Main stages
- [ ] Au moins 1 avec `flags: ["final"]`
- [ ] Au moins 1 avec `flags: ["final", "accept"]` OU `["final", "reject"]`
- [ ] Chaque transition a `id` unique dans tout le fichier
- [ ] `resultMainStages` liste les fins possibles

### Stages UI
- [ ] Fichiers `.json` existent dans `ui/`
- [ ] Exceptions `REQUIRED_FIELDS_MISSING` et `FIELDS_NOT_ALLOWED` mappées

### Stages UXP
- [ ] Fichiers `.xml`/`.json` existent dans `uxp/`
- [ ] Au moins `CONNECTION_FAILED` et `CALL_FAILED` mappées

### Stages PDF
- [ ] Fichiers `.html` existent dans `pdf/`
- [ ] `pathToNameInData` pointe vers chemin valide
- [ ] `bindToFile` (si présent) correspond à ID FILE_UPLOAD

### Stages email
- [ ] Fichiers `.txt` existent dans `email/`
- [ ] `target.type` et propriétés cohérentes
- [ ] `subject` défini avec langues

### Stages SMS
- [ ] Fichiers `.txt` existent dans `notification/`
- [ ] Templates ≤ 160 caractères
- [ ] `target.type` et propriétés cohérentes

### Transitions
- [ ] Tous les `nextStage` pointent vers stages existants
- [ ] Aucun `nextStage` ne pointe vers `start`
- [ ] Stages `blacklist` utilisent obligatoirement `map-by-meta`
- [ ] Map-by-meta a TOUTES les valeurs possibles

### Permissions
- [ ] Tous les stages/transitions ont `permissions` définies
- [ ] `meta-array` avec `metaPathToArray` correct
- [ ] `hardcoded-array` avec groupes/NPIs valides
- [ ] `public` utilisé avec précaution

---

**Dernière mise à jour:** 24 octobre 2025
**Voir aussi:**
- [02-SERVICECONF-PART1.md](02-SERVICECONF-PART1.md) (structure de base)
- [03-UI-CONFIGURATION.md](03-UI-CONFIGURATION.md)
- [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md)
- [05-NOTIFICATION-TEMPLATES.md](05-NOTIFICATION-TEMPLATES.md)
- [06-PDF-TEMPLATES.md](06-PDF-TEMPLATES.md)
- [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md)
