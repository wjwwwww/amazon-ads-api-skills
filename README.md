# Amazon Ads API Assistance Skills

A collection of [Kiro](https://kiro.dev) skills that help developers with Amazon Ads API integration — including migrating from legacy APIs, adopting new ad formats, and generating test commands.

Official docs are the reference dictionary; this project distills scattered API specs into actionable, agent-consumable knowledge — covering migration, new feature integration, and test validation — so developers get executable answers instead of reading pages of specs. Note: LLMs can hallucinate; when in doubt, verify against [official documentation](https://advertising.amazon.com/API/docs/en-us/guides/overview) or contact the Ads API Support Center.

## Demo

https://github.com/user-attachments/assets/34fb1c60-a76d-410a-a55a-bcb2cbed4679

## Available Skills

These skills cover three key aspects of API integration:

| Category | Skill | Use When |
|----------|-------|----------|
| **Migration** | `unified-api-migration-guide` | Assessing migration scope from SP v3 / SB v4 |
| **Migration** | `unified-sp-migration` | Migrating Sponsored Products to Unified API |
| **Migration** | `unified-sb-migration` | Migrating Sponsored Brands to Unified API |
| **New API Adoption** | `amazon-ads-sb-collections` | Building SB Collections campaigns |
| **New API Adoption** | `amazon-ads-spglobal` | Managing SP campaigns across multiple marketplaces |
| **CLI Testing** | `unified-api-cli-testing` | Generating curl commands and test scripts |

## Quick Start

### Prerequisites

1. Install [Kiro CLI](https://kiro.dev):
   ```bash
   brew install kiro-cli
   ```
   Or download from [kiro.dev/downloads](https://kiro.dev/downloads).

2. Verify:
   ```bash
   kiro-cli --version
   ```

### Install & Use

```bash
# Clone this repo
git clone https://github.com/wjwwwww/amazon-ads-api-skills.git
cd amazon-ads-api-skills

# Install skills + agent into your project
./install.sh /path/to/your/project

# Start using
cd /path/to/your/project
kiro-cli chat --agent ads-api-migration-assistant
```

This installs skills (to `.kiro/skills/`) and the migration agent (to `.kiro/agents/`).

### Or install a single skill

```bash
mkdir -p /path/to/your/project/.kiro/skills
cp -r skills/unified-sp-migration /path/to/your/project/.kiro/skills/
```

### Verify it works

After installation, try these questions:

> "I want to migrate from SP v3. Which endpoints have equivalents in adsApi/v1?"

> "Generate a curl command to create an SP Global campaign across US, UK, and DE"

> "How do I create a Sponsored Brands Collection ad?"

## How It Works

The agent uses a two-layer knowledge system:

```
User asks question
        │
        ▼
┌─────────────────┐
│  SKILL.md files │ ← Primary (loaded as context)
│  (guides, maps) │
└────────┬────────┘
         │ Need more detail?
         ▼
┌─────────────────────────────┐
│  api-specs/*.json           │ ← Fallback (searched on demand)
│  (full OpenAPI schemas)     │
└─────────────────────────────┘
```

Skills provide migration guides, endpoint mappings, and code examples. When the agent needs precise field types, enum values, or schema details, it automatically searches the OpenAPI spec files.

## API Spec Sync

The specs change frequently. Keep them up-to-date:

```bash
./scripts/sync-api-specs.sh
```

This downloads the latest specs from Amazon's CDN, compares with previous versions, and reports changes that may affect skills.

For CI integration, use `--check` mode (exits with code 1 if specs changed):

```bash
./scripts/sync-api-specs.sh --check
```

## Project Structure

```
amazon-ads-api-skills/
├── .kiro/agents/                    ← Agent config (with API Spec Fallback)
├── skills/                          ← Skill source files (SKILL.md each)
├── api-specs/                       ← OpenAPI specs (auto-synced)
│   ├── unified-api-sp.json
│   ├── unified-api-sb.json
│   ├── unified-api-spglobal.json
│   └── enums-unified-api.json
├── scripts/                         ← Sync & maintenance tools
├── install.sh                       ← One-command installer
├── skills.json                      ← Machine-readable manifest
└── README.md
```

## Using with Other AI Tools

You can also use the SKILL.md files with non-Kiro AI tools:

```python
# Load as context for any LLM agent
with open("skills/unified-sp-migration/SKILL.md", "r") as f:
    skill_content = f.read()
```

```yaml
# Or reference in agent config (Cursor, Cline, etc.)
context_files:
  - skills/unified-sp-migration/SKILL.md
  - skills/unified-sb-migration/SKILL.md
```

## Requirements

- [Kiro CLI](https://kiro.dev) (or any AI coding assistant that supports context files)
- `curl`, `jq` (for spec sync script)
- bash 4+

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
