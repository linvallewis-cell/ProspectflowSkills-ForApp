---
name: find-skills
description: Helps users discover relevant skills and prepare selected ones for Enter's draft-confirm flow when they ask questions like "how do I do X", "find a skill for X", "is there a skill that can...", or express interest in extending capabilities.
---

# Find Skills

This skill helps you discover skills from the open agent skills ecosystem and, in Enter, prepare selected ones for preview/save through the draft-confirm flow.

## Enter Workflow Additions

When this skill is used inside Enter, keep the original discovery and recommendation flow, but switch to Enter's draft-confirm preview flow after the user chooses which skill or skills they want.

- Search and recommend skills the usual way first. Do not skip the leaderboard, search, or quality checks.
- Treat requests like "install", "add", "enable", "use this skill in Enter", or "add it to the current workspace" as Enter install intent.
- The user may choose one skill or multiple skills from the recommended options.
- Before preparing each selected skill for Enter's preview/save flow, call `list_skills(scope="workspace")` to inspect the current workspace-visible skills.
- Compare each selected skill's exact `skill_key` against that workspace-visible list.
- If a selected skill already exists in the workspace, tell the user it already exists and skip installation for that skill.
- If a selected skill does not exist, you MUST prepare it as a `custom` draft under `/workspace/skill-drafts/custom/{skill_key}@1/` before doing anything else.
- When preparing a selected skill for Enter, the only filesystem location you may write is `/workspace/skill-drafts/custom/{skill_key}@1/`.
- That is the physical filesystem draft directory. When you call `confirm_skill`, pass the logical path `skill-drafts/custom/{skill_key}@1/` instead of the filesystem path.
- `confirm_skill` only hands the draft to Enter's Save / Discard UI. It does not publish or persist the skill by itself.
- Before the user clicks Save, the selected skill is still only a draft. Do not say it is already installed, enabled, added to the workspace, or persisted.
- Do not try to invent or paraphrase the draft metadata in tool args; Enter resolves the canonical `name` and `description` from `SKILL.md`.
- If the user selected multiple skills, process them one by one. Do not batch multiple selected skills into one `confirm_skill` call.

## When to Use This Skill

Use this skill when the user:

- Asks "how do I do X" where X might be a common task with an existing skill
- Says "find a skill for X" or "is there a skill for X"
- Asks "can you do X" where X is a specialized capability
- Expresses interest in extending agent capabilities
- Wants to search for tools, templates, or workflows
- Mentions they wish they had help with a specific domain (design, testing, deployment, etc.)

## What is the Skills CLI?

The Skills CLI (`npx skills`) is the package manager for the open agent skills ecosystem. Skills are modular packages that extend agent capabilities with specialized knowledge, workflows, and tools.

**Key commands:**

- `npx skills find [query]` - Search for skills interactively or by keyword
- `npx skills add <package>` - Install a skill from GitHub or other sources
- `npx skills check` - Check for skill updates
- `npx skills update` - Update all installed skills

**Browse skills at:** https://skills.sh/

## How to Help Users Find Skills

### Step 1: Understand What They Need

When a user asks for help with something, identify:

1. The domain (e.g., React, testing, design, deployment)
2. The specific task (e.g., writing tests, creating animations, reviewing PRs)
3. Whether this is a common enough task that a skill likely exists

### Step 2: Check the Leaderboard First

Before running a CLI search, check the [skills.sh leaderboard](https://skills.sh/) to see if a well-known skill already exists for the domain. The leaderboard ranks skills by total installs, surfacing the most popular and battle-tested options.

For example, top skills for web development include:
- `vercel-labs/agent-skills` — React, Next.js, web design (100K+ installs each)
- `anthropics/skills` — Frontend design, document processing (100K+ installs)

### Step 3: Search for Skills

If the leaderboard doesn't cover the user's need, run the find command:

```bash
npx skills find [query]
```

For example:

- User asks "how do I make my React app faster?" → `npx skills find react performance`
- User asks "can you help me with PR reviews?" → `npx skills find pr review`
- User asks "I need to create a changelog" → `npx skills find changelog`

### Step 4: Verify Quality Before Recommending

**Do not recommend a skill based solely on search results.** Always verify:

1. **Install count** — Prefer skills with 1K+ installs. Be cautious with anything under 100.
2. **Source reputation** — Official sources (`vercel-labs`, `anthropics`, `microsoft`) are more trustworthy than unknown authors.
3. **GitHub stars** — Check the source repository. A skill from a repo with <100 stars should be treated with skepticism.

### Step 5: Present Options to the User

When you find relevant skills, present them to the user with:

1. The skill name and what it does
2. The install count and source
3. The upstream package or ecosystem install command as reference
4. A link to learn more at skills.sh
5. A clear invitation to choose one or more skills if they want you to prepare Enter's preview/save flow for them

Example response:

```
I found a skill that might help! The "react-best-practices" skill provides
React and Next.js performance optimization guidelines from Vercel Engineering.
(185K installs)

Upstream package:
npx skills add vercel-labs/agent-skills@react-best-practices

Learn more: https://skills.sh/vercel-labs/agent-skills/react-best-practices

Inside Enter, I must first write any selected skill to
`/workspace/skill-drafts/custom/{skill_key}@1/` and then call `confirm_skill`.

If you'd like, choose one or more skills and I can prepare that preview/save flow for them.
```

### Step 6: Prepare Preview/Save Flow in Enter

If the user wants to proceed in Enter, prepare each selected skill individually using the Enter draft-and-confirm flow.

1. Take the selected skill or skills the user chose from your recommendations.
2. For each selected skill, identify its exact `skill_key`.
3. If the user expresses Enter install intent, call `list_skills(scope="workspace")` before preparing it.
4. If that exact `skill_key` is already visible in the workspace, tell the user it already exists and skip it.
5. If that exact `skill_key` is not visible, you MUST prepare the draft under:

```text
/workspace/skill-drafts/custom/{skill_key}@1/
```

6. This is the only filesystem location you may write for that Enter preparation flow.
7. This is the physical draft directory. After preparing the draft, hand it to Enter with the logical path:

```text
confirm_skill(skill_path="skill-drafts/custom/{skill_key}@1/")
```

8. `confirm_skill` only hands the draft to Enter's Save / Discard UI. It does not publish or persist the skill by itself.
9. Before the user clicks Save, the skill is still only a draft. Do not tell the user it is already installed, enabled, added to the workspace, or persisted.
10. If the user selected multiple skills, repeat the same flow for each selected skill.
11. Do not batch multiple selected skills into a single confirmation. Each newly prepared skill gets its own `confirm_skill` call.
12. Do not try to invent or paraphrase the draft metadata in tool args; Enter resolves the canonical `name` and `description` from `SKILL.md`.

The generic CLI install command remains useful context when explaining the broader ecosystem, but inside Enter the rule is strict: first prepare the selected skill at `/workspace/skill-drafts/custom/{skill_key}@1/`, then call `confirm_skill`.

## Common Skill Categories

When searching, consider these common categories:

| Category        | Example Queries                          |
| --------------- | ---------------------------------------- |
| Web Development | react, nextjs, typescript, css, tailwind |
| Testing         | testing, jest, playwright, e2e           |
| DevOps          | deploy, docker, kubernetes, ci-cd        |
| Documentation   | docs, readme, changelog, api-docs        |
| Code Quality    | review, lint, refactor, best-practices   |
| Design          | ui, ux, design-system, accessibility     |
| Productivity    | workflow, automation, git                |

## Tips for Effective Searches

1. **Use specific keywords**: "react testing" is better than just "testing"
2. **Try alternative terms**: If "deploy" doesn't work, try "deployment" or "ci-cd"
3. **Check popular sources**: Many skills come from `vercel-labs/agent-skills` or `ComposioHQ/awesome-claude-skills`

## When No Skills Are Found

If no relevant skills exist:

1. Acknowledge that no existing skill was found
2. Offer to help with the task directly using your general capabilities
3. Suggest the user could create their own skill with `npx skills init`

Do not enter the Enter install flow if no relevant skills were found.

Example:

```
I searched for skills related to "xyz" but didn't find any matches.
I can still help you with this task directly! Would you like me to proceed?

If this is something you do often, you could create your own skill:
npx skills init my-xyz-skill
```
