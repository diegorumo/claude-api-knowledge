# Skills API

> **Last updated:** 2026-08-31  
> **Status:** GA (out of beta as of 2026-08-19)  
> **SDK support:** Python v0.121.0+ / TypeScript v0.116.0+

## Overview

The Skills API lets you upload reusable, versioned skill packages (collections of files including a `SKILL.md` descriptor) that can be referenced by Managed Agents sessions. Skills extend what an agent can do without modifying the agent definition itself.

Two skill sources exist:
- **Anthropic skills** — curated skills maintained by Anthropic (e.g., `"xlsx"`, `"pdf"`)
- **Custom skills** — your own packaged skills uploaded via the API

## Endpoints

| Operation | Endpoint | Description |
|-----------|----------|-------------|
| `create` | `POST /v1/skills` (multipart) | Upload skill files; creates a new skill |
| `retrieve` | `GET /v1/skills/{id}` | Get skill metadata |
| `list` | `GET /v1/skills` | List all skills (paginated) |
| `delete` | `DELETE /v1/skills/{id}` | Delete a skill |

## Skill Object

```json
{
  "id": "skill_01XJ5...",
  "type": "skill",
  "source": "custom",
  "display_title": "My Analysis Skill",
  "latest_version": "1.0.0",
  "created_at": "2026-08-10T12:00:00Z",
  "updated_at": "2026-08-10T12:00:00Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | `str` | Unique skill ID (`skill_01…`). Format may change. |
| `type` | `"skill"` | Always `"skill"` |
| `source` | `"custom" \| "anthropic"` | Origin of the skill |
| `display_title` | `str \| null` | Human-readable label (not included in prompts) |
| `latest_version` | `str \| null` | Most recent version identifier |
| `created_at` | `str` | ISO 8601 creation timestamp |
| `updated_at` | `str` | ISO 8601 last-modified timestamp |

## Creating a Skill

A skill upload is a **multipart form upload**. All files must be in the same top-level directory and must include a `SKILL.md` at the root of that directory.

### SKILL.md format

`SKILL.md` is the skill descriptor that the model reads. It typically describes what the skill does, how to invoke it, and any parameters.

```markdown
# My Analysis Skill

## Purpose
Perform statistical analysis on tabular data.

## Usage
Call the `analyze_data` tool with a CSV string.
```

### Python

```python
import anthropic

client = anthropic.Anthropic()

# GA: use client.skills (not client.beta.skills)
with open("skill/SKILL.md", "rb") as skill_md, \
     open("skill/helpers.py", "rb") as helpers:
    skill = client.skills.create(
        files=[
            ("files", ("SKILL.md", skill_md, "text/markdown")),
            ("files", ("helpers.py", helpers, "text/plain")),
        ],
        display_title="My Analysis Skill",
    )

print(skill.id, skill.latest_version)  # skill_01... 1.0.0
```

### TypeScript

```typescript
import Anthropic from '@anthropic-ai/sdk';
import { readFileSync } from 'fs';

const client = new Anthropic();

// GA: use client.skills (not client.beta.skills)
const skill = await client.skills.create({
  files: [
    new File([readFileSync('skill/SKILL.md')], 'SKILL.md', { type: 'text/markdown' }),
    new File([readFileSync('skill/helpers.py')], 'helpers.py', { type: 'text/plain' }),
  ],
  display_title: 'My Analysis Skill',
});

console.log(skill.id, skill.latest_version);
```

## Listing Skills

```python
# GA: use client.skills (not client.beta.skills)
skills = client.skills.list()
for skill in skills.data:
    print(skill.id, skill.source, skill.display_title)
```

## Using Skills in Sessions

Once uploaded, reference skills when creating a Managed Agents session. See [Managed Agents](./managed-agents.md) for the full session API.

### Anthropic skills (built-in)

```python
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    skills=[
        {
            "type": "anthropic",
            "skill_id": "xlsx",        # Anthropic-managed skill ID
            "version": "latest",       # optional; defaults to latest
        },
    ],
)
```

### Custom skills (user-uploaded)

```python
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    skills=[
        {
            "type": "custom",
            "skill_id": "skill_01XJ5...",  # your skill ID from upload
            "version": "1.0.0",            # optional; pin to version
        },
    ],
)
```

You can mix Anthropic and custom skills in the same session.

## GitHub Repository Auto-Loading

Sessions can pull skills directly from a GitHub repository via `BetaManagedAgentsGitHubRepositoryResourceConfig`. The session automatically loads skills found in the repo.

```python
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    github_repository={
        "type": "github",
        # GitHub repo details configured server-side via the resource config
    },
)
```

> **Note:** GitHub auto-loading requires the repository resource config to be set up on the environment or agent. Consult the Managed Agents docs for full details.

## Gotchas

- All files must be in the same top-level directory; the `SKILL.md` must be at the root of that directory
- `display_title` is for human-readable UI purposes only — it is NOT included in the system prompt or model context
- Custom skill IDs start with `skill_01…`; format may change over time
- Uploading a new version of a skill creates a new `latest_version`; existing pinned sessions are unaffected
- **GA as of 2026-08-19**: no beta header required; use `client.skills.*` (not `client.beta.skills.*`) in SDK v1.0.0+
- **SDK v1.2.0+ (Python) / v0.122.0+ (TypeScript)**: `client.beta.skills.*` now uses the GA shape and no longer sends the `skills-2025-10-02` beta header. `client.beta.skills.delete()` now deletes **all versions** of the skill. The beta Messages type `BetaSkill` (the container Skill reference) is renamed **`BetaContainerSkill`**. Update any code that imports or references `BetaSkill` for the container context.
- Requests that send the old `skills-2025-10-02` beta header continue to work and return the previous shape
- Check SDK changelogs for new capabilities

## Related

- [Managed Agents](./managed-agents.md)
- [Tool Use](./tool-use.md)
- [Files API](./files-api.md)
