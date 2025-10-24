# 🎯 TEMPLATE_BASE - Template Exhaustif E-Service Portail Citoyen

Ce repository contient **le template le plus complet** pour créer rapidement n'importe quel e-service sur le Portail Citoyen Béninois.

---

## 📦 CONTENU

### Fichiers de référence (racine)
- **TEMPLATE_BASE.EXHAUSTIF.jsonc** : Référence complète avec TOUS les types de stages (15 types) documentés
- **TEMPLATE_BASE.serviceconf.json** : Configuration minimale fonctionnelle (workflow de base)
- **README.md** : Guide complet d'utilisation (vous êtes ici)

### Structure complète (0.1/)
```
TEMPLATE_BASE/
├── 0.1/
│   ├── TEMPLATE_BASE.serviceconf.json      ⭐ Config principale (minimal)
│   ├── TEMPLATE_BASE.EXHAUSTIF.jsonc       📚 Référence exhaustive
│   ├── README.md                            📖 Guide détaillé
│   │
│   ├── ui/                                  🎨 Formulaires utilisateur
│   │   ├── citizen-input.json              ✅ Exemple complet
│   │   └── official-review.json            ✅ Avec branchement
│   │
│   ├── email/                               📧 Templates email
│   │   ├── citizen-approved.txt            ✅ Notification approbation
│   │   ├── citizen-rejected.txt            ✅ Notification rejet
│   │   ├── citizen-changes-requested.txt   ✅ Demande modifications
│   │   ├── official-notification.txt       ✅ Notification officiel
│   │   └── official-emails.txt             ✅ Liste destinataires
│   │
│   ├── notification/                        📱 Templates SMS
│   │   ├── citizen-approved.txt            ✅ SMS approbation
│   │   └── citizen-rejected.txt            ✅ SMS rejet
│   │
│   ├── pdf/                                 📄 Templates PDF
│   │   ├── header-template.html            ✅ En-tête
│   │   ├── footer-template.html            ✅ Pied de page
│   │   └── certificate-template.html       ✅ Corps certificat
│   │
│   └── uxp/                                 🔗 Intégrations UXP
│       ├── get-company.json                ✅ Exemple REST
│       ├── check-eligibility.xml           ✅ Exemple SOAP
│       └── README.md                        📖 Guide UXP
```

---

## 🚀 DÉMARRAGE RAPIDE

### 1. Copier le template

```bash
# Copier tout le dossier TEMPLATE_BASE
cp -r TEMPLATE_BASE/ PS<VOTRE_ID>/

# Se positionner dans le nouveau service
cd PS<VOTRE_ID>/0.1/

# Renommer le fichier principal
mv TEMPLATE_BASE.serviceconf.json PS<VOTRE_ID>.serviceconf.json
```

### 2. Modifier la configuration de base

Éditer `PS<VOTRE_ID>.serviceconf.json` :

```json
{
  "serviceId": "PS<VOTRE_ID>",     // ⚠️ Changer ici
  "serviceVersion": "0.1",
  "public": true,
  "stages": {
    "start": { ... },
    // Adapter selon votre workflow
  }
}
```

### 3. Consulter la référence exhaustive

Pour ajouter des stages supplémentaires :

1. Ouvrir `TEMPLATE_BASE.EXHAUSTIF.jsonc`
2. Chercher le type de stage voulu (15 types disponibles)
3. Copier/coller dans votre `PS<VOTRE_ID>.serviceconf.json`
4. Adapter les valeurs

### 4. Créer vos templates

Selon les stages utilisés :

- **UI stages** → créer fichiers dans `ui/`
- **Email stages** → créer fichiers dans `email/`
- **SMS stages** → créer fichiers dans `notification/`
- **PDF stages** → créer fichiers dans `pdf/`
- **UXP stages** → créer fichiers dans `uxp/`

---

## 📚 TYPES DE STAGES DISPONIBLES

### Obligatoires
| Type | Description | Exemple fichier |
|------|-------------|-----------------|
| `start` | Point de départ (1 seul par service) | Ligne 43 EXHAUSTIF.jsonc |
| `main` | État majeur du workflow | Ligne 167 EXHAUSTIF.jsonc |

### Manuels (interaction utilisateur)
| Type | Description | Exemple fichier |
|------|-------------|-----------------|
| `ui` | Formulaire de saisie | Ligne 226 EXHAUSTIF.jsonc |
| `pay` | Paiement KKiaPay | Ligne 773 EXHAUSTIF.jsonc |

### Automatiques
| Type | Description | Exemple fichier |
|------|-------------|-----------------|
| `uxp-rest` | Intégration REST/JSON | Ligne 386 EXHAUSTIF.jsonc |
| `uxp` | Intégration SOAP/XML | Ligne 464 EXHAUSTIF.jsonc |
| `gen-document` | Génération PDF | Ligne 544 EXHAUSTIF.jsonc |
| `email` | Envoi email | Ligne 622 EXHAUSTIF.jsonc |
| `notification` | Envoi SMS | Ligne 730 EXHAUSTIF.jsonc |
| `person-data` | Récup données citoyen | Ligne 831 EXHAUSTIF.jsonc |
| `hardcoded-data` | Injection données fixes | Ligne 919 EXHAUSTIF.jsonc |
| `blacklist` | Suppression champs | Ligne 971 EXHAUSTIF.jsonc |
| `alert` | Message utilisateur | Ligne 1021 EXHAUSTIF.jsonc |
| `share-files` | Partage ZIP | Ligne 1122 EXHAUSTIF.jsonc |
| `share-one-pdf` | Partage PDF unique | Ligne 1168 EXHAUSTIF.jsonc |

---

## 🎨 EXEMPLES DE WORKFLOWS

### Workflow 1: Simple (citoyen → officiel → certificat)

```
start → citizen-input → REQUESTED → official-review → gen-certificate → APPROVED
                                                     → REJECTED
```

**Fichiers nécessaires:**
- `ui/citizen-input.json`
- `ui/official-review.json`
- `pdf/certificate-template.html`

### Workflow 2: Avec UXP (vérification externe)

```
start → get-citizen-data → uxp-check → citizen-input → payment → gen-document → APPROVED
                                     → REJECTED
```

**Fichiers nécessaires:**
- `uxp/check-eligibility.xml` ou `.json`
- `ui/citizen-input.json`
- `pdf/certificate-template.html`

### Workflow 3: Avec modifications (boucle)

```
start → citizen-input → REQUESTED → official-review → gen-certificate → APPROVED
                                                     → alert-changes → CHANGES-REQUESTED
                                                                      
CHANGES-REQUESTED → citizen-resubmit → REQUESTED
```

**Fichiers nécessaires:**
- `ui/citizen-input.json`
- `ui/official-review.json`
- `ui/citizen-resubmit.json`
- `email/citizen-changes-requested.txt`

---

## 📖 GUIDES DÉTAILLÉS

### Guide complet
👉 **Voir `0.1/README.md`** pour :
- Documentation exhaustive de chaque stage
- Exemples UI avec tous les types de champs
- Exemples email/SMS/PDF/UXP
- Patterns de workflow courants
- Pièges à éviter
- Checklist avant déploiement

### Guide UXP
👉 **Voir `0.1/uxp/README.md`** pour :
- Format REST/JSON
- Format SOAP/XML
- Mapping des réponses
- Bonnes pratiques

---

## ⚙️ CONFIGURATION

### serviceId
**Format:** `PSxxxxx` (ex: PS00565, PS01259)
- Doit correspondre au nom du dossier
- Doit être enregistré dans CatIS

### serviceVersion
**Format:** `x.y` (ex: 0.1, 0.2, 1.0)
- Correspond au sous-dossier de version
- Incrémenter pour nouvelles versions

### public
- `true`: service accessible sans authentification (citoyens)
- `false`: authentification requise

---

## 🔒 PERMISSIONS

### Types disponibles

#### 1. meta-array (tableau dans metaData)
```json
"permissions": {
  "type": "meta-array",
  "actor": "CITIZEN",
  "metaPathToArray": ["citizen"],
  "public": true
}
```
**Usage:** citoyens propriétaires (créateurs de la demande)

#### 2. hardcoded-array (liste fixe)
```json
"permissions": {
  "type": "hardcoded-array",
  "actor": "OFFICIAL",
  "array": ["ReviewGroup", "ManagerGroup"]
}
```
**Usage:** agents/officiels spécifiques

#### 3. public (tous)
```json
"permissions": {
  "type": "public"
}
```
**Usage:** rare, pour stages accessibles à tous

---

## 🎯 TRANSITIONS

### Type: single (destination unique)
```json
"transitions": {
  "type": "single",
  "nextStage": "next-stage-id"
}
```

### Type: map-by-meta (branchement conditionnel)
```json
"transitions": {
  "type": "map-by-meta",
  "metaPathToKey": ["official", "choice"],
  "map": {
    "approve": "gen-certificate",
    "reject": "REJECTED",
    "request-changes": "alert-changes"
  }
}
```

---

## 🛡️ GESTION D'ERREURS

### Exception mappable (onStageException)

```json
"transitions": {
  "type": "single",
  "nextStage": "success-stage",
  "onStageException": {
    "EXCEPTION_TYPE": "error-stage"
  }
}
```

### Exceptions courantes par stage

| Stage Type | Exceptions mappables |
|------------|---------------------|
| `ui` | REQUIRED_FIELDS_MISSING, FIELDS_NOT_ALLOWED |
| `uxp-rest` | PROCESSING_ERROR, MISSING_REQUIRED_NODE, CONNECTION_ERROR, WEB_APPLICATION_EXCEPTION |
| `uxp` | CONFIG_ERROR, CONNECTION_ERROR, DATA_ERROR, DATA_WARNING |
| `gen-document` | NO_TEMPLATE, API_EXCEPTION, CONNECTION_ERROR |
| `email` | NO_TARGET_CONFIGURED, NOTIFICATION_SERVICE_ERROR |
| `notification` | INVALID_PHONE_NUMBER, NOTIFICATION_SERVICE_ERROR |
| `person-data` | CANNOT_GET_NPI, CANNOT_FIND_PERSON |
| `share-files` | CANT_GENERATE_UNIQUE_CODE |

⚠️ **Stage `pay` (paiement) : AUCUNE EXCEPTION MAPPABLE**

---

## ⚠️ PIÈGES À ÉVITER

### 1. Shallow Merge
❌ UI et hardcoded-data font un **shallow merge** :
```javascript
// État initial
data.person = {name: "X", age: 30, city: "Cotonou"}

// UI envoie
data.person = {name: "Y"}

// Résultat
data.person = {name: "Y"}  // ❌ age et city PERDUS!
```

✅ **Solution:** Toujours renvoyer TOUS les champs d'un objet

### 2. Variables manquantes
❌ Template utilise `${data.person.firstname}`
✅ Mais le vrai champ est `data.person.first_name` (underscore!)

### 3. Boucles infinies
❌ Branchement map-by-meta qui reboucle sur lui-même
✅ Toujours vérifier que chaque branche a une sortie

### 4. Oubli flag "final"
❌ Stage terminal sans `"flags": ["final"]` → demande jamais terminée!
✅ TOUJOURS ajouter `"flags": ["final"]` aux stages finaux

---

## ✅ CHECKLIST AVANT DÉPLOIEMENT

- [ ] `serviceId` correspond au nom du dossier
- [ ] 1 seul stage `"type": "start"`
- [ ] Au moins 1 stage final avec `"flags": ["final"]`
- [ ] Tous les fichiers référencés existent (ui/, email/, pdf/, uxp/)
- [ ] Gestion `onStageException` pour stages critiques
- [ ] Permissions cohérentes
- [ ] Variables `${}` correspondent aux vrais chemins
- [ ] Aucune boucle infinie
- [ ] Mode sandbox activé pour paiements (si applicable)

---

## 📚 DOCUMENTATION OFFICIELLE

- **Guide développement:** `bj-citizen-portal-service-development.md` (2466 lignes)
- **Spécifications UI:** `bj-citizen-portal-service-ui-spec-complet.md` (1059 lignes)
- **Intégration UXP:** `bj-citizen-portal-uxp-integration.adoc`
- **Génération PDF:** `bj-citizen-portal-pdf-generation.adoc`

---

## 📊 SERVICES EXISTANTS (pour référence)

Exemples complets dans ce workspace :
- **PS00565** : Service standard avec workflow complet
- **PS01259** : Utilise UXP-REST
- **PS01333** : Workflow avec paiement
- **PS00906** : Exemple génération PDF complexe
- **PS01056** : Exemple avec modifications demandées

---

## 🆘 SUPPORT

1. **Consulter README.md dans 0.1/** (guide exhaustif)
2. **Examiner TEMPLATE_BASE.EXHAUSTIF.jsonc** (tous les exemples)
3. **Analyser services existants** (PS00565, PS01259...)
4. **Contacter équipe technique** Portail Citoyen

---

## 🎓 FORMATION

### Niveau 1 : Workflow simple
1. Copier template
2. Modifier start + 1 ui + 1 main (REQUESTED) + 1 main final (APPROVED)
3. Créer 1 fichier ui/
4. Tester

### Niveau 2 : Branchement
1. Ajouter ui avec map-by-meta
2. Créer plusieurs chemins (approve/reject)
3. Ajouter alerts pour erreurs

### Niveau 3 : Intégrations
1. Ajouter stages uxp-rest ou uxp
2. Créer templates dans uxp/
3. Mapper réponses

### Niveau 4 : Complet
1. Combiner UI + UXP + PDF + Email + SMS
2. Gérer toutes les exceptions
3. Optimiser workflow

---

**Bon développement ! 🚀**

*Template créé par analyse exhaustive de 48+ services existants et aligné strictement avec la documentation officielle.*
