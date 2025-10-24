# Analyse Complète de la Codebase Open Lovable

## 📋 Table des Matières
1. [Vue d'ensemble](#vue-densemble)
2. [Architecture Frontend](#architecture-frontend)
3. [Architecture Backend](#architecture-backend)
4. [Système de Génération IA](#système-de-génération-ia)
5. [Outils et Collaboration](#outils-et-collaboration)
6. [Guide de Customisation](#guide-de-customisation)

---

## 🎯 Vue d'ensemble

**Open Lovable** est un générateur de code React piloté par IA qui permet de créer des applications web en temps réel via conversation naturelle. C'est un clone open-source de Lovable.dev créé par l'équipe Firecrawl.

### Technologies Principales
- **Frontend**: Next.js 15 (App Router), React 19, TypeScript
- **Styling**: Tailwind CSS, Framer Motion
- **IA**: Multiple providers (Anthropic Claude, OpenAI GPT-5, Google Gemini, Groq)
- **Sandbox**: Vercel Sandbox ou E2B pour l'exécution isolée
- **Scraping**: Firecrawl pour extraire le contenu des sites web

### Flux Principal
```
Utilisateur → Interface Chat → IA (génération) → Sandbox (exécution) → Preview temps réel
                    ↓
              Firecrawl (scraping de sites web pour inspiration)
```

---

## 🎨 Architecture Frontend

### Structure des Pages

#### 1. **Page d'Accueil (`app/page.tsx` & `app/landing.tsx`)**
- Interface Hero avec input pour URL ou recherche
- Sélecteur de style (Glassmorphism, Neumorphism, Brutalism, etc.)
- Sélecteur de modèle IA
- Recherche web intégrée pour trouver des sites d'inspiration

**Composants clés:**
```tsx
- HeroInput: Input principal avec recherche
- HomeHeroPixi: Animation de fond avec Pixi.js
- HeroFlame: Effets de flammes animées
- HeaderDropdownWrapper: Menu de navigation
```

#### 2. **Page de Génération (`app/generation/page.tsx`)**
C'est le **cœur** de l'application - 3539 lignes !

**Fonctionnalités principales:**
- Chat conversationnel avec l'IA
- Preview en temps réel dans iframe
- Explorateur de fichiers interactif
- Gestion de l'état de génération avec streaming
- Détection automatique des erreurs Vite/HMR

**États principaux:**
```typescript
interface GenerationState {
  sandboxData: SandboxData | null;          // Infos sandbox
  chatMessages: ChatMessage[];              // Historique conversation
  generationProgress: {
    isGenerating: boolean;
    status: string;
    files: Array<FileInfo>;                 // Fichiers en cours de génération
    streamedCode: string;                   // Code reçu en streaming
    isThinking: boolean;                    // IA en réflexion
  };
  codeApplicationState: CodeApplicationState; // État application du code
  conversationContext: {                    // Contexte conversation
    scrapedWebsites: Array<...>;
    generatedComponents: Array<...>;
    appliedCode: Array<...>;
  };
}
```

**Flux de génération:**
1. L'utilisateur envoie un message
2. Analyse de l'intention (édition vs création)
3. Streaming de la réponse IA
4. Parsing du code généré
5. Application dans le sandbox
6. Refresh du preview

### Composants UI Réutilisables

**`components/ui/`** - Composants Shadcn/UI customisés:
- Button, Dialog, Tabs, ScrollArea, etc.
- Design system cohérent avec Tailwind

**`components/shared/`** - Composants partagés:
- Header avec navigation
- Effets visuels (flame, ascii-background)
- Layout components (curvy-rect, lockBody)

**`components/CodeApplicationProgress.tsx`** - Feedback visuel:
Affiche les étapes de l'application du code (parsing, création fichiers, installation packages)

---

## ⚙️ Architecture Backend

### API Routes (Next.js App Router)

Toutes les routes sont dans `app/api/`. Voici les principales:

#### 1. **Génération de Code (`generate-ai-code-stream/route.ts`)**

**Endpoint le plus complexe** - 1896 lignes !

**Fonctionnalités:**
- Streaming de réponses IA en temps réel
- Contexte conversationnel intelligent
- Sélection agentique de fichiers à éditer
- Support multi-modèles IA
- Gestion de la mémoire (limitée aux 20 derniers messages)

**Flux détaillé:**
```typescript
1. Réception du prompt utilisateur
2. Détermination du mode (édition vs création)
   
   SI MODE ÉDITION:
   a. Récupération du manifest de fichiers
   b. Analyse de l'intention avec IA (/api/analyze-edit-intent)
   c. Création d'un plan de recherche (SearchPlan)
   d. Exécution de la recherche (file-search-executor)
   e. Sélection du fichier cible exact
   f. Création d'un contexte "chirurgical"
   
   SI MODE CRÉATION:
   a. Utilisation du contexte complet
   b. Scraping éventuel de sites web
   
3. Construction du prompt système avec:
   - Règles critiques (préservation code existant)
   - Historique de conversation
   - Fichiers récemment créés/édités
   - Préférences utilisateur déduites
   
4. Streaming vers l'IA avec streamText()
5. Envoi des chunks au client via SSE
```

**Prompts système critiques:**
```typescript
// Le système inclut des règles TRÈS strictes:
- "DO EXACTLY WHAT IS ASKED - NOTHING MORE"
- "CHECK App.jsx FIRST"
- "USE STANDARD TAILWIND CLASSES ONLY"
- "NEVER TRUNCATE FILES"
- "PRESERVE 99% of original code in edits"
```

**Providers IA supportés:**
```typescript
const groq = createGroq({ apiKey: process.env.GROQ_API_KEY });
const anthropic = createAnthropic({ apiKey: process.env.ANTHROPIC_API_KEY });
const googleGenerativeAI = createGoogleGenerativeAI({ apiKey: process.env.GEMINI_API_KEY });
const openai = createOpenAI({ apiKey: process.env.OPENAI_API_KEY });
```

#### 2. **Application du Code (`apply-ai-code-stream/route.ts`)**

Applique le code généré dans le sandbox.

**Flux:**
```typescript
1. Parse la réponse IA (extraction des fichiers)
2. Détection automatique des packages nécessaires
   - Analyse des imports ES6
   - Exclusion de React/React-DOM (déjà présents)
3. Écriture des fichiers dans le sandbox
4. Installation des packages manquants
5. Détection des erreurs de compilation
6. Tentative de récupération en cas de troncature
```

**Parsing intelligent:**
```typescript
// Supporte plusieurs formats:
<file path="src/App.jsx">
  // code...
</file>

// ou markdown:
```file path="src/Hero.jsx"
// code...
```

// Gère les duplications (garde la version la plus complète)
// Détecte les troncatures (absence de balise </file>)
```

#### 3. **Gestion du Sandbox (`create-ai-sandbox-v2/route.ts`)**

Crée et configure les environnements d'exécution isolés.

**Factory Pattern:**
```typescript
// lib/sandbox/factory.ts
class SandboxFactory {
  static create(provider?: string): SandboxProvider {
    const selectedProvider = provider || process.env.SANDBOX_PROVIDER || 'vercel';
    
    switch (selectedProvider) {
      case 'e2b': return new E2BProvider();
      case 'vercel': return new VercelProvider();
    }
  }
}
```

**Interface commune:**
```typescript
interface SandboxProvider {
  createSandbox(): Promise<SandboxInfo>;
  runCommand(command: string): Promise<CommandResult>;
  writeFile(path: string, content: string): Promise<void>;
  readFile(path: string): Promise<string>;
  listFiles(directory: string): Promise<string[]>;
  installPackages(packages: string[]): Promise<CommandResult>;
  setupViteApp(): Promise<void>;
  terminate(): Promise<void>;
}
```

**Providers:**
- **E2BProvider**: Utilise le SDK E2B pour des sandboxes cloud
- **VercelProvider**: Utilise Vercel Sandbox (intégration native Next.js)

#### 4. **Scraping Web (`scrape-website/route.ts`)**

Intégration avec Firecrawl pour extraire le contenu des sites.

```typescript
const app = new FirecrawlApp({ apiKey: process.env.FIRECRAWL_API_KEY });

const scrapeResult = await app.scrape(url, {
  formats: ['markdown', 'html'],
  onlyMainContent: true,
  waitFor: 2000,
  timeout: 30000
});
```

**Retourne:**
- Markdown du contenu principal
- HTML brut
- Screenshot optionnel
- Métadonnées (title, description, links)

#### 5. **Autres API Routes**

- **`analyze-edit-intent/`**: Analyse l'intention de l'utilisateur avec IA
- **`detect-and-install-packages/`**: Détection et installation auto de packages
- **`get-sandbox-files/`**: Récupère tous les fichiers du sandbox
- **`monitor-vite-logs/`**: Surveillance des logs Vite en temps réel
- **`restart-vite/`**: Redémarre le serveur Vite
- **`run-command/`**: Exécute des commandes dans le sandbox
- **`sandbox-status/`**: Vérifie l'état du sandbox

---

## 🤖 Système de Génération IA

### Architecture Agentique Multi-Étapes

Le système utilise une approche **agentique** sophistiquée pour les éditions de code:

#### Étape 1: Analyse de l'Intention

**Fichier**: `lib/edit-intent-analyzer.ts`

```typescript
function analyzeEditIntent(prompt: string, manifest: FileManifest): EditIntent {
  // Patterns d'intention
  const patterns: IntentPattern[] = [
    {
      patterns: [/update\s+(the\s+)?(\w+)/i, /change\s+(the\s+)?(\w+)/i],
      type: EditType.UPDATE_COMPONENT,
      fileResolver: (p, m) => findComponentByContent(p, m)
    },
    {
      patterns: [/add\s+(a\s+)?new\s+(\w+)/i, /create\s+(a\s+)?(\w+)/i],
      type: EditType.ADD_FEATURE,
      fileResolver: (p, m) => findFeatureInsertionPoints(p, m)
    },
    // ... autres patterns
  ];
}
```

**Types d'éditions détectés:**
- `UPDATE_COMPONENT`: Modification d'un composant existant
- `ADD_FEATURE`: Ajout d'une nouvelle fonctionnalité
- `FIX_ISSUE`: Correction de bug
- `UPDATE_STYLE`: Changement de style/CSS
- `REFACTOR`: Refactorisation de code
- `FULL_REBUILD`: Reconstruction complète
- `ADD_DEPENDENCY`: Ajout de package

#### Étape 2: Sélection de Contexte

**Fichier**: `lib/context-selector.ts`

```typescript
function selectFilesForEdit(userPrompt: string, manifest: FileManifest): FileContext {
  const editIntent = analyzeEditIntent(userPrompt, manifest);
  
  return {
    primaryFiles: editIntent.targetFiles,      // Fichiers à éditer
    contextFiles: otherRelevantFiles,          // Fichiers pour contexte
    systemPrompt: buildSystemPrompt(...),      // Prompt enrichi
    editIntent: editIntent
  };
}
```

**Priorisation intelligente:**
- Toujours inclure `App.jsx` pour comprendre la structure
- Inclure `tailwind.config.js` pour le contexte de style
- Inclure `package.json` pour les dépendances
- Limiter le contexte pour éviter la surcharge

#### Étape 3: Recherche Agentique de Code

**Fichier**: `lib/file-search-executor.ts`

Recherche **exacte** du code à modifier avant l'édition:

```typescript
interface SearchPlan {
  editType: string;
  reasoning: string;
  searchTerms: string[];           // Termes de recherche
  regexPatterns?: string[];        // Patterns regex optionnels
  fileTypesToSearch?: string[];    // Extensions de fichiers
  expectedMatches?: number;        // Nombre de résultats attendus
  fallbackSearch?: {               // Recherche de secours
    terms: string[];
    patterns?: string[];
  };
}

function executeSearchPlan(searchPlan: SearchPlan, files: Record<string, string>): SearchExecutionResult {
  // Recherche dans tous les fichiers
  // Retourne les localisations exactes (fichier + numéro de ligne)
  // Inclut le contexte (3 lignes avant/après)
  // Calcule un score de confiance
}
```

**Exemple de résultat:**
```typescript
{
  filePath: "/home/user/app/src/Hero.jsx",
  lineNumber: 45,
  lineContent: "  <div className=\"bg-gray-900\">",
  confidence: "high",
  contextBefore: ["function Hero() {", "  return (", "    <section>"],
  contextAfter: ["      <h1>Title</h1>", "    </div>", "  </section>"]
}
```

#### Étape 4: Génération avec Streaming

Le code est généré en **streaming** pour un feedback temps réel:

```typescript
const stream = streamText({
  model: selectedModel,
  system: enhancedSystemPrompt,
  messages: conversationHistory,
  temperature: 0.7,
  maxTokens: 8000
});

for await (const chunk of stream.textStream) {
  await sendProgress({
    type: 'stream',
    content: chunk,
    files: extractedFiles
  });
}
```

### Gestion de la Mémoire Conversationnelle

**État global persistant:**
```typescript
global.conversationState = {
  conversationId: `conv-${Date.now()}`,
  startedAt: Date.now(),
  lastUpdated: Date.now(),
  context: {
    messages: [],              // Historique des messages
    edits: [],                 // Historique des éditions
    projectEvolution: {        // Évolution du projet
      majorChanges: []
    },
    userPreferences: {}        // Préférences déduites
  }
};
```

**Nettoyage automatique:**
- Garde seulement les 15 derniers messages
- Garde seulement les 8 dernières éditions
- Limite le contexte à 2000 caractères

**Analyse des préférences:**
```typescript
function analyzeUserPreferences(messages: ConversationMessage[]): {
  commonPatterns: string[];           // Patterns récurrents
  preferredEditStyle: 'targeted' | 'comprehensive';
} {
  // Compte les éditions ciblées vs complètes
  // Détecte les demandes récurrentes (hero, header, colors, etc.)
  // Adapte le comportement futur
}
```

### Prompts Système

Le système utilise des prompts **extrêmement détaillés** avec des règles strictes:

**Structure du prompt:**
```
1. Identité et rôle de l'IA
2. Contexte conversationnel
3. 🚨 RÈGLES CRITIQUES (les plus importantes)
4. Instruction de réflexion (pensée étape par étape)
5. Contraintes et limites
6. Exemples de bonnes/mauvaises pratiques
7. Contexte technique (fichiers, structure)
8. Instructions spécifiques au mode (édition/création)
```

**Règles critiques (extraits):**
```markdown
🚨 CRITICAL RULES:
1. DO EXACTLY WHAT IS ASKED - NOTHING MORE, NOTHING LESS
2. CHECK App.jsx FIRST - see what components exist
3. NAVIGATION LIVES IN Header.jsx - Don't create Nav.jsx
4. USE STANDARD TAILWIND CLASSES ONLY (bg-white, NOT bg-background)
5. FILE COUNT LIMITS:
   - Simple change = 1 file ONLY
   - New component = 2 files MAX
6. NEVER TRUNCATE FILES - Include EVERY line
7. NO ellipsis (...) to skip content
8. SURGICAL PRECISION - Change ONLY what's requested
```

**Mode édition chirurgicale:**
```markdown
SURGICAL EDIT INSTRUCTIONS:
You have been given the EXACT location of the code to edit.
- File: /home/user/app/src/Hero.jsx
- Line: 45
- Reason: Background color change

Make ONLY the change requested. Do not modify any other code.
```

---

## 🛠️ Outils et Collaboration

### 1. Firecrawl (Web Scraping)

**Utilisation:**
```typescript
const app = new FirecrawlApp({ apiKey: process.env.FIRECRAWL_API_KEY });

const result = await app.scrape(url, {
  formats: ['markdown', 'html'],
  onlyMainContent: true,
  waitFor: 2000
});
```

**Intégration dans le flux:**
1. L'utilisateur entre une URL
2. Firecrawl extrait le contenu
3. Le contenu est stocké dans `sessionStorage`
4. L'IA utilise ce contenu comme référence pour générer du code similaire

**Cas d'usage:**
- Cloner un site web existant
- S'inspirer d'un design
- Extraire du contenu pour génération

### 2. Sandbox Providers

#### **E2B (Code Interpreter)**

Sandbox cloud complet avec environnement Python/Node.

**Avantages:**
- Environnement complet (Python + Node)
- Filesystem persistant
- Bonne isolation
- Timeout configurable (30 min par défaut)

**Limitations:**
- Coût par utilisation
- Latence réseau

**Configuration:**
```typescript
// config/app.config.ts
e2b: {
  timeoutMinutes: 30,
  vitePort: 5173,
  viteStartupDelay: 10000,
  workingDirectory: '/home/user/app'
}
```

#### **Vercel Sandbox**

Sandbox optimisé pour Next.js/React.

**Avantages:**
- Intégration native avec Vercel
- Très rapide (local dans Vercel)
- OIDC automatique avec `vercel link`
- Pas de coût additionnel

**Limitations:**
- Runtime limité (Node 22, quelques autres)
- Moins de flexibilité qu'E2B

**Configuration:**
```typescript
// config/app.config.ts
vercelSandbox: {
  timeoutMinutes: 15,
  devPort: 3000,
  devServerStartupDelay: 7000,
  runtime: 'node22'
}
```

### 3. Détection et Installation de Packages

**Détection automatique:**
```typescript
// Dans apply-ai-code-stream
function extractPackagesFromCode(content: string): string[] {
  const importRegex = /import\s+.*from\s+['"]([^'"]+)['"]/g;
  
  // Filtre:
  // - Imports relatifs (./ ou /)
  // - React/React-DOM (déjà présents)
  // - Alias de chemin (@/)
  
  return uniquePackages;
}
```

**Installation intelligente:**
```typescript
// api/install-packages-v2/route.ts
async function installPackages(packages: string[]) {
  // Évite les duplications
  // Utilise --legacy-peer-deps si configuré
  // Redémarre Vite automatiquement après installation
  // Surveille les logs pour détecter les erreurs
}
```

### 4. Monitoring des Erreurs Vite

**HMRErrorDetector (`components/HMRErrorDetector.tsx`):**

Détecte automatiquement les erreurs de compilation dans l'iframe:

```typescript
useEffect(() => {
  const handleMessage = (event: MessageEvent) => {
    if (event.data.type === 'vite:error') {
      // Extraire l'erreur
      // Parser pour détecter les packages manquants
      // Afficher dans l'UI
      // Proposer l'installation automatique
    }
  };
  
  window.addEventListener('message', handleMessage);
}, []);
```

**Auto-correction:**
- Détecte les packages manquants
- Propose l'installation automatique
- Redémarre Vite après installation
- Rafraîchit le preview

### 5. Application du Code avec MorphLLM (Fast Apply)

**Option avancée** pour éditions rapides:

```typescript
// lib/morph-fast-apply.ts
async function applyMorphEditToFile(
  fileContent: string,
  editInstruction: string,
  aiProvider: any
): Promise<string> {
  // Utilise MorphLLM pour éditions chirurgicales ultra-rapides
  // Plus rapide que régénérer le fichier entier
  // Parfait pour petits changements
}
```

**Désactivé par défaut** car en phase expérimentale.

---

## 🎨 Guide de Customisation

### Pour Créer un Agent pour un Framework Propriétaire

Votre framework utilise **JSON, HTML, et TXT simples**. Voici comment adapter Open Lovable:

#### **1. Créer un Nouveau Provider Sandbox**

**Créer** `lib/sandbox/providers/custom-provider.ts`:

```typescript
import { SandboxProvider, SandboxInfo, CommandResult } from '../types';

export class CustomFrameworkProvider extends SandboxProvider {
  async createSandbox(): Promise<SandboxInfo> {
    // Créer un environnement pour votre framework
    // Peut être un container Docker, un process Node, etc.
    
    return {
      sandboxId: generateId(),
      url: `http://localhost:${port}`,
      provider: 'custom-framework',
      createdAt: new Date()
    };
  }

  async setupFrameworkApp(): Promise<void> {
    // Au lieu de setupViteApp(), créer votre structure
    
    // Structure de base:
    await this.writeFile('config.json', JSON.stringify({
      name: 'My Custom App',
      version: '1.0.0',
      framework: 'custom'
    }));
    
    await this.writeFile('index.html', `
<!DOCTYPE html>
<html>
  <head>
    <title>Custom Framework App</title>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
    `);
    
    // Démarrer votre serveur de développement
    await this.runCommand('custom-framework serve');
  }

  async writeFile(path: string, content: string): Promise<void> {
    // Implémentation spécifique à votre environnement
    // Exemple: écriture dans un filesystem local
    const fs = require('fs').promises;
    await fs.writeFile(path, content, 'utf-8');
  }

  // ... autres méthodes requises
}
```

**Enregistrer** dans `lib/sandbox/factory.ts`:

```typescript
import { CustomFrameworkProvider } from './providers/custom-provider';

export class SandboxFactory {
  static create(provider?: string): SandboxProvider {
    const selectedProvider = provider || process.env.SANDBOX_PROVIDER || 'e2b';
    
    switch (selectedProvider) {
      case 'e2b': return new E2BProvider();
      case 'vercel': return new VercelProvider();
      case 'custom-framework': return new CustomFrameworkProvider(); // ← Nouveau
      default: throw new Error(`Unknown provider: ${selectedProvider}`);
    }
  }
}
```

#### **2. Adapter les Prompts Système**

**Modifier** `app/api/generate-ai-code-stream/route.ts`:

Remplacer les prompts React/Vite par des prompts spécifiques à votre framework:

```typescript
let systemPrompt = `You are an expert in the Custom Framework.

Custom Framework uses simple JSON, HTML, and TXT files:
- JSON files for configuration and data structures
- HTML files for templates and layouts
- TXT files for content and metadata

**CRITICAL FRAMEWORK RULES:**

1. **File Structure:**
   - config.json: Main configuration
   - templates/*.html: HTML templates
   - data/*.json: Data files
   - content/*.txt: Text content

2. **JSON Format Rules:**
   - Use valid JSON syntax
   - All strings must be properly escaped
   - No trailing commas
   - Include all required fields:
     {
       "id": "unique-id",
       "type": "component-type",
       "properties": {},
       "children": []
     }

3. **HTML Template Rules:**
   - Use semantic HTML5
   - Include data-bind attributes for dynamic content:
     <div data-bind="text: title"></div>
   - Use standard CSS classes (no external frameworks)
   - Templates must be valid HTML

4. **TXT Content Rules:**
   - UTF-8 encoding
   - Plain text with optional markdown
   - One section per file
   - Use headers (# Title) for structure

5. **File Output Format:**
   <file path="config.json">
   {
     "name": "Component Name",
     "version": "1.0.0"
   }
   </file>

   <file path="templates/main.html">
   <!DOCTYPE html>
   <html>...</html>
   </file>

   <file path="content/about.txt">
   # About Us
   This is our content...
   </file>

6. **When Creating Components:**
   - Always create config.json first
   - Define templates in templates/ directory
   - Store data in data/ directory
   - Keep content in content/ directory

7. **Common Patterns:**

   **Simple Component:**
   - config.json (component definition)
   - templates/component.html (layout)
   - data/component-data.json (data)

   **Page with Multiple Sections:**
   - config.json (page config)
   - templates/header.html
   - templates/content.html
   - templates/footer.html
   - content/sections/*.txt

8. **Integration Points:**
   - Components reference each other via "component-id"
   - Templates can include other templates: {{include "header.html"}}
   - Data binding: Use {{variable}} syntax in templates

**EXAMPLES:**

Example 1: Simple Card Component
<file path="components/card/config.json">
{
  "id": "simple-card",
  "type": "component",
  "template": "card.html",
  "data": "card-data.json"
}
</file>

<file path="components/card/templates/card.html">
<div class="card">
  <h2 data-bind="text: title"></h2>
  <p data-bind="text: description"></p>
</div>
</file>

<file path="components/card/data/card-data.json">
{
  "title": "Card Title",
  "description": "Card description text"
}
</file>

${conversationContext}

${isEdit ? `
🚨 EDIT MODE - SURGICAL CHANGES ONLY:
- DO NOT regenerate entire application
- ONLY edit the specific files mentioned
- Preserve ALL existing structure and content
- Make minimal changes to achieve the requested modification
` : ''}

USER REQUEST: "${prompt}"

Generate ONLY the files needed for this specific request.
Use the file format shown above.
`;
```

#### **3. Adapter le Parser de Code**

**Modifier** `app/api/apply-ai-code-stream/route.ts`:

Le parser actuel fonctionne déjà avec le format `<file path="">`, mais vous pouvez ajouter des validations spécifiques:

```typescript
function parseAIResponse(response: string): ParsedResponse {
  const sections = {
    files: [] as Array<{ path: string; content: string }>,
    // ... autres champs
  };

  // Parser les fichiers (déjà fonctionnel)
  const fileRegex = /<file path="([^"]+)">([\s\S]*?)<\/file>/g;
  let match;
  
  while ((match = fileRegex.exec(response)) !== null) {
    const filePath = match[1];
    const content = match[2].trim();
    
    // VALIDATION SPÉCIFIQUE À VOTRE FRAMEWORK:
    
    if (filePath.endsWith('.json')) {
      // Valider le JSON
      try {
        JSON.parse(content);
      } catch (error) {
        console.error(`Invalid JSON in ${filePath}:`, error);
        // Optionnel: tenter de corriger ou rejeter
        continue;
      }
    }
    
    if (filePath.endsWith('.html')) {
      // Valider le HTML (basique)
      if (!content.includes('<!DOCTYPE html>') && filePath.includes('templates/')) {
        // Peut être un fragment, c'est OK
      }
    }
    
    if (filePath.endsWith('.txt')) {
      // Valider l'encodage
      // Vérifier qu'il n'y a pas de caractères invalides
    }
    
    sections.files.push({ path: filePath, content });
  }
  
  return sections;
}
```

#### **4. Créer une Documentation de Framework**

**Créer** `docs/CUSTOM_FRAMEWORK_GUIDE.md`:

```markdown
# Custom Framework Guide

## Structure de Fichiers

```
my-app/
├── config.json          # Configuration principale
├── templates/           # Templates HTML
│   ├── layout.html
│   ├── header.html
│   └── footer.html
├── data/                # Données JSON
│   ├── navigation.json
│   └── content.json
└── content/             # Contenu texte
    ├── home.txt
    └── about.txt
```

## Format JSON

Tous les fichiers JSON doivent suivre cette structure:

```json
{
  "id": "unique-identifier",
  "type": "component|page|layout",
  "version": "1.0.0",
  "properties": {
    "key": "value"
  },
  "children": []
}
```

## Templates HTML

Les templates utilisent des data-bindings:

```html
<div data-bind="text: title"></div>
<div data-bind="html: content"></div>
<div data-bind="foreach: items">
  <span data-bind="text: name"></span>
</div>
```

## Fichiers de Contenu (.txt)

Format simple avec markdown optionnel:

```
# Titre de Section

Paragraphe de contenu...

## Sous-titre

Plus de contenu...
```

## Exemples Complets

### Exemple 1: Page Simple

**config.json**
```json
{
  "id": "homepage",
  "type": "page",
  "template": "main.html",
  "sections": ["hero", "features", "contact"]
}
```

**templates/main.html**
```html
<!DOCTYPE html>
<html>
<head>
  <title>{{page.title}}</title>
</head>
<body>
  {{include "header.html"}}
  <main>
    {{include "hero.html"}}
    {{include "features.html"}}
  </main>
  {{include "footer.html"}}
</body>
</html>
```

### Exemple 2: Composant Réutilisable

**components/button/config.json**
```json
{
  "id": "primary-button",
  "type": "component",
  "template": "button.html",
  "styles": "button.css",
  "defaults": {
    "text": "Click Me",
    "color": "blue"
  }
}
```
```

**Référencer** cette doc dans vos prompts:

```typescript
const frameworkDocs = await fs.readFile('docs/CUSTOM_FRAMEWORK_GUIDE.md', 'utf-8');

systemPrompt = `${systemPrompt}

## Custom Framework Documentation

${frameworkDocs}

Follow the framework guidelines precisely.
`;
```

#### **5. Adapter le Manifeste de Fichiers**

**Modifier** `types/file-manifest.ts`:

Ajouter des types spécifiques à votre framework:

```typescript
export interface FileManifest {
  entryPoint: string;
  files: Record<string, FileInfo>;
  routes: RouteInfo[];
  components: ComponentInfo[];
  // NOUVEAU: Spécifique à Custom Framework
  configs: ConfigFileInfo[];
  templates: TemplateInfo[];
  dataFiles: DataFileInfo[];
  contentFiles: ContentFileInfo[];
}

// Nouveaux types
export interface ConfigFileInfo {
  path: string;
  id: string;
  type: 'page' | 'component' | 'layout';
  version: string;
}

export interface TemplateInfo {
  path: string;
  name: string;
  includes: string[];  // Templates inclus
  dataBindings: string[];  // Variables utilisées
}

export interface DataFileInfo {
  path: string;
  schema: any;  // Structure du JSON
}

export interface ContentFileInfo {
  path: string;
  sections: string[];  // Titres de sections
  wordCount: number;
}
```

**Créer un analyseur** de manifest spécifique:

```typescript
// lib/custom-framework-analyzer.ts

export async function analyzeCustomFramework(files: Record<string, string>): Promise<FileManifest> {
  const manifest: FileManifest = {
    entryPoint: 'config.json',
    files: {},
    routes: [],
    components: [],
    configs: [],
    templates: [],
    dataFiles: [],
    contentFiles: []
  };

  for (const [path, content] of Object.entries(files)) {
    if (path.endsWith('.json') && !path.includes('package.json')) {
      // Analyser les fichiers de config
      try {
        const config = JSON.parse(content);
        manifest.configs.push({
          path,
          id: config.id,
          type: config.type,
          version: config.version
        });
      } catch (error) {
        console.error(`Failed to parse JSON: ${path}`);
      }
    }
    
    if (path.endsWith('.html')) {
      // Analyser les templates
      const includes = extractIncludes(content);  // {{include "file.html"}}
      const dataBindings = extractDataBindings(content);  // data-bind="..."
      
      manifest.templates.push({
        path,
        name: path.split('/').pop() || '',
        includes,
        dataBindings
      });
    }
    
    if (path.endsWith('.txt')) {
      // Analyser le contenu
      const sections = extractSections(content);  // Titres markdown
      const wordCount = content.split(/\s+/).length;
      
      manifest.contentFiles.push({
        path,
        sections,
        wordCount
      });
    }
    
    // Ajouter à manifest.files pour compatibilité
    manifest.files[path] = {
      path,
      type: determineFileType(path),
      content,
      size: content.length
    };
  }

  return manifest;
}

function extractIncludes(html: string): string[] {
  const regex = /\{\{include\s+"([^"]+)"\}\}/g;
  const matches = [];
  let match;
  while ((match = regex.exec(html)) !== null) {
    matches.push(match[1]);
  }
  return matches;
}

function extractDataBindings(html: string): string[] {
  const regex = /data-bind="[^"]*:\s*([^"]+)"/g;
  const matches = [];
  let match;
  while ((match = regex.exec(html)) !== null) {
    matches.push(match[1].trim());
  }
  return [...new Set(matches)];  // Unique values
}

function extractSections(text: string): string[] {
  const lines = text.split('\n');
  return lines
    .filter(line => line.startsWith('#'))
    .map(line => line.replace(/^#+\s*/, '').trim());
}
```

**Utiliser** dans l'API:

```typescript
// app/api/get-sandbox-files/route.ts

import { analyzeCustomFramework } from '@/lib/custom-framework-analyzer';

export async function GET() {
  const files = await sandbox.listAllFiles();
  const manifest = await analyzeCustomFramework(files);
  
  return NextResponse.json({
    success: true,
    files,
    manifest
  });
}
```

#### **6. Ajouter des Exemples au Contexte IA**

**Créer** `lib/custom-framework-examples.ts`:

```typescript
export const customFrameworkExamples = `
## Custom Framework Examples

### Example 1: Creating a New Page

USER: "Create a homepage with a hero section and features"

CORRECT OUTPUT:

<file path="pages/home/config.json">
{
  "id": "homepage",
  "type": "page",
  "title": "Home",
  "template": "home.html",
  "sections": ["hero", "features"]
}
</file>

<file path="pages/home/templates/home.html">
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>{{page.title}}</title>
</head>
<body>
  {{include "components/header.html"}}
  
  <section data-section="hero">
    <div data-bind="html: hero.content"></div>
  </section>
  
  <section data-section="features">
    <div data-bind="foreach: features">
      <div class="feature">
        <h3 data-bind="text: title"></h3>
        <p data-bind="text: description"></p>
      </div>
    </div>
  </section>
  
  {{include "components/footer.html"}}
</body>
</html>
</file>

<file path="pages/home/data/home-data.json">
{
  "hero": {
    "content": "<h1>Welcome to Our Site</h1><p>Your journey starts here</p>"
  },
  "features": [
    {
      "title": "Feature 1",
      "description": "Description of feature 1"
    },
    {
      "title": "Feature 2",
      "description": "Description of feature 2"
    }
  ]
}
</file>

### Example 2: Editing Existing Content

USER: "Change the hero title to 'Hello World'"

CORRECT OUTPUT:

<file path="pages/home/data/home-data.json">
{
  "hero": {
    "content": "<h1>Hello World</h1><p>Your journey starts here</p>"
  },
  "features": [
    {
      "title": "Feature 1",
      "description": "Description of feature 1"
    },
    {
      "title": "Feature 2",
      "description": "Description of feature 2"
    }
  ]
}
</file>

WRONG - DO NOT DO THIS:
- Creating new files when only data needs to change
- Modifying the template when only content changes
- Regenerating the entire page structure

### Example 3: Adding a New Component

USER: "Add a newsletter signup form"

CORRECT OUTPUT:

<file path="components/newsletter/config.json">
{
  "id": "newsletter-signup",
  "type": "component",
  "template": "newsletter.html",
  "data": "newsletter-data.json"
}
</file>

<file path="components/newsletter/templates/newsletter.html">
<div class="newsletter-signup">
  <h3 data-bind="text: heading"></h3>
  <form>
    <input type="email" placeholder="{{placeholder}}" required>
    <button type="submit" data-bind="text: buttonText"></button>
  </form>
</div>
</file>

<file path="components/newsletter/data/newsletter-data.json">
{
  "heading": "Subscribe to our Newsletter",
  "placeholder": "Enter your email",
  "buttonText": "Subscribe"
}
</file>

### Key Patterns:

1. **Config First**: Always create/update config.json to define structure
2. **Separate Concerns**: Templates for structure, data for content
3. **Minimal Changes**: In edit mode, change only what's requested
4. **Reuse Components**: Reference existing components rather than duplicating
5. **Valid Syntax**: Ensure all JSON is valid, all HTML is well-formed
`;
```

**Ajouter** aux prompts:

```typescript
import { customFrameworkExamples } from '@/lib/custom-framework-examples';

systemPrompt = `${systemPrompt}

${customFrameworkExamples}

Study these examples carefully. Your output must follow the same patterns.
`;
```

#### **7. Configuration de l'Environnement**

**Mettre à jour** `.env.local`:

```env
# Sandbox Provider
SANDBOX_PROVIDER=custom-framework

# Custom Framework Configuration (optionnel)
CUSTOM_FRAMEWORK_PORT=8080
CUSTOM_FRAMEWORK_WORKSPACE=/var/custom-app

# ... autres configs
```

**Mettre à jour** `config/app.config.ts`:

```typescript
export const appConfig = {
  // ... configs existants
  
  customFramework: {
    port: process.env.CUSTOM_FRAMEWORK_PORT || 8080,
    workspace: process.env.CUSTOM_FRAMEWORK_WORKSPACE || '/app',
    
    // Timeouts
    startupDelay: 5000,
    timeoutMinutes: 20,
    
    // Structure de fichiers par défaut
    defaultStructure: {
      'config.json': {
        name: 'Custom App',
        version: '1.0.0',
        framework: 'custom'
      },
      'templates/': {},
      'data/': {},
      'content/': {}
    },
    
    // Commandes
    commands: {
      start: 'custom-framework serve',
      build: 'custom-framework build',
      validate: 'custom-framework validate'
    }
  }
};
```

#### **8. Tester le Système**

**Créer** un test simple:

```typescript
// tests/custom-framework.test.ts

import { CustomFrameworkProvider } from '@/lib/sandbox/providers/custom-provider';

describe('Custom Framework Provider', () => {
  let provider: CustomFrameworkProvider;
  
  beforeEach(() => {
    provider = new CustomFrameworkProvider();
  });
  
  test('creates sandbox successfully', async () => {
    const sandbox = await provider.createSandbox();
    expect(sandbox.sandboxId).toBeDefined();
    expect(sandbox.provider).toBe('custom-framework');
  });
  
  test('writes and reads files correctly', async () => {
    await provider.createSandbox();
    
    const testContent = { id: 'test', type: 'component' };
    await provider.writeFile('config.json', JSON.stringify(testContent));
    
    const readContent = await provider.readFile('config.json');
    expect(JSON.parse(readContent)).toEqual(testContent);
  });
  
  test('validates JSON files', async () => {
    await provider.createSandbox();
    
    // Valide JSON
    await expect(
      provider.writeFile('valid.json', '{"key": "value"}')
    ).resolves.not.toThrow();
    
    // JSON invalide
    await expect(
      provider.writeFile('invalid.json', '{key: value}')
    ).rejects.toThrow();
  });
});
```

---

### Résumé de la Customisation

Pour adapter Open Lovable à votre framework propriétaire:

1. ✅ **Créer un SandboxProvider personnalisé** pour votre environnement d'exécution
2. ✅ **Adapter les prompts système** avec les règles spécifiques à votre framework
3. ✅ **Créer un analyseur de manifest** pour comprendre votre structure de fichiers
4. ✅ **Ajouter des exemples concrets** pour guider l'IA
5. ✅ **Valider les formats** (JSON, HTML, TXT) lors du parsing
6. ✅ **Documenter votre framework** dans un guide dédié
7. ✅ **Configurer l'environnement** avec les bons paramètres
8. ✅ **Tester** avec des cas réels

**Points clés:**
- Le système de prompts est **très flexible** - adaptez-le à vos besoins
- Le parser de fichiers `<file path="">` fonctionne déjà - ajoutez juste vos validations
- La structure agentique (analyse intention → recherche → génération) s'applique à n'importe quel framework
- Privilégiez des **exemples concrets** dans vos prompts plutôt que des explications abstraites

**Prochaines étapes recommandées:**
1. Commencer petit : créer un provider simple qui écrit des fichiers localement
2. Tester avec des prompts basiques ("créer une page simple")
3. Itérer sur les prompts en fonction des résultats
4. Ajouter progressivement des validations et contraintes
5. Documenter les patterns qui fonctionnent bien

---

## 📚 Ressources Additionnelles

### Documentation Interne

- `docs/PACKAGE_DETECTION_GUIDE.md` - Guide de détection automatique de packages
- `docs/STREAMING_FIXES_SUMMARY.md` - Corrections du streaming
- `docs/TOOL_CALL_FIX_SUMMARY.md` - Corrections des appels de tools
- `docs/UI_FEEDBACK_DEMO.md` - Démo du feedback utilisateur

### Fichiers de Configuration

- `config/app.config.ts` - Configuration centralisée
- `tailwind.config.ts` - Configuration Tailwind
- `next.config.ts` - Configuration Next.js

### Types TypeScript

- `types/conversation.ts` - Types pour la conversation
- `types/file-manifest.ts` - Types pour le manifest de fichiers
- `types/sandbox.ts` - Types pour les sandboxes

### Bibliothèques Clés

- **AI SDK**: `ai` package de Vercel pour streaming
- **E2B**: `@e2b/code-interpreter` pour sandboxes
- **Vercel Sandbox**: `@vercel/sandbox` pour environnements locaux
- **Firecrawl**: `@mendable/firecrawl-js` pour scraping
- **Framer Motion**: Animations fluides
- **Jotai**: State management léger
- **Sonner**: Notifications toast

---

## 🎓 Concepts Avancés

### 1. Streaming avec Server-Sent Events (SSE)

Le système utilise SSE pour le feedback temps réel:

```typescript
// Côté serveur
const encoder = new TextEncoder();
const stream = new TransformStream();
const writer = stream.writable.getWriter();

await writer.write(encoder.encode(`data: ${JSON.stringify(data)}\n\n`));

// Côté client
const eventSource = new EventSource('/api/generate-ai-code-stream');
eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  // Traiter les données
};
```

### 2. Global State Management

Variables globales pour l'état partagé entre requêtes:

```typescript
declare global {
  var sandboxState: SandboxState;
  var conversationState: ConversationState | null;
  var activeSandboxProvider: any;
}

// Accessible dans toutes les API routes
global.conversationState = { ... };
```

**Attention:** Les variables globales persistent entre les requêtes en développement mais peuvent être réinitialisées en production (serverless).

### 3. Gestion d'Erreurs et Récupération

Le système inclut plusieurs mécanismes de récupération:

- **Truncation Recovery**: Détecte les réponses tronquées et demande la suite
- **Package Auto-Install**: Détecte les imports manquants et installe automatiquement
- **Vite Error Detection**: Surveille les logs pour identifier les problèmes
- **Fallback Search**: Si la recherche principale échoue, essaie une recherche de secours

### 4. Optimisations de Performance

- **Context Limiting**: Limite le contexte à 2000 caractères pour éviter les timeouts
- **Message Trimming**: Garde seulement les 15 derniers messages
- **Lazy Loading**: Les composants lourds sont chargés à la demande
- **Debouncing**: Les inputs sont debounced pour réduire les requêtes

---

## 🐛 Debugging et Troubleshooting

### Logs Utiles

```typescript
// Active les logs détaillés
console.log('[generate-ai-code-stream] Received request:', { prompt, isEdit });
console.log('[file-search] Search results:', results);
console.log('[apply-ai-code-stream] Detected packages:', packages);
```

### Outils de Développement

- **React DevTools**: Pour inspecter les composants
- **Next.js DevTools**: Pour voir les requêtes API
- **Network Tab**: Pour suivre le streaming SSE
- **Console**: Les logs sont préfixés par module

### Problèmes Courants

1. **"Sandbox not created"**: Vérifier les API keys (E2B ou Vercel)
2. **"Package not found"**: Vérifier que npm install fonctionne dans le sandbox
3. **"Truncated response"**: Augmenter maxTokens ou activer truncation recovery
4. **"Context too large"**: Réduire maxRecentMessagesContext dans config

---

## 🚀 Conclusion

**Open Lovable** est un système sophistiqué qui combine:
- 🤖 **IA multi-modèles** pour génération de code intelligente
- 🔍 **Recherche agentique** pour éditions chirurgicales précises
- 📦 **Sandboxes isolées** pour exécution sécurisée
- 🌐 **Web scraping** pour inspiration basée sur des sites réels
- 💬 **Conversation contextuelle** avec mémoire et préférences
- ⚡ **Streaming temps réel** pour feedback immédiat

**Points forts:**
- Architecture modulaire et extensible
- Prompts système ultra-détaillés et guidés par exemples
- Gestion intelligente du contexte et de la mémoire
- Auto-correction et récupération d'erreurs
- Support multi-providers (E2B, Vercel)

**Pour customiser:**
- Créer un nouveau SandboxProvider pour votre environnement
- Adapter les prompts système à vos besoins
- Ajouter vos propres validations et contraintes
- Documenter vos patterns avec des exemples concrets

**Ressources:**
- GitHub: https://github.com/mendableai/open-lovable
- Firecrawl: https://firecrawl.dev
- E2B: https://e2b.dev
- Vercel Sandbox: https://vercel.com/docs/sandbox

---

*Document créé le 23 octobre 2025*
*Dernière mise à jour: 23 octobre 2025*
