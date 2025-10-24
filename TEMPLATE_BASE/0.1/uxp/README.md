# UXP Templates

Ce dossier contient les templates pour les intégrations UXP (Unified eXchange Platform).

## Types de templates

### 1. REST/JSON (*.json)

Format JSON pour requêtes HTTP REST.

**Structure:**
```json
{
  "method": "POST|GET|PUT|DELETE",
  "url": "https://...",
  "headers": { ... },
  "body": { ... }
}
```

**Variables supportées:**
- `${data.field}`: accès aux données dans `data`
- `${metaData.field}`: accès aux données dans `metaData`
- `${serviceId}`: ID du service
- `${serviceVersion}`: version du service

**Exemple: get-company.json**

### 2. SOAP/XML (*.xml)

Format XML pour requêtes SOAP (X-Road).

**Namespaces courants:**
- `soapenv`: http://schemas.xmlsoap.org/soap/envelope/
- `wsdl`: http://x-road.eu/xsd/connector/service/v1
- `iden`: http://x-road.eu/xsd/identifiers

**Variables supportées:** mêmes que REST/JSON.

**Exemple: check-eligibility.xml**

## Mapping des réponses

Le mapping se fait dans le fichier serviceconf.json :

### REST (responseMapping)

```json
"responseMapping": {
  "data.company.name": {
    "pointer": "/company/name",
    "required": true
  }
}
```

### SOAP (responseMap avec XPath)

```json
"template": {
  "namespaces": {
    "soapenv": "http://schemas.xmlsoap.org/soap/envelope/",
    "bj": "http://gouv.bj/schemas/eligibility/v1"
  },
  "responseMap": {
    "metaData.eligible": {
      "xpath": "/soapenv:Envelope/soapenv:Body/bj:checkResponse/bj:eligible",
      "required": true
    }
  }
}
```

## Bonnes pratiques

1. **Toujours inclure les headers requis** (Content-Type, Accept, API keys)
2. **Utiliser des variables** pour rendre les templates dynamiques
3. **Marquer les champs requis** avec `"required": true` dans le mapping
4. **Tester avec différentes données** pour valider le mapping
5. **Documenter les APIs externes** (URL, authentification, format réponse)

## Sécurité

⚠️ **NE JAMAIS stocker de secrets en dur dans les templates!**

- API keys: utiliser des variables d'environnement
- Tokens: générer dynamiquement
- Credentials: stocker dans configuration sécurisée

## Voir aussi

- Documentation complète: `bj-citizen-portal-uxp-integration.adoc`
- Exemples de services: PS00565, PS01259, PS01333
