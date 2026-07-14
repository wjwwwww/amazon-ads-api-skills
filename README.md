# Amazon Ads API Migration Skills for Kiro

A collection of [Kiro](https://kiro.dev) skills that help developers migrate from legacy Amazon Ads APIs (SP v3, SB v4) to the Unified API (`/adsApi/v1/`).

## What Are Skills?

Kiro skills are structured knowledge files (SKILL.md) that teach AI assistants domain-specific expertise. When installed in your workspace, Kiro automatically uses these skills to provide accurate, context-aware guidance for API migration tasks.

## Available Skills

| Skill | Description | Use When |
|-------|-------------|----------|
| `unified-api-migration-guide` | High-level migration assessment and planning | Starting a migration, assessing scope |
| `unified-sp-migration` | SP v3 → Unified API field-level mapping | Migrating Sponsored Products code |
| `unified-sb-migration` | SB v4 → Unified API field-level mapping | Migrating Sponsored Brands code |
| `amazon-ads-sb-collections` | SBC ad format integration guide | Building new SBC campaigns or migrating from Product Collections |

## Quick Start

### Option 1: Install to Kiro CLI project (`.kiro/skills/`)

```bash
# Clone and install into your project
git clone https://github.com/YOUR_USER/amazon-ads-api-skills.git
cd amazon-ads-api-skills
./install.sh /path/to/your/project
```

Skills are installed to `/path/to/your/project/.kiro/skills/` and automatically loaded when you run `kiro` in that project directory.

### Option 2: Install a single skill

```bash
# Copy just the skill you need
mkdir -p /path/to/your/project/.kiro/skills
cp -r skills/unified-sp-migration /path/to/your/project/.kiro/skills/
```

### Option 3: Add to a custom AI agent / MCP agent

If you are building or configuring a custom agent (e.g., Amazon Q, Cursor, Cline, or other LLM-based agents), you can include the SKILL.md content as part of the agent's system prompt or context:

```python
# Example: Load skill as context for your agent
with open("skills/unified-sp-migration/SKILL.md", "r") as f:
    skill_content = f.read()

system_prompt = f"""
You are an Amazon Ads API migration assistant.

{skill_content}
"""
```

Or reference it in your agent configuration file:

```yaml
# Example: agent config (varies by platform)
agent:
  name: ads-api-migration-helper
  context_files:
    - skills/unified-api-migration-guide/SKILL.md
    - skills/unified-sp-migration/SKILL.md
    - skills/unified-sb-migration/SKILL.md
```

### How Skills Are Loaded

| Environment | Where to put skills | How they're loaded |
|-------------|--------------------|--------------------|
| **Kiro CLI** | `.kiro/skills/<name>/SKILL.md` in your project | Auto-loaded based on `description` field matching |
| **Kiro IDE** | Same `.kiro/skills/` directory | Auto-loaded in workspace context |
| **Custom Agent** | Anywhere accessible | Manually include as system prompt / context |
| **RAG pipeline** | Index the `skills/` directory | Retrieve relevant skill based on user query |
| **GitHub Copilot** | `.github/copilot-instructions.md` or custom instructions | Copy skill content into instructions |

### Choosing Which Skills to Install

| Your Use Case | Skills to Install |
|---------------|-------------------|
| Migrating SP campaigns | `unified-api-migration-guide` + `unified-sp-migration` |
| Migrating SB campaigns | `unified-api-migration-guide` + `unified-sb-migration` |
| Building new SBC ads | `amazon-ads-sb-collections` |
| Testing API endpoints | `unified-api-cli-testing` |
| Maintaining this repo | `update-migration-skills` |
| Everything | `./install.sh /path/to/project` (installs all) |

### Verify installation

After installation, test with these questions:

**Question 1** (migration scope):
> "I want to migrate from SP v3. Which endpoints have equivalents in adsApi/v1?"

Expected: The agent should list the full endpoint mapping table (campaigns, ad groups, product ads, keywords → targets, negative keywords → targets with negative=true, etc.) and clearly indicate which v3 endpoints do NOT have Unified API equivalents (budget rules, recommendations, etc.)

**Question 2** (targeting type differentiation):
> "Generate a curl command to create an SP campaign"

Expected: The agent should explain the Unified API uses `targetType` + `matchType` instead of expression format, and provide the mapping:
- `targetType: "PRODUCT"` + `matchType: "PRODUCT_EXACT"` = ASIN exact targeting
- `targetType: "PRODUCT"` + `matchType: "PRODUCT_SIMILAR"` = ASIN expanded targeting
- `targetType: "PRODUCT_CATEGORY"` = Category targeting

**Question 3** (API CLI Generation):
Expected:  The agent should return curl command to create an SP campaign using adsapi/v1 based on API Spec. 

## API Spec Sync

The migration skills reference the Amazon Ads OpenAPI specifications. Since these specs change frequently, we include a sync tool to detect changes that may affect the skills.

### Pull latest specs and check for changes

```bash
./scripts/sync-api-specs.sh
```

### CI mode (for GitHub Actions)

```bash
# Exits with code 1 if specs have changed (useful in CI pipelines)
./scripts/sync-api-specs.sh --check
```

### What it does

1. Downloads the latest OpenAPI specs from Amazon's CDN:
   - `unified-api-sp.json` — Sponsored Products Unified API
   - `unified-api-sb.json` — Sponsored Brands Unified API
2. Pretty-prints JSON for stable diffing
3. Compares with previous versions
4. Reports: new/removed endpoints, schema changes
5. Flags which skills need review

### GitHub Actions Example

```yaml
# .github/workflows/spec-sync.yml
name: API Spec Sync Check
on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9am UTC
  workflow_dispatch:

jobs:
  check-specs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check for API spec changes
        run: ./scripts/sync-api-specs.sh --check
      - name: Create issue on change
        if: failure()
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: '⚠️ API specs have changed — review skills',
              body: 'The weekly spec sync detected changes. Run `./scripts/sync-api-specs.sh` locally to see details.',
              labels: ['api-change', 'needs-review']
            })
```

## Project Structure

```
amazon-ads-api-skills/
├── README.md                     ← You are here
├── CONTRIBUTING.md               ← How to add/update skills
├── install.sh                    ← One-command installer
├── skills.json                   ← Machine-readable skill manifest
├── skills/
│   ├── unified-api-migration-guide/
│   │   └── SKILL.md
│   ├── unified-sp-migration/
│   │   └── SKILL.md
│   ├── unified-sb-migration/
│   │   └── SKILL.md
│   └── amazon-ads-sb-collections/
│       └── SKILL.md
├── api-specs/                    ← Downloaded OpenAPI specs (git-tracked)
│   ├── unified-api-sp.json
│   └── unified-api-sb.json
├── scripts/
│   └── sync-api-specs.sh        ← API spec sync tool
├── docs/
│   └── how-skills-work.md
└── examples/
    └── (usage examples)
```

## Requirements

- [Kiro CLI](https://kiro.dev) or Kiro IDE extension
- `curl`, `jq` (for spec sync script)
- bash 4+ (for associative arrays in sync script)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on:
- Adding new skills
- Updating existing skills when API specs change
- Skill template and quality standards

## License

MIT — see [LICENSE](LICENSE).
