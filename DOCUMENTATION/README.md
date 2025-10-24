# 📚 DOCUMENTATION COMPLÈTE - INDEX

**Guide de navigation dans la documentation exhaustive du template e-service.**

---

## 🎯 OBJECTIF

Cette documentation vous permet de **créer et modifier des e-services rapidement** en suivant les règles et exemples pratiques fournis.

**Chaque fichier est autonome** : exemples, règles officielles, warnings, checklists.

---

## 📖 STRUCTURE DE LA DOCUMENTATION

### 1️⃣ **Structure et Organisation**
📄 [**01-STRUCTURE-PROJET.md**](01-STRUCTURE-PROJET.md) (~500 lignes)

**Contenu:**
- Structure complète des dossiers (`PSxxxxx/x.y/`)
- Règles de nommage strictes
- Organisation des fichiers (ui/, email/, pdf/, uxp/)
- Déploiement et versioning
- Permissions et accès
- Erreurs courantes

**Quand le consulter:**
- ✅ Création d'un nouveau service
- ✅ Questions sur arborescence
- ✅ Problèmes de déploiement
- ✅ Erreurs de nommage

---

### 2️⃣ **Configuration Workflow**
📄 [**02-SERVICECONF-PART1.md**](02-SERVICECONF-PART1.md) (~2000 lignes)  
📄 [**02-SERVICECONF-PART2.md**](02-SERVICECONF-PART2.md) (~1800 lignes)

**Part 1 - Fondamentaux:**
- Structure racine (serviceId, serviceVersion, public, backOffice)
- Stage `start` (point d'entrée)
- Stage `main` (orchestration)
- Stage `ui` (formulaires citoyens)
- Stage `pay` (paiements)

**Part 2 - Stages avancés:**
- `uxp` / `uxp-rest` (appels APIs)
- `gen-document` (génération PDF)
- `email` / `notification` (communications)
- `person-data` / `hardcoded-data` (manipulation données)
- `blacklist` / `alert` (contrôles)
- `share-files` / `share-one-pdf` (partage documents)

**Quand le consulter:**
- ✅ Création du workflow principal
- ✅ Ajout d'étapes (stages)
- ✅ Configuration transitions
- ✅ Gestion permissions
- ✅ Mapping exceptions

---

### 3️⃣ **Formulaires UI**
📄 [**03-UI-CONFIGURATION-PART1.md**](03-UI-CONFIGURATION-PART1.md) (~2200 lignes)  
📄 [**03-UI-CONFIGURATION-PART2.md**](03-UI-CONFIGURATION-PART2.md) (~2500 lignes)

**Part 1 - Bases:**
- Propriétés communes (id, label, path, helpText, required...)
- Champs simples: TEXT, TEXT_AREA, TEXT_EDITOR
- Champs spécialisés: NUMBER, DATE, BOOLEAN
- Upload: FILE_UPLOAD

**Part 2 - Fonctionnalités avancées:**
- Listes: RADIO_LIST, STATIC_RADIO_LIST
- Répétables: REPEATABLE (sous-formulaires)
- Affichage: MARKDOWN_DESCRIPTION
- Actions: SUBMIT
- 14 types de validations (MIN_LENGTH, REGEX, EMAIL, FILE_SIZE...)
- displayConditions (affichage conditionnel)

**Quand le consulter:**
- ✅ Création formulaires citoyens
- ✅ Ajout validations
- ✅ Conditions d'affichage
- ✅ Champs répétables
- ✅ Upload fichiers

---

### 4️⃣ **Templates Email**
📄 [**04-EMAIL-TEMPLATES.md**](04-EMAIL-TEMPLATES.md) (~1700 lignes)

**Contenu:**
- Format .txt avec variables `${}`
- Variables: data, metaData, _i18n
- 5 exemples complets (approbation, rejet, modifications...)
- Multilingue (fr/en)
- Bonnes pratiques (structure, tone, instructions)
- Checklist validation

**Quand le consulter:**
- ✅ Création emails automatiques
- ✅ Notifications citoyens/agents
- ✅ Messages d'erreur
- ✅ Support multilingue

---

### 5️⃣ **Templates SMS**
📄 [**05-NOTIFICATION-TEMPLATES.md**](05-NOTIFICATION-TEMPLATES.md) (~1900 lignes)

**Contenu:**
- Contraintes SMS (160 caractères, pas d'accents)
- Variables dynamiques
- 6 exemples optimisés
- Stratégies de concision
- Problèmes encodage
- URLs courtes
- Checklist SMS

**Quand le consulter:**
- ✅ Création SMS notifications
- ✅ Optimisation longueur
- ✅ Problèmes caractères spéciaux
- ✅ Multilingue SMS

---

### 6️⃣ **Templates PDF**
📄 [**06-PDF-TEMPLATES.md**](06-PDF-TEMPLATES.md) (~1800 lignes)

**Contenu:**
- Structure HTML/CSS
- Header/Footer automatiques
- Variables et substitution
- Images (base64 vs URL)
- Mise en page (@page, marges)
- 3 exemples complets (certificat, reçu, lettre)
- Styles professionnels
- Checklist PDF

**Quand le consulter:**
- ✅ Génération certificats
- ✅ Reçus de paiement
- ✅ Documents officiels
- ✅ Problèmes affichage PDF

---

### 7️⃣ **Intégrations UXP**
📄 [**07-UXP-INTEGRATION.md**](07-UXP-INTEGRATION.md) (~2100 lignes)

**Contenu:**
- Appels REST (JSON)
- Appels SOAP (XML)
- Authentification (Bearer, API Key, Basic)
- Mapping réponses (JSON Pointer, XPath)
- Gestion exceptions
- 5 exemples complets (NPI, RCCM, IFU...)
- Sécurité et bonnes pratiques
- Checklist UXP

**Quand le consulter:**
- ✅ Vérification données externes (NPI, RCCM, IFU)
- ✅ Appels APIs REST ou SOAP
- ✅ Mapping réponses complexes
- ✅ Gestion erreurs API

---

## 🚀 GUIDE RAPIDE PAR CAS D'USAGE

### Je veux créer un nouveau service
1. Lire [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md)
2. Copier `TEMPLATE_BASE/`
3. Configurer [02-SERVICECONF-PART1.md](02-SERVICECONF-PART1.md)

### Je veux ajouter un formulaire
1. Lire [03-UI-CONFIGURATION-PART1.md](03-UI-CONFIGURATION-PART1.md)
2. Ajouter champs avancés [03-UI-CONFIGURATION-PART2.md](03-UI-CONFIGURATION-PART2.md)
3. Créer `ui/my-form.json`

### Je veux envoyer un email
1. Lire [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md)
2. Créer `email/my-email.txt`
3. Ajouter stage `email` dans serviceconf

### Je veux envoyer un SMS
1. Lire [05-NOTIFICATION-TEMPLATES.md](05-NOTIFICATION-TEMPLATES.md)
2. Créer `notification/my-sms.txt` (≤160 chars)
3. Ajouter stage `notification`

### Je veux générer un PDF
1. Lire [06-PDF-TEMPLATES.md](06-PDF-TEMPLATES.md)
2. Créer `pdf/my-document.html`
3. Ajouter stage `gen-document`

### Je veux appeler une API externe
1. Lire [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md)
2. Créer `uxp/my-api-call.json` (REST) ou `.soap` (SOAP)
3. Ajouter stage `uxp`

---

## 🔍 RECHERCHE PAR MOT-CLÉ

| Je cherche... | Voir fichier |
|---------------|--------------|
| **Structure dossiers** | 01-STRUCTURE-PROJET.md |
| **Nommage PSxxxxx** | 01-STRUCTURE-PROJET.md |
| **serviceId, serviceVersion** | 02-SERVICECONF-PART1.md |
| **Transitions, permissions** | 02-SERVICECONF-PART1.md, PART2 |
| **Exceptions** | 02-SERVICECONF-PART2.md |
| **Champs TEXT, NUMBER, DATE** | 03-UI-CONFIGURATION-PART1.md |
| **REPEATABLE, RADIO_LIST** | 03-UI-CONFIGURATION-PART2.md |
| **Validations (REGEX, EMAIL...)** | 03-UI-CONFIGURATION-PART2.md |
| **displayConditions** | 03-UI-CONFIGURATION-PART2.md |
| **Templates email** | 04-EMAIL-TEMPLATES.md |
| **Templates SMS** | 05-NOTIFICATION-TEMPLATES.md |
| **Templates PDF/HTML** | 06-PDF-TEMPLATES.md |
| **Appels REST/SOAP** | 07-UXP-INTEGRATION.md |
| **JSON Pointer, XPath** | 07-UXP-INTEGRATION.md |
| **Variables ${data.xxx}** | Tous les fichiers (contexte) |
| **Multilingue** | 04-EMAIL, 05-NOTIFICATION |

---

## ✅ CHECKLISTS DISPONIBLES

Chaque fichier contient une **checklist de validation** complète en fin de document :

- ✅ **01-STRUCTURE:** Checklist structure/nommage
- ✅ **02-SERVICECONF:** Checklist workflow complet
- ✅ **03-UI:** Checklist formulaire UI
- ✅ **04-EMAIL:** Checklist template email
- ✅ **05-NOTIFICATION:** Checklist SMS
- ✅ **06-PDF:** Checklist template PDF
- ✅ **07-UXP:** Checklist intégration API

---

## 🎓 PARCOURS D'APPRENTISSAGE

### Niveau 1 - Débutant
1. [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md) - Comprendre l'organisation
2. [02-SERVICECONF-PART1.md](02-SERVICECONF-PART1.md) - Créer workflow basique
3. [03-UI-CONFIGURATION-PART1.md](03-UI-CONFIGURATION-PART1.md) - Formulaire simple

### Niveau 2 - Intermédiaire
4. [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md) - Notifications email
5. [05-NOTIFICATION-TEMPLATES.md](05-NOTIFICATION-TEMPLATES.md) - SMS
6. [02-SERVICECONF-PART2.md](02-SERVICECONF-PART2.md) - Stages avancés

### Niveau 3 - Avancé
7. [03-UI-CONFIGURATION-PART2.md](03-UI-CONFIGURATION-PART2.md) - UI complexe
8. [06-PDF-TEMPLATES.md](06-PDF-TEMPLATES.md) - Documents PDF
9. [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md) - Intégrations externes

---

## 📊 STATISTIQUES DOCUMENTATION

| Fichier | Lignes | Exemples | Warnings |
|---------|--------|----------|----------|
| 01-STRUCTURE | ~500 | 15+ | 10+ |
| 02-SERVICECONF-1 | ~2000 | 20+ | 15+ |
| 02-SERVICECONF-2 | ~1800 | 25+ | 12+ |
| 03-UI-1 | ~2200 | 30+ | 18+ |
| 03-UI-2 | ~2500 | 35+ | 20+ |
| 04-EMAIL | ~1700 | 5+ | 10+ |
| 05-NOTIFICATION | ~1900 | 6+ | 12+ |
| 06-PDF | ~1800 | 3+ | 8+ |
| 07-UXP | ~2100 | 5+ | 15+ |
| **TOTAL** | **~17000** | **140+** | **120+** |

---

## 🆘 DÉPANNAGE

### Problème de déploiement
→ [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md) - Section "Déploiement"

### Workflow ne démarre pas
→ [02-SERVICECONF-PART1.md](02-SERVICECONF-PART1.md) - Section "Stage start"

### Formulaire ne s'affiche pas
→ [03-UI-CONFIGURATION-PART1.md](03-UI-CONFIGURATION-PART1.md) - Section "Erreurs courantes"

### Email non envoyé
→ [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md) - Section "Erreurs courantes"

### SMS tronqué
→ [05-NOTIFICATION-TEMPLATES.md](05-NOTIFICATION-TEMPLATES.md) - Section "Contraintes SMS"

### PDF vide ou mal formaté
→ [06-PDF-TEMPLATES.md](06-PDF-TEMPLATES.md) - Section "Erreurs courantes"

### API retourne erreur
→ [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md) - Section "Gestion exceptions"

---

## 🔗 NAVIGATION RAPIDE

**Fichiers de documentation:**
1. [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md)
2. [02-SERVICECONF-PART1.md](02-SERVICECONF-PART1.md)
3. [02-SERVICECONF-PART2.md](02-SERVICECONF-PART2.md)
4. [03-UI-CONFIGURATION-PART1.md](03-UI-CONFIGURATION-PART1.md)
5. [03-UI-CONFIGURATION-PART2.md](03-UI-CONFIGURATION-PART2.md)
6. [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md)
7. [05-NOTIFICATION-TEMPLATES.md](05-NOTIFICATION-TEMPLATES.md)
8. [06-PDF-TEMPLATES.md](06-PDF-TEMPLATES.md)
9. [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md)

**Template de référence:**
- [../TEMPLATE_BASE/](../TEMPLATE_BASE/) - Arborescence complète
- [../TEMPLATE_BASE/README.md](../TEMPLATE_BASE/README.md) - Guide principal template

---

**💡 Conseil:** Utilisez Ctrl+F (Cmd+F sur Mac) pour rechercher dans les fichiers.

**Dernière mise à jour:** 24 octobre 2025
