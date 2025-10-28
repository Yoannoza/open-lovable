# Guide Méthodologique du Développeur E-Service

## Table des Matières

1. [Introduction](#introduction)
2. [Philosophie de Conception](#philosophie-de-conception)
3. [De l'Expression Client à l'E-Service](#de-lexpression-client-à-leservice)
4. [Méthodologie de Réflexion](#méthodologie-de-réflexion)
5. [Les Questions Clés à Poser](#les-questions-clés-à-poser)
6. [Démarche de Construction](#démarche-de-construction)
7. [Cas d'Usage et Scénarios](#cas-dusage-et-scénarios)
8. [Checklist de Validation](#checklist-de-validation)

---

## Introduction

### Objectif de ce Guide

Ce guide n'est **PAS** un manuel technique. C'est un **guide de réflexion** pour vous aider à :

- **Comprendre** ce que veut réellement le client
- **Structurer** votre pensée avant de coder
- **Identifier** les composants nécessaires
- **Anticiper** les besoins non-exprimés
- **Créer** un e-service robuste et complet

### Principe Fondamental

> **Vous partez TOUJOURS du TEMPLATE_BASE comme fondation.**
> 
> Vous ne créez jamais un e-service de zéro. Le template est votre point de départ.
> Votre travail consiste à **adapter**, **compléter** et **configurer** ce template selon les besoins.

### Documentation Technique vs Guide Méthodologique

- **Ce guide** : Comment PENSER la création d'un e-service
- **Les docs techniques** : Comment ÉCRIRE chaque fichier (syntaxe, structure)

**Règle d'or** : Avant de créer/modifier un fichier, référez-vous à sa documentation technique spécifique.

---

## Philosophie de Conception

### Comprendre l'Essence d'un E-Service

Un e-service est un **parcours** que suit une demande (application) du citoyen. Ce parcours est composé de :

1. **Étapes principales (Main Stages)** : Les grandes phases du processus
2. **Étapes intermédiaires (Intermediate Stages)** : Les actions concrètes entre les phases
3. **Transitions** : Les chemins et conditions pour passer d'une étape à l'autre
4. **Permissions** : Qui peut faire quoi à chaque étape

### Les Trois Piliers de Réflexion

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  1. QUOI ?          2. QUI ?           3. COMMENT ?        │
│                                                             │
│  Quelle info        Qui intervient     Quel traitement     │
│  collecter ?        à quel moment ?    effectuer ?         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## De l'Expression Client à l'E-Service

### Typologie des Demandes Clients

Les clients s'expriment de différentes manières. Voici comment décoder leurs besoins :

#### Type 1 : Expression par Formulaire

**Client dit** : *"Je veux un formulaire pour que le citoyen entre son nom, prénom, NPI et raison de la demande"*

**Vous devez penser** :
- ✅ Étape UI nécessaire
- ✅ Quels champs exactement ? (Types de données)
- ✅ Y a-t-il des validations spécifiques ?
- ✅ Ces données vont où ? (`data` ou `metaData` ?)
- ✅ Qui remplit ce formulaire ? (Citoyen, officiel, les deux ?)
- ✅ À quel moment du processus ?

**Actions** :
1. Identifier l'étape intermédiaire → **UI Stage**
2. Se référer à : `03-UI-CONFIGURATION-PART1.md` et `03-UI-CONFIGURATION-PART2.md`
3. Créer le fichier JSON dans `ui/`

---

#### Type 2 : Expression par Traitement

**Client dit** : *"Après validation, il faut vérifier l'éligibilité du demandeur dans le système X"*

**Vous devez penser** :
- ✅ Intégration externe nécessaire
- ✅ Quel système ? (UXP, REST, autre ?)
- ✅ Quelles données envoyer ?
- ✅ Quelle réponse attendre ?
- ✅ Que faire si échec ? (Transition conditionnelle)
- ✅ Automatique ou manuel ?

**Actions** :
1. Identifier l'étape intermédiaire → **UXP Integration Stage** ou **UXP REST Stage**
2. Se référer à : `07-UXP-INTEGRATION.md`
3. Configurer l'intégration dans `uxp/` et dans `.serviceconf.json`

---

#### Type 3 : Expression par Document

**Client dit** : *"Le citoyen doit recevoir un certificat PDF avec ses informations"*

**Vous devez penser** :
- ✅ Génération de document nécessaire
- ✅ Quelles données apparaissent sur le PDF ?
- ✅ Quel template visuel ?
- ✅ À quel moment générer ? (Après validation ? À la fin ?)
- ✅ Le PDF est-il téléchargeable ? Envoyé par email ?

**Actions** :
1. Identifier l'étape intermédiaire → **Document Generation Stage**
2. Se référer à : `06-PDF-TEMPLATES.md`
3. Créer les templates HTML dans `pdf/`

---

#### Type 4 : Expression par Notification

**Client dit** : *"L'officiel doit être notifié quand une nouvelle demande arrive"*

**Vous devez penser** :
- ✅ Communication nécessaire
- ✅ Par quel canal ? (Email, notification interne, SMS ?)
- ✅ À qui exactement ? (Rôle, personne spécifique, liste ?)
- ✅ Quel message ?
- ✅ À quel moment déclencher ?

**Actions** :
1. Identifier l'étape(s) intermédiaire(s) → **Email Stage** et/ou **Notification Stage**
2. Se référer à : `04-EMAIL-TEMPLATES.md` et `05-NOTIFICATION-TEMPLATES.md`
3. Créer les templates dans `email/` et/ou `notification/`

---

#### Type 5 : Expression par Paiement

**Client dit** : *"Le service coûte 5000 FCFA"*

**Vous devez penser** :
- ✅ Étape de paiement nécessaire
- ✅ Montant fixe ou variable ?
- ✅ À quel moment ? (Début ? Milieu ? Selon conditions ?)
- ✅ Que faire si paiement échoue ?
- ✅ Remboursement possible ?

**Actions** :
1. Identifier l'étape intermédiaire → **Payment Stage**
2. Se référer à : `02-SERVICECONF-PART1.md` (section Payment Stage)
3. Configurer dans `.serviceconf.json`

---

#### Type 6 : Expression par Validation

**Client dit** : *"Un officiel doit approuver ou rejeter la demande"*

**Vous devez penser** :
- ✅ Étape manuelle nécessaire
- ✅ Qui peut valider ? (Permissions)
- ✅ Sur quels critères ? (Données à afficher)
- ✅ Formulaire de décision nécessaire ?
- ✅ Branches différentes selon décision (approuvé vs rejeté)
- ✅ Commentaires à ajouter ?

**Actions** :
1. Identifier l'étape intermédiaire → **UI Stage** (pour officiel)
2. Identifier les **Main Stages** de sortie (APPROVED, REJECTED)
3. Configurer les **transitions conditionnelles**
4. Se référer à : `03-UI-CONFIGURATION-PART1.md`

---

## Méthodologie de Réflexion

### Étape 1 : Écouter et Extraire

**Ne commencez JAMAIS à coder directement.**

1. **Notez** toutes les informations que le client mentionne
2. **Identifiez** les acteurs (citoyen, officiel, système externe)
3. **Repérez** les actions (saisir, valider, envoyer, générer, payer)
4. **Listez** les données mentionnées
5. **Détectez** les conditions et branches ("si... alors...")

### Étape 2 : Cartographier le Parcours

Dessinez mentalement (ou sur papier) le flux :

```
[NOUVEAU] → [Formulaire citoyen] → [DEMANDÉ] → [Paiement] → [PAYÉ] 
          → [Vérification externe] → [Décision officiel] 
          → [APPROUVÉ] ou [REJETÉ]
```

**Questions à ce stade** :
- Combien de **Main Stages** (états principaux) ?
- Combien de chemins possibles ?
- Y a-t-il des boucles (retour en arrière) ?

### Étape 3 : Décomposer Chaque Segment

Pour chaque segment du parcours :

```
[Main Stage A] → ??? → [Main Stage B]
                  │
                  └→ Quelles étapes intermédiaires ?
```

**Exemple** :
```
[DEMANDÉ] → ??? → [PAYÉ]
             │
             └→ 1. Étape de paiement (Payment Stage)
```

### Étape 4 : Identifier les Composants Techniques

Pour chaque étape intermédiaire identifiée, demandez-vous :

| Question | Impact |
|----------|--------|
| Est-ce automatique ou manuel ? | Définit si UI nécessaire |
| Qui peut l'exécuter ? | Définit les permissions |
| Quelles données utilise-t-elle ? | Définit les inputs |
| Quelles données produit-elle ? | Définit où stocker (data/metaData) |
| Que se passe-t-il en cas d'erreur ? | Définit les transitions d'exception |
| Plusieurs chemins possibles ? | Définit les transitions conditionnelles |

### Étape 5 : Anticiper les Besoins Non-Dits

Le client ne dit pas toujours tout. **Posez des questions proactives** :

#### Sur les Données
- "Avez-vous besoin de conserver l'historique des modifications ?"
- "Y a-t-il des données sensibles ?"
- "Ces données viennent-elles d'un système existant ?"

#### Sur le Processus
- "Que se passe-t-il si le paiement échoue ?"
- "Peut-on annuler une demande en cours ?"
- "Y a-t-il des délais à respecter ?"

#### Sur les Utilisateurs
- "Plusieurs personnes peuvent-elles travailler sur la même demande ?"
- "Y a-t-il des niveaux d'approbation différents ?"
- "Le citoyen peut-il modifier sa demande après soumission ?"

#### Sur la Communication
- "Qui doit être informé à chaque étape ?"
- "Le citoyen doit-il recevoir des rappels ?"
- "Y a-t-il des rapports à générer ?"

---

## Les Questions Clés à Poser

### Questions Générales (Début de Projet)

1. **Quel est l'objectif final du service ?**
   - Délivrer un document ?
   - Enregistrer une demande ?
   - Obtenir une autorisation ?

2. **Qui sont les acteurs ?**
   - Citoyen uniquement ?
   - Plusieurs types d'officiels ?
   - Systèmes externes ?

3. **Y a-t-il un coût associé ?**
   - Fixe ou variable ?
   - Payable quand ?

4. **Quelle est la durée typique du processus ?**
   - Instantané ?
   - Quelques jours ?
   - Plusieurs semaines ?

### Questions sur les Formulaires (UI)

5. **Quelles informations le citoyen doit-il fournir ?**
   - Données personnelles ?
   - Documents à uploader ?
   - Choix à faire ?

6. **Y a-t-il des pré-remplissages possibles ?**
   - Depuis le profil utilisateur ?
   - Depuis une demande précédente ?

7. **Quelles validations sur les données ?**
   - Format spécifique ?
   - Plages de valeurs ?
   - Dépendances entre champs ?

### Questions sur les Traitements

8. **Quels systèmes externes doivent être consultés ?**
   - Pour vérifier quoi ?
   - Synchrone ou asynchrone ?

9. **Quelles décisions automatiques peuvent être prises ?**
   - Selon quels critères ?
   - Avec quelle fiabilité ?

10. **Quelles étapes nécessitent une intervention humaine ?**
    - Pour faire quoi exactement ?
    - Avec quel niveau d'expertise ?

### Questions sur les Sorties

11. **Quels documents doivent être générés ?**
    - Certificat ?
    - Reçu ?
    - Attestation ?

12. **Qui doit recevoir quelles notifications ?**
    - À quels moments ?
    - Par quels canaux ?

13. **Y a-t-il des données à archiver longtemps ?**
    - Pour conformité légale ?
    - Pour statistiques ?

### Questions sur les Cas Exceptionnels

14. **Que faire si un traitement échoue ?**
    - Réessayer automatiquement ?
    - Alerter quelqu'un ?
    - Annuler la demande ?

15. **Peut-on faire marche arrière ?**
    - Annuler une demande ?
    - Corriger après validation ?
    - Rouvrir après clôture ?

16. **Y a-t-il des cas de rejet ?**
    - Sur quels critères ?
    - Avec possibilité de réessayer ?

---

## Démarche de Construction

### Phase 0 : Préparation

1. **Dupliquer le TEMPLATE_BASE**
   ```
   cp -r TEMPLATE_BASE/0.1 templates/MON_SERVICE/0.1
   ```

2. **Renommer les fichiers**
   - `TEMPLATE_BASE.serviceconf.json` → `MON_SERVICE.serviceconf.json`

3. **Ouvrir la documentation technique**
   - Avoir sous les yeux les docs correspondantes

### Phase 1 : Configuration du Service (serviceconf.json)

#### 1.1 Métadonnées de Base

```json
{
  "serviceId": "MON_SERVICE",
  "serviceVersion": "0.1"
}
```

**Référence** : `02-SERVICECONF-PART1.md` (Section 3)

#### 1.2 Définir les Main Stages

**Réflexion** :
- Quels sont les grands états de ma demande ?
- Départ : Toujours "start"
- Arrivée(s) : COMPLETED ? ISSUED ? REJECTED ? APPROVED ?
- Intermédiaires : REQUESTED ? PENDING ? IN_REVIEW ?

**Exemple de progression typique** :
```
start → REQUESTED → PENDING → APPROVED/REJECTED → COMPLETED
```

Pour chaque Main Stage :
- `shortTitle` : Mot court pour la liste des demandes
- `title` : Description complète pour la timeline
- `description` : Explications pour l'utilisateur
- `transitions` : Array de chemins possibles

**Référence** : `02-SERVICECONF-PART1.md` (Section 6)

#### 1.3 Définir le Start Stage

**Réflexion** :
- Qui peut créer une demande ? (Généralement le citoyen)
- Quelle est la première étape après création ?

```json
"start": {
  "type": "start",
  "shortTitle": "Nouveau",
  "title": "Nouvelle demande",
  "description": "...",
  "permissions": {
    "type": "metaArray",
    "actor": "citizen",
    "metaPathToArray": "citizen"
  },
  "nextStage": "premiere-etape-intermediate"
}
```

**Référence** : `02-SERVICECONF-PART2.md` (Section 7.1)

#### 1.4 Définir les Intermediate Stages

Pour chaque action identifiée dans votre parcours :

**Checklist par type** :

- **UI Stage** (Formulaire)
  - [ ] Qui remplit ? (citizen, official)
  - [ ] Quel fichier UI ? (`uiConfiguration`)
  - [ ] Transition unique ou conditionnelle ?
  - **Référence** : `02-SERVICECONF-PART2.md` (Section 7.2) + `03-UI-CONFIGURATION-PART1.md`

- **Payment Stage** (Paiement)
  - [ ] Montant fixe ou variable ? (`amountPath` ou valeur directe)
  - [ ] Transition après succès ?
  - **Référence** : `02-SERVICECONF-PART2.md` (Section 7.3)

- **UXP Integration Stage** (Intégration externe)
  - [ ] Quel système cible ? (`memberClass`, `subsystemCode`)
  - [ ] Quelle requête ? (`requestFile`)
  - [ ] Mapping des réponses ? (`variableMapping`)
  - [ ] Gestion des erreurs ?
  - **Référence** : `02-SERVICECONF-PART2.md` (Section 7.4) + `07-UXP-INTEGRATION.md`

- **Document Generation Stage** (PDF)
  - [ ] Quel template ? (`templateFile`)
  - [ ] Données à injecter ?
  - [ ] Header/Footer ?
  - **Référence** : `02-SERVICECONF-PART2.md` (Section 7.6) + `06-PDF-TEMPLATES.md`

- **Email Stage** (Email)
  - [ ] À qui ? (`targetPath`)
  - [ ] Quel template ? (`templateFile`)
  - [ ] Variables dynamiques ?
  - **Référence** : `02-SERVICECONF-PART2.md` (Section 7.7) + `04-EMAIL-TEMPLATES.md`

- **Notification Stage** (Notification interne)
  - [ ] À qui ? (configuration `target`)
  - [ ] Quel message ? (`templateFile`)
  - **Référence** : `02-SERVICECONF-PART2.md` (Section 7.8) + `05-NOTIFICATION-TEMPLATES.md`

#### 1.5 Définir les Transitions

**Réflexion pour chaque étape intermédiaire** :

**Transition Simple** (un seul chemin) :
```json
"transitions": {
  "nextStage": "etape-suivante"
}
```

**Transition Conditionnelle** (plusieurs chemins) :
```json
"transitions": {
  "outcomes": [
    {
      "name": "approved",
      "condition": {
        "type": "expression",
        "expression": "metaData.decision == 'approve'"
      },
      "nextStage": "APPROVED"
    },
    {
      "name": "rejected",
      "condition": {
        "type": "expression",
        "expression": "metaData.decision == 'reject'"
      },
      "nextStage": "REJECTED"
    }
  ]
}
```

**Référence** : `02-SERVICECONF-PART2.md` (Section 8)

#### 1.6 Définir les Permissions

**Réflexion** :
- Qui peut démarrer chaque chemin depuis un Main Stage ?

**Types de permissions** :

1. **Public** (tout le monde)
   ```json
   "permissions": { "type": "public" }
   ```

2. **MetaArray** (acteurs listés dans metaData)
   ```json
   "permissions": {
     "type": "metaArray",
     "actor": "citizen",
     "metaPathToArray": "citizen"
   }
   ```

3. **Hardcoded** (rôles spécifiques)
   ```json
   "permissions": {
     "type": "hardcoded",
     "actors": [
       {"type": "employeeRole", "role": "VALIDATOR"}
     ]
   }
   ```

**Référence** : `02-SERVICECONF-PART2.md` (Section 9)

---

### Phase 2 : Création des Fichiers UI

Pour chaque UI Stage définie :

1. **Créer le fichier JSON dans `ui/`**
   - Exemple : `ui/citizen-input.json`

2. **Référence obligatoire** : `03-UI-CONFIGURATION-PART1.md` et `03-UI-CONFIGURATION-PART2.md`

3. **Réflexion sur la structure** :
   - Quels onglets ? (tabs)
   - Quels champs ? (components)
   - Quelles validations ? (validators)
   - Affichage conditionnel ? (requiredIf, hiddenIf)
   - Pré-remplissage ? (defaultValue, defaultValuePath)

4. **Choix des types de composants** :
   - Texte simple : `textbox`
   - Texte long : `textarea`
   - Date : `date`
   - Fichier : `file`
   - Choix unique : `radio`, `dropdown`
   - Choix multiples : `checkbox`
   - Tableau : `table`
   - Etc.

5. **Déterminer les paths** :
   - Les données vont où ?
   - `data.` ou `metaData.` ?
   - **Règle** : Si utilisé dans les transitions → `metaData.`

---

### Phase 3 : Création des Templates Email/Notification

Pour chaque Email ou Notification Stage :

1. **Créer le fichier .txt dans `email/` ou `notification/`**

2. **Référence obligatoire** : 
   - Emails : `04-EMAIL-TEMPLATES.md`
   - Notifications : `05-NOTIFICATION-TEMPLATES.md`

3. **Structure du template** :
   ```
   Subject: [Sujet de l'email]
   
   [Corps du message avec variables ${...}]
   ```

4. **Utiliser les variables** :
   - `${data.nomCitoyen}` pour accéder aux données
   - `${metaData.numeroApplication}` pour accéder aux métadonnées

---

### Phase 4 : Création des Templates PDF

Pour chaque Document Generation Stage :

1. **Créer les fichiers HTML dans `pdf/`**
   - Template principal : `mon-certificat.html`
   - Header (optionnel) : `header.html`
   - Footer (optionnel) : `footer.html`

2. **Référence obligatoire** : `06-PDF-TEMPLATES.md`

3. **Structure HTML** :
   - HTML valide
   - CSS inline ou dans `<style>`
   - Variables Thymeleaf : `${data.field}` ou `th:text="${data.field}"`
   - Pas de JavaScript

4. **Considérations** :
   - Pagination
   - Marges pour header/footer
   - Polices disponibles
   - Images (base64 ou URL)

---

### Phase 5 : Configuration des Intégrations UXP

Pour chaque UXP Integration Stage :

1. **Créer le fichier de requête dans `uxp/`**
   - XML pour UXP classique
   - JSON pour UXP REST

2. **Référence obligatoire** : `07-UXP-INTEGRATION.md`

3. **Définir** :
   - Le service cible (memberClass, subsystemCode, serviceCode)
   - Les paramètres d'entrée (avec variables `${...}`)
   - Le mapping de sortie (quelles données récupérer et où les stocker)

4. **Tester** :
   - Vérifier que le service externe existe
   - Valider le format de réponse attendu

---

### Phase 6 : Test et Validation

#### 6.1 Vérification de la Configuration

**Checklist** :
- [ ] `serviceId` et `serviceVersion` corrects
- [ ] Exactement **un** `start` stage
- [ ] Tous les stages référencés existent
- [ ] Toutes les transitions pointent vers des stages existants
- [ ] Pas de boucles infinies involontaires
- [ ] Permissions cohérentes à chaque étape
- [ ] Tous les fichiers référencés existent (`uiConfiguration`, `templateFile`, etc.)

#### 6.2 Validation des Chemins

Pour chaque chemin possible :
1. **Suivre manuellement le flux** :
   - start → étape1 → étape2 → ... → Main Stage final
2. **Vérifier les données** :
   - Chaque étape a-t-elle les données nécessaires ?
   - Les données sont-elles créées avant d'être utilisées ?
3. **Vérifier les permissions** :
   - La bonne personne peut-elle exécuter chaque étape ?

#### 6.3 Tests de Cas Limites

**Scénarios à tester** :
- Paiement échoué
- Système externe indisponible
- Données manquantes
- Rejet de la demande
- Annulation en cours de route

---

## Cas d'Usage et Scénarios

### Scénario 1 : Service Simple (Demande de Certificat)

**Expression Client** :
> "Je veux un service où le citoyen demande un certificat de résidence. Il remplit un formulaire, paie 2000 FCFA, un officiel valide, et il reçoit un PDF par email."

**Décomposition** :

1. **Main Stages** :
   - start
   - REQUESTED (après formulaire)
   - PAID (après paiement)
   - APPROVED / REJECTED (après validation)
   - ISSUED (après génération PDF)

2. **Intermediate Stages** :
   - `citizen-input` (UI) : Formulaire citoyen
   - `payment` (Payment) : Paiement de 2000 FCFA
   - `official-review` (UI) : Validation par officiel
   - `generate-certificate` (Document Generation) : Génération du PDF
   - `send-certificate-email` (Email) : Envoi du PDF par email

3. **Fichiers à créer** :
   - `MON_SERVICE.serviceconf.json`
   - `ui/citizen-input.json`
   - `ui/official-review.json`
   - `pdf/certificate.html`
   - `email/certificate-email.txt`

4. **Flux** :
   ```
   start 
     → citizen-input → REQUESTED 
     → payment → PAID 
     → official-review → APPROVED ou REJECTED
     → (si APPROVED) generate-certificate → send-certificate-email → ISSUED
   ```

---

### Scénario 2 : Service avec Vérification Externe

**Expression Client** :
> "Le citoyen demande une autorisation. On vérifie d'abord s'il est éligible dans le système national, puis un officiel décide."

**Décomposition** :

1. **Main Stages** :
   - start
   - REQUESTED
   - VERIFIED
   - APPROVED / REJECTED

2. **Intermediate Stages** :
   - `citizen-input` (UI)
   - `check-eligibility` (UXP Integration) : Vérification externe
   - `official-decision` (UI)
   - `notify-citizen` (Email)

3. **Fichiers supplémentaires** :
   - `uxp/check-eligibility.xml` ou `.json`

4. **Flux** :
   ```
   start 
     → citizen-input → REQUESTED 
     → check-eligibility → VERIFIED
     → official-decision → APPROVED ou REJECTED
     → notify-citizen
   ```

5. **Gestion d'erreur** :
   - Si `check-eligibility` échoue → transition vers une étape d'erreur ou REJECTED
   - Utiliser `onStageException` dans les transitions

---

### Scénario 3 : Service Complexe Multi-Acteurs

**Expression Client** :
> "Une entreprise demande une licence. Un premier officiel vérifie les documents, un second valide le paiement, un troisième approuve. À chaque étape, notifier la personne concernée."

**Décomposition** :

1. **Main Stages** :
   - start
   - SUBMITTED
   - DOCUMENTS_VERIFIED
   - PAYMENT_VERIFIED
   - APPROVED / REJECTED
   - ISSUED

2. **Acteurs** :
   - Entreprise (citizen dans ce cas)
   - Officiel 1 (vérificateur documents)
   - Officiel 2 (vérificateur paiement)
   - Officiel 3 (approbateur final)

3. **Intermediate Stages** :
   - `company-input` (UI)
   - `notify-verifier1` (Notification)
   - `verify-documents` (UI)
   - `notify-verifier2` (Notification)
   - `verify-payment` (UI)
   - `notify-approver` (Notification)
   - `final-approval` (UI)
   - `generate-license` (Document Generation)
   - `notify-company` (Email)

4. **Permissions** :
   - Différents rôles d'employés pour chaque étape de validation
   - `"type": "hardcoded", "actors": [{"type": "employeeRole", "role": "DOC_VERIFIER"}]`

5. **Flux** :
   ```
   start 
     → company-input → SUBMITTED 
     → notify-verifier1 → verify-documents → DOCUMENTS_VERIFIED
     → notify-verifier2 → verify-payment → PAYMENT_VERIFIED
     → notify-approver → final-approval → APPROVED ou REJECTED
     → (si APPROVED) generate-license → notify-company → ISSUED
   ```

---

### Scénario 4 : Service avec Boucle de Correction

**Expression Client** :
> "L'officiel peut demander des corrections au citoyen avant d'approuver."

**Décomposition** :

1. **Main Stages** :
   - start
   - REQUESTED
   - CORRECTIONS_NEEDED
   - RESUBMITTED
   - APPROVED / REJECTED

2. **Intermediate Stages** :
   - `citizen-input` (UI)
   - `official-review` (UI) : Avec 3 choix (approve, reject, request-changes)
   - `notify-changes-needed` (Email)
   - `citizen-corrections` (UI)

3. **Transitions conditionnelles** :
   - Depuis `official-review` :
     - Si "approve" → APPROVED
     - Si "reject" → REJECTED
     - Si "request-changes" → CORRECTIONS_NEEDED
   - Depuis `citizen-corrections` → RESUBMITTED
   - Depuis RESUBMITTED → Retour à `official-review`

4. **Flux** :
   ```
   start 
     → citizen-input → REQUESTED 
     → official-review 
       ├→ (approve) APPROVED
       ├→ (reject) REJECTED
       └→ (request-changes) notify-changes-needed → CORRECTIONS_NEEDED
          → citizen-corrections → RESUBMITTED
          → official-review (boucle)
   ```

---

## Checklist de Validation

### Avant de Soumettre le Service

#### Configuration Générale

- [ ] Le dossier est bien nommé : `templates/[serviceId]/[serviceVersion]/`
- [ ] Le fichier principal existe : `[serviceId].serviceconf.json`
- [ ] Les `serviceId` et `serviceVersion` correspondent au nom du dossier
- [ ] Un seul stage de type `start` existe
- [ ] Au moins un Main Stage de fin existe

#### Stages

- [ ] Tous les stages ont un `type` valide
- [ ] Tous les stages ont des `transitions` définies
- [ ] Aucun stage n'est référencé mais non défini
- [ ] Les IDs de stages sont uniques
- [ ] Les Main Stages ont `shortTitle`, `title`, `description`
- [ ] Les Intermediate Stages ont les champs requis selon leur type

#### Transitions

- [ ] Toutes les transitions pointent vers des stages existants
- [ ] Les conditions sont correctes (syntaxe, paths de données)
- [ ] Les `onStageException` sont utilisés quand nécessaire
- [ ] Pas de boucles infinies non intentionnelles
- [ ] Les transitions depuis Main Stages ont des permissions définies

#### Permissions

- [ ] Chaque transition depuis un Main Stage a des permissions
- [ ] Le stage `start` a des permissions définies
- [ ] Les permissions correspondent aux acteurs réels du processus
- [ ] Les `metaPathToArray` pointent vers des chemins existants

#### Fichiers Référencés

- [ ] Tous les `uiConfiguration` existent dans `ui/`
- [ ] Tous les `templateFile` (email) existent dans `email/`
- [ ] Tous les `templateFile` (notification) existent dans `notification/`
- [ ] Tous les `templateFile` (pdf) existent dans `pdf/`
- [ ] Tous les `requestFile` (uxp) existent dans `uxp/`

#### UI Configurations

- [ ] Les fichiers JSON sont valides
- [ ] Les `targetPath` sont cohérents (data vs metaData)
- [ ] Les validations nécessaires sont présentes
- [ ] Les types de composants sont appropriés
- [ ] Les labels sont traduits (ou traduisibles)

#### Templates Email/Notification

- [ ] Les fichiers ont une ligne Subject (pour emails)
- [ ] Les variables `${...}` correspondent à des données existantes
- [ ] Le contenu est clair et professionnel
- [ ] Les traductions sont fournies si nécessaire

#### Templates PDF

- [ ] Le HTML est valide
- [ ] Les variables Thymeleaf correspondent à des données existantes
- [ ] Le style est approprié (marges, polices, etc.)
- [ ] Les images sont accessibles
- [ ] Le rendu est testé

#### Intégrations UXP

- [ ] Les services externes sont confirmés existants
- [ ] Les paramètres d'entrée sont corrects
- [ ] Le `variableMapping` est complet
- [ ] Les cas d'erreur sont gérés

### Après Tests

- [ ] Tous les chemins possibles ont été testés manuellement
- [ ] Les cas d'erreur fonctionnent correctement
- [ ] Les permissions sont respectées (bon acteur au bon moment)
- [ ] Les emails/notifications sont reçus
- [ ] Les PDFs sont générés correctement
- [ ] Les intégrations externes fonctionnent
- [ ] Les paiements sont traités
- [ ] Les données sont stockées correctement (data vs metaData)

---

## Conseils Pratiques

### 1. Commencez Simple

Ne créez pas d'emblée un service complexe. Commencez par :
1. Un formulaire simple
2. Une transition simple vers un Main Stage
3. Testez

Puis ajoutez progressivement :
- Paiement
- Validation officiel
- PDF
- Intégrations
- Etc.

### 2. Documentez au Fur et à Mesure

Créez un fichier `README.md` dans votre dossier de service qui explique :
- L'objectif du service
- Les acteurs
- Le flux général
- Les particularités

### 3. Utilisez des Noms Explicites

**Mauvais** :
```json
"stage1": { "type": "ui" }
"stage2": { "type": "ui" }
```

**Bon** :
```json
"citizen-personal-info": { "type": "ui" }
"official-validation-decision": { "type": "ui" }
```

### 4. Groupez les Données Logiquement

Dans vos formulaires UI, organisez les données par thème :

```json
{
  "data": {
    "personalInfo": {
      "name": "...",
      "npi": "..."
    },
    "requestDetails": {
      "reason": "...",
      "documents": []
    }
  }
}
```

### 5. Prévoyez les Erreurs

Pour chaque étape automatique, demandez-vous :
- Que faire si ça échoue ?
- Faut-il notifier quelqu'un ?
- Peut-on réessayer ?

Utilisez `onStageException` dans les transitions.

### 6. Testez avec des Données Réalistes

Ne testez pas qu'avec "Test Test" et "123456". Utilisez :
- Vrais formats de NPI
- Vrais noms (accents, espaces)
- Fichiers de différentes tailles
- Montants réels

### 7. Revoyez la Documentation Régulièrement

À chaque fois que vous devez créer un nouveau type de fichier :
1. Ouvrez la doc technique correspondante
2. Lisez-la en entier (même si vous l'avez déjà lue)
3. Créez le fichier en suivant les exemples

**Ne faites JAMAIS confiance à votre mémoire pour la syntaxe.**

### 8. Validez Progressivement

Ne créez pas tout le service d'un coup puis testez. Validez par étapes :
1. Créez le `.serviceconf.json` avec juste start et un Main Stage → Testez
2. Ajoutez un formulaire → Testez
3. Ajoutez le paiement → Testez
4. Etc.

### 9. Demandez des Clarifications

Si le client dit quelque chose d'ambigu, **ne devinez pas**. Posez des questions :
- "Quand vous dites X, voulez-vous dire Y ou Z ?"
- "Dans quel cas ce scénario se produit-il ?"
- "Qui exactement doit recevoir cette notification ?"

### 10. Gardez le Template de Base Intact

Ne modifiez **JAMAIS** le `TEMPLATE_BASE` directement. Copiez-le pour chaque nouveau service.

Le template est votre référence. Si vous devez l'améliorer, faites-le dans un processus séparé.

---

## Conclusion

### Ce que Vous Devez Retenir

1. **Écoutez d'abord, codez ensuite**
   - Comprenez le besoin avant de toucher au code

2. **Partez toujours du TEMPLATE_BASE**
   - Ne réinventez pas la roue

3. **Référez-vous à la documentation technique**
   - Pour chaque fichier à créer, lisez la doc correspondante

4. **Pensez en termes de parcours**
   - Main Stages = États
   - Intermediate Stages = Actions
   - Transitions = Chemins

5. **Anticipez les cas exceptionnels**
   - Erreurs, rejets, annulations

6. **Testez progressivement**
   - Pas tout d'un coup

7. **Documentez votre travail**
   - Pour vous et pour les autres

### Ressources

Avant de créer un fichier, consultez :

| Fichier à créer | Documentation à lire |
|----------------|---------------------|
| `.serviceconf.json` | `02-SERVICECONF-PART1.md` + `02-SERVICECONF-PART2.md` |
| `ui/*.json` | `03-UI-CONFIGURATION-PART1.md` + `03-UI-CONFIGURATION-PART2.md` |
| `email/*.txt` | `04-EMAIL-TEMPLATES.md` |
| `notification/*.txt` | `05-NOTIFICATION-TEMPLATES.md` |
| `pdf/*.html` | `06-PDF-TEMPLATES.md` |
| `uxp/*.xml` ou `*.json` | `07-UXP-INTEGRATION.md` |

### Vous Êtes Prêt

Avec ce guide méthodologique et la documentation technique, vous avez tous les outils pour créer des e-services robustes et complets.

**La clé du succès** : Réfléchir avant d'agir, comprendre avant de coder.

Bon développement ! 🚀
