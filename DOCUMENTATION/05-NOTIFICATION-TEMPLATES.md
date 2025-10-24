# 📱 NOTIFICATION TEMPLATES (SMS) - DOCUMENTATION COMPLÈTE

**Documentation exhaustive des templates SMS (.txt).**

---

## 🎯 RÔLE DES TEMPLATES NOTIFICATION

Les fichiers dans `notification/` définissent le **contenu des SMS** envoyés automatiquement lors du workflow.

**Caractéristiques:**
- ✅ Messages courts (160 caractères recommandés)
- ✅ Texte brut uniquement
- ✅ Variables dynamiques
- ✅ Pas d'accents (recommandé pour compatibilité)

---

## 📁 EMPLACEMENT

```
PSxxxxx/
└── x.y/
    └── notification/
        ├── citizen-approved.txt      ← SMS citoyen approuvé
        ├── citizen-rejected.txt      ← SMS citoyen rejeté
        └── alert.txt                 ← SMS alerte
```

**Règles strictes:**
- ✅ Extension `.txt` OBLIGATOIRE
- ✅ Encodage UTF-8
- ✅ Texte brut (pas de HTML)
- ✅ Nom libre (référencé dans serviceconf.json)

---

## 📝 FORMAT DU FICHIER

### Structure de base

```
Demande ${metaData.request_id} approuvee. Telechargez votre certificat sur le portail.
```

**⚠️ CONTRAINTES SMS:**
1. **160 caractères MAX recommandé**
2. **Éviter accents** (problèmes encodage selon opérateurs)
3. **Pas de retours à la ligne** (ignorés dans SMS)
4. **Variables courtes**

---

## 🔧 VARIABLES DISPONIBLES

### Variables depuis `data`

**Syntaxe:** `${data.chemin}`

**Exemples:**
```
${data.person.first_name}
${data.company.name}
${data.document.type}
```

---

### Variables depuis `metaData`

**Syntaxe:** `${metaData.chemin}`

**Exemples:**
```
${metaData.request_id}
${metaData.approved_date}
${metaData.citizen.0}
```

---

### Variables système

**Syntaxe:** `${_i18n.variable}`

**Exemples:**
```
${_i18n.current_date}
${_i18n.current_time}
```

---

## ⚠️ CONTRAINTES TECHNIQUES

### 1. Limite de caractères

**Standard SMS: 160 caractères**

**Au-delà:**
- Message envoyé en **plusieurs SMS**
- Coût multiplié
- Réception peut être fragmentée

**Exemples:**

**✅ BON (142 caractères):**
```
Demande ${metaData.request_id} approuvee le ${_i18n.current_date}. Certificat disponible sur portal.gouv.bj. Ref: ${metaData.request_id}
```

**❌ MAUVAIS (223 caractères):**
```
Bonjour ${data.person.first_name}, nous avons le plaisir de vous informer que votre demande de certificat reference ${metaData.request_id} soumise le ${metaData.submission_date} a ete approuvee. Vous pouvez maintenant telecharger votre certificat.
```

**Compter les caractères:**
```bash
echo "Votre message ici" | wc -c
```

---

### 2. Encodage - Accents

**Problème:** Certains opérateurs SMS ne supportent pas bien les accents.

**❌ Risques avec accents:**
```
Demande approuvée.    → Peut s'afficher: "Demande approuv?e."
État modifié.         → Peut s'afficher: "?tat modifi?."
```

**✅ RECOMMANDÉ: Pas d'accents**
```
é → e
è → e
à → a
ê → e
ç → c
```

**Exemples:**

**❌ Avec accents:**
```
Demande approuvée. Téléchargez votre certificat.
```

**✅ Sans accents:**
```
Demande approuvee. Telechargez votre certificat.
```

**⚠️ Compromis:**
Si vous voulez absolument les accents, **testez d'abord** avec tous les opérateurs cibles (MTN, Moov, etc.).

---

### 3. Retours à la ligne

**⚠️ Ignorés dans SMS!**

**MAUVAIS (ne fonctionne pas):**
```
Demande approuvee.
Ref: ${metaData.request_id}
Telechargez sur le portail.
```

**Résultat SMS:**
```
Demande approuvee. Ref: REQ123 Telechargez sur le portail.
```

**✅ SOLUTION: Tout sur une ligne**
```
Demande approuvee. Ref: ${metaData.request_id}. Telechargez sur le portail.
```

---

### 4. Caractères spéciaux

**Éviter:**
- Emojis (⚠️ ✅ ❌) → peuvent ne pas s'afficher
- Symboles complexes
- Guillemets typographiques (" " ' ')

**Utiliser:**
- Tirets simples (-)
- Apostrophes simples (')
- Guillemets droits (")

---

## 📱 EXEMPLES DE TEMPLATES

### Exemple 1: Approbation simple

**Fichier:** `notification/citizen-approved.txt`

```
Demande ${metaData.request_id} approuvee. Certificat disponible sur portal.gouv.bj
```

**Caractères:** 82 ✅

**Usage:**
```json
"sms-approved": {
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

---

### Exemple 2: Rejet avec motif court

**Fichier:** `notification/citizen-rejected.txt`

```
Demande ${metaData.request_id} rejetee. Motif: ${metaData.short_rejection_reason}. Contact: support@gouv.bj
```

**⚠️ Variable `short_rejection_reason` doit être COURTE!**

**Dans workflow:**
```json
// Stage avant SMS: définit motif court
"set-short-reason": {
  "type": "hardcoded-data",
  "writeData": {
    "metaData": {
      "short_rejection_reason": "Documents incomplets"    // Max 20 caractères
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "sms-rejected"
  }
}
```

---

### Exemple 3: Modifications demandées

**Fichier:** `notification/citizen-changes-requested.txt`

```
Modifications requises sur demande ${metaData.request_id}. Details par email. Delai: ${metaData.days_remaining} jours.
```

**Caractères:** ~110 ✅

---

### Exemple 4: Confirmation de soumission

**Fichier:** `notification/citizen-submission-confirmation.txt`

```
Demande ${metaData.request_id} recue. Traitement sous 5-7 jours. Suivi: portal.gouv.bj
```

**Caractères:** 85 ✅

---

### Exemple 5: Alerte agent

**Fichier:** `notification/official-alert.txt`

```
Nouvelle demande ${metaData.request_id} a examiner. Type: ${data.document.type}. Acces: portal.gouv.bj/official
```

**Caractères:** 105 ✅

**Usage:**
```json
"alert-official": {
  "type": "notification",
  "templateTxt": "official-alert.txt",
  "target": {
    "type": "hardcoded-nr",
    "nr": "+22997123456"    // Numéro agent
  },
  "transitions": {
    "type": "single",
    "nextStage": "REQUESTED"
  }
}
```

---

### Exemple 6: Rappel échéance

**Fichier:** `notification/citizen-deadline-reminder.txt`

```
Rappel: Demande ${metaData.request_id} expire dans ${metaData.days_remaining} jours. Modifiez sur portal.gouv.bj
```

**Caractères:** 104 ✅

---

## 🎯 STRATÉGIES DE CONCISION

### 1. Abréviations acceptables

| Complet | Abrégé | Gain |
|---------|--------|------|
| référence | ref | 5 char |
| demande | dem | 3 char |
| téléchargez | telechargez | 0 (mais retire accent) |
| disponible | dispo | 5 char |
| modifications | modifs | 5 char |
| informations | infos | 5 char |

**✅ Exemple:**
```
// Long (95 caractères)
Informations: demande reference ${metaData.request_id} disponible

// Court (70 caractères)
Infos: dem ref ${metaData.request_id} dispo
```

---

### 2. URLs courtes

**❌ Long:**
```
https://portal.gouv.bj/citizen/requests/${metaData.request_id}/download
```

**✅ Court:**
```
portal.gouv.bj/r/${metaData.request_id}
```

**⚠️ Nécessite:**
- Redirections configurées côté serveur
- URL shortener

---

### 3. Éliminer le superflu

**❌ Verbeux:**
```
Bonjour ${data.person.first_name}, nous avons le plaisir de vous informer que...
```

**✅ Concis:**
```
Demande ${metaData.request_id} approuvee. Certificat dispo sur portail.
```

---

### 4. Variables courtes

**❌ Variables longues:**
```
${metaData.official_full_name}             // "Jean-Baptiste KOUASSI"
${data.company.commercial_name}            // "Société Internationale..."
```

**✅ Définir versions courtes:**

```json
// Dans workflow
"prepare-sms": {
  "type": "hardcoded-data",
  "writeData": {
    "metaData": {
      "sms_ref": "${metaData.request_id}",
      "sms_status": "OK"    // Au lieu de "Approuvé le XX par YY"
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "send-sms"
  }
}
```

---

## 🌍 MULTILINGUE

**⚠️ Templates SMS = Une seule langue**

**❌ NE FONCTIONNE PAS:**
```
${if lang == "fr"}Approuvé${else}Approved${endif}
```

**✅ SOLUTIONS:**

### Solution 1: Templates séparés

```
notification/
├── citizen-approved-fr.txt
└── citizen-approved-en.txt
```

```json
// Branchement selon langue
"route-sms": {
  "type": "map-by-meta",
  "metaPathToKey": ["user_lang"],
  "map": {
    "fr": "sms-approved-fr",
    "en": "sms-approved-en"
  }
}

"sms-approved-fr": {
  "type": "notification",
  "templateTxt": "citizen-approved-fr.txt",
  ...
}

"sms-approved-en": {
  "type": "notification",
  "templateTxt": "citizen-approved-en.txt",
  ...
}
```

---

### Solution 2: Langue par défaut unique

**Contexte Bénin: Français par défaut**

```
notification/citizen-approved.txt:
Demande ${metaData.request_id} approuvee. Certificat dispo sur portail.
```

**Pas de version anglaise** (simplifie maintenance).

---

## ⚠️ ERREURS COURANTES

### 1. Message trop long

**❌ ERREUR (201 caractères):**
```
Bonjour ${data.person.first_name}, nous avons le plaisir de vous informer que votre demande de certificat numero ${metaData.request_id} a ete approuvee par nos services et vous pouvez desormais telecharger votre document.
```

**Résultat:**
- 2 SMS envoyés (coût x2)
- Réception fragmentée

**✅ SOLUTION (82 caractères):**
```
Demande ${metaData.request_id} approuvee. Certificat sur portal.gouv.bj
```

---

### 2. Variable inexistante

**❌ ERREUR:**
```
Demande ${metaData.ref_number} traitee.
```

**Si `ref_number` n'existe pas:**
```
SMS reçu: "Demande undefined traitee."
```

**✅ SOLUTION:**
Vérifier variables dans workflow avant SMS.

---

### 3. Accents mal affichés

**❌ Problème:**
```
Template: Demande approuvée.
SMS reçu: Demande approuv?e.
```

**✅ SOLUTION:**
```
Template: Demande approuvee.
```

---

### 4. Retours à la ligne

**❌ ERREUR:**
```
Demande approuvee.
Ref: ${metaData.request_id}.
Portail: portal.gouv.bj
```

**SMS reçu:**
```
Demande approuvee. Ref: REQ123. Portail: portal.gouv.bj
```

**✅ Prévoir espacement:**
```
Demande approuvee. Ref: ${metaData.request_id}. Portail: portal.gouv.bj
```

---

## 📋 BONNES PRATIQUES

### 1. Toujours inclure référence

**✅ BON:**
```
Demande ${metaData.request_id} approuvee. Certificat dispo.
```

**❌ MAUVAIS:**
```
Votre demande est approuvee.    // Quelle demande?
```

---

### 2. Action claire

**✅ BON:**
```
Demande ${metaData.request_id} approuvee. Telechargez sur portal.gouv.bj
```

**❌ MAUVAIS:**
```
Demande ${metaData.request_id} traitee.    // Que faire ensuite?
```

---

### 3. Contact si nécessaire

**Pour rejets/problèmes:**
```
Demande ${metaData.request_id} rejetee. Details par email. Contact: support@gouv.bj
```

**Pour succès simple:**
```
Demande ${metaData.request_id} approuvee. Certificat dispo sur portail.
```

---

### 4. Tester longueur

**Avant production:**
```bash
# Compter caractères (sans variables)
echo "Demande REQ123 approuvee. Certificat dispo." | wc -c
# Résultat: 46

# Avec variables max (simuler)
echo "Demande REQ-2025-10-24-123456 approuvee. Certificat dispo." | wc -c
# Résultat: 62 ✅
```

---

### 5. Variables courtes

**Définir dans workflow:**
```json
"prepare-sms-vars": {
  "type": "hardcoded-data",
  "writeData": {
    "metaData": {
      "sms_ref": "REQ${metaData.request_number}",    // Court
      "sms_status": "OK",                             // Au lieu de long texte
      "sms_action": "Telechargez"                     // Réutilisable
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "send-sms"
  }
}
```

**Template:**
```
${metaData.sms_status}: Dem ${metaData.sms_ref}. ${metaData.sms_action} sur portail.
```

---

## 🔄 WORKFLOW COMPLET EXEMPLE

**Scénario:** Approbation avec SMS et email

```json
// Stage 1: Prépare variables SMS courtes
"prepare-notifications": {
  "type": "hardcoded-data",
  "writeData": {
    "metaData": {
      "sms_ref": "${metaData.request_id}",
      "approved_date_short": "${_i18n.current_date}"
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "send-sms"
  }
},

// Stage 2: Envoie SMS court
"send-sms": {
  "type": "notification",
  "templateTxt": "citizen-approved.txt",
  "target": {
    "type": "path-to-npi",
    "path": "metaData.citizen.0"
  },
  "transitions": {
    "type": "single",
    "nextStage": "send-email"
  }
},

// Stage 3: Envoie email détaillé
"send-email": {
  "type": "email",
  "templateTxt": "citizen-approved-detailed.txt",
  "target": {
    "type": "path-to-email",
    "path": "data.person.email"
  },
  "subject": {
    "en": "Application approved",
    "fr": "Demande approuvée"
  },
  "transitions": {
    "type": "single",
    "nextStage": "APPROVED"
  }
}
```

**SMS:** `notification/citizen-approved.txt`
```
Demande ${metaData.sms_ref} approuvee le ${metaData.approved_date_short}. Certificat sur portal.gouv.bj
```

**Email:** `email/citizen-approved-detailed.txt`
```
Bonjour ${data.person.first_name},

Nous avons le plaisir de vous informer que votre demande a été approuvée.

[... contenu détaillé ...]
```

**Résultat:**
- SMS **immédiat** et **court** pour notification rapide
- Email **complet** avec tous les détails

---

## ✅ CHECKLIST TEMPLATE SMS

**Avant de créer un template:**

- [ ] Nom de fichier clair
- [ ] Extension `.txt`
- [ ] Encodage UTF-8
- [ ] Référencé dans serviceconf.json

**Contenu:**

- [ ] **≤ 160 caractères** (avec variables max)
- [ ] **Pas d'accents** (ou testé sur tous opérateurs)
- [ ] **Référence demande** incluse (`${metaData.request_id}`)
- [ ] **Action claire** (télécharger, contacter, modifier...)
- [ ] **Contact** si nécessaire (email ou tél)
- [ ] **Pas de retours à la ligne**
- [ ] **Variables existent** dans workflow
- [ ] **Concis mais compréhensible**

**Test:**

- [ ] Compté caractères (avec variables simulées)
- [ ] Testé affichage sans accents
- [ ] Vérifié variables définies avant SMS
- [ ] Testé sur téléphone réel (si possible)

---

## 📊 TABLEAU RÉCAPITULATIF

| Critère | Email | SMS |
|---------|-------|-----|
| **Longueur** | 100-300 mots | **≤160 caractères** |
| **Format** | Texte structuré | **Une seule ligne** |
| **Accents** | ✅ OK | ⚠️ Éviter |
| **HTML** | ❌ Non | ❌ Non |
| **Retours ligne** | ✅ OK | ❌ Ignorés |
| **Détails** | ✅ Complets | ❌ Minimal |
| **Variables** | Nombreuses | **Courtes uniquement** |
| **Coût** | Gratuit | **Par SMS** |
| **Usage** | Informations détaillées | **Notification rapide** |

---

**Dernière mise à jour:** 24 octobre 2025
**Voir aussi:**
- [04-EMAIL-TEMPLATES.md](04-EMAIL-TEMPLATES.md)
- [02-SERVICECONF-PART2.md](02-SERVICECONF-PART2.md)
- [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md)
