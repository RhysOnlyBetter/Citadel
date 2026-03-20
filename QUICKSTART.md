# Quickstart

From install to first `/do` command in 5 minutes.

## 1. Copy the harness into your project

```bash
# Clone the harness
git clone https://github.com/DevMoses/claude-harness.git

# Copy harness files into your project
cp -r claude-harness/.claude your-project/.claude
cp -r claude-harness/.planning your-project/.planning
cp -r claude-harness/scripts your-project/scripts

# If you don't have a CLAUDE.md yet, copy the starter
cp claude-harness/CLAUDE.md your-project/CLAUDE.md
```

Or copy manually — the harness is just files, no build step.

## 2. Run setup

Open your project in Claude Code, then:

```
/do setup
```

This will:
- Detect your language and framework
- Configure the typecheck hook for your stack
- Generate `.claude/harness.json` with your settings
- Create the planning directory structure
- Run a quick demo on your code

## 3. Start using it

```
/do review src/main.ts          # Code review
/do generate tests for utils    # Test generation
/do refactor the auth module    # Safe refactoring
/do scaffold a new API module   # Project-aware scaffolding
```

Or let the router figure it out:

```
/do fix the login bug
/do what's wrong with the API
/do build a caching layer
```

## 4. Create your first custom skill

```
/create-skill
```

It'll ask what patterns you keep repeating and generate a skill file
that captures your knowledge permanently.

## What's Next

- Add your project's conventions to `CLAUDE.md` — the more specific, the better
- Read `docs/SKILLS.md` to understand how skills work
- Try `/marshal "audit the codebase"` for a multi-step investigation
- Try `/archon "build [large feature]"` for multi-session campaigns
- Try `/fleet "overhaul all three modules"` for parallel execution
