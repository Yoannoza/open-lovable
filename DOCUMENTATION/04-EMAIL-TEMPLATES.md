# 📧 EMAIL TEMPLATES - DOCUMENTATION COMPLÈTE

**Documentation exhaustive des templates email (.txt).**

---

## 🎯 RÔLE DES TEMPLATES EMAIL

Les fichiers dans `email/` définissent le **contenu des emails** envoyés automatiquement lors du workflow.

**Ils contiennent:**
- ✅ Le corps du message (texte brut)
- ✅ Variables dynamiques (remplacées au runtime)
- ✅ Texte personnalisé selon le contexte

---

## 📁 EMPLACEMENT

```
PSxxxxx/
└── x.y/
    └── email/
        ├── citizen-approved.txt        ← Email citoyen approuvé
        ├── citizen-rejected.txt        ← Email citoyen rejeté
        ├── official-notification.txt   ← Email agent
        └── official-emails.txt         ← Liste d'emails
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
Bonjour ${data.person.first_name},

Votre demande de certificat a été traitée.

Référence: ${metaData.request_id}
Statut: Approuvé

Vous pouvez télécharger votre certificat depuis votre espace citoyen.

Cordialement,
L'équipe du Portail Citoyen
```

**Règles:**
- ✅ Texte brut uniquement (pas de balises HTML)
- ✅ Variables entourées de `${}` 
- ✅ Retours à la ligne natifs (`\n`)
- ✅ Caractères accentués supportés (UTF-8)

---

## 🔧 VARIABLES DISPONIBLES

### Variables depuis `data`

**Syntaxe:** `${data.chemin.vers.valeur}`

**Exemples:**
```
${data.person.first_name}       // Prénom
${data.person.last_name}        // Nom
${data.person.email}            // Email
${data.company.name}            // Nom entreprise
${data.document.type}           // Type document
${data.files.certificate}       // ID fichier
```

**Exemple dans template:**
```
Bonjour ${data.person.first_name} ${data.person.last_name},

Votre entreprise "${data.company.name}" a été enregistrée avec succès.
```

---

### Variables depuis `metaData`

**Syntaxe:** `${metaData.chemin.vers.valeur}`

**Exemples:**
```
${metaData.request_id}          // Référence demande
${metaData.citizen.0}           // NPI citoyen
${metaData.approved_date}       // Date approbation
${metaData.official.name}       // Nom agent
${metaData.rejection_reason}    // Motif rejet
```

**Exemple dans template:**
```
Référence de votre demande: ${metaData.request_id}
Traitée par: ${metaData.official.name}
Date: ${metaData.approved_date}
```

---

### Variables système `_i18n`

**Syntaxe:** `${_i18n.variable}`

**Variables disponibles:**

| Variable | Description | Exemple |
|----------|-------------|---------|
| `${_i18n.current_date}` | Date actuelle | 24/10/2025 |
| `${_i18n.current_time}` | Heure actuelle | 14:30:45 |
| `${_i18n.current_datetime}` | Date et heure | 24/10/2025 14:30 |
| `${_i18n.current_year}` | Année | 2025 |

**Exemple:**
```
Date d'envoi: ${_i18n.current_datetime}

© ${_i18n.current_year} Portail Citoyen du Bénin
```

---

### Variables conditionnelles

**⚠️ PAS de logique conditionnelle dans templates!**

**MAUVAIS (ne fonctionne pas):**
```
${if metaData.approved}
  Votre demande est approuvée.
${else}
  Votre demande est rejetée.
${endif}
```

**✅ SOLUTION: Templates séparés**
```
email/
├── citizen-approved.txt     // Template si approuvé
└── citizen-rejected.txt     // Template si rejeté
```

```json
// Dans serviceconf.json
"email-approved": {
  "type": "email",
  "templateTxt": "citizen-approved.txt",    // Template spécifique
  ...
}

"email-rejected": {
  "type": "email",
  "templateTxt": "citizen-rejected.txt",    // Autre template
  ...
}
```

---

## 📧 TYPES DE TEMPLATES

### 1. Template citoyen - Approbation

**Fichier:** `email/citizen-approved.txt`

**Contenu:**
```
Bonjour ${data.person.first_name},

Nous avons le plaisir de vous informer que votre demande de certificat a été approuvée.

DÉTAILS DE LA DEMANDE
----------------------
Référence: ${metaData.request_id}
Type: ${data.document.type}
Date de traitement: ${metaData.approved_date}
Traité par: ${metaData.official.name}

PROCHAINES ÉTAPES
-----------------
Votre certificat est maintenant disponible dans votre espace citoyen.
Vous pouvez le télécharger à l'adresse suivante:
https://portal.gouv.bj/citizen/requests/${metaData.request_id}

Pour toute question, veuillez nous contacter:
- Email: support@gouv.bj
- Téléphone: +229 21 12 34 56

Cordialement,
Le Service des Citoyens
République du Bénin

---
Cet email a été envoyé automatiquement. Merci de ne pas y répondre.
```

**Usage dans serviceconf.json:**
```json
"email-citizen-approved": {
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

---

### 2. Template citoyen - Rejet

**Fichier:** `email/citizen-rejected.txt`

**Contenu:**
```
Bonjour ${data.person.first_name},

Nous sommes au regret de vous informer que votre demande de certificat a été rejetée.

DÉTAILS DE LA DEMANDE
----------------------
Référence: ${metaData.request_id}
Type: ${data.document.type}
Date de traitement: ${metaData.rejected_date}

MOTIF DU REJET
--------------
${metaData.rejection_reason}

QUE FAIRE MAINTENANT ?
-----------------------
Vous pouvez soumettre une nouvelle demande en corrigeant les points mentionnés ci-dessus.

Si vous pensez qu'il s'agit d'une erreur, vous pouvez faire un recours en contactant:
- Email: recours@gouv.bj
- Téléphone: +229 21 12 34 56

Référence à mentionner: ${metaData.request_id}

Cordialement,
Le Service des Citoyens
République du Bénin

---
Cet email a été envoyé automatiquement. Merci de ne pas y répondre.
```

---

### 3. Template citoyen - Modifications demandées

**Fichier:** `email/citizen-changes-requested.txt`

**Contenu:**
```
Bonjour ${data.person.first_name},

Votre demande de certificat nécessite des modifications avant d'être approuvée.

DÉTAILS DE LA DEMANDE
----------------------
Référence: ${metaData.request_id}
Type: ${data.document.type}
Date de révision: ${metaData.review_date}

MODIFICATIONS REQUISES
-----------------------
${metaData.changes_requested}

ACTION REQUISE
--------------
Veuillez vous connecter à votre espace citoyen et modifier votre demande:
https://portal.gouv.bj/citizen/requests/${metaData.request_id}/edit

DÉLAI
-----
Vous disposez de 15 jours pour effectuer les modifications.
Passé ce délai, votre demande sera automatiquement annulée.

Pour toute question:
- Email: support@gouv.bj
- Téléphone: +229 21 12 34 56

Cordialement,
Le Service des Citoyens
```

---

### 4. Template agent - Nouvelle demande

**Fichier:** `email/official-notification.txt`

**Contenu:**
```
Nouvelle demande à examiner

DEMANDE
-------
Référence: ${metaData.request_id}
Type: ${data.document.type}
Demandeur: ${data.person.first_name} ${data.person.last_name}
NPI: ${metaData.citizen.0}
Date de soumission: ${metaData.submission_date}

DÉTAILS
-------
Entreprise: ${data.company.name}
Catégorie: ${data.request_category}

ACCÈS
-----
Examiner la demande:
https://portal.gouv.bj/official/requests/${metaData.request_id}

---
Portail Citoyen - Back Office
```

**Usage:**
```json
"notify-officials": {
  "type": "email",
  "templateTxt": "official-notification.txt",
  "target": {
    "type": "email-file",
    "fileName": "official-emails.txt"
  },
  "subject": {
    "en": "New application to review - ${metaData.request_id}",
    "fr": "Nouvelle demande à examiner - ${metaData.request_id}"
  },
  "transitions": {
    "type": "single",
    "nextStage": "REQUESTED"
  }
}
```

---

### 5. Fichier liste d'emails

**Fichier:** `email/official-emails.txt`

**Format:** 1 email par ligne

**Contenu:**
```
reviewer1@gouv.bj
reviewer2@gouv.bj
manager@gouv.bj
admin@gouv.bj
```

**⚠️ RÈGLES:**
- 1 email par ligne
- Pas de commentaires
- Lignes vides ignorées
- Emails invalides ignorés silencieusement

**Usage:**
```json
"target": {
  "type": "email-file",
  "fileName": "official-emails.txt"
}
```

**Email envoyé à TOUS les destinataires de la liste.**

---

## ✨ BONNES PRATIQUES

### 1. Structure claire

**✅ BON:**
```
Bonjour ${data.person.first_name},

SECTION 1
---------
Contenu...

SECTION 2
---------
Contenu...

Cordialement,
Signature
```

**❌ MAUVAIS:**
```
Bonjour ${data.person.first_name}, votre demande ${metaData.request_id} du ${metaData.date} concernant ${data.type} a été traitée et vous pouvez maintenant télécharger votre certificat depuis le portail en vous connectant avec votre NPI ${metaData.citizen.0} et en allant dans la section mes demandes puis en cliquant sur la référence ${metaData.request_id}.
```

---

### 2. Informations essentielles

**Toujours inclure:**
- ✅ Référence de demande (traçabilité)
- ✅ Date de traitement
- ✅ Action attendue (si applicable)
- ✅ Contact support

**Exemple:**
```
Référence: ${metaData.request_id}
Date: ${metaData.processed_date}

[ACTION REQUISE ou INFORMATION]

Contact: support@gouv.bj
```

---

### 3. Ton professionnel mais accessible

**✅ BON:**
```
Bonjour ${data.person.first_name},

Nous avons le plaisir de vous informer que...
```

**❌ MAUVAIS:**
```
Salut ${data.person.first_name},

Cool ! Ta demande est OK...
```

---

### 4. Instructions claires

**✅ BON:**
```
PROCHAINES ÉTAPES
-----------------
1. Connectez-vous à votre espace citoyen
2. Accédez à "Mes demandes"
3. Téléchargez votre certificat

Lien direct: https://portal.gouv.bj/citizen/requests/${metaData.request_id}
```

**❌ MAUVAIS:**
```
Allez sur le portail et téléchargez votre certificat quelque part.
```

---

### 5. Variables vérifiées

**⚠️ AVANT d'utiliser une variable, vérifier qu'elle existe!**

**Workflow pour variables sûres:**

```json
// Stage 1: hardcoded-data (initialise variables)
"set-defaults": {
  "type": "hardcoded-data",
  "writeData": {
    "metaData": {
      "request_id": "REQ-${_i18n.current_timestamp}",
      "submission_date": "${_i18n.current_date}"
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "citizen-input"
  }
}

// Stage 2: email (utilise variables définies)
"send-confirmation": {
  "type": "email",
  "templateTxt": "confirmation.txt",
  ...
}
```

**Si variable absente/undefined:**
- Affichage dans email: `(vide)` ou `undefined`
- ❌ Mauvaise expérience utilisateur

---

### 6. Longueur raisonnable

**Recommandations:**
- ✅ 100-300 mots (1-2 écrans)
- ✅ Paragraphes courts
- ❌ Éviter emails > 500 mots

**Trop long:**
```
Bonjour,

[10 paragraphes de contexte]
[5 paragraphes d'explication]
[3 paragraphes de disclaimers]
...
```

**✅ Meilleur:**
```
Bonjour,

[1-2 paragraphes essentiels]

Pour plus d'informations: [lien]
```

---

### 7. Disclaimer et footer

**Toujours inclure:**
```
---
Cet email a été envoyé automatiquement. Merci de ne pas y répondre.
Pour toute question, contactez: support@gouv.bj
```

**Protection légale:**
```
---
AVERTISSEMENT: Ce message et toutes les pièces jointes sont confidentiels
et destinés exclusivement au(x) destinataire(s). Si vous avez reçu ce
message par erreur, merci d'en informer l'expéditeur et de le détruire.
```

---

## 🌍 MULTILINGUE

**⚠️ Templates = Une seule langue**

**MAUVAIS (ne fonctionne pas):**
```
// Dans citizen-approved.txt
${if lang == "fr"}
  Bonjour
${else}
  Hello
${endif}
```

**✅ SOLUTION 1: Templates séparés**
```
email/
├── citizen-approved-en.txt     // Version anglaise
└── citizen-approved-fr.txt     // Version française
```

```json
// Choix du template selon langue
"email-citizen-approved": {
  "type": "email",
  "templateTxt": "${metaData.user_lang == 'en' ? 'citizen-approved-en.txt' : 'citizen-approved-fr.txt'}",
  ...
}
```

**⚠️ Vérifier support de ternaire dans votre version!**

**✅ SOLUTION 2: Branches séparées**
```json
// Français
"email-approved-fr": {
  "type": "email",
  "templateTxt": "citizen-approved-fr.txt",
  ...
}

// Anglais
"email-approved-en": {
  "type": "email",
  "templateTxt": "citizen-approved-en.txt",
  ...
}

// Choix avec map-by-meta
"route-email": {
  "type": "map-by-meta",
  "metaPathToKey": ["user_lang"],
  "map": {
    "fr": "email-approved-fr",
    "en": "email-approved-en"
  }
}
```

---

## 📋 EXEMPLES COMPLETS

### Exemple 1: Confirmation de soumission

**Fichier:** `email/citizen-submission-confirmation.txt`

```
Bonjour ${data.person.first_name},

Nous avons bien reçu votre demande de ${data.document.type}.

VOTRE DEMANDE
-------------
Référence: ${metaData.request_id}
Date de soumission: ${metaData.submission_date}
Type: ${data.document.type}

PROCHAINES ÉTAPES
-----------------
Votre demande est en cours d'examen par nos services.
Délai de traitement: 5 à 7 jours ouvrables.

Vous recevrez un email dès que votre demande aura été traitée.

SUIVI DE VOTRE DEMANDE
-----------------------
Vous pouvez suivre l'état de votre demande en temps réel:
https://portal.gouv.bj/citizen/requests/${metaData.request_id}

BESOIN D'AIDE ?
---------------
Email: support@gouv.bj
Téléphone: +229 21 12 34 56
Horaires: Lundi-Vendredi, 8h-17h

Cordialement,
Le Service des Citoyens
République du Bénin

---
Cet email a été envoyé automatiquement. Merci de ne pas y répondre.
Référence: ${metaData.request_id}
```

---

### Exemple 2: Alerte agent - Document manquant

**Fichier:** `email/official-missing-document.txt`

```
ALERTE - Document manquant

DEMANDE CONCERNÉE
-----------------
Référence: ${metaData.request_id}
Demandeur: ${data.person.first_name} ${data.person.last_name}
NPI: ${metaData.citizen.0}
Type: ${data.document.type}

PROBLÈME DÉTECTÉ
----------------
Le document suivant est manquant ou invalide:
${metaData.missing_document_name}

Raison: ${metaData.missing_reason}

ACTION REQUISE
--------------
Veuillez contacter le demandeur pour obtenir le document manquant.

Contact du demandeur:
- Email: ${data.person.email}
- Téléphone: ${data.person.phone}

ACCÈS À LA DEMANDE
-------------------
https://portal.gouv.bj/official/requests/${metaData.request_id}

---
Portail Citoyen - Système d'alerte
Envoyé le: ${_i18n.current_datetime}
```

---

### Exemple 3: Rappel échéance

**Fichier:** `email/citizen-deadline-reminder.txt`

```
Bonjour ${data.person.first_name},

RAPPEL - Action requise sur votre demande

VOTRE DEMANDE
-------------
Référence: ${metaData.request_id}
Type: ${data.document.type}
Statut: En attente de modifications

MODIFICATIONS REQUISES
-----------------------
Des modifications ont été demandées sur votre demande le ${metaData.changes_requested_date}.

Détails: ${metaData.changes_requested}

⚠️ ÉCHÉANCE
-----------
Vous disposez encore de ${metaData.days_remaining} jours pour effectuer les modifications.
Date limite: ${metaData.deadline_date}

Passé ce délai, votre demande sera automatiquement annulée.

MODIFIER VOTRE DEMANDE
-----------------------
Cliquez ici pour modifier:
https://portal.gouv.bj/citizen/requests/${metaData.request_id}/edit

BESOIN D'AIDE ?
---------------
Si vous avez des questions sur les modifications requises:
- Email: support@gouv.bj
- Téléphone: +229 21 12 34 56

Cordialement,
Le Service des Citoyens

---
Référence: ${metaData.request_id}
```

---

## ⚠️ ERREURS COURANTES

### 1. Variable inexistante

**❌ ERREUR:**
```
Bonjour ${data.user.name},
```

**Résultat:** `Bonjour undefined,`

**✅ SOLUTION:**
Vérifier que la variable existe dans le workflow.

---

### 2. Mauvais chemin

**❌ ERREUR:**
```
Référence: ${request_id}    // Manque metaData.
```

**✅ CORRECT:**
```
Référence: ${metaData.request_id}
```

---

### 3. HTML dans template

**❌ ERREUR:**
```
<h1>Bonjour</h1>
<p>Votre demande <strong>${metaData.request_id}</strong> est approuvée.</p>
```

**Résultat:** HTML affiché comme texte brut

**✅ SOLUTION:**
Templates sont texte brut uniquement. Pas de HTML.

---

### 4. Ligne trop longue

**❌ MAUVAIS:**
```
Nous avons le plaisir de vous informer que votre demande de certificat référence ${metaData.request_id} soumise le ${metaData.submission_date} a été approuvée par ${metaData.official.name} le ${metaData.approved_date} et vous pouvez maintenant télécharger votre certificat.
```

**✅ BON:**
```
Nous avons le plaisir de vous informer que votre demande de certificat
a été approuvée.

Référence: ${metaData.request_id}
Date de soumission: ${metaData.submission_date}
Approuvée par: ${metaData.official.name}
Date d'approbation: ${metaData.approved_date}

Vous pouvez maintenant télécharger votre certificat.
```

---

## ✅ CHECKLIST TEMPLATE EMAIL

**Avant de créer un template:**

- [ ] Nom de fichier clair et descriptif
- [ ] Extension `.txt`
- [ ] Encodage UTF-8
- [ ] Référencé correctement dans serviceconf.json

**Contenu:**

- [ ] Salutation personnalisée (avec prénom si possible)
- [ ] Référence de demande (`${metaData.request_id}`)
- [ ] Date de traitement
- [ ] Information principale (approbation, rejet, modification...)
- [ ] Action attendue (télécharger, modifier, contacter...)
- [ ] Lien vers le portail (si applicable)
- [ ] Contact support (email + téléphone)
- [ ] Signature professionnelle
- [ ] Disclaimer automatique

**Variables:**

- [ ] Toutes les variables existent dans le workflow
- [ ] Chemins corrects (`data.` ou `metaData.`)
- [ ] Pas de variables conditionnelles (utiliser templates séparés)

**Style:**

- [ ] Ton professionnel
- [ ] Paragraphes courts
- [ ] Sections clairement séparées
- [ ] Instructions claires et actionnables
- [ ] Longueur raisonnable (100-300 mots)

---

**Dernière mise à jour:** 24 octobre 2025
**Voir aussi:**
- [05-NOTIFICATION-TEMPLATES.md](05-NOTIFICATION-TEMPLATES.md)
- [02-SERVICECONF-PART2.md](02-SERVICECONF-PART2.md)
- [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md)
