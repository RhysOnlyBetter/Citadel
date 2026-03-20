---
title: Public Harness Quality Audit
date: 2026-03-20
auditor: Claude (Marshal-level audit)
files_reviewed: 59
---

# Public Harness Quality Audit

59 files reviewed. Every file read in full. No summaries, no skipping.

---

## 1. Cold Start Test

**Simulating a brand new user: clone -> first `/do` command.**

### What works

- README quickstart is above the fold (line 11-17). Three commands. Clear.
- QUICKSTART.md expands with numbered steps. The sequence is logical: copy -> setup -> use -> create-skill.
- `/do setup` as the entry point is smart -- it's the system demonstrating itself.
- The setup skill auto-detects language, generates harness.json, runs a live demo. This is well-designed onboarding.

### What will confuse or block a new user

1. **CRITICAL: `cp -r` path uses `your-project/` placeholder.** The README says `cp -r claude-harness/.claude your-project/.claude`. A beginner will literally type `your-project` and wonder why it fails. Should use `.` or explicitly say "replace `your-project` with your project directory." QUICKSTART repeats the same issue.

2. **Missing prerequisite: Claude Code itself.** Neither README nor QUICKSTART mentions that Claude Code must be installed. A user who finds this on GitHub might not have Claude Code yet. The first line should link to Claude Code installation. The README says "An agent orchestration system for Claude Code" but never says "you need Claude Code installed first."

3. **Missing prerequisite: Node.js.** The hooks are JavaScript files. `package.json` exists. The scripts require `node`. Nowhere does it say "requires Node.js 18+." A Python-only developer cloning this will hit `node: command not found` on the first hook fire and have no idea why.

4. **QUICKSTART Step 2 says "Open your project in Claude Code, then: `/do setup`."** But how? A new user doesn't know how to "open a project in Claude Code." This needs a one-liner: `cd your-project && claude` or equivalent.

5. **The CLAUDE.md copy step is conditional** ("If you don't have a CLAUDE.md yet") but the README quickstart omits this. A user following only the README will miss their chance to get a starter CLAUDE.md.

6. **No mention of `.gitignore` entries.** The harness ships a `.gitignore` that covers runtime state files. But when copying into an existing project, the user needs to merge `.gitignore` entries, not overwrite. This is not mentioned anywhere.

7. **`/do setup` confirmation question** (Step 1, Q1) asks "What's your project?" but a brand new user who just ran setup might not realize this is a real question requiring their input vs. a rhetorical prompt. Minor.

### Verdict: A user can get from clone to first `/do` **if** they already have Claude Code and Node.js installed and know what `cp -r` does. The path is 80% there but has two hard blockers (Claude Code and Node.js prerequisites) and one soft blocker (placeholder paths).

---

## 2. Skill Quality Audit

### /review (Code Review)

- **Identity**: Excellent. "You are not a linter -- you find the problems that tools miss." Clear differentiation.
- **Protocol**: Exceptionally detailed. 5 passes with specific scan criteria per pass. Each pass lists 8-15 concrete things to check. This is not a template -- this is a real protocol.
- **Quality Gates**: Strong. "Every finding is actionable," "No false positives from skimming," "Severity is calibrated," "Line numbers are accurate." All verifiable.
- **Exit Protocol**: Complete. Includes verdict criteria (PASS/CONDITIONAL/FAIL with thresholds).
- **Would it produce good output?** Yes. This is production quality. The pass structure prevents the common "random list of findings" anti-pattern.
- **Rating: A.** Ship it.

### /test-gen (Test Generation)

- **Identity**: Good. "Tests that run on the first try" is the right commitment. "Mock only what you must" is a strong stance.
- **Protocol**: Very thorough. Framework detection (Step 1) is comprehensive -- covers Jest, Vitest, Mocha, pytest, Go, Rust. The three-category structure (happy path, edge cases, error paths) with specific boundary values is excellent.
- **Quality Gates**: Strong. "No snapshot-only tests," "No implementation coupling," "No test interdependence," "Mocks are minimal." All checkable.
- **Exit Protocol**: Complete with coverage reporting and source bug documentation.
- **One concern**: Step 5 says "Target only the generated file -- do not run the entire suite." But some test frameworks (pytest, Go) don't support single-file targeting well. The skill should note fallback behavior.
- **Rating: A-.** Near-production quality.

### /doc-gen (Documentation Generator)

- **Identity**: Good. "Never produce boilerplate that a reader could derive faster by reading the code." This is the right philosophy.
- **Protocol**: Well-structured three-mode system. The "Trivial skip" rule (skip obvious getters/setters) prevents over-documentation. Style detection phase is thorough.
- **Quality Gates**: "Information density" gate is excellent. "Trivial skip rate" is novel and correct.
- **IP Leak**: Lines 93 and 159 reference "Realm entity ID" and `GET /api/realms/:id`. These are Tailored Realms-specific examples. **Must fix before launch.** Replace with generic examples like "User account ID, used for DB lookup" and `GET /api/users/:id`.
- **Rating: B+.** Good skill, has one IP leak.

### /refactor (Safe Refactoring)

- **Identity**: Excellent. "Safety as a hard constraint, not a best effort." The contract metaphor (typecheck + tests before/after) is exactly right.
- **Protocol**: Best protocol in the set. Six phases (Baseline -> Plan -> Execute -> Verify -> Fix -> Revert) with specific instructions per refactoring type (rename, extract, move, split, merge). The execution order (create new files first, update importers, update source, delete last) shows real experience.
- **Quality Gates**: "Zero new type errors," "Zero new test failures," "Minimal diff," "Plan matches execution." All excellent.
- **Exit Protocol**: Complete with structured report.
- **Rating: A+.** This is genuinely useful and would work on a real codebase.

### /scaffold (Project-Aware Scaffolding)

- **Identity**: Good. "Indistinguishable from hand-written code" is the right bar. The "find exemplars" approach is correct.
- **Protocol**: The exemplar-finding strategy table is excellent. The "Do NOT generate" list (no empty placeholders, no TODO-only test files) shows taste. The wiring step (Step 5) is crucial and often missed -- this skill handles it.
- **Quality Gates**: Thorough. "Found 2+ exemplar files," "No placeholder comments," "Main file is wired into the project." All verifiable.
- **One concern**: The quality gate "Generated files match the project's import/export style exactly" is vague. How does a machine verify "exactly"? Could use a more concrete gate like "Generated imports use the same alias style as exemplars."
- **Rating: A-.** Would produce useful output.

### /create-skill (Skill Creator)

- **Identity**: Excellent. "Decision-compression artifact" is a precise description. The teacher role is important -- "A skill the user cannot maintain is a liability, not an asset."
- **Protocol**: The three-question interview (What do you repeat? What mistakes happen? What does done look like?) is well-designed. The analysis step (2a-2e) is thorough. The "test on a real target" step (Step 5) is critical and well-handled.
- **Quality Gates**: 13 gates. Very thorough. "Trigger keywords do not conflict with existing skills," "Skill was tested on a real target," "SKILL.md is under 500 lines."
- **Exit Protocol**: Complete with invocation example.
- **One concern**: The skill says "Ask these three questions and wait for the user to respond to each one." But the three questions are asked sequentially, which could feel like an interrogation. A user might prefer to answer all three at once. Consider allowing batch answers.
- **Rating: A.** This is the most important skill in the harness and it shows.

### Overall Skill Assessment

All six skills are substantive, not templates. They would produce genuinely useful output on real codebases. The quality is remarkably consistent across all six. The weakest is doc-gen (B+ due to IP leak), and the strongest are refactor (A+) and review (A).

---

## 3. /do Router Completeness

### Tier 0: Pattern Match

| Pattern | Assessment |
|---|---|
| "typecheck" / "type check" | Good |
| "build" | Good |
| "test" / "tests" | Good |
| "status" | Good |
| "continue" / "keep going" | Good |
| "setup" | Good |
| "--list" / "list" | Good |
| "fix typo in X" / "rename X to Y" | Good |
| "commit" | Good |

**Missing patterns worth considering:**
- "lint" / "format" -- common quick tasks
- "what changed" / "diff" -- common query
- "undo" / "revert" -- common recovery

**Verdict**: Tier 0 is comprehensive for the most common quick tasks. The missing patterns are nice-to-have, not critical.

### Tier 1: Active State Detection

Logic: Read campaigns/ for `Status: active`, read fleet/ for active sessions, match input scope to active campaign.

**Issue**: The skill says "If input scope matches an active campaign -> `/archon continue`" but doesn't define how "scope matching" works. If I say `/do fix the login page` and there's an active campaign about auth, does that match? The matching criteria are vague.

**Issue**: Step 2 checks fleet sessions for `status: active` or `needs-continue` but the fleet session template uses capitalized `Status:` while the regex in the code checks lowercase. The pre-compact hook uses `/status:\s*(active|needs-continue)/mi` (case-insensitive), but the skill text should be explicit about case handling to avoid confusion.

**Verdict**: Logic is sound but scope matching criteria need sharpening.

### Tier 2: Skill Keyword Match

| Input Contains | Route To | Coverage |
|---|---|---|
| "review", "code review" | /review | Good -- covers trigger_keywords |
| "test", "generate tests", "write tests" | /test-gen | Good |
| "document", "docs", "docstring", "readme" | /doc-gen | Good |
| "refactor", "rename", "extract", "split file" | /refactor | Good |
| "scaffold", "new module", "new component", "bootstrap" | /scaffold | Good -- "bootstrap" is a nice addition |
| "create skill", "new skill", "repeated pattern" | /create-skill | Good |
| "handoff", "session summary" | /session-handoff | Good |

**Missing from Tier 2 that are in the skill frontmatter:**
- `/review` frontmatter has "review PR" and "review this" -- not in Tier 2 table
- `/test-gen` frontmatter has "add tests" -- not in Tier 2 table
- `/doc-gen` frontmatter has "jsdoc", "api docs" -- not in Tier 2 table
- `/refactor` frontmatter has "inline", "merge files" -- not in Tier 2 table
- `/scaffold` frontmatter has "generate component", "generate module", "new route", "create component", "stub out" -- not in Tier 2 table
- `/create-skill` frontmatter has "teach the harness", "custom skill", "my own skill", "automate this pattern" -- not in Tier 2 table

**Assessment**: The Tier 2 table in the /do skill is a subset of actual trigger_keywords. The router says it "scans `.claude/skills/*/SKILL.md` frontmatter" so the actual routing would use the full keyword set, not just the abbreviated table. The table is documentation, not the implementation. **This is fine** but could confuse someone reading the /do SKILL.md who doesn't realize the full keyword set is in the individual skills.

### Tier 3: LLM Classifier

- Six dimensions are well-chosen: SCOPE, COMPLEXITY, INTENT, REQUIRES_PERSISTENCE, REQUIRES_PARALLEL, REQUIRES_TASTE.
- Routing rules table is clear and ordered by complexity.
- The `Confidence < 0.7 -> /marshal` fallback is a good safe default.
- The "repeated pattern complaint routes to /create-skill" rule shows insight.

**Concern**: The classifier prompt itself isn't shown in the skill -- it says "classify across 6 dimensions" but doesn't show the actual prompt that would be sent to the LLM. This means the classification quality depends entirely on how Claude interprets these dimensions, which is fine in practice but makes the skill harder to debug.

### Bias-Down Principle

Explicitly stated: "The router biases aggressively toward the cheapest path -- under-routing (skill fails, user re-invokes) is far cheaper than over-routing." This is clear and correct.

### /do status, /do continue, /do --list

All three have clear handling with specific output formats. `/do status` shows campaigns, fleet sessions, and intake count. `/do --list` shows all skills grouped by category. `/do continue` resumes the most recent active work.

### Escape Hatches

Documented: "Direct invocation ALWAYS works and bypasses the router." This is important and well-placed.

### Verdict: Router is well-designed. The four-tier classification is sound. Minor gaps in Tier 1 scope matching and Tier 2 keyword documentation completeness.

---

## 4. Hook Integrity

### post-edit.js (Per-file typecheck)

- **Language-adaptive**: Yes. Handles TypeScript, Python, Go, Rust. Each has its own function with language-specific invocation.
- **Graceful degradation**: If no typecheck command is configured (harness.json), returns 0 (pass). If the command fails for non-typecheck reasons, catches the error and returns 0. Good.
- **Performance lint**: Cross-language web anti-pattern detection (confirm/alert/prompt, transition-all). Language-appropriate -- only checks JS/TS files for confirm/alert.
- **No Tailored Realms references**: Clean.
- **Issue**: The TypeScript check runs `npx tsc --noEmit --pretty false 2>&1` on every edit. On large codebases, this can take 10-25 seconds. The 25-second timeout is appropriate, but the docs should mention that per-file typecheck means "run full tsc, filter errors to this file" not "check only this file." Users might expect fast single-file checks.
- **Issue**: The `performanceLint` function checks for `transition-all` in ALL source files (including Python, Go, Rust) where that pattern is irrelevant. Should be gated to CSS/JS/TS files only for that specific check. Currently it is: `if (/\.(ts|tsx|js|jsx)$/.test(filePath))` gates confirm/alert, but `transition-all` check is outside that gate and runs on all files. Minor -- unlikely to false-positive in Python/Go but still sloppy.

### circuit-breaker.js

- **Logic**: Tracks `consecutiveFailures`. At 3: suggests alternatives. At 5 lifetime trips: escalates to "stop and rethink." Counter resets after each trip. This is sound.
- **State management**: Atomic write (write to .tmp, rename). Good.
- **No Tailored Realms references**: Clean.
- **Issue**: The circuit breaker resets `consecutiveFailures` to 0 after tripping at THRESHOLD (3). But it doesn't reset on SUCCESS. The `PostToolUseFailure` event only fires on failures, so there is no mechanism to reset the counter when a tool succeeds. This means: fail, fail, succeed, fail -> counter is at 3 (not 1), which would trip the breaker. **Wait** -- actually, looking at the hook event: `PostToolUseFailure` only fires on failures. There is no `PostToolUseSuccess` hook registered. So the counter only ever increments. It resets to 0 only when it hits the threshold. This means: 2 failures, then 50 successes, then 1 failure = counter at 3 = trip. **This is a bug.** The counter should reset on any successful tool use. But since there's no PostToolUse hook that resets it (the PostToolUse hook is post-edit.js for typecheck), this is a known limitation. Documenting it would be sufficient.
- **Mitigation**: The impact is low because the counter tracks the most recent consecutive failures across all tools, so a long-running session with scattered failures would eventually trip. But the messaging says "3 consecutive failures" which is misleading if there were successes in between.

### quality-gate.js

- **Logic**: Scans git diff for changed files, applies built-in and custom rules. Clean.
- **Configurable**: Yes, via harness.json `qualityRules.builtIn` and `qualityRules.custom`.
- **Custom rules**: Regex-based with file pattern filtering. Well-designed.
- **No Tailored Realms references**: Clean.
- **Issue**: The "no-magic-intervals" rule regex is `setInterval\s*\(\s*\w+\s*,\s*\d+\s*\)`. This requires the callback to be a single word (`\w+`). `setInterval(() => tick(), 1000)` would NOT match because the callback is an arrow function. The regex should be more permissive.
- **Issue**: The Stop hook checks `ctx.stop_hook_active` to prevent infinite loops, but Stop hooks in Claude Code don't re-trigger themselves. This is defensive coding, which is fine but unnecessary.

### protect-files.js

- **Logic**: Reads protected patterns from harness.json, blocks Edit/Write to matching files. Exit code 2 blocks the tool.
- **Pattern matching**: Supports exact match, `/*` wildcard (single directory level), and `/` prefix (directory prefix). No `**` recursive. Documented.
- **No Tailored Realms references**: Clean.
- **Issue**: The default protected files (`['.claude/settings.json', '.claude/hooks/*']`) make sense for preventing agents from modifying the harness itself. But this means agents can't fix hook bugs either. The message tells the user how to unprotect: "Remove the pattern from harness.json." Good.

### intake-scanner.js

- **Logic**: Reads `.planning/intake/`, skips files starting with `_` or `.`, parses frontmatter for title/status, reports pending and in-progress items.
- **No Tailored Realms references**: Clean.
- **Issue**: None significant. Clean, simple, does one thing.

### pre-compact.js and restore-compact.js

- **Logic**: pre-compact saves active campaign/fleet session state to `.claude/compact-state.json`. restore-compact re-injects it on SessionStart with `compact` matcher.
- **No Tailored Realms references**: Clean.
- **Issue**: The regex for extracting Active Context uses `\Z` which in JavaScript regex is not valid (it's Perl/Python). In JS, end-of-string is `$`. The regex is `/(## Active Context\s*\n([\s\S]*?)(?=\n## |\n---|\Z))/`. In practice, `\Z` is treated as a literal `Z` character, so the regex falls back to the other alternations (`\n## ` or `\n---`) which work fine for well-formatted campaign files. **But**: if Active Context is the LAST section in the file (no following `##` or `---`), the match will fail silently. The fix: replace `\Z` with `$`.

### worktree-setup.js

- **Logic**: Auto-installs dependencies in new worktrees (npm/pnpm/yarn/bun), creates Python venvs, copies .env files.
- **Language-adaptive**: Handles Node (4 package managers) and Python (venv + pip). Does NOT handle Go or Rust worktree setup, but those typically don't need it (Go modules are cached globally, Rust uses cargo cache).
- **No Tailored Realms references**: Clean.
- **Issue**: Python venv activation uses `.venv/bin/pip` which is Unix-only. On Windows, it's `.venv\Scripts\pip`. The harness has Windows users (the project root is `C:\Users\gammo\Desktop\public-harness`). **Should fix** with a cross-platform path.

### harness-health-util.js

- **Logic**: Shared utilities. PROJECT_ROOT from env or cwd. Config reading with defaults. Stack detection. Typecheck config generation.
- **No Tailored Realms references**: Clean.
- **Issue**: `detectStack()` checks for `tsconfig.app.json` (Angular convention) in addition to `tsconfig.json`. Good attention to detail.
- **Issue**: Framework detection for Python is fragile: `exists('app.py')` -> Flask, `exists('main.py')` -> FastAPI. But `main.py` is common in any Python project, and `app.py` could be anything. This will misdetect framework in many cases. **Should note** that framework detection is best-effort and users should verify in harness.json.

### Overall Hook Assessment

All hooks are clean of Tailored Realms references. They degrade gracefully when tools aren't installed. The main issues are: circuit breaker counter never resets on success (design bug), pre-compact uses `\Z` instead of `$` (regex bug), worktree-setup uses Unix-only Python paths (platform bug), and post-edit's transition-all check isn't language-gated (minor).

---

## 5. Campaign and Fleet Infrastructure

### Campaign Creation

The template at `.planning/_templates/campaign.md` is well-structured with HTML comments explaining each section. A new user reading the template would understand what goes where. The example at `examples/campaign-example.md` is excellent -- a realistic, complete campaign (JWT auth overhaul) with 6 phases, decision log entries with reasoning, and feature ledger entries.

**Issue**: The template includes `src/api/auth/` and `src/middleware/` as example claimed scope. These are obviously examples but could be confusing if copy-pasted literally. Should use `{src/your/scope/}` placeholder syntax consistent with other placeholders in the template.

### Wave Coordination and Discovery Compression

`scripts/compress-discovery.cjs` is well-implemented:
- Extracts HANDOFF blocks, decisions, file paths, and failures
- Produces structured briefs under ~500 tokens
- Logs compression stats to telemetry

The discovery relay concept is well-documented in `docs/FLEET.md` with a concrete example (rate limiting discovery in Wave 1 informing frontend throttling in Wave 2).

**Issue**: `docs/FLEET.md` says "Start with 2 agents per wave, scale up as you trust scope separation" but doesn't give guidance on WHEN to scale up. What signals indicate trust?

### Worktree Isolation

Explained in `docs/FLEET.md`:
- Separate working directory
- Independent git branch
- Dependencies auto-installed
- Environment files copied

**Issue**: The explanation assumes the user understands git worktrees. A beginner who has never used `git worktree` will need more context. The docs should either explain worktrees briefly or link to git documentation.

### Verdict: Campaign infrastructure is well-designed and well-documented. The example campaign is particularly strong. Fleet documentation is thorough for intermediate+ users but assumes git worktree knowledge.

---

## 6. Documentation Gaps

### Capabilities mentioned in README but not in docs/

| Capability | README | docs/ | Gap? |
|---|---|---|---|
| Orchestration ladder | Yes | ARCHITECTURE.md | No |
| /do router | Yes | ARCHITECTURE.md | No |
| Skills | Yes | SKILLS.md | No |
| Hooks | Yes | HOOKS.md | No |
| Campaigns | Yes | CAMPAIGNS.md | No |
| Fleet | Yes | FLEET.md | No |
| `/do setup` | Yes | setup SKILL.md | No |
| `/create-skill` | Yes | create-skill SKILL.md | No |
| `harness.json` | Yes | ARCHITECTURE.md, HOOKS.md | No |
| Telemetry | Mentioned | Underdocumented | **Yes** |

**Telemetry gap**: README mentions telemetry scripts in the project structure, and package.json has `telemetry:report` and `telemetry:log` scripts, but there is no `docs/TELEMETRY.md`. The telemetry-report.cjs has three modes (agent summary, hook timing, compression stats) that are not documented anywhere except the script's own comments.

### Capabilities in code not mentioned anywhere

1. **`harness-health-util.js` `detectStack()` function** -- this is called by the setup skill but its detection logic (which files trigger which language, framework detection heuristics) is not documented. Users who want to understand why their stack was detected incorrectly have nowhere to look.

2. **`coordination.js` CLI** -- the full command set (generate-id, register, unregister, heartbeat, claim, release, check-overlap, sweep, status) is documented in the script comments and in `docs/FLEET.md` script table, but there is no dedicated coordination doc. The ARCHITECTURE.md mentions coordination briefly.

3. **`parse-handoff.cjs`** -- mentioned in `docs/FLEET.md` script table but usage is not explained anywhere. When would a user run this manually?

### Docs assuming knowledge a beginner might lack

1. **ARCHITECTURE.md** assumes understanding of "context windows," "token cost," and "compaction." These are Claude Code-specific terms.
2. **FLEET.md** assumes understanding of git worktrees.
3. **CAMPAIGNS.md** mentions `/autopilot brief` without explaining what "brief" means in this context.
4. **HOOKS.md** mentions "exit code 2 prevents the tool from executing" without explaining why 2 specifically (Claude Code convention).

### Docs too surface-level for advanced users

1. **ARCHITECTURE.md** is good for overview but lacks depth on how the /do router actually classifies intent. The skill has the detail; the doc just summarizes.
2. **HOOKS.md** doesn't explain how to write a CUSTOM hook from scratch (only how to add custom quality rules). An advanced user wanting to add a new lifecycle hook has to reverse-engineer settings.json.

---

## 7. IP Leak Check

### Files scanned: ALL 59

### Findings

| File | Line | Issue | Severity |
|---|---|---|---|
| `.claude/skills/doc-gen/SKILL.md` | 93 | `@param id - Realm entity ID, used for DB lookup` -- "Realm entity" is Tailored Realms terminology | **Must fix** |
| `.claude/skills/doc-gen/SKILL.md` | 159 | `GET /api/realms/:id` -- "realms" is the Tailored Realms primary entity | **Must fix** |

### Clean checks (no matches found)

- "Tailored Realms" (case-insensitive): **Clean**
- Production skill names (sandbox, battlemap, omni-canvas, sim-director, lyra): **Clean**
- Domain-specific terms (spatial rendering, entity system, procedural generation, world-building, OmniCanvas): **Clean**
- Archon profile/taste layer references: **Clean**
- Observatory dashboard references: **Clean**
- Research agent / /experiment: **Clean**
- Hardcoded paths (D:\Tailored-Realms): **Clean**
- Private layer aliases (@kernel/*, @os/*, @domains/*, @canvas/*, @design-system/*): **Clean**

### Assessment

Two minor IP leaks in doc-gen examples. The word "Realm" in `Realm entity ID` and `GET /api/realms/:id` are direct references to the Tailored Realms entity model. While "realm" is a common English word, combined with "entity ID" and an API endpoint pattern, it's recognizably from the private codebase.

**Fix**: Replace line 93 with: `@param id - User account ID, used for DB lookup`
**Fix**: Replace line 159 with: `GET /api/users/:id`

The rest of the repo is remarkably clean. No accidental private paths, no private skill names, no domain-specific architecture references. The extraction from private to public was done carefully.

---

## 8. Beginner to Advanced Spectrum

### Beginner (just started with Claude Code)

**Can they get value in 10 minutes?**

Conditional yes. If they already have Claude Code and Node.js:
1. Clone (1 min)
2. Copy files (1 min)
3. `/do setup` (3 min with questions)
4. `/review [file]` demo runs automatically (2 min)
5. They've seen the system work (7 min total)

**Blockers for true beginners:**
- No Claude Code installation link
- No Node.js requirement stated
- `cp -r` command uses placeholder paths
- "Open your project in Claude Code" doesn't explain how

**Verdict**: 10-minute value is achievable for someone who already uses Claude Code. For true beginners, add 15-20 minutes of setup friction.

### Intermediate (uses CLAUDE.md, maybe tried a skill)

**Clear progression path?**

Yes. The progression is natural:
1. Use individual skills (/review, /test-gen)
2. Use /do router to stop choosing manually
3. Use /marshal for multi-step work
4. Create custom skills with /create-skill
5. Use /archon for multi-session campaigns

The documentation supports this progression:
- QUICKSTART ends with "What's Next" pointing to marshal, archon, fleet
- docs/SKILLS.md explains how to write custom skills
- docs/CAMPAIGNS.md explains multi-session persistence

**Gap**: No explicit "progression guide" document. The progression is implicit from reading multiple docs. A "What to Learn Next" section in the README or a dedicated `docs/PROGRESSION.md` would help.

### Advanced (already running parallel agents)

**Enough depth?**

Yes. The advanced features are genuinely advanced:
- Fleet wave coordination with discovery relay
- Multi-instance coordination with scope claim system
- Telemetry and performance reporting
- Campaign persistence across context windows
- Custom hooks and quality rules
- The coordination.js CLI with overlap detection

**What advanced users might miss:**
- How to tune the circuit breaker threshold (hardcoded at 3)
- How to customize the /do router's Tier 3 classification
- How to integrate with CI/CD (no mention)
- How to run Fleet without worktrees (e.g., in environments where worktrees aren't available)

**Verdict**: Good depth. The advanced tier has real sophistication (coordination system, discovery relay, telemetry). Missing CI/CD integration guidance and some configurability.

---

## 9. README First Impression

### Is it immediately clear what this is and why it matters?

**First line**: "An agent orchestration system for Claude Code." Clear what.
**Second line**: "Route any task through the right tool at the right scale -- from a one-line fix to a multi-day parallel campaign." Clear why.
**Third paragraph**: "Built from running 198 autonomous agents across 32 parallel sessions on a production codebase. 27 postmortems worth of lessons baked into every hook and skill." Establishes credibility.

**The tagline "The harness is simple. The knowledge that shaped it isn't." is excellent.** It sets the right expectation: the files are plain, the value is in what they encode.

### Do the character cards work as visual introduction?

The SVG cards are well-designed:
- Skill: Blue, book icon, "Domain Expert," "Zero tokens inactive"
- Marshal: Red/orange, chain links, "Session Commander"
- Archon: Purple, eye of providence, "Autonomous Strategist"
- Fleet: Gold/red, constellation/network, "Parallel Coordinator"

The visual hierarchy works: the cards introduce the four tiers before the user reads about them. The icons are thematically appropriate. The color scheme is consistent and professional.

**Issue**: GitHub renders SVGs inline, but some email clients and social previews will not. Consider adding PNG fallbacks or a screenshot. Not critical for GitHub.

### Is quickstart findable within 5 seconds?

Yes. It's the second section, right after the title. Three lines of code. Hard to miss.

### Does anything feel like fluff vs substance?

**Potential fluff:**
- The FAQ section is useful but the "How much does this cost in tokens?" question answers itself in a way that might feel like marketing rather than documentation.
- The "Video" section links to a coming-soon video. **Remove or hide this until the video exists.** A placeholder link looks unfinished.
- "Every capability shown in the video is something you can install and use right now." -- This is marketing copy in a technical README. Fine for a landing page, slightly out of place here.

**Everything else is substance.** The README is information-dense without being overwhelming. The tables are scannable. The code examples are concrete.

### Overall: Strong first impression. Professional, credible, well-structured. The character cards are a memorable differentiator. Two cleanup items (coming-soon video link, FAQ marketing tone).

---

## 10. Recommendations

### Must Fix Before Launch

1. **IP Leak in doc-gen/SKILL.md** (lines 93, 159): Replace "Realm entity ID" with "User account ID" and `GET /api/realms/:id` with `GET /api/users/:id`. [Section 7]

2. **Add Claude Code prerequisite** to README and QUICKSTART: "Requires: [Claude Code](https://claude.ai/claude-code) installed." First line of Quickstart. [Section 1]

3. **Add Node.js prerequisite**: "Requires: Node.js 18+ (for hooks and scripts)." [Section 1]

4. **Fix `your-project` placeholder** in README and QUICKSTART: Replace with `.` or add explicit instruction: "Replace `your-project` with the path to your project directory." [Section 1]

5. **Remove or hide the coming-soon video link** in README. A placeholder link signals "unfinished." [Section 9]

### Should Fix

6. **Fix `\Z` regex in pre-compact.js** (line 43): Replace `\Z` with `$`. Currently fails silently when Active Context is the last section in a campaign file. [Section 4]

7. **Fix Windows Python path in worktree-setup.js** (line 57): Use cross-platform path for venv activation (`path.join('.venv', process.platform === 'win32' ? 'Scripts' : 'bin', 'pip')`). [Section 4]

8. **Gate transition-all lint to CSS/JS/TS only** in post-edit.js (line 225): Move the `transition-all` check inside the `/\.(ts|tsx|js|jsx|css|scss)$/` guard. Currently runs on Python/Go/Rust files needlessly. [Section 4]

9. **Document circuit breaker counter behavior**: Note in docs/HOOKS.md that the counter tracks failures since last trip, not strictly consecutive failures (no success-reset mechanism). [Section 4]

10. **Add "How to open a project in Claude Code" one-liner** to QUICKSTART: `cd your-project && claude` or equivalent. [Section 1]

11. **Fix no-magic-intervals regex** in quality-gate.js: The pattern `\w+` for callback misses arrow functions and inline functions. Use a more permissive pattern. [Section 4]

12. **Add brief git worktree explanation** to docs/FLEET.md for users who haven't used worktrees before. [Section 5]

### Nice to Have

13. **Add `docs/TELEMETRY.md`** documenting the three telemetry report modes and how to interpret results. [Section 6]

14. **Add progression guide** to README or as `docs/PROGRESSION.md`: explicit "start here, then try this, then advance to that" path. [Section 8]

15. **Add PNG fallback images** for SVG cards (for social previews and email clients). [Section 9]

16. **Add `.gitignore merge instructions`** to QUICKSTART: "If your project already has a .gitignore, append the entries from the harness .gitignore rather than overwriting." [Section 1]

17. **Improve Python framework detection** in harness-health-util.js: Add note that detection is best-effort and users should verify harness.json after setup. [Section 4]

18. **Add CI/CD integration guidance** for advanced users: how to run quality-gate in a CI pipeline, how to use telemetry-report in CI. [Section 8]

19. **Make circuit breaker threshold configurable** via harness.json instead of hardcoded `const THRESHOLD = 3`. [Section 4]

20. **Clarify Tier 1 scope matching** in the /do skill: define what "input scope matches an active campaign" means concretely. [Section 3]

---

## Summary

The public harness is in strong shape. The core skills are production quality -- not templates, not skeletons, but genuine protocols that would produce useful output on real codebases. The hooks are well-engineered with graceful degradation. The campaign and fleet infrastructure shows real experience with multi-agent orchestration.

**Two hard blockers**: the doc-gen IP leak and missing prerequisite documentation. Both are 5-minute fixes.

**The biggest risk is not quality but onboarding friction.** The harness assumes the user already has Claude Code and Node.js and knows their way around a terminal. The first 5 minutes for a true beginner could be rough. The content is there; the hand-holding for the very first steps is not.

The repo is clean of private Tailored Realms references except for two examples in doc-gen. No private paths, no private skill names, no domain-specific architecture leaks. The extraction was done carefully.

**Ready for launch after fixing the 5 must-fix items.**
