# 📄 PDF TEMPLATES - DOCUMENTATION COMPLÈTE

**Documentation exhaustive des templates PDF (.html).**

---

## 🎯 RÔLE DES TEMPLATES PDF

Les fichiers dans `pdf/` définissent la **structure HTML** des documents PDF générés automatiquement.

**Ils contiennent:**
- ✅ Structure HTML/CSS du document
- ✅ Header/Footer (optionnels, résolution automatique)
- ✅ Variables dynamiques (remplacées au runtime)
- ✅ Images (base64 ou URL)
- ✅ Mise en page professionnelle

---

## 📁 EMPLACEMENT

```
PSxxxxx/
└── x.y/
    └── pdf/
        ├── header-template.html       ← En-tête (réutilisable)
        ├── footer-template.html       ← Pied de page (réutilisable)
        ├── certificate-template.html  ← Corps du certificat
        └── receipt-template.html      ← Autre document
```

**Règles strictes:**
- ✅ Extension `.html` OBLIGATOIRE
- ✅ HTML valide
- ✅ Encodage UTF-8
- ✅ CSS inline ou dans `<style>`

---

## 🏗️ STRUCTURE HTML

### Template minimal

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Certificat</title>
  <style>
    body { font-family: Arial, sans-serif; }
  </style>
</head>
<body>
  <h1>CERTIFICAT</h1>
  <p>Délivré à: ${data.person.full_name}</p>
  <p>Date: ${_i18n.current_date}</p>
</body>
</html>
```

---

### Template complet avec header/footer

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Certificat d'Enregistrement</title>
  <style>
    @page {
      size: A4;
      margin: 2cm 1.5cm;
    }
    
    body {
      font-family: 'Arial', sans-serif;
      font-size: 12pt;
      line-height: 1.6;
      color: #333;
    }
    
    .header {
      text-align: center;
      border-bottom: 3px solid #00539F;
      padding-bottom: 20px;
      margin-bottom: 30px;
    }
    
    .logo {
      width: 100px;
      height: auto;
    }
    
    .title {
      font-size: 24pt;
      font-weight: bold;
      color: #00539F;
      margin: 20px 0;
      text-transform: uppercase;
    }
    
    .content {
      margin: 30px 0;
    }
    
    .info-table {
      width: 100%;
      border-collapse: collapse;
      margin: 20px 0;
    }
    
    .info-table td {
      padding: 10px;
      border: 1px solid #ddd;
    }
    
    .info-table td:first-child {
      font-weight: bold;
      background-color: #f5f5f5;
      width: 40%;
    }
    
    .signature-section {
      margin-top: 50px;
      text-align: right;
    }
    
    .footer {
      text-align: center;
      font-size: 10pt;
      color: #666;
      margin-top: 40px;
      border-top: 1px solid #ddd;
      padding-top: 10px;
    }
  </style>
</head>
<body>
  <!-- HEADER -->
  <div class="header">
    <img src="data:image/png;base64,iVBORw0KGgoAAAANS..." class="logo" alt="Logo">
    <div class="title">République du Bénin</div>
    <div>Ministère de l'Économie et des Finances</div>
  </div>
  
  <!-- CONTENT -->
  <div class="content">
    <h1 class="title">Certificat d'Enregistrement</h1>
    
    <p>Nous soussignés, autorités compétentes de la République du Bénin, 
    attestons par le présent document que:</p>
    
    <table class="info-table">
      <tr>
        <td>Nom complet</td>
        <td>${data.person.full_name}</td>
      </tr>
      <tr>
        <td>NPI</td>
        <td>${metaData.citizen.0}</td>
      </tr>
      <tr>
        <td>Entreprise</td>
        <td>${data.company.name}</td>
      </tr>
      <tr>
        <td>Type d'activité</td>
        <td>${data.company.activity_type}</td>
      </tr>
      <tr>
        <td>Date d'enregistrement</td>
        <td>${data.company.registration_date}</td>
      </tr>
      <tr>
        <td>Numéro de certificat</td>
        <td>${metaData.certificate_number}</td>
      </tr>
    </table>
    
    <p>Ce certificat est délivré pour servir et valoir ce que de droit.</p>
    
    <div class="signature-section">
      <p>Fait à Cotonou, le ${_i18n.current_date}</p>
      <p style="margin-top: 40px;">
        <strong>Le Directeur</strong><br>
        [Signature officielle]
      </p>
    </div>
  </div>
  
  <!-- FOOTER -->
  <div class="footer">
    <p>Référence: ${metaData.request_id} | 
    Document généré le ${_i18n.current_datetime}</p>
    <p>Ce document est authentique et vérifié par les autorités compétentes.</p>
    <p>Page <span class="page"></span></p>
  </div>
</body>
</html>
```

---

## 🔧 ÉLÉMENTS TECHNIQUES

### 1. Déclaration @page

**Rôle:** Définit les dimensions et marges du PDF.

```css
@page {
  size: A4;              /* Format: A4, Letter, Legal */
  margin: 2cm 1.5cm;     /* Haut/Bas Gauche/Droite */
}
```

**Formats disponibles:**
```css
size: A4;              /* 21cm x 29.7cm */
size: A4 landscape;    /* A4 paysage */
size: Letter;          /* 21.6cm x 27.9cm */
size: 21cm 29.7cm;     /* Dimensions personnalisées */
```

**Marges:**
```css
margin: 2cm;                 /* Toutes les marges */
margin: 2cm 1.5cm;           /* Vertical | Horizontal */
margin: 2cm 1.5cm 2cm 1.5cm; /* Haut Droite Bas Gauche */
```

---

### 2. Header et Footer automatiques

**⚠️ RÉSOLUTION AUTOMATIQUE!**

Le système cherche automatiquement:
1. `pdf/header-template.html` → En-tête de TOUTES les pages
2. `pdf/footer-template.html` → Pied de page de TOUTES les pages

**Fichier:** `pdf/header-template.html`
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <style>
    body {
      margin: 0;
      padding: 10px;
      font-family: Arial, sans-serif;
      font-size: 10pt;
    }
    .header-container {
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-bottom: 2px solid #00539F;
      padding-bottom: 10px;
    }
    .logo {
      height: 40px;
    }
  </style>
</head>
<body>
  <div class="header-container">
    <img src="data:image/png;base64,..." class="logo" alt="Logo">
    <div>
      <strong>République du Bénin</strong><br>
      Portail Citoyen
    </div>
  </div>
</body>
</html>
```

**Fichier:** `pdf/footer-template.html`
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <style>
    body {
      margin: 0;
      padding: 10px;
      font-family: Arial, sans-serif;
      font-size: 9pt;
      text-align: center;
      color: #666;
    }
    .footer-line {
      border-top: 1px solid #ddd;
      padding-top: 10px;
    }
  </style>
</head>
<body>
  <div class="footer-line">
    <p>Document officiel - République du Bénin | 
    Page <span class="page"></span> sur <span class="topage"></span></p>
    <p>Généré le ${_i18n.current_datetime} | 
    Référence: ${metaData.request_id}</p>
  </div>
</body>
</html>
```

**⚠️ IMPORTANT:**
- Header/Footer sont **réutilisés** pour TOUS les PDF du service
- Présents sur **CHAQUE page** du document
- Variables disponibles dans header/footer

---

### 3. Variables spéciales pagination

**Dans footer uniquement:**

| Variable | Description | Exemple |
|----------|-------------|---------|
| `<span class="page"></span>` | Numéro page actuelle | 1 |
| `<span class="topage"></span>` | Nombre total pages | 3 |

**Exemple:**
```html
<p>Page <span class="page"></span> sur <span class="topage"></span></p>
```

**Résultat:**
```
Page 1 sur 3
Page 2 sur 3
Page 3 sur 3
```

---

### 4. Images

**2 méthodes:**

#### Méthode 1: Base64 (recommandé)

```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUA..." 
     alt="Logo" 
     style="width: 100px;">
```

**Avantages:**
- ✅ Intégré dans le HTML (autonome)
- ✅ Fonctionne offline
- ✅ Pas de dépendances externes

**Conversion:**
```bash
# Linux/Mac
base64 logo.png

# Ou en ligne
https://www.base64-image.de/
```

#### Méthode 2: URL externe

```html
<img src="https://portal.gouv.bj/assets/logo.png" 
     alt="Logo" 
     style="width: 100px;">
```

**⚠️ Risques:**
- ❌ Nécessite connexion internet
- ❌ URL peut devenir invalide
- ❌ Lent si serveur distant

**Recommandation:** Base64 pour logos officiels fixes.

---

### 5. Styles CSS

**3 options:**

#### Option 1: Inline (élément par élément)
```html
<p style="color: red; font-weight: bold;">Texte</p>
```

**⚠️ Verbeux et répétitif**

#### Option 2: Balise `<style>` (recommandé)
```html
<head>
  <style>
    .title { color: #00539F; font-size: 24pt; }
    .highlight { background-color: yellow; }
  </style>
</head>
<body>
  <h1 class="title">Titre</h1>
  <p class="highlight">Texte</p>
</body>
```

**✅ Meilleur choix**

#### Option 3: Fichier CSS externe
```html
<link rel="stylesheet" href="styles.css">
```

**⚠️ Non recommandé:**
- Dépendance externe
- Peut ne pas fonctionner selon config

---

## 🔧 VARIABLES DISPONIBLES

### Variables depuis `data`

```html
<p>Nom: ${data.person.full_name}</p>
<p>Entreprise: ${data.company.name}</p>
<p>Email: ${data.person.email}</p>
<p>Téléphone: ${data.person.phone}</p>
```

---

### Variables depuis `metaData`

```html
<p>Référence: ${metaData.request_id}</p>
<p>Citoyen NPI: ${metaData.citizen.0}</p>
<p>Date approbation: ${metaData.approved_date}</p>
<p>Numéro certificat: ${metaData.certificate_number}</p>
```

---

### Variables système

```html
<p>Date: ${_i18n.current_date}</p>
<p>Heure: ${_i18n.current_time}</p>
<p>Date et heure: ${_i18n.current_datetime}</p>
<p>Année: ${_i18n.current_year}</p>
```

---

### Variables calculées

**Définir dans workflow avant gen-document:**

```json
"prepare-pdf-data": {
  "type": "hardcoded-data",
  "writeData": {
    "metaData": {
      "certificate_number": "CERT-${_i18n.current_year}-${metaData.request_number}",
      "validity_date": "${_date.add_years(1)}",
      "full_address": "${data.person.street}, ${data.person.city}"
    }
  },
  "transitions": {
    "type": "single",
    "nextStage": "gen-certificate"
  }
}
```

**Utilisation dans template:**
```html
<p>Numéro: ${metaData.certificate_number}</p>
<p>Valide jusqu'au: ${metaData.validity_date}</p>
<p>Adresse: ${metaData.full_address}</p>
```

---

## 🎨 EXEMPLES COMPLETS

### Exemple 1: Certificat simple

**Fichier:** `pdf/certificate-template.html`

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Certificat</title>
  <style>
    @page { size: A4; margin: 2cm; }
    body {
      font-family: 'Times New Roman', serif;
      text-align: center;
      padding: 50px 0;
    }
    .border {
      border: 5px double #00539F;
      padding: 40px;
      margin: 20px;
    }
    .title {
      font-size: 32pt;
      font-weight: bold;
      color: #00539F;
      margin: 30px 0;
      text-transform: uppercase;
    }
    .subtitle {
      font-size: 18pt;
      margin: 20px 0;
    }
    .recipient {
      font-size: 24pt;
      font-weight: bold;
      margin: 40px 0;
      text-decoration: underline;
    }
    .body-text {
      font-size: 14pt;
      line-height: 2;
      margin: 30px 40px;
      text-align: justify;
    }
    .signature {
      margin-top: 60px;
      text-align: right;
      padding-right: 100px;
    }
  </style>
</head>
<body>
  <div class="border">
    <div class="title">République du Bénin</div>
    <div class="subtitle">Ministère de l'Économie et des Finances</div>
    
    <div class="title" style="margin-top: 50px;">Certificat</div>
    <div class="subtitle">d'Enregistrement Commercial</div>
    
    <div class="recipient">${data.person.full_name}</div>
    
    <div class="body-text">
      Nous soussignés, autorités compétentes de la République du Bénin, 
      certifions que <strong>${data.company.name}</strong>, entreprise 
      enregistrée sous le numéro <strong>${data.company.registration_number}</strong>,
      exerçant dans le domaine de <strong>${data.company.activity_type}</strong>,
      est dûment enregistrée et autorisée à exercer ses activités 
      sur le territoire national.
    </div>
    
    <div class="body-text">
      En foi de quoi, le présent certificat est délivré pour servir 
      et valoir ce que de droit.
    </div>
    
    <div class="signature">
      <p>Fait à Cotonou, le ${_i18n.current_date}</p>
      <p style="margin-top: 60px;">
        <strong>Le Directeur Général</strong><br>
        [Signature]
      </p>
    </div>
    
    <div style="margin-top: 40px; font-size: 10pt; color: #666;">
      Référence: ${metaData.request_id} | 
      Certificat N° ${metaData.certificate_number}
    </div>
  </div>
</body>
</html>
```

---

### Exemple 2: Reçu de paiement

**Fichier:** `pdf/receipt-template.html`

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Reçu</title>
  <style>
    @page { size: A4; margin: 1.5cm; }
    body {
      font-family: Arial, sans-serif;
      font-size: 11pt;
    }
    .header {
      display: flex;
      justify-content: space-between;
      border-bottom: 3px solid #333;
      padding-bottom: 15px;
      margin-bottom: 30px;
    }
    .company-info {
      flex: 1;
    }
    .receipt-info {
      text-align: right;
    }
    .receipt-number {
      font-size: 18pt;
      font-weight: bold;
      color: #d32f2f;
    }
    .section-title {
      background-color: #f5f5f5;
      padding: 10px;
      font-weight: bold;
      margin-top: 20px;
      border-left: 4px solid #00539F;
    }
    .info-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin: 15px 0;
    }
    .info-item {
      padding: 8px;
      border-bottom: 1px solid #eee;
    }
    .info-label {
      font-weight: bold;
      color: #666;
    }
    .amount-section {
      background-color: #f9f9f9;
      border: 2px solid #00539F;
      padding: 20px;
      margin: 30px 0;
      text-align: center;
    }
    .amount {
      font-size: 24pt;
      font-weight: bold;
      color: #00539F;
      margin: 10px 0;
    }
    .footer-note {
      margin-top: 40px;
      padding: 15px;
      background-color: #fff3cd;
      border-left: 4px solid #ffc107;
    }
  </style>
</head>
<body>
  <div class="header">
    <div class="company-info">
      <h2 style="margin: 0;">République du Bénin</h2>
      <p style="margin: 5px 0;">Ministère de l'Économie et des Finances</p>
      <p style="margin: 5px 0;">Direction des Services Citoyens</p>
    </div>
    <div class="receipt-info">
      <div class="receipt-number">REÇU</div>
      <p>N° ${metaData.receipt_number}</p>
      <p>${_i18n.current_date}</p>
    </div>
  </div>
  
  <div class="section-title">Informations du payeur</div>
  <div class="info-grid">
    <div class="info-item">
      <div class="info-label">Nom complet</div>
      <div>${data.person.full_name}</div>
    </div>
    <div class="info-item">
      <div class="info-label">NPI</div>
      <div>${metaData.citizen.0}</div>
    </div>
    <div class="info-item">
      <div class="info-label">Email</div>
      <div>${data.person.email}</div>
    </div>
    <div class="info-item">
      <div class="info-label">Téléphone</div>
      <div>${data.person.phone}</div>
    </div>
  </div>
  
  <div class="section-title">Détails du paiement</div>
  <div class="info-grid">
    <div class="info-item">
      <div class="info-label">Service</div>
      <div>${data.service_name}</div>
    </div>
    <div class="info-item">
      <div class="info-label">Référence demande</div>
      <div>${metaData.request_id}</div>
    </div>
    <div class="info-item">
      <div class="info-label">Date de paiement</div>
      <div>${metaData.payment_date}</div>
    </div>
    <div class="info-item">
      <div class="info-label">Mode de paiement</div>
      <div>${metaData.payment_method}</div>
    </div>
  </div>
  
  <div class="amount-section">
    <div style="font-size: 14pt; color: #666;">MONTANT TOTAL PAYÉ</div>
    <div class="amount">${metaData.amount} FCFA</div>
    <div style="font-size: 12pt; color: #666; margin-top: 10px;">
      (${metaData.amount_in_words})
    </div>
  </div>
  
  <div class="footer-note">
    <strong>Note importante:</strong> Ce reçu atteste du paiement effectué 
    pour le service mentionné ci-dessus. Veuillez le conserver précieusement. 
    En cas de litige, présentez ce document comme preuve de paiement.
  </div>
  
  <div style="text-align: center; margin-top: 50px; color: #666; font-size: 10pt;">
    <p>Document généré automatiquement le ${_i18n.current_datetime}</p>
    <p>Ce reçu est valide sans signature</p>
  </div>
</body>
</html>
```

---

### Exemple 3: Lettre officielle

**Fichier:** `pdf/official-letter.html`

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Lettre officielle</title>
  <style>
    @page { size: A4; margin: 2.5cm 2cm; }
    body {
      font-family: Arial, sans-serif;
      font-size: 11pt;
      line-height: 1.6;
    }
    .letterhead {
      text-align: center;
      border-bottom: 2px solid #00539F;
      padding-bottom: 15px;
      margin-bottom: 30px;
    }
    .ref-line {
      margin: 20px 0;
      font-size: 10pt;
    }
    .recipient {
      margin: 30px 0;
    }
    .subject {
      font-weight: bold;
      text-decoration: underline;
      margin: 20px 0;
    }
    .body-paragraph {
      text-align: justify;
      margin: 15px 0;
    }
    .signature-block {
      margin-top: 50px;
      float: right;
      text-align: center;
    }
  </style>
</head>
<body>
  <div class="letterhead">
    <strong>RÉPUBLIQUE DU BÉNIN</strong><br>
    <em>Fraternité - Justice - Travail</em><br>
    <br>
    MINISTÈRE DE L'ÉCONOMIE ET DES FINANCES<br>
    Direction des Services Citoyens
  </div>
  
  <div class="ref-line">
    <strong>Référence:</strong> ${metaData.letter_ref}<br>
    <strong>Objet:</strong> ${metaData.letter_subject}
  </div>
  
  <div style="text-align: right; margin: 20px 0;">
    Cotonou, le ${_i18n.current_date}
  </div>
  
  <div class="recipient">
    <strong>À l'attention de:</strong><br>
    ${data.person.full_name}<br>
    ${data.person.address}<br>
    ${data.person.city}
  </div>
  
  <div class="subject">
    Objet: ${metaData.letter_subject}
  </div>
  
  <div class="body-paragraph">
    ${data.salutation},
  </div>
  
  <div class="body-paragraph">
    Nous accusons réception de votre demande de certificat enregistrée 
    sous la référence <strong>${metaData.request_id}</strong>.
  </div>
  
  <div class="body-paragraph">
    ${metaData.letter_body}
  </div>
  
  <div class="body-paragraph">
    Pour toute information complémentaire, veuillez contacter nos services 
    au +229 21 12 34 56 ou par email à support@gouv.bj.
  </div>
  
  <div class="body-paragraph">
    Nous vous prions d'agréer, ${data.salutation_short}, 
    l'expression de nos salutations distinguées.
  </div>
  
  <div class="signature-block">
    <p><strong>Le Directeur</strong></p>
    <p style="margin-top: 60px;">[Signature]</p>
    <p>${metaData.official.name}</p>
  </div>
  
  <div style="clear: both;"></div>
  
  <div style="margin-top: 30px; font-size: 9pt; color: #666;">
    <p>Pièces jointes: ${metaData.attachments_count}</p>
  </div>
</body>
</html>
```

---

## ✅ BONNES PRATIQUES

### 1. Polices sûres

**✅ Polices web-safe:**
```css
font-family: Arial, sans-serif;
font-family: 'Times New Roman', serif;
font-family: 'Courier New', monospace;
font-family: Georgia, serif;
font-family: Verdana, sans-serif;
```

**❌ Éviter polices personnalisées:**
```css
font-family: 'MonaLisa', cursive;    /* Peut ne pas s'afficher */
```

---

### 2. Couleurs professionnelles

**✅ Palette officielle Bénin:**
```css
#00539F   /* Bleu officiel */
#009639   /* Vert */
#FCD116   /* Jaune */
#E8112D   /* Rouge */
```

**✅ Nuances neutres:**
```css
#333333   /* Texte principal */
#666666   /* Texte secondaire */
#999999   /* Texte tertiaire */
#F5F5F5   /* Fond gris clair */
```

---

### 3. Mise en page responsive

**❌ Dimensions fixes:**
```css
width: 800px;    /* Peut déborder sur PDF */
```

**✅ Dimensions relatives:**
```css
width: 100%;
width: 80%;
max-width: 600px;
```

---

### 4. Sauts de page

**Contrôler les sauts:**
```css
.section {
  page-break-after: always;    /* Saut après */
}

.no-break {
  page-break-inside: avoid;    /* Pas de saut à l'intérieur */
}
```

**Exemple:**
```html
<div class="section">
  <h2>Section 1</h2>
  <p>Contenu...</p>
</div>
<!-- Nouvelle page -->
<div class="section">
  <h2>Section 2</h2>
  <p>Contenu...</p>
</div>
```

---

### 5. Tableaux professionnels

```css
table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: left;
}

th {
  background-color: #00539F;
  color: white;
  font-weight: bold;
}

tr:nth-child(even) {
  background-color: #f9f9f9;
}
```

---

## ⚠️ ERREURS COURANTES

### 1. CSS externe non chargé

**❌ ERREUR:**
```html
<link rel="stylesheet" href="styles.css">
```

**✅ SOLUTION:**
```html
<style>
  /* CSS inline */
</style>
```

---

### 2. Images non affichées

**❌ Chemin relatif:**
```html
<img src="../../assets/logo.png">
```

**✅ Base64 ou URL absolue:**
```html
<img src="data:image/png;base64,iVBORw0...">
<!-- OU -->
<img src="https://portal.gouv.bj/assets/logo.png">
```

---

### 3. Variable inexistante

**❌ Variable undefined:**
```html
<p>Nom: ${data.full_name}</p>
```

**Si `data.full_name` n'existe pas:**
```
Nom: undefined
```

**✅ Vérifier dans workflow:**
```json
"prepare-pdf": {
  "type": "hardcoded-data",
  "writeData": {
    "data": {
      "full_name": "${data.person.first_name} ${data.person.last_name}"
    }
  }
}
```

---

## ✅ CHECKLIST PDF TEMPLATE

**Avant de créer un template:**

- [ ] Extension `.html`
- [ ] Encodage UTF-8
- [ ] DOCTYPE et balises HTML valides
- [ ] `@page` défini (taille, marges)
- [ ] Polices web-safe
- [ ] CSS dans `<style>` (pas externe)
- [ ] Images en base64 ou URL absolue
- [ ] Toutes variables existent dans workflow
- [ ] Testé génération PDF (pas seulement HTML)

**Header/Footer (si utilisés):**

- [ ] `header-template.html` créé
- [ ] `footer-template.html` créé
- [ ] Variables pagination (`<span class="page"></span>`)
- [ ] Cohérence visuelle avec corps

**Contenu:**

- [ ] Titre clair et professionnel
- [ ] Informations essentielles présentes
- [ ] Référence demande visible
- [ ] Date de génération
- [ ] Signature (si applicable)
- [ ] Footer avec infos légales

---

**Dernière mise à jour:** 24 octobre 2025
**Voir aussi:**
- [02-SERVICECONF-PART2.md](02-SERVICECONF-PART2.md)
- [01-STRUCTURE-PROJET.md](01-STRUCTURE-PROJET.md)
- [07-UXP-INTEGRATION.md](07-UXP-INTEGRATION.md)
