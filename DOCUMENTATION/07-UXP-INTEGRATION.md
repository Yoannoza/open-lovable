# 🔌 UXP INTEGRATION - DOCUMENTATION COMPLÈTE

**Documentation exhaustive de l'intégration avec les systèmes externes (UXP).**

---

## 🎯 RÔLE DES INTÉGRATIONS UXP

Les fichiers dans `uxp/` définissent les **appels vers des APIs externes** (REST ou SOAP).

**Ils permettent:**
- ✅ Vérifier données externes (NPI, RCCM, IFU)
- ✅ Enrichir demande avec données officielles
- ✅ Valider éligibilité
- ✅ Synchroniser avec systèmes tiers
- ✅ Consulter bases de données externes

---

## 📁 EMPLACEMENT

```
PSxxxxx/
└── x.y/
    └── uxp/
        ├── verify-company.json       ← Appel REST
        ├── check-eligibility.soap    ← Appel SOAP
        └── README.md                 ← Documentation (optionnel)
```

**Règles strictes:**
- ✅ Extension `.json` (REST) ou `.soap` (SOAP)
- ✅ Format JSON valide
- ✅ Encodage UTF-8
- ✅ 1 fichier = 1 appel

---

## 🏗️ STRUCTURE GÉNÉRALE

### Format minimal (REST)

```json
{
  "method": "POST",
  "url": "https://api.gouv.bj/companies/verify",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer ${env.API_TOKEN}"
  },
  "body": {
    "company_id": "${data.company.rccm}"
  },
  "responseMapping": {
    "data": {
      "company_name": "/name",
      "company_status": "/status",
      "company_address": "/address/full"
    }
  }
}
```

---

## 🌐 APPELS REST/JSON

### Structure complète

```json
{
  "method": "POST",
  "url": "https://api.example.com/endpoint",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer ${env.API_TOKEN}",
    "X-Custom-Header": "value"
  },
  "body": {
    "field1": "${data.path.to.value}",
    "field2": "static value"
  },
  "responseMapping": {
    "data": {
      "destination_field": "/json/pointer/path"
    },
    "metaData": {
      "meta_field": "/another/path"
    }
  }
}
```

---

### Propriété `method`

**Méthodes HTTP supportées:**

| Méthode | Usage | Body autorisé |
|---------|-------|---------------|
| `GET` | Lecture | ❌ Non |
| `POST` | Création | ✅ Oui |
| `PUT` | Mise à jour | ✅ Oui |
| `PATCH` | Modification partielle | ✅ Oui |
| `DELETE` | Suppression | ❌ Non |

**Exemples:**

```json
{
  "method": "GET",
  "url": "https://api.example.com/user/${data.user_id}"
}
```

```json
{
  "method": "POST",
  "url": "https://api.example.com/verify",
  "body": { "id": "${data.npi}" }
}
```

---

### Propriété `url`

**URL complète de l'API:**

```json
{
  "url": "https://api.gouv.bj/v1/companies/search"
}
```

**Variables dans URL:**
```json
{
  "url": "https://api.example.com/users/${data.user_id}/profile"
}
```

**Query parameters:**
```json
{
  "url": "https://api.example.com/search?type=${data.search_type}&limit=10"
}
```

---

### Propriété `headers`

**Headers courants:**

```json
{
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer ${env.API_TOKEN}",
    "Accept": "application/json",
    "X-Request-ID": "${metaData.request_id}",
    "User-Agent": "BeninPortal/1.0"
  }
}
```

**Types d'authentification:**

#### 1. Bearer Token
```json
{
  "headers": {
    "Authorization": "Bearer ${env.API_TOKEN}"
  }
}
```

#### 2. API Key
```json
{
  "headers": {
    "X-API-Key": "${env.API_KEY}"
  }
}
```

#### 3. Basic Auth
```json
{
  "headers": {
    "Authorization": "Basic ${env.BASIC_AUTH_ENCODED}"
  }
}
```

**⚠️ IMPORTANT:** Jamais de secrets en dur, toujours `${env.xxx}`.

---

### Propriété `body`

**Corps de la requête (POST/PUT/PATCH):**

```json
{
  "body": {
    "npi": "${metaData.citizen.0}",
    "company_id": "${data.company.rccm}",
    "request_date": "${_i18n.current_date}",
    "nested": {
      "field1": "${data.value1}",
      "field2": "static"
    },
    "array": [
      "${data.item1}",
      "${data.item2}"
    ]
  }
}
```

**Variables disponibles:**
- `${data.xxx}` - Données utilisateur
- `${metaData.xxx}` - Métadonnées
- `${env.xxx}` - Variables environnement
- `${_i18n.xxx}` - Variables système

---

### Propriété `responseMapping`

**Mapper la réponse dans data/metaData:**

#### Syntaxe JSON Pointer

**Réponse API:**
```json
{
  "status": "success",
  "data": {
    "company": {
      "name": "ABC Corp",
      "address": {
        "street": "Avenue Clozel",
        "city": "Cotonou"
      }
    },
    "valid": true
  }
}
```

**Mapping:**
```json
{
  "responseMapping": {
    "data": {
      "company_name": "/data/company/name",
      "company_city": "/data/company/address/city"
    },
    "metaData": {
      "is_valid": "/data/valid",
      "api_status": "/status"
    }
  }
}
```

**Résultat dans workflow:**
```json
{
  "data": {
    "company_name": "ABC Corp",
    "company_city": "Cotonou"
  },
  "metaData": {
    "is_valid": true,
    "api_status": "success"
  }
}
```

**⚠️ JSON Pointer:**
- Commence par `/`
- Séparateur: `/`
- Path: `/parent/child/grandchild`

---

## 🧼 APPELS SOAP/XML

### Structure complète

```json
{
  "method": "SOAP",
  "url": "https://soap.example.com/service",
  "soapAction": "urn:VerifyCompany",
  "namespace": {
    "soap": "http://schemas.xmlsoap.org/soap/envelope/",
    "ns": "http://example.com/services"
  },
  "body": {
    "soap:Envelope": {
      "@xmlns:soap": "http://schemas.xmlsoap.org/soap/envelope/",
      "@xmlns:ns": "http://example.com/services",
      "soap:Body": {
        "ns:VerifyCompanyRequest": {
          "ns:CompanyID": "${data.company.rccm}",
          "ns:RequestDate": "${_i18n.current_date}"
        }
      }
    }
  },
  "responseMapping": {
    "data": {
      "company_name": "//ns:CompanyName",
      "company_status": "//ns:Status"
    }
  }
}
```

---

### Propriété `method`

**SOAP uniquement:**
```json
{
  "method": "SOAP"
}
```

---

### Propriété `soapAction`

**Action SOAP (header HTTP):**

```json
{
  "soapAction": "urn:VerifyCompany"
}
```

**Varie selon le service SOAP.**

---

### Propriété `namespace`

**Namespaces XML:**

```json
{
  "namespace": {
    "soap": "http://schemas.xmlsoap.org/soap/envelope/",
    "ns1": "http://example.com/service/v1",
    "ns2": "http://example.com/common"
  }
}
```

**Utilisés dans body et responseMapping.**

---

### Propriété `body` (SOAP)

**Structure SOAP Envelope:**

```json
{
  "body": {
    "soap:Envelope": {
      "@xmlns:soap": "http://schemas.xmlsoap.org/soap/envelope/",
      "@xmlns:ns": "http://example.com/services",
      "soap:Header": {
        "ns:Authentication": {
          "ns:Token": "${env.SOAP_TOKEN}"
        }
      },
      "soap:Body": {
        "ns:GetCompanyRequest": {
          "ns:CompanyID": "${data.company.rccm}",
          "ns:IncludeDetails": "true"
        }
      }
    }
  }
}
```

**⚠️ Attributs XML:** Préfixe `@` pour attributs.

**XML généré:**
```xml
<soap:Envelope 
  xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
  xmlns:ns="http://example.com/services">
  <soap:Header>
    <ns:Authentication>
      <ns:Token>TOKEN_VALUE</ns:Token>
    </ns:Authentication>
  </soap:Header>
  <soap:Body>
    <ns:GetCompanyRequest>
      <ns:CompanyID>RCCM_VALUE</ns:CompanyID>
      <ns:IncludeDetails>true</ns:IncludeDetails>
    </ns:GetCompanyRequest>
  </soap:Body>
</soap:Envelope>
```

---

### Propriété `responseMapping` (SOAP)

**Syntaxe XPath:**

**Réponse SOAP:**
```xml
<soap:Envelope xmlns:soap="...">
  <soap:Body>
    <ns:GetCompanyResponse xmlns:ns="...">
      <ns:Company>
        <ns:Name>ABC Corp</ns:Name>
        <ns:Status>Active</ns:Status>
        <ns:Address>
          <ns:City>Cotonou</ns:City>
        </ns:Address>
      </ns:Company>
    </ns:GetCompanyResponse>
  </soap:Body>
</soap:Envelope>
```

**Mapping XPath:**
```json
{
  "responseMapping": {
    "data": {
      "company_name": "//ns:Name",
      "company_status": "//ns:Status",
      "company_city": "//ns:Address/ns:City"
    }
  }
}
```

**⚠️ XPath:**
- Commence par `//`
- Utilise les namespaces définis
- Path: `//ns:Parent/ns:Child`

---

## 🎯 EXEMPLES COMPLETS

### Exemple 1: Vérifier NPI (REST)

**Fichier:** `uxp/verify-npi.json`

```json
{
  "method": "POST",
  "url": "https://api.gouv.bj/npi/verify",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer ${env.NPI_API_TOKEN}",
    "X-Request-ID": "${metaData.request_id}"
  },
  "body": {
    "npi": "${metaData.citizen.0}",
    "include_details": true
  },
  "responseMapping": {
    "data": {
      "person_verified_name": "/data/full_name",
      "person_verified_birth_date": "/data/birth_date",
      "person_verified_gender": "/data/gender"
    },
    "metaData": {
      "npi_verification_status": "/status",
      "npi_verification_date": "/verification_date",
      "npi_is_active": "/data/is_active"
    }
  }
}
```

**Workflow:**
```json
"verify-citizen-npi": {
  "type": "uxp",
  "uxpFile": "verify-npi.json",
  "exceptions": {
    "CALL_FAILED": "npi-verification-failed",
    "INVALID_RESPONSE": "npi-verification-error"
  },
  "transitions": {
    "type": "map-by-meta",
    "key": "npi_is_active",
    "mapping": {
      "true": "citizen-input",
      "false": "npi-inactive-error"
    }
  }
}
```

---

### Exemple 2: Rechercher entreprise (REST)

**Fichier:** `uxp/search-company.json`

```json
{
  "method": "GET",
  "url": "https://api.gouv.bj/rccm/search?id=${data.company.rccm}&format=json",
  "headers": {
    "Authorization": "Bearer ${env.RCCM_API_TOKEN}",
    "Accept": "application/json"
  },
  "responseMapping": {
    "data": {
      "company_official_name": "/result/name",
      "company_registration_date": "/result/created_at",
      "company_activity_type": "/result/activity/type",
      "company_address": "/result/address/street",
      "company_city": "/result/address/city",
      "company_legal_form": "/result/legal_form"
    },
    "metaData": {
      "company_verified": "/result/verified",
      "company_last_updated": "/result/updated_at"
    }
  }
}
```

**Workflow:**
```json
"search-company-info": {
  "type": "uxp",
  "uxpFile": "search-company.json",
  "exceptions": {
    "CONNECTION_FAILED": "api-connection-error",
    "CALL_FAILED": "company-not-found",
    "INVALID_RESPONSE": "api-error"
  },
  "transitions": {
    "type": "map-by-meta",
    "key": "company_verified",
    "mapping": {
      "true": "company-input",
      "false": "company-verification-failed"
    }
  }
}
```

---

### Exemple 3: Vérifier IFU (SOAP)

**Fichier:** `uxp/verify-ifu.soap`

```json
{
  "method": "SOAP",
  "url": "https://soap.impots.bj/VerificationService",
  "soapAction": "urn:VerifyIFU",
  "namespace": {
    "soap": "http://schemas.xmlsoap.org/soap/envelope/",
    "ns": "http://impots.bj/verification/v1"
  },
  "body": {
    "soap:Envelope": {
      "@xmlns:soap": "http://schemas.xmlsoap.org/soap/envelope/",
      "@xmlns:ns": "http://impots.bj/verification/v1",
      "soap:Header": {
        "ns:Authentication": {
          "ns:Username": "${env.SOAP_USERNAME}",
          "ns:Password": "${env.SOAP_PASSWORD}"
        }
      },
      "soap:Body": {
        "ns:VerifyIFURequest": {
          "ns:IFU": "${data.company.ifu}",
          "ns:RequestDate": "${_i18n.current_date}"
        }
      }
    }
  },
  "responseMapping": {
    "data": {
      "ifu_company_name": "//ns:CompanyName",
      "ifu_status": "//ns:Status",
      "ifu_registration_date": "//ns:RegistrationDate"
    },
    "metaData": {
      "ifu_verified": "//ns:IsValid",
      "ifu_verification_message": "//ns:Message"
    }
  }
}
```

**Workflow:**
```json
"verify-company-ifu": {
  "type": "uxp",
  "uxpFile": "verify-ifu.soap",
  "exceptions": {
    "CONNECTION_FAILED": "ifu-api-unavailable",
    "CALL_FAILED": "ifu-verification-failed",
    "INVALID_RESPONSE": "ifu-api-error"
  },
  "transitions": {
    "type": "map-by-meta",
    "key": "ifu_verified",
    "mapping": {
      "true": "payment-stage",
      "false": "ifu-invalid"
    }
  }
}
```

---

### Exemple 4: Vérifier éligibilité (REST avancé)

**Fichier:** `uxp/check-eligibility.json`

```json
{
  "method": "POST",
  "url": "https://api.gouv.bj/eligibility/check",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer ${env.ELIGIBILITY_API_TOKEN}",
    "X-Request-ID": "${metaData.request_id}",
    "X-Service-ID": "${serviceId}"
  },
  "body": {
    "citizen": {
      "npi": "${metaData.citizen.0}",
      "birth_date": "${data.person.birth_date}",
      "nationality": "${data.person.nationality}"
    },
    "service": {
      "id": "${serviceId}",
      "type": "license",
      "category": "${data.license.category}"
    },
    "requirements": {
      "minimum_age": 18,
      "required_documents": [
        "id_card",
        "proof_of_residence"
      ]
    }
  },
  "responseMapping": {
    "data": {
      "eligibility_age": "/checks/age/value",
      "eligibility_documents": "/checks/documents/missing"
    },
    "metaData": {
      "is_eligible": "/eligible",
      "eligibility_message": "/message",
      "eligibility_age_check": "/checks/age/passed",
      "eligibility_docs_check": "/checks/documents/passed",
      "eligibility_nationality_check": "/checks/nationality/passed"
    }
  }
}
```

**Workflow:**
```json
"check-citizen-eligibility": {
  "type": "uxp",
  "uxpFile": "check-eligibility.json",
  "exceptions": {
    "CONNECTION_FAILED": "eligibility-api-down",
    "CALL_FAILED": "eligibility-check-failed",
    "INVALID_RESPONSE": "eligibility-api-error"
  },
  "transitions": {
    "type": "map-by-meta",
    "key": "is_eligible",
    "mapping": {
      "true": "citizen-input",
      "false": "not-eligible"
    }
  }
}
```

---

### Exemple 5: Appel avec array (REST)

**Fichier:** `uxp/validate-documents.json`

```json
{
  "method": "POST",
  "url": "https://api.gouv.bj/documents/validate",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer ${env.DOC_API_TOKEN}"
  },
  "body": {
    "request_id": "${metaData.request_id}",
    "documents": [
      {
        "type": "id_card",
        "number": "${data.person.id_card_number}",
        "issued_date": "${data.person.id_card_date}"
      },
      {
        "type": "proof_residence",
        "file_url": "${data.documents.proof_residence_url}"
      }
    ]
  },
  "responseMapping": {
    "metaData": {
      "all_documents_valid": "/all_valid",
      "id_card_status": "/documents/0/status",
      "proof_residence_status": "/documents/1/status"
    }
  }
}
```

---

## ⚠️ GESTION DES EXCEPTIONS

### Types d'exceptions

| Exception | Signification | Exemple |
|-----------|---------------|---------|
| `CONNECTION_FAILED` | Impossible de joindre l'API | Serveur down, timeout |
| `CALL_FAILED` | Appel réussi mais erreur métier | 404, 401, 500 |
| `INVALID_RESPONSE` | Réponse mal formée | JSON invalide, XML corrompu |
| `MISSING_EXPECTED_FIELDS` | Champs manquants dans réponse | Mapping impossible |

---

### Configuration dans workflow

```json
"call-external-api": {
  "type": "uxp",
  "uxpFile": "verify-data.json",
  "exceptions": {
    "CONNECTION_FAILED": "api-connection-error",
    "CALL_FAILED": "api-call-failed",
    "INVALID_RESPONSE": "api-invalid-response",
    "MISSING_EXPECTED_FIELDS": "api-missing-fields"
  },
  "transitions": {
    "type": "single",
    "nextStage": "process-result"
  }
}
```

---

### Stages d'erreur correspondants

```json
"api-connection-error": {
  "type": "email",
  "to": "admin@gouv.bj",
  "emailFile": "api-connection-failed.txt",
  "transitions": {
    "type": "single",
    "nextStage": "FAILED"
  }
},

"api-call-failed": {
  "type": "ui",
  "uiFile": "api-error-message.json",
  "permission": "public",
  "transitions": {
    "type": "single",
    "nextStage": "retry-or-cancel"
  }
},

"api-invalid-response": {
  "type": "notification",
  "to": "${metaData.admin_phone}",
  "notificationFile": "api-malformed-response.txt",
  "transitions": {
    "type": "single",
    "nextStage": "FAILED"
  }
}
```

---

## 🔐 SÉCURITÉ

### 1. Variables d'environnement

**✅ TOUJOURS utiliser `${env.xxx}`:**
```json
{
  "headers": {
    "Authorization": "Bearer ${env.API_TOKEN}"
  }
}
```

**❌ JAMAIS en dur:**
```json
{
  "headers": {
    "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6..."
  }
}
```

**⚠️ Variables env configurées par administrateurs système.**

---

### 2. Validation réponses

**Toujours mapper champs critiques:**
```json
{
  "responseMapping": {
    "metaData": {
      "api_status": "/status",
      "api_error_message": "/error/message",
      "api_timestamp": "/timestamp"
    }
  }
}
```

**Vérifier dans workflow:**
```json
"check-api-result": {
  "type": "main",
  "transitions": {
    "type": "map-by-meta",
    "key": "api_status",
    "mapping": {
      "success": "continue-process",
      "error": "handle-api-error"
    }
  }
}
```

---

### 3. Timeout et retry

**⚠️ Pas configurable dans UXP JSON.**

**Géré au niveau infrastructure.**

**Bonnes pratiques workflow:**
```json
"call-api-with-retry": {
  "type": "uxp",
  "uxpFile": "api-call.json",
  "exceptions": {
    "CONNECTION_FAILED": "retry-api-call"
  }
},

"retry-api-call": {
  "type": "hardcoded-data",
  "writeData": {
    "metaData": {
      "retry_count": "${metaData.retry_count + 1}"
    }
  },
  "transitions": {
    "type": "map-by-meta",
    "key": "retry_count",
    "mapping": {
      "1": "call-api-with-retry",
      "2": "call-api-with-retry",
      "3": "max-retries-reached"
    }
  }
}
```

---

### 4. Données sensibles

**❌ Ne pas logger:**
```json
{
  "body": {
    "password": "${data.user.password}",
    "ssn": "${data.person.ssn}"
  }
}
```

**✅ Préférer tokens temporaires:**
```json
{
  "body": {
    "session_token": "${metaData.temp_session_token}"
  }
}
```

---

## ✅ BONNES PRATIQUES

### 1. Nommage fichiers

**✅ Noms explicites:**
```
verify-npi.json
search-company-rccm.json
check-eligibility.json
validate-ifu.soap
get-citizen-data.json
```

**❌ Noms génériques:**
```
api1.json
call.json
request.json
```

---

### 2. Documentation inline

**Ajouter README.md dans uxp/:**

```markdown
# UXP Integrations

## verify-npi.json
Vérifie le NPI du citoyen via l'API nationale.
- **Endpoint:** POST /npi/verify
- **Auth:** Bearer token
- **Timeout:** 10s

## search-company.json
Recherche entreprise dans RCCM.
- **Endpoint:** GET /rccm/search
- **Auth:** API Key
- **Response:** JSON

## verify-ifu.soap
Vérification IFU via service SOAP impôts.
- **Endpoint:** SOAP /VerificationService
- **Auth:** Username/Password
- **Response:** XML
```

---

### 3. Gestion erreurs exhaustive

**Mapper tous les cas:**
```json
{
  "exceptions": {
    "CONNECTION_FAILED": "api-down-error",
    "CALL_FAILED": "api-business-error",
    "INVALID_RESPONSE": "api-format-error",
    "MISSING_EXPECTED_FIELDS": "api-incomplete-response"
  }
}
```

**Créer stages d'erreur dédiés.**

---

### 4. Logging et traçabilité

**Inclure request_id:**
```json
{
  "headers": {
    "X-Request-ID": "${metaData.request_id}"
  },
  "body": {
    "trace_id": "${metaData.request_id}"
  }
}
```

**Mapper timestamps:**
```json
{
  "responseMapping": {
    "metaData": {
      "api_called_at": "${_i18n.current_datetime}",
      "api_response_timestamp": "/timestamp"
    }
  }
}
```

---

### 5. Tester endpoints

**Avant mise en prod:**

1. **Tester avec Postman/Insomnia**
2. **Vérifier structure réponse**
3. **Valider JSON Pointers/XPath**
4. **Tester cas d'erreur**
5. **Vérifier timeout**

---

## 🧪 TESTS ET DÉBOGAGE

### 1. Tester JSON Pointer

**Réponse API:**
```json
{
  "data": {
    "user": {
      "name": "John Doe",
      "age": 30
    }
  }
}
```

**Mapping:**
```json
{
  "responseMapping": {
    "data": {
      "user_name": "/data/user/name",
      "user_age": "/data/user/age"
    }
  }
}
```

**Vérifier:**
- `/data/user/name` → "John Doe" ✅
- `/user/name` → ERROR (manque `/data`) ❌
- `/data/user` → { "name": "John Doe", "age": 30 } ✅

---

### 2. Tester XPath (SOAP)

**Réponse XML:**
```xml
<soap:Envelope xmlns:soap="..." xmlns:ns="...">
  <soap:Body>
    <ns:Response>
      <ns:User>
        <ns:Name>John Doe</ns:Name>
      </ns:User>
    </ns:Response>
  </soap:Body>
</soap:Envelope>
```

**Mapping:**
```json
{
  "responseMapping": {
    "data": {
      "user_name": "//ns:Name"
    }
  }
}
```

**Vérifier:**
- `//ns:Name` → "John Doe" ✅
- `//Name` → ERROR (manque namespace) ❌
- `//ns:User/ns:Name` → "John Doe" ✅

---

### 3. Logs erreurs

**CONNECTION_FAILED:**
```
ERROR: Cannot connect to https://api.example.com
Reason: Connection timeout after 30s
```

**CALL_FAILED:**
```
ERROR: API returned status 404
Response: {"error": "Company not found"}
```

**INVALID_RESPONSE:**
```
ERROR: Response is not valid JSON
Body: <html>Error 500</html>
```

**MISSING_EXPECTED_FIELDS:**
```
ERROR: Field "/data/user/name" not found in response
Response: {"data": {"user": {"email": "..."}}}
```

---

## 📋 CHECKLIST UXP

**Avant de créer un fichier UXP:**

- [ ] Extension `.json` (REST) ou `.soap` (SOAP)
- [ ] `method` défini correctement
- [ ] `url` complète et testée
- [ ] `headers` avec authentification
- [ ] Variables `${env.xxx}` pour secrets
- [ ] `body` avec variables dynamiques
- [ ] `responseMapping` avec chemins valides
- [ ] JSON Pointer (REST) ou XPath (SOAP) testés
- [ ] Tous les cas d'exception mappés
- [ ] Stages d'erreur créés dans workflow
- [ ] Documentation dans README.md

**Tests:**

- [ ] Appel réussi en dev
- [ ] Réponse correctement mappée
- [ ] Cas d'erreur gérés (404, 500, timeout)
- [ ] Variables substituées correctement
- [ ] Authentification fonctionne
- [ ] Timeout acceptable (<30s)

---

## 🎯 RÉCAPITULATIF

### REST vs SOAP

| Critère | REST | SOAP |
|---------|------|------|
| **Format** | JSON | XML |
| **Méthodes** | GET, POST, PUT, DELETE | SOAP |
| **Headers** | `headers` object | Inclus dans XML |
| **Body** | JSON object | SOAP Envelope |
| **Mapping** | JSON Pointer (`/path/to/field`) | XPath (`//ns:Field`) |
| **Auth** | Bearer, API Key | Username/Password, Token |
| **Namespaces** | ❌ Non | ✅ Oui |

---

### Variables disponibles

| Source | Syntaxe | Exemple |
|--------|---------|---------|
| User data | `${data.xxx}` | `${data.person.name}` |
| Metadata | `${metaData.xxx}` | `${metaData.request_id}` |
| Environment | `${env.xxx}` | `${env.API_TOKEN}` |
| System | `${_i18n.xxx}` | `${_i18n.current_date}` |

---

### Exceptions UXP

| Exception | Quand | Action |
|-----------|-------|--------|
| `CONNECTION_FAILED` | API injoignable | Notifier admin |
| `CALL_FAILED` | Erreur métier (404, 500) | Afficher message user |
| `INVALID_RESPONSE` | JSON/XML invalide | Log erreur |
| `MISSING_EXPECTED_FIELDS` | Champ absent | Vérifier mapping |

---

**Dernière mise à jour:** 24 octobre 2025
**Voir aussi:**
- [02-SERVICECONF-PART2.md](02-SERVICECONF-PART2.md)
- [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md)
- [03-UI-CONFIGURATION-PART1.md](03-UI-CONFIGURATION-PART1.md)
