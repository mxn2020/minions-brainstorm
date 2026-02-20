**MINIONS BRAINSTORM — IMPLEMENTATION SPEC**

You are tasked with creating the complete initial foundation for `minions-brainstorm` — a structured thinking and ideation system designed for divergent and convergent exploration. This is part of the Minions ecosystem, a universal structured object system designed for building AI-native tools.

---

**PROJECT OVERVIEW**

`minions-brainstorm` provides structured brainstorming, mind mapping, and ideation workflows. It enables tracking of idea lineage through `inspired_by` relations, clustering related thoughts, measuring exploration entropy, and promoting ideas to more structured formats like notes or documents.

The core concept: ideas should be captured, connected, explored in multiple directions, then converged and refined. Agents can use this to explore problem spaces, generate variations, cluster related concepts, and identify the most promising directions.

---

**CONCEPT OVERVIEW**

This project is built on the Minions SDK (`minions-sdk`), which provides the foundational primitives: Minion (structured object instance), Minion Type (schema), and Relation (typed link between minions).

Ideas can inspire other ideas via `inspired_by` relations, forming inspiration chains. Thoughts can be clustered by tags or content similarity. Connections explicitly link related concepts. Clusters group related ideas. Insights represent synthesized conclusions from multiple ideas.

The system supports both TypeScript and Python SDKs with cross-language interoperability (both serialize to the same JSON format). All documentation includes dual-language code examples with tabbed interfaces.

---

**CORE PRIMITIVES**

This project defines the following Minion Types:

- `idea` — A single atomic idea or concept with optional context and sources
- `thought` — A free-form thinking fragment, less structured than an idea
- `connection` — An explicit link between two ideas with a relationship description
- `cluster` — A named grouping of related ideas, thoughts, or connections
- `insight` — A synthesized conclusion or realization derived from multiple ideas

---

**MINIONS SDK REFERENCE — REQUIRED DEPENDENCY**

This project depends on `minions-sdk`, a published package that provides the foundational primitives. The GH Agent building this project MUST install it from the public registries and use the APIs documented below — do NOT reimplement minions primitives from scratch.

**Installation:**
```bash
# TypeScript (npm)
npm install minions-sdk
# or: pnpm add minions-sdk

# Python (PyPI) — package name is minions-sdk, but you import as "minions"
pip install minions-sdk
```

**TypeScript SDK — Core Imports:**
```typescript
import {
  // Core types
  type Minion, type MinionType, type Relation,
  type FieldDefinition, type FieldValidation, type FieldType,
  type CreateMinionInput, type UpdateMinionInput, type CreateRelationInput,
  type MinionStatus, type MinionPriority, type RelationType,
  type ExecutionResult, type Executable,
  type ValidationError, type ValidationResult,

  // Validation
  validateField, validateFields,

  // Built-in Schemas (10 MinionType instances — reuse where applicable)
  noteType, linkType, fileType, contactType,
  agentType, teamType, thoughtType, promptTemplateType, testCaseType, taskType,
  builtinTypes,

  // Registry — stores and retrieves MinionTypes by id or slug
  TypeRegistry,

  // Relations — in-memory directed graph with traversal utilities
  RelationGraph,

  // Lifecycle — CRUD operations with validation
  createMinion, updateMinion, softDelete, hardDelete, restoreMinion,

  // Evolution — migrate minions when schemas change (preserves removed fields in _legacy)
  migrateMinion,

  // Utilities
  generateId, now, SPEC_VERSION,
} from 'minions-sdk';
```

**Python SDK — Core Imports:**
```python
from minions import (
    # Types
    Minion, MinionType, Relation, FieldDefinition, FieldValidation,
    CreateMinionInput, UpdateMinionInput, CreateRelationInput,
    ExecutionResult, Executable, ValidationError, ValidationResult,
    # Validation
    validate_field, validate_fields,
    # Built-in Schemas (10 types)
    note_type, link_type, file_type, contact_type,
    agent_type, team_type, thought_type, prompt_template_type,
    test_case_type, task_type, builtin_types,
    # Registry
    TypeRegistry,
    # Relations
    RelationGraph,
    # Lifecycle
    create_minion, update_minion, soft_delete, hard_delete, restore_minion,
    # Evolution
    migrate_minion,
    # Utilities
    generate_id, now, SPEC_VERSION,
)
```

**Key Concepts:**
- A `MinionType` defines a schema (list of `FieldDefinition`s) — each field has `name`, `type`, `label`, `required`, `defaultValue`, `options`, `validation`
- A `Minion` is an instance with `id`, `title`, `minionTypeId`, `fields` (dict), `status`, `tags`, timestamps
- A `Relation` is a typed directional link (12 types: `parent_of`, `depends_on`, `implements`, `relates_to`, `inspired_by`, `triggers`, `references`, `blocks`, `alternative_to`, `part_of`, `follows`, `integration_link`)
- Field types: `string`, `number`, `boolean`, `date`, `select`, `multi-select`, `url`, `email`, `textarea`, `tags`, `json`, `array`
- `TypeRegistry` auto-loads 10 built-in types; register custom types with `registry.register(myType)`
- `createMinion(input, type)` validates fields against the schema and returns `{ minion, validation }` (TS) or `(minion, validation)` tuple (Python)
- Both SDKs serialize to identical camelCase JSON; Python provides `to_dict()` / `from_dict()` for conversion

**IMPORTANT:** Do NOT recreate these primitives. Import them from `minions-sdk` (npm) / `minions` (PyPI). Build your domain-specific types and utilities ON TOP of the SDK.

---

**WHAT YOU NEED TO CREATE**

**1. THE SPECIFICATION** (`/spec`)

Write a complete markdown specification document covering:

- Motivation and goals — why structured ideation matters for creative and analytical work
- Glossary of terms specific to brainstorming and ideation
- Core type definitions for all five minion types with full field schemas
- Inspiration chain semantics — how `inspired_by` relations form idea lineage
- Divergence vs. convergence modes — exploring outward vs. synthesizing inward
- Clustering algorithms — tag-based, semantic similarity, manual grouping
- Entropy scoring — measuring breadth of exploration across tags and themes
- Promotion workflows — converting ideas to notes or other structured formats
- Integration patterns with `minions-notes` and `minions-docs`
- Best practices for agent-driven ideation
- Conformance checklist for implementations

**2. THE CORE LIBRARY** (`/packages/core`)

A framework-agnostic TypeScript library built on `minions-sdk`. Must include:

- Full TypeScript type definitions for all brainstorm-specific types
- `ThoughtGraph` class extending `RelationGraph` from `minions-sdk`
  - `diverge(seedId, depth)` — traverse `inspired_by` chains outward from seed idea
  - `converge(ideaIds[])` — find common ancestor ideas across multiple branches
  - `findOrphans()` — identify ideas with no relations
  - `getLineage(ideaId)` — get full inspiration ancestry
- `IdeaClusterer` class — group related ideas
  - `clusterByTags(ideas[])` — group ideas sharing tags
  - `clusterBySimilarity(ideas[], threshold)` — semantic clustering (requires embeddings)
  - `manualCluster(ideaIds[], clusterName)` — create explicit cluster minion
- `EntropyCalculator` utility — measure exploration breadth
  - `calculateEntropy(sessionId)` — score based on tag diversity and relation patterns
  - `diversityScore(ideas[])` — measure unique themes explored
  - `depthScore(ideas[])` — measure how deep inspiration chains go
- `IdeaPromoter` class — convert ideas to other formats
  - `promoteToNote(ideaId)` — create `note` minion from idea
  - `promoteToDocument(ideaIds[])` — create structured document from idea cluster
  - `exportToMarkdown(clusterIds[])` — export clusters as markdown
- `BrainstormSession` class — manage brainstorming sessions
  - `createSession(topic)` — initialize new session
  - `recordIdea(sessionId, content)` — add idea to session
  - `getSessionStats(sessionId)` — count ideas, connections, clusters
- Clean public API with comprehensive JSDoc documentation
- Zero storage opinions — works with any backend

**3. THE PYTHON SDK** (`/packages/python`)

A complete Python port of the core library with identical functionality:

- Python type hints for all classes and methods
- `ThoughtGraph`, `IdeaClusterer`, `EntropyCalculator`, `IdeaPromoter`, `BrainstormSession` classes
- Same method signatures as TypeScript version (following Python naming conventions)
- Serializes to identical JSON format as TypeScript SDK (cross-language interoperability)
- Full docstrings compatible with Sphinx documentation generation
- Published to PyPI as `minions-brainstorm`

**4. THE CLI** (`/packages/cli`)

A command-line tool called `brainstorm` that provides:

```bash
brainstorm new "Product pricing models"
# Create new brainstorm session with topic

brainstorm idea add <session-id> "Freemium with usage tiers"
# Add idea to session

brainstorm expand <idea-id>
# Generate related ideas (interactive prompt or AI-assisted)

brainstorm connect <id-a> <id-b> "Both target SMB market"
# Create explicit connection between ideas

brainstorm cluster <session-id>
# Auto-cluster ideas by tags

brainstorm entropy <session-id>
# Calculate exploration entropy score

brainstorm lineage <idea-id>
# Show inspiration ancestry

brainstorm promote <idea-id> --to note
# Convert idea to note minion

brainstorm export <session-id> --format markdown
# Export session as markdown
```

Additional features:
- Interactive mode for rapid idea capture
- Visual tree output for inspiration chains
- Tag-based filtering and search
- Session statistics dashboard
- JSON output mode for programmatic usage
- Config file support (`.brainstormrc.json`)

**5. THE DOCUMENTATION SITE** (`/apps/docs`)

Built with Astro Starlight. Must include:

- Landing page — "Structured thinking for humans and agents" positioning
- Getting started guide with both TypeScript and Python examples
- Core concepts:
  - Ideas vs. thoughts vs. insights
  - Inspiration chains and lineage
  - Divergent vs. convergent exploration
  - Clustering strategies
  - Entropy and exploration metrics
- API reference for both TypeScript and Python
  - Dual-language code tabs for all examples
  - Auto-generated from JSDoc/docstrings where possible
- Guides:
  - Running an effective brainstorm session
  - Using entropy to know when to converge
  - Agent-assisted ideation workflows
  - Promoting ideas to structured formats
  - Integrating with notes and documents
- CLI reference with example commands
- Integration examples:
  - Agent divergent exploration (generate variations)
  - Agent convergent synthesis (find common themes)
  - Brainstorm-to-document pipeline
  - Collaborative brainstorming
- Best practices for structured ideation
- Contributing guide

**6. OPTIONAL: THE WEB APP** (`/apps/web`)

A visual brainstorming canvas (optional but recommended):

- Infinite canvas for idea placement
- Drag-and-drop idea cards
- Visual inspiration chains (arrows between ideas)
- Real-time clustering visualization
- Entropy meter showing exploration breadth
- Export interface (markdown, JSON, note promotion)
- Built with Next.js or SvelteKit

---

**PROJECT STRUCTURE**

Standard Minions ecosystem monorepo structure:

```
minions-brainstorm/
  packages/
    core/                 # TypeScript core library
      src/
        types.ts          # Type definitions
        ThoughtGraph.ts
        IdeaClusterer.ts
        EntropyCalculator.ts
        IdeaPromoter.ts
        BrainstormSession.ts
        index.ts          # Public API surface
      test/
      package.json
    python/               # Python SDK
      minions_brainstorm/
        __init__.py
        types.py
        thought_graph.py
        idea_clusterer.py
        entropy_calculator.py
        idea_promoter.py
        brainstorm_session.py
      tests/
      pyproject.toml
    cli/                  # CLI tool
      src/
        commands/
          new.ts
          idea.ts
          expand.ts
          connect.ts
          cluster.ts
          entropy.ts
          lineage.ts
          promote.ts
          export.ts
        index.ts
      package.json
  apps/
    docs/                 # Astro Starlight documentation
      src/
        content/
          docs/
            index.md
            getting-started.md
            concepts/
            guides/
            api/
              typescript/
              python/
            cli/
      astro.config.mjs
      package.json
    web/                  # Optional canvas app
      src/
      package.json
  spec/
    v0.1.md              # Full specification
  examples/
    typescript/
      simple-session.ts
      diverge-converge.ts
      cluster-promote.ts
    python/
      simple_session.py
      diverge_converge.py
      cluster_promote.py
  .github/
    workflows/
      ci.yml             # Lint, test, build for both TS and Python
      publish.yml        # Publish to npm and PyPI
  README.md
  LICENSE                # AGPL-3.0
  package.json           # Workspace root
```

---

**BEYOND STANDARD PATTERN**

These utilities and classes are specific to `@minions-brainstorm/sdk`:

**ThoughtGraph**
- Extends `RelationGraph` from `minions-sdk`
- `diverge(seedId, depth)` traverses `inspired_by` relations outward
- `converge(ideaIds[])` finds lowest common ancestor in inspiration chains
- `findOrphans()` identifies ideas with no incoming or outgoing relations
- `getLineage(ideaId)` returns full ancestry path to root idea

**IdeaClusterer**
- `clusterByTags()` groups ideas sharing tag overlap above threshold
- `clusterBySimilarity()` uses embeddings for semantic grouping (optional embedding provider)
- `manualCluster()` creates explicit `cluster` minion linking ideas via `part_of` relations
- Returns cluster statistics: size, density, average tag overlap

**EntropyCalculator**
- `calculateEntropy()` measures exploration breadth using Shannon entropy on tag distributions
- `diversityScore()` counts unique themes/tags explored
- `depthScore()` measures average and max depth of inspiration chains
- Higher entropy = broader exploration, lower entropy = focused refinement

**IdeaPromoter**
- `promoteToNote()` creates `note` minion from idea, linking via `relates_to`
- `promoteToDocument()` generates structured document from cluster of ideas
- `exportToMarkdown()` renders clusters as markdown with hierarchy
- Preserves relations and metadata in promoted formats

**BrainstormSession**
- Container for managing related ideas within a single brainstorming context
- `createSession(topic)` returns session ID
- `recordIdea()` adds idea and links to session via `part_of`
- `getSessionStats()` returns counts and metrics for session

---

**CLI COMMANDS**

All commands with detailed specifications:

**`brainstorm new <topic>`**
- Creates new brainstorm session
- Accepts topic as title
- Optional flags: `--tags`, `--description`
- Returns session ID

**`brainstorm idea add <session-id> <content>`**
- Adds new idea to session
- Links via `part_of` relation to session
- Optional flags: `--tags`, `--inspired-by <id>`
- Returns idea ID

**`brainstorm expand <idea-id>`**
- Interactive mode to generate related ideas
- Prompts for variations, alternatives, extensions
- Each new idea links via `inspired_by` to source
- Can integrate with AI for suggestion generation

**`brainstorm connect <id-a> <id-b> [description]`**
- Creates explicit `connection` minion
- Links both ideas via `relates_to`
- Description explains relationship
- Returns connection ID

**`brainstorm cluster <session-id>`**
- Auto-clusters ideas in session by tag similarity
- Creates `cluster` minions for each group
- Links ideas to clusters via `part_of`
- Displays cluster statistics

**`brainstorm entropy <session-id>`**
- Calculates and displays entropy score
- Shows tag distribution histogram
- Reports depth and breadth metrics
- Suggests when to shift from diverge to converge

**`brainstorm lineage <idea-id>`**
- Displays full inspiration ancestry
- Tree format showing `inspired_by` chain
- Includes timestamps and authors
- Can output as JSON graph

**`brainstorm promote <idea-id> --to <note|document>`**
- Converts idea to specified format
- Creates new minion of target type
- Links via `relates_to` relation
- Preserves tags and metadata

**`brainstorm export <session-id> --format <markdown|json>`**
- Exports session to specified format
- Markdown includes hierarchy and clusters
- JSON includes full graph structure
- Outputs to stdout or file

---

**DUAL SDK REQUIREMENTS**

Critical cross-language compatibility requirements:

**Serialization Parity**
- Both TypeScript and Python SDKs must serialize minions to identical JSON format
- Field names, types, and structure must match exactly
- Relation types and metadata must be interchangeable

**API Consistency**
- Same method names (adjusted for language conventions: TypeScript camelCase, Python snake_case)
- Same parameters and return types
- Same class hierarchies and interfaces

**Documentation Parity**
- Every code example in docs must have both TypeScript and Python versions
- Use Astro Starlight's code tabs: `<Tabs><TabItem label="TypeScript">...</TabItem><TabItem label="Python">...</TabItem></Tabs>`
- API reference must document both languages side by side

**Testing Parity**
- Shared test fixtures (JSON files) that both SDKs can consume
- Identical test case coverage
- Cross-language integration tests (TypeScript SDK creates minion, Python SDK reads it)

---

**FIELD SCHEMAS**

Define these Minion Types with full JSON Schema definitions:

**`idea`**
```typescript
{
  id: string;
  title: string;
  content: string;              // The idea description
  context?: string;             // Background or rationale
  source?: string;              // Where the idea came from
  tags?: string[];
  confidence?: number;          // 0-100 confidence in idea quality
  status?: 'raw' | 'refined' | 'validated' | 'promoted';
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `inspired_by` → another `idea`
Relations: `part_of` → `cluster` or `brainstorm-session`

**`thought`**
```typescript
{
  id: string;
  title: string;
  content: string;              // Free-form thinking
  stream?: string;              // Associated thought stream or session
  tags?: string[];
  createdAt: Date;
  updatedAt: Date;
}
```

**`connection`**
```typescript
{
  id: string;
  title: string;
  description: string;          // Nature of the relationship
  connectionType?: 'similar' | 'opposite' | 'builds-on' | 'alternative' | 'prerequisite';
  strength?: number;            // 0-100 strength of connection
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `relates_to` → two `idea` or `thought` minions

**`cluster`**
```typescript
{
  id: string;
  title: string;
  description?: string;
  clusteringMethod?: 'manual' | 'tag-based' | 'semantic';
  tags?: string[];
  statistics?: {                // Computed stats
    size: number;
    avgTagOverlap: number;
    density: number;
  };
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `part_of` ← multiple `idea` or `thought` minions

**`insight`**
```typescript
{
  id: string;
  title: string;
  content: string;              // Synthesized conclusion
  synthesis: string;            // How this was derived
  sourceCount?: number;         // Number of ideas this synthesizes
  confidence?: number;          // 0-100 confidence in insight
  tags?: string[];
  createdAt: Date;
  updatedAt: Date;
}
```
Relations: `references` → multiple `idea` minions that contributed to insight

---

**TONE AND POSITIONING**

This is a tool for structured creative thinking. Position it as:

- **Divergent exploration with convergent synthesis** — both generate and refine
- **Lineage tracking** — understand where ideas came from
- **Entropy-guided exploration** — know when you've explored enough
- **Agent-native ideation** — agents can brainstorm autonomously

Avoid:
- Generic mind mapping language
- Over-promising on creativity enhancement
- Complexity without clear value

The README should open with a concrete example: a brainstorm session, divergence from a seed idea, clustering, entropy calculation, and promotion to notes.

---

**INTEGRATION EXAMPLES**

Include working examples for:

**Agent Divergent Exploration** (TypeScript)
```typescript
import { ThoughtGraph, BrainstormSession } from '@minions-brainstorm/sdk';

const session = new BrainstormSession();
const sessionId = await session.createSession('AI Agent Architectures');
const seedId = await session.recordIdea(sessionId, 'Modular agent with swappable tools');

const graph = new ThoughtGraph();
const variations = await graph.diverge(seedId, 2); // 2 levels deep
// Returns all ideas inspired by seed, and ideas inspired by those
```

**Entropy-Guided Convergence** (Python)
```python
from minions_brainstorm import EntropyCalculator, IdeaClusterer

calculator = EntropyCalculator()
entropy = calculator.calculate_entropy(session_id)

if entropy > 0.8:  # High entropy = broad exploration
    print("Good divergence. Ready to converge.")
    clusterer = IdeaClusterer()
    clusters = clusterer.cluster_by_tags(idea_ids)
    # Synthesize insights from clusters
```

---

**DELIVERABLES**

Produce all files necessary to bootstrap this project completely:

1. **Full specification** (`/spec/v0.1.md`) — complete enough to implement from
2. **TypeScript core library** (`/packages/core`) — fully functional, well-tested
3. **Python SDK** (`/packages/python`) — feature parity with TypeScript
4. **CLI tool** (`/packages/cli`) — all commands working with helpful output
5. **Documentation site** (`/apps/docs`) — complete with dual-language examples
6. **README** — compelling, clear, with concrete examples
7. **Examples** — working code in both TypeScript and Python
8. **CI/CD setup** — lint, test, and publish workflows for both languages

Every file should be production quality — not stubs, not placeholders. The spec should be complete. The core libraries should be fully functional. The docs should be ready to publish. The CLI should be ready to install and use.

---

**START SYSTEMATICALLY**

1. Write the specification first — nail down the field schemas and graph algorithms
2. Implement TypeScript core library with full type definitions
3. Port to Python maintaining exact serialization compatibility
4. Build CLI using the core library
5. Write documentation with dual-language examples throughout
6. Create working examples demonstrating diverge/converge workflows
7. Write the README with concrete ideation use cases

This is a foundational tool for creative and analytical thinking in the Minions ecosystem. Agents can use this to explore problem spaces systematically. Get it right.
