# Amazon Ads API Assistance Skills

A collection of [Kiro](https://kiro.dev) skills that help developers with Amazon Ads API integration — including migrating from legacy APIs, adopting new ad formats, and generating test commands.

## Demo

https://github.com/user-attachments/assets/34fb1c60-a76d-410a-a55a-bcb2cbed4679

## What Are Skills?

Kiro skills are structured knowledge files (SKILL.md) that teach AI assistants domain-specific expertise. When installed in your workspace, Kiro automatically uses these skills to provide accurate, context-aware guidance for API migration tasks.

## Available Skills

These skills cover three key aspects of API integration:

### a. Unified API Migration

Help developers migrate from legacy APIs (SP v3, SB v4) to the Unified API (`/adsApi/v1/`).

| Skill | Description | Use When |
|-------|-------------|----------|
| `unified-api-migration-guide` | High-level migration assessment and planning | Starting a migration, assessing scope |
| `unified-sp-migration` | SP v3 → Unified API field-level mapping | Migrating Sponsored Products code |
| `unified-sb-migration` | SB v4 → Unified API field-level mapping | Migrating Sponsored Brands code |

### b. New Ads API Adoption

Help developers integrate new Amazon Ads API features and ad formats.

| Skill | Description | Use When |
|-------|-------------|----------|
| `amazon-ads-sb-collections` | SBC ad format integration guide | Building new SBC campaigns or migrating from Product Collections |

### c. CLI Generation for Testing

Help developers generate and validate API requests.

| Skill | Description | Use When |
|-------|-------------|----------|
| `unified-api-cli-testing` | Generate curl commands and test scripts from OpenAPI specs | Testing endpoints, validating payloads |

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

### Verify installation

After installation, test with these questions:

**Question 1** (migration scope):
> "I want to migrate from SP v3. Which endpoints have equivalents in adsApi/v1?"

Expected: The agent should list the full endpoint mapping table (campaigns, ad groups, product ads, keywords → targets, negative keywords → targets with negative=true, etc.) and clearly indicate which v3 endpoints do NOT have Unified API equivalents (budget rules, recommendations, etc.)

**Question 2** (targeting type differentiation):
> "Unable to distinguish between different targeting types (e.g., ASIN exact, broad, category, etc.) in ads API V1"

Expected: The agent should explain the Unified API uses `targetType` + `matchType` instead of expression format, and provide the mapping:
- `targetType: "PRODUCT"` + `matchType: "PRODUCT_EXACT"` = ASIN exact targeting
- `targetType: "PRODUCT"` + `matchType: "PRODUCT_SIMILAR"` = ASIN expanded targeting
- `targetType: "PRODUCT_CATEGORY"` = Category targeting

**Question 3** (API CLI Generation):
> "Generate a curl command to create an SP campaign"
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

## Project Structure

```
amazon-ads-api-skills/
├── .kiro/
│   └── agents/
│       └── ads-api-migration-assistant.json  ← Agent config with API Spec Fallback
├── skills/                                   ← Skill source files
│   ├── unified-api-migration-guide/
│   │   └── SKILL.md
│   ├── unified-sp-migration/
│   │   └── SKILL.md
│   ├── unified-sb-migration/
│   │   └── SKILL.md
│   ├── amazon-ads-sb-collections/
│   │   └── SKILL.md
│   ├── unified-api-cli-testing/
│   │   └── SKILL.md
│   └── update-migration-skills/
│       └── SKILL.md                          ← Maintenance skill for updating skills from spec diffs
├── api-specs/                                ← OpenAPI specs (source of truth)
│   ├── unified-api-sp.json                   ← Sponsored Products Unified API
│   ├── unified-api-sb.json                   ← Sponsored Brands Unified API
│   ├── enums-unified-api.json                ← Extracted enum value mappings
│   └── diff/                                 ← Generated diffs after sync
│       ├── unified-api-sp.changes.md
│       ├── unified-api-sp.old.json
│       └── unified-api-sp.new.json
├── scripts/
│   ├── sync-api-specs.sh                     ← Download & diff latest specs
│   ├── extract-enums.sh                      ← Extract enum values from docs
│   └── fetch-enums-page.py                   ← Helper for enum extraction
├── docs/
├── examples/
├── install.sh                                ← One-command installer (skills + agent)
├── skills.json                               ← Machine-readable skill manifest
├── CONTRIBUTING.md
├── LICENSE
└── README.md                                 ← You are here
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
