# 📁 STRUCTURE DE PROJET E-SERVICE

**Documentation exhaustive de l'organisation des fichiers et dossiers d'un e-service.**

---

## 🎯 STRUCTURE COMPLÈTE

```
PSxxxxx/                                    # Dossier racine du service
│
├── 0.1/                                    # Version 0.1
│   │
│   ├── PSxxxxx.serviceconf.json           # ⭐ FICHIER PRINCIPAL
│   │                                       # Configuration complète du workflow
│   │                                       # Voir: 02-SERVICECONF.md
│   │
│   ├── ui/                                 # 🎨 FORMULAIRES UTILISATEUR
│   │   ├── citizen-input.json             # Formulaire citoyen
│   │   ├── official-review.json           # Formulaire agent
│   │   ├── citizen-resubmit.json          # Formulaire modifications
│   │   └── ...                            # Autres formulaires
│   │                                       # Voir: 03-UI-CONFIGURATION.md
│   │
│   ├── email/                              # 📧 TEMPLATES EMAIL
│   │   ├── citizen-approved.txt           # Email approbation citoyen
│   │   ├── citizen-rejected.txt           # Email rejet
│   │   ├── official-notification.txt      # Email notification agent
│   │   ├── official-emails.txt            # Liste emails agents
│   │   └── ...                            # Autres emails
│   │                                       # Voir: 04-EMAIL-TEMPLATES.md
│   │
│   ├── notification/                       # 📱 TEMPLATES SMS
│   │   ├── citizen-approved.txt           # SMS approbation
│   │   ├── citizen-rejected.txt           # SMS rejet
│   │   └── ...                            # Autres SMS
│   │                                       # Voir: 05-NOTIFICATION-TEMPLATES.md
│   │
│   ├── pdf/                                # 📄 TEMPLATES PDF
│   │   ├── header-template.html           # En-tête (toutes pages)
│   │   ├── footer-template.html           # Pied de page (toutes pages)
│   │   ├── certificate-template.html      # Corps certificat
│   │   ├── receipt-template.html          # Reçu
│   │   └── ...                            # Autres PDF
│   │                                       # Voir: 06-PDF-TEMPLATES.md
│   │
│   └── uxp/                                # 🔗 INTÉGRATIONS UXP
│       ├── get-company.json               # Requête REST
│       ├── check-eligibility.xml          # Requête SOAP
│       └── ...                            # Autres intégrations
│                                           # Voir: 07-UXP-INTEGRATION.md
│
├── 0.2/                                    # Version 0.2 (si applicable)
│   └── ... (même structure que 0.1)
│
└── 1.0/                                    # Version 1.0 (si applicable)
    └── ... (même structure que 0.1)
```

---

## 📋 RÈGLES DE NOMMAGE

### 1. Dossier racine du service

**Format:** `PSxxxxx` où `xxxxx` = numéro à 5 chiffres

**Exemples valides:**
- ✅ `PS00565`
- ✅ `PS01259`
- ✅ `PS00906`

**Exemples invalides:**
- ❌ `ps00565` (minuscules)
- ❌ `PS565` (moins de 5 chiffres)
- ❌ `PS-00565` (tiret)

**⚠️ IMPORTANT:**
- Le nom du dossier DOIT correspondre au `serviceId` dans le fichier `.serviceconf.json`
- Le `serviceId` DOIT être enregistré dans **CatIS** (Catalogue of Interoperable Solutions)

---

### 2. Dossier de version

**Format:** `x.y` (version sémantique simplifiée)

**Exemples:**
- `0.1` - Première version en développement
- `0.2` - Deuxième itération (corrections/améliorations)
- `1.0` - Version stable/production
- `2.0` - Version majeure

**Règles:**
- Chaque version = dossier séparé
- Permet versioning incrémental
- Migrations gérées par le moteur du portail

---

### 3. Fichier serviceconf.json

**Format:** `PSxxxxx.serviceconf.json`

**Règles strictes:**
- ✅ Nom = `serviceId` + `.serviceconf.json`
- ✅ Exemple: si `serviceId = "PS00565"` → fichier = `PS00565.serviceconf.json`
- ❌ PAS d'autres noms acceptés
- ❌ PAS d'extension différente

**Emplacement:**
```
PSxxxxx/
└── x.y/
    └── PSxxxxx.serviceconf.json    ← ICI
```

---

### 4. Fichiers UI

**Format:** `nom-descriptif.json` (kebab-case recommandé)

**Exemples:**
- ✅ `citizen-input.json`
- ✅ `official-review.json`
- ✅ `company-info-form.json`
- ⚠️ `CitizenInput.json` (fonctionne mais non recommandé)

**Emplacement:**
```
PSxxxxx/x.y/ui/nom-descriptif.json
```

**Référence dans serviceconf.json:**
```json
{
  "type": "ui",
  "uiConfiguration": "citizen-input.json"
}
```

---

### 5. Fichiers Email

**Deux types de fichiers:**

#### A. Templates email (`.txt`)
**Format:** `nom-descriptif.txt`

**Exemples:**
- `citizen-approved.txt`
- `official-notification.txt`
- `admin-alert.txt`

**Emplacement:**
```
PSxxxxx/x.y/email/nom-descriptif.txt
```

**Référence dans serviceconf.json:**
```json
{
  "type": "email",
  "bodyTemplatePath": "citizen-approved.txt"
}
```

#### B. Listes d'emails (`.txt`)
**Format:** `nom-liste.txt` (1 email par ligne)

**Exemple:** `official-emails.txt`
```
admin@ministry.bj
reviewer1@ministry.bj
reviewer2@ministry.bj
```

**Référence dans serviceconf.json:**
```json
{
  "type": "email",
  "targetType": "email-file",
  "targetsFile": "official-emails.txt"
}
```

---

### 6. Fichiers Notification (SMS)

**Format:** `nom-descriptif.txt`

**Exemples:**
- `citizen-approved.txt`
- `reminder.txt`
- `urgent-alert.txt`

**Emplacement:**
```
PSxxxxx/x.y/notification/nom-descriptif.txt
```

**Référence dans serviceconf.json:**
```json
{
  "type": "notification",
  "template": "citizen-approved.txt"
}
```

---

### 7. Fichiers PDF

**Trois types de templates:**

#### A. Header (`.html`)
**Nommage:** `header-{template-name}.html`

**Exemples:**
- `header-certificate.html`
- `header-receipt.html`
- `header-template.html` (générique)

#### B. Footer (`.html`)
**Nommage:** `footer-{template-name}.html`

**Exemples:**
- `footer-certificate.html`
- `footer-receipt.html`
- `footer-template.html` (générique)

#### C. Body (`.html`)
**Nommage:** `{template-name}.html`

**Exemples:**
- `certificate.html`
- `receipt.html`
- `certificate-template.html`

**Emplacement:**
```
PSxxxxx/x.y/pdf/
├── header-certificate.html
├── footer-certificate.html
└── certificate.html
```

**⚠️ RÈGLE DE RECHERCHE AUTOMATIQUE:**

Si dans serviceconf.json vous avez:
```json
{
  "type": "gen-document",
  "template": "certificate.html"
}
```

Le moteur cherche automatiquement:
1. `pdf/certificate.html` (body - OBLIGATOIRE)
2. `pdf/header-certificate.html` (si existe - OPTIONNEL)
3. `pdf/footer-certificate.html` (si existe - OPTIONNEL)

**Fallback:**
- Si `header-certificate.html` absent → cherche `header-template.html`
- Si `footer-certificate.html` absent → cherche `footer-template.html`

---

### 8. Fichiers UXP

**Deux formats:**

#### A. REST/JSON (`.json`)
**Format:** `nom-descriptif.json`

**Exemples:**
- `get-company.json`
- `check-person.json`
- `verify-eligibility.json`

**Emplacement:**
```
PSxxxxx/x.y/uxp/get-company.json
```

**Référence dans serviceconf.json:**
```json
{
  "type": "uxp-rest",
  "templateFile": "get-company.json"
}
```

#### B. SOAP/XML (`.xml`)
**Format:** `nom-descriptif.xml`

**Exemples:**
- `check-eligibility.xml`
- `get-citizen-data.xml`
- `verify-company.xml`

**Emplacement:**
```
PSxxxxx/x.y/uxp/check-eligibility.xml
```

**Référence dans serviceconf.json:**
```json
{
  "type": "uxp",
  "template": {
    "templateFile": "check-eligibility.xml"
  }
}
```

---

## 🔄 WORKFLOW DE DÉPLOIEMENT

### 1. Développement local

```bash
# Structure de travail
/workspace/
└── PSxxxxx/
    └── 0.1/
        ├── PSxxxxx.serviceconf.json
        ├── ui/...
        ├── email/...
        ├── notification/...
        ├── pdf/...
        └── uxp/...
```

### 2. Déploiement sur serveur

**Chemin de déploiement:**
```
/etc/citizen-portal/templates/PSxxxxx/0.1/...
```

**Commande de déploiement:**
```bash
# Copier tout le dossier
sudo cp -r PSxxxxx/ /etc/citizen-portal/templates/

# Vérifier permissions
sudo chown -R portal:portal /etc/citizen-portal/templates/PSxxxxx/
sudo chmod -R 755 /etc/citizen-portal/templates/PSxxxxx/
```

### 3. Activation du service

Enregistrer dans **CatIS** avec:
- `serviceId`: PSxxxxx
- `serviceVersion`: 0.1
- Métadonnées descriptives

---

## 📦 VERSIONING

### Stratégie de versioning

**Version 0.x - Développement/Test:**
- `0.1` : Première version de développement
- `0.2` : Corrections après tests
- `0.3` : Améliorations fonctionnelles
- ...

**Version 1.x - Production:**
- `1.0` : Première version stable
- `1.1` : Corrections mineures
- `1.2` : Améliorations mineures

**Version 2.x - Évolution majeure:**
- `2.0` : Refonte complète du workflow
- `2.1` : Ajustements

### Migration entre versions

**Scénario 1: Nouvelle version (0.1 → 0.2)**

```bash
# 1. Copier la version précédente
cp -r PS00565/0.1/ PS00565/0.2/

# 2. Modifier serviceVersion dans 0.2/PS00565.serviceconf.json
vim PS00565/0.2/PS00565.serviceconf.json
# Changer "serviceVersion": "0.1" → "serviceVersion": "0.2"

# 3. Effectuer les modifications nécessaires

# 4. Déployer
sudo cp -r PS00565/0.2/ /etc/citizen-portal/templates/PS00565/
```

**Scénario 2: Gestion des demandes en cours**

Le moteur du portail gère automatiquement:
- Demandes créées en v0.1 → continuent en v0.1
- Nouvelles demandes → utilisent v0.2

**⚠️ JAMAIS modifier une version déjà en production!**
- ❌ Modifier `0.1/` si des demandes actives existent
- ✅ Créer `0.2/` avec les changements

---

## 🔒 PERMISSIONS ET SÉCURITÉ

### Permissions recommandées

```bash
# Dossiers
chmod 755 PSxxxxx/
chmod 755 PSxxxxx/0.1/
chmod 755 PSxxxxx/0.1/{ui,email,notification,pdf,uxp}/

# Fichiers
chmod 644 PSxxxxx/0.1/PSxxxxx.serviceconf.json
chmod 644 PSxxxxx/0.1/ui/*.json
chmod 644 PSxxxxx/0.1/email/*.txt
chmod 644 PSxxxxx/0.1/notification/*.txt
chmod 644 PSxxxxx/0.1/pdf/*.html
chmod 644 PSxxxxx/0.1/uxp/*.{json,xml}
```

### Propriétaire

```bash
# Sur le serveur de production
chown -R portal:portal /etc/citizen-portal/templates/PSxxxxx/
```

---

## ✅ CHECKLIST STRUCTURE

Avant déploiement, vérifier:

- [ ] Nom dossier racine = `PSxxxxx` (5 chiffres)
- [ ] Dossier version = `x.y` (format valide)
- [ ] Fichier principal = `PSxxxxx.serviceconf.json`
- [ ] `serviceId` dans JSON = nom du dossier racine
- [ ] `serviceVersion` dans JSON = nom du dossier version
- [ ] Tous les fichiers référencés existent
- [ ] Nommage cohérent (kebab-case recommandé)
- [ ] Pas de fichiers inutiles/temporaires
- [ ] Permissions correctes (755/644)
- [ ] Encodage UTF-8 pour tous les fichiers texte

---

## 🚨 ERREURS FRÉQUENTES

### 1. Nom de fichier serviceconf incorrect

❌ **Erreur:**
```
PS00565/0.1/service.json
```

✅ **Correct:**
```
PS00565/0.1/PS00565.serviceconf.json
```

---

### 2. serviceId ne correspond pas au dossier

❌ **Erreur:**
```
Dossier: PS00565/
Fichier: PS00565.serviceconf.json
Contenu: "serviceId": "PS00566"  ← ERREUR!
```

✅ **Correct:**
```
Dossier: PS00565/
Fichier: PS00565.serviceconf.json
Contenu: "serviceId": "PS00565"  ← OK!
```

---

### 3. Fichier référencé inexistant

❌ **Erreur dans serviceconf.json:**
```json
{
  "type": "ui",
  "uiConfiguration": "citizen-form.json"
}
```

Mais fichier réel: `ui/citizen-input.json` ← ERREUR!

✅ **Correct:**
```json
{
  "type": "ui",
  "uiConfiguration": "citizen-input.json"
}
```

Et fichier existe: `ui/citizen-input.json` ← OK!

---

### 4. Header/Footer PDF non trouvés

❌ **Erreur:**
```
pdf/
├── certificate.html
└── header.html              ← Nom générique
```

Template référencé: `certificate.html`
→ Cherche `header-certificate.html` (absent!)
→ Cherche `header-template.html` (absent!)
→ Pas de header appliqué

✅ **Solution 1 (spécifique):**
```
pdf/
├── certificate.html
├── header-certificate.html  ← OK!
└── footer-certificate.html
```

✅ **Solution 2 (générique):**
```
pdf/
├── certificate.html
├── header-template.html     ← Fallback OK!
└── footer-template.html
```

---

## 📚 VOIR AUSSI

- **Configuration principale:** [02-SERVICECONF.md](02-SERVICECONF.md)
- **Formulaires UI:** [03-UI-CONFIGURATION.md](03-UI-CONFIGURATION.md)
- **Templates Email:** [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md)
- **Templates SMS:** [05-NOTIFICATION-TEMPLATES.md](05-NOTIFICATION-TEMPLATES.md)
- **Templates PDF:** [06-PDF-TEMPLATES.md](06-PDF-TEMPLATES.md)
- **Intégrations UXP:** [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md)

---

**Dernière mise à jour:** 24 octobre 2025
