**KPMG Claude Code Foundation Training**

**Guided Hands-On Lab Manual**

# How to Use This Lab Manual

**Who these labs are for**

This manual accompanies the Claude Code Foundation Training. It is written for professional software developers who build applications for clients in Healthcare and Banking & Finance. You are assumed to be comfortable with a programming language, an IDE, Git, and automated testing - what is new to you is Claude and Claude Code, not software engineering.

Because you ship code into regulated domains, the labs use domain-credible tasks (FHIR/HL7, PHI handling, decimal money, time-zone-safe scheduling, KYC/AML, idempotency, audit logging) rather than toy examples. Every hands-on activity in the syllabus has a matching lab here.

**Language and stack**

Runnable examples use Python with pytest so a room on mixed stacks can validate results the same way. The Claude Code techniques are identical in Java, C#, TypeScript, or Go - adapt the prompts to your stack by naming your language and test framework. Where a lab says 'create a file', let Claude create it; the point is the workflow, not the typing.

**How to read a lab**

- Objective - the Claude Code capability you will be able to apply afterwards.

- Scenario - a realistic feature or defect from a Healthcare or Finance application.

- Steps / prompt boxes - the exact prompts and commands to run, in order.

- Checkpoint - how to confirm you completed the lab correctly.

**The golden rule for regulated client work**

> **Read this before every lab:** Never paste a client's real PHI, PII, PAN, card or account numbers, production database rows, credentials, or private keys into any AI tool. All labs use synthetic fixtures. When you need Claude to work against real data, have it write the code and run that code yourself inside the client's approved environment. Treat this as a contractual and regulatory obligation (HIPAA, PCI-DSS, DPDP/GDPR), not a lab convention.

**Conventions**

- A monospace box is a prompt or a command. A line starting with \$ is a shell command; a line starting with / is a Claude Code slash command; anything else is natural language to type inside a running Claude session.

- Tip: press Shift+Tab inside Claude Code to cycle modes (normal, auto-accept edits, and plan mode). Plan mode makes Claude propose a plan without editing files - use it whenever a change is non-trivial.

# Lab Map

Every hands-on activity in the syllabus maps to a lab below. Module 5 (Contrast with Claude Desktop) is discussion-based with no hands-on, so it has no lab.

| **Lab** | **Syllabus module**                                              | **Domain**                   |
|---------|------------------------------------------------------------------|------------------------------|
| 1       | Module 1 - Introduction to Claude Code                           | Healthcare                   |
| 2       | Module 2 - Installation across Terminal, Web, VS Code, JetBrains | Environment & team setup     |
| 3       | Module 3 - Basic Usage of Claude Code                            | Banking & Finance            |
| 4       | Module 4 - Advanced Prompt Engineering                           | Healthcare                   |
| 5       | Module 6 - Claude Models Overview & Selection                    | Banking & Finance            |
| 6       | Module 7 - Token & Cost Optimisation                             | Working in a real repository |
| 7       | Module 8 - Authentication, Security                              | Healthcare + Finance         |
| 8       | Module 9 - Responsible Usage: DOs & DON'Ts                       | Healthcare + Finance         |
| 9       | Module 10 - Review and Own the Outcome                           | Banking & Finance            |
| 10      | Module 11 - Code Analysis                                        | Healthcare                   |
| 11      | Module 12 - Specification Based Development                      | Banking & Finance            |
| 12      | Module 13 - Defect Analysis, Fixing & Resolution                 | Healthcare                   |
| 13      | Module 14 - Refactoring Codebase                                 | Banking & Finance            |
| 14      | Module 15 - Basics of Agentic Workflows & Automation             | Healthcare                   |
| 15      | Module 16 - Explore and Rollback                                 | Banking & Finance            |

# Lab 1: First Interaction & Simple Prompt Execution

*Module 1 - Introduction to Claude Code*

| **Domain focus**  | Healthcare                                                           |
|-------------------|----------------------------------------------------------------------|
| **Level**         | Foundation                                                           |
| **Duration**      | 20-25 minutes                                                        |
| **Prerequisites** | Claude Code installed and logged in (Lab 2 covers install if needed) |

**Objective**

- Launch Claude Code inside a project and understand the prompt/response loop.

- Distinguish a knowledge prompt from a code-generation prompt.

- Produce a small, tested unit of code on the first interaction.

**Scenario**

You are evaluating Claude Code for your team, which builds integration services for hospital clients. You will get oriented and generate one small, tested validator.

**Steps**

**Step 1.** Create a working folder and start an interactive session.

> Terminal:
> $ mkdir cc-labs && cd cc-labs
> $ claude

**Step 2.** Orient yourself - ask how Claude fits your SDLC.

> ▶ Prompt to try (copy & paste into Claude Code):
> I build backend integration services for healthcare clients. In 4 bullets,
> where in the SDLC can you help me, and where should I stay in control?

**Step 3.** Run a knowledge prompt (no code) to see Claude explain a domain concept.

> ▶ Prompt to try (copy & paste into Claude Code):
> In ~100 words for a backend dev new to healthcare integration, contrast
> HL7 v2 messaging with FHIR R4 REST resources - when is each used?

**Step 4.** Run your first code-generation prompt. Claude will ask before creating the file - approve it.

> ▶ Prompt to try (copy & paste into Claude Code):
> Create patient_validation.py with validate_patient(payload: dict) -> list[str]
> that checks a minimal FHIR R4 Patient: resourceType == 'Patient', a non-empty
> id, and at least one name entry containing a 'family' field. Return a list of
> human-readable error strings (empty list means valid). Add pytest tests with
> one valid fixture and two invalid ones. Use only synthetic data.

> **Note:** Claude Code always asks permission before creating or editing files and before running shell commands. Read what it proposes, then approve or deny - this permission loop is your primary safety control.

**Checkpoint**

- You can point to which reply was explanation and which produced code.

- patient_validation.py and its tests exist; running pytest passes.

- You saw and consciously approved the file-write permission prompt.

> **Trainer tip:** Type /help for commands and /exit to leave. Encourage full-sentence prompts with explicit acceptance criteria - developers who prompt like they write a ticket get better first drafts.

# Lab 2: Install, Configure & Validate Claude Code

*Module 2 - Installation across Terminal, Web, VS Code, JetBrains*

| **Domain focus**  | Environment & team setup                                                              |
|-------------------|---------------------------------------------------------------------------------------|
| **Level**         | Foundation                                                                            |
| **Duration**      | 30-40 minutes                                                                         |
| **Prerequisites** | A Pro, Max, Team, Enterprise, or Console account (the free plan excludes Claude Code) |

**Objective**

- Install Claude Code and authenticate on your work machine.

- Validate the install and initialise a real repository with a CLAUDE.md.

- Wire Claude Code into VS Code or a JetBrains IDE.

**Part A - Install**

Use the recommended native installer for your OS. Package-manager installs (Homebrew/WinGet/apt/dnf) and npm are also supported.

| **Environment**     | **Install command**                             |
|---------------------|-------------------------------------------------|
| macOS / Linux / WSL | curl -fsSL https://claude.ai/install.sh \| bash |
| Windows PowerShell  | irm https://claude.ai/install.ps1 \| iex        |
| Homebrew (macOS)    | brew install --cask claude-code                 |
| WinGet (Windows)    | winget install Anthropic.ClaudeCode             |
| npm (Node 22+)      | npm install -g @anthropic-ai/claude-code        |

> **Note:** Avoid sudo npm install -g (permission and security issues). Native installs auto-update in the background; Homebrew/WinGet/apt/dnf/npm require manual updates.

**Part B - Verify**

> Terminal:
> $ claude --version # prints e.g. 2.1.211 (Claude Code)
> $ claude doctor # read-only install / auth / config diagnostics

**Part C - Authenticate**

Run claude and complete the browser login for interactive use. For CI or headless machines, set ANTHROPIC_API_KEY instead of using the browser flow (covered in Lab 7).

> Terminal:
> $ claude # first launch opens the browser to log in
> $ claude whoami # confirm the authenticated account

**Part D - Initialise a real repository**

Open an existing service repo (or clone one) and run /init. It scans the project and writes a CLAUDE.md that Claude reads on every future session - your place to encode team conventions.

> Inside the session:
> /init

**Step 1.** Enrich CLAUDE.md with domain conventions so future prompts inherit them.

> ▶ Prompt to try (copy & paste into Claude Code):
> Update CLAUDE.md to record these house rules: money is handled with Decimal
> never float; all datetimes are timezone-aware UTC; every state change writes
> an audit log entry; and no real PHI/PII ever appears in code or fixtures.

**Part E - IDE integration**

- VS Code: install the Claude Code extension, then run claude in the integrated terminal - it shares your editor context and shows diffs inline.

- JetBrains (IntelliJ / PyCharm / WebStorm / Rider): install the Claude Code plugin from the Marketplace and sign in when prompted.

**Checkpoint**

- claude --version and claude doctor both report a healthy, authenticated install.

- CLAUDE.md exists and contains your team's domain conventions.

- You can launch Claude Code from inside your IDE.

> **Trainer tip:** If the wrong version launches or claude persists after an uninstall, claude doctor names the active install type so you can remove a conflicting npm copy or a stale shell alias.

# Lab 3: Generate Code from Simple Prompts & Iterate

*Module 3 - Basic Usage of Claude Code*

| **Domain focus**  | Banking & Finance                   |
|-------------------|-------------------------------------|
| **Level**         | Foundation                          |
| **Duration**      | 30 minutes                          |
| **Prerequisites** | Lab 1; a running session in cc-labs |

**Objective**

- Generate a working module from a plain-English specification.

- Refine it across turns by referencing prior work instead of restating it.

**Scenario**

You need a small money primitive for a payments service. Money bugs are almost always float bugs - you will build it correctly with Decimal, then harden it in passes.

**Pass 1 - Generate**

> ▶ Prompt to try (copy & paste into Claude Code):
> Create money.py with a Money value object: fields amount (Decimal) and
> currency (str, ISO-4217). Constructor accepts amount as str or Decimal and
> rejects float. Provide add() and subtract() that require matching currency,
> and a __str__ that renders exactly 2 decimal places. Raise clear exceptions
> for currency mismatch and invalid input.

**Pass 2 - Harden**

> ▶ Prompt to try (copy & paste into Claude Code):
> Add rounding control: a quantize_to(cents=2) method using Decimal.quantize
> with ROUND_HALF_EVEN (banker's rounding). Explain in a docstring why
> ROUND_HALF_UP would bias totals in a large ledger.

**Pass 3 - Prove it**

> ▶ Prompt to try (copy & paste into Claude Code):
> Add pytest tests: constructing from float raises, 0.10 + 0.20 == 0.30
> exactly, adding USD to EUR raises, and banker's rounding turns 2.675 into
> 2.68 at 2 dp. Run the tests and show me the result.

> **Note:** The generate -\> harden -\> prove loop is the core Claude Code rhythm. You are the reviewer and specifier; Claude does the mechanical work. Notice you refined by describing deltas, not re-pasting the file.

**Checkpoint**

- money.py uses Decimal throughout and rejects float.

- The 0.10 + 0.20 == 0.30 test passes exactly - the classic float trap is gone.

- You can explain why banker's rounding matters at ledger scale.

# Lab 4: Prompt Templates, Few-Shot & Chain-of-Thought

*Module 4 - Advanced Prompt Engineering*

| **Domain focus**  | Healthcare    |
|-------------------|---------------|
| **Level**         | Foundation    |
| **Duration**      | 35-40 minutes |
| **Prerequisites** | Labs 1-3      |

**Objective**

- Build a reusable prompt template that encodes your team's conventions.

- Use few-shot examples to pin an output format.

- Use chain-of-thought to enumerate edge cases before coding.

**Part A - A reusable 'endpoint' template**

Templates make generation consistent across a team. Have Claude build one you can save in your repo's prompt library.

> ▶ Prompt to try (copy & paste into Claude Code):
> Create a reusable prompt template (save-ready, with labelled placeholders)
> that asks you to generate a REST endpoint in our house style. Placeholders:
> {method}, {path}, {request_model}, {response_model}, {validation_rules},
> {audit_event}. House rules: pydantic models, structured error responses,
> an audit log line on success, and a matching pytest. Then show one filled-in
> example for GET /patients/{id} returning a FHIR Patient summary.

**Part B - Few-shot to fix an output contract**

Worked examples are the most reliable way to lock a format. Here we standardise validation-error output that your API returns to clients.

> ▶ Prompt to try (copy & paste into Claude Code):
> Emit validation errors in EXACTLY this shape. Examples:
> Input: missing patient id -> {"field":"id","code":"REQUIRED","msg":"id is required"}
> Input: bad birthDate 13/13 -> {"field":"birthDate","code":"FORMAT","msg":"expected YYYY-MM-DD"}
> Now produce the error objects for: (1) empty name array, (2) gender value
> 'unknown-value' not in the allowed set, (3) telecom.system missing.

> **Note:** If output drifts from the contract, add one more example rather than writing longer prose instructions - few-shot beats verbose description.

**Part C - Chain-of-thought for edge cases**

For branchy logic, ask Claude to reason step by step first. This surfaces edge cases you would otherwise find in production.

> ▶ Prompt to try (copy & paste into Claude Code):
> We must decide if a patient is eligible for a subsidised screening: age 50-74
> inclusive, no screening in the last 24 months, and registered at a partner
> clinic. Before writing any code, think step by step and list every edge case
> and boundary you can find (age exactly 50 or 74, screening exactly 24 months
> ago, missing registration, unknown DOB). Then propose the function signature.

**Checkpoint**

- You have a saved, reusable endpoint-generation template.

- Few-shot output matches the error contract exactly.

- The chain-of-thought pass produced an edge-case list before any code.

# Lab 5: Compare Outputs Across Claude Models

*Module 6 - Claude Models Overview & Selection*

| **Domain focus**  | Banking & Finance                                          |
|-------------------|------------------------------------------------------------|
| **Level**         | Foundation                                                 |
| **Duration**      | 25 minutes                                                 |
| **Prerequisites** | A plan with model switching (Opus needs Max or API access) |

**Objective**

- Switch models mid-session and feel the speed/cost/depth trade-off.

- Apply task-based model selection to real workloads.

**Model line-up (at time of writing)**

| **Model**        | **Reach for it when...**                                                                           |
|------------------|----------------------------------------------------------------------------------------------------|
| Claude Opus 4.8  | Deep reasoning: architecture, subtle concurrency/decimal bugs, large refactors, long agentic runs. |
| Claude Sonnet 5  | The default for most implementation, review, and analysis - fast and highly capable.               |
| Claude Haiku 4.5 | High-volume, mechanical work: renames, boilerplate, simple classification, quick lookups.          |

**Steps**

**Step 1.** List models, select Sonnet, and pose a genuine design task.

> Inside the session:
> /model
> /model sonnet

> ▶ Prompt to try (copy & paste into Claude Code):
> Design the core data model for a double-entry ledger: entities, invariants
> (debits == credits per transaction), and how you would prevent unbalanced
> or partially-written transactions. Give it as a concise design note.

**Step 2.** Switch to Haiku, run the identical prompt, and note the speed vs depth.

> ▶ Prompt to try (copy & paste into Claude Code):
> /model haiku

**Step 3.** Switch to Opus, run the identical prompt, and note the extra rigour on invariants and failure modes.

> ▶ Prompt to try (copy & paste into Claude Code):
> /model opus

> **Note:** Claude Code also offers /model opusplan: Opus plans the hard part, then Sonnet implements it - Opus-grade thinking at closer to Sonnet cost. Ideal for 'figure out what to change, then change it' tasks.

**Checkpoint**

- You switched models at least twice with /model.

- You can justify one task each for Haiku, Sonnet, and Opus from your own backlog.

# Lab 6: Token & Context Cost Optimisation

*Module 7 - Token & Cost Optimisation*

| **Domain focus**  | Working in a real repository               |
|-------------------|--------------------------------------------|
| **Level**         | Foundation                                 |
| **Duration**      | 25-30 minutes                              |
| **Prerequisites** | A session with history (do Labs 3-5 first) |

**Objective**

- Inspect and control context usage in a real codebase.

- Keep sessions cheap and sharp at team/enterprise scale.

**Background (brief)**

You pay for input tokens (everything Claude reads: prompt, attached files, and history) and output tokens (what it writes). In a large repo, uncontrolled context is where cost and quality both degrade - irrelevant files dilute the model's attention.

**Steps**

**Step 1.** Inspect current context and session cost.

> Inside the session:
> /context # what is currently loaded into context
> /cost # tokens and spend for this session

**Step 2.** Practise scoping: ask for a change but constrain what Claude reads.

> ▶ Prompt to try (copy & paste into Claude Code):
> Only look at money.py and its test file - do not scan the rest of the repo.
> Add a multiply_by_units(count: int) method to Money and a test for it.

**Step 3.** Compact a long session to reclaim context without losing the thread.

> Inside the session:
> /compact

> **Note:** Habits that cut cost in regulated codebases: keep durable conventions in CLAUDE.md so you never re-explain them; name the specific files a task needs; use /compact instead of restarting when you want to keep continuity; and write precise acceptance criteria so Claude does not over-produce output tokens.

**Checkpoint**

- You viewed /context and /cost and can read them.

- You scoped a change to specific files rather than the whole repo.

- You used /compact on a long session.

# Lab 7: Authentication, Secrets & Secure Prompting

*Module 8 - Authentication, Security*

| **Domain focus**  | Healthcare + Finance              |
|-------------------|-----------------------------------|
| **Level**         | Foundation                        |
| **Duration**      | 30 minutes                        |
| **Prerequisites** | Claude Code installed; a Git repo |

**Objective**

- Configure interactive and CI/headless authentication safely.

- Prevent secrets and client data from leaking into Git or prompts.

- Set tool permissions for a regulated project.

**Part A - Authentication paths**

- Interactive dev machine: run claude and complete the browser login.

- CI / headless: provide ANTHROPIC_API_KEY via your secrets manager - never in the repo or a Dockerfile.

> CI example (key injected by the secrets manager, NOT committed):
> $ export ANTHROPIC_API_KEY="$CI_SECRET_ANTHROPIC_KEY"
> $ claude -p "run the test suite and summarise failures" # headless / print mode

**Part B - Keep secrets and PHI out of Git**

**Step 1.** Have Claude generate a hardening .gitignore and a pre-commit guard.

> ▶ Prompt to try (copy & paste into Claude Code):
> Create a .gitignore that excludes .env, *.key, *.pem, and any *.csv/*.xlsx
> data files. Then add a simple pre-commit hook (bash) that blocks a commit if
> a staged file matches patterns for PAN, Aadhaar, or 16-digit card numbers,
> printing which file and line triggered it. Explain its limits - it is a
> safety net, not a guarantee.

**Part C - Scope tool permissions**

Claude Code asks before running commands or editing files. For a regulated project you can pre-approve safe tools and deny risky ones in settings, so an agentic run cannot, say, touch production config.

> ▶ Prompt to try (copy & paste into Claude Code):
> Explain how Claude Code permissions and allowed-tools settings work, and
> propose a conservative .claude/settings.json for a HIPAA-scoped repo: allow
> reading files and running the test runner, but require explicit approval for
> any shell command that writes outside the repo or makes network calls.

**Part D - Secure prompting**

The most common leak is pasting real data into a prompt. Practise the safe pattern: describe structure, use synthetic rows, and run the generated code yourself against real data.

> UNSAFE - do NOT do this:
> Here is our patient export, find duplicates:
> Ramesh Kumar, 1961-03-12, Aadhaar 1234-5678-9012, HbA1c 8.2 ...(4,000 real rows)

> SAFE - the professional pattern:
> I have a CSV with columns name,dob,patient_id,hba1c. Write Python that finds
> duplicate patients by (name,dob) and prints each duplicate group's patient_ids.
> Test against this synthetic sample:
> Sample One,1961-03-12,P001,8.2
> Sample One,1961-03-12,P047,7.9
> Sample Two,1975-11-02,P002,6.1
> I will run it against the real export inside the client's environment.

**Checkpoint**

- You can describe interactive vs CI authentication and where the key lives in CI.

- .gitignore and the pre-commit guard exist; you understand the guard's limits.

- You produced a conservative permissions config and a safe rewrite of the leaking prompt.

# Lab 8: Identify Unsafe Prompts & Apply Corrections

*Module 9 - Responsible Usage: DOs & DON'Ts*

| **Domain focus**  | Healthcare + Finance    |
|-------------------|-------------------------|
| **Level**         | Foundation              |
| **Duration**      | 30 minutes              |
| **Prerequisites** | Lab 7 recommended first |

**Objective**

- Recognise the three failure modes that hurt regulated clients: data leakage, over-trust in generated output, and hallucination.

- Rewrite unsafe prompts into responsible ones you can defend in an audit.

**DOs and DON'Ts for building regulated client software**

| **DON'T**                                                                         | **DO**                                                                             |
|-----------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| Paste client production data, PHI, PAN, card/account numbers.                     | Describe the schema; use synthetic fixtures; run code in the client's environment. |
| Ship generated code to a client without review and tests.                         | Review, test, and own every line before it leaves your machine.                    |
| Ask Claude to invent a billing code, regulation, or citation.                     | Ask how to look it up in the authoritative source; verify before use.              |
| Let an agent take irreversible actions unattended (deletes, migrations, deploys). | Keep a human approval gate on anything irreversible or regulated.                  |

**Exercise - fix each unsafe prompt**

For each unsafe prompt, write a safe rewrite, then run your rewrite to confirm it still delivers.

> Unsafe 1 (finance):
> Here are 500 real customer transactions with names and account numbers -
> tell me which customers look like money-laundering risks.

> Unsafe 2 (healthcare):
> Give me the exact CPT/reimbursement code for this procedure - just make one
> up if you're unsure, I need to submit the claim today.

> Unsafe 3 (either):
> Connect to the production database and delete all records older than 2 years
> however you think best - you don't need to check with me.

> **Note:** Reference corrections: (1) request the detection logic over synthetic data and run it yourself; (2) ask Claude how to resolve the code against the official schedule and never submit an invented code; (3) request a dry-run that lists what WOULD be deleted, backs up first, and requires an explicit confirmation flag - run against non-prod.

**Prove one corrected prompt**

> ▶ Prompt to try (copy & paste into Claude Code):
> Write a Python function flag_transactions(rows) that flags amounts over a
> configurable threshold and rapid repeated transfers between the same pair.
> Explain each rule so a compliance analyst can review it. Test with synthetic
> rows: (acc_A,acc_B,950000),(acc_A,acc_C,980000). I will run it on real data
> inside the client's environment.

**Checkpoint**

- You produced a safe rewrite for all three unsafe prompts.

- You can articulate why 'just make one up' is a liability in a regulated deliverable.

# Lab 9: Review the Output & Spot the Hallucination

*Module 10 - Review and Own the Outcome*

| **Domain focus**  | Banking & Finance |
|-------------------|-------------------|
| **Level**         | Foundation        |
| **Duration**      | 35 minutes        |
| **Prerequisites** | Labs 1-3          |

**Objective**

- Review generated code critically instead of accepting it.

- Detect a hallucinated (invented) library API and a subtle logic bug.

- Correct both and lock the fix with a test.

**The principle**

You own the outcome, not Claude. Models produce confident, plausible, sometimes wrong output. For a banking client an unreviewed rounding or API error is a defect you shipped. Human-in-the-loop review is mandatory, and tests are how you make review repeatable.

**Part A - Seed a realistic defect**

**Step 1.** Ask Claude to write code with a deliberate, realistic mistake for you to catch.

> ▶ Prompt to try (copy & paste into Claude Code):
> Create interest.py with simple_interest(principal, rate_pct, years) that
> returns principal * rate_pct * years - deliberately forgetting to divide
> rate_pct by 100. Add a comment claiming the result is correct.

**Part B - Review it**

> ▶ Prompt to try (copy & paste into Claude Code):
> Review simple_interest for correctness. Work the maths for principal=10000,
> rate_pct=8.5, years=2, state the value it returns vs the correct value, and
> explain the bug precisely. Do not change code yet.

**Part C - Spot the hallucination**

Now bait an invented API and require Claude to admit non-existence.

> ▶ Prompt to try (copy & paste into Claude Code):
> Is there a stdlib/pandas function that parses SWIFT MT103 messages directly,
> e.g. pandas.read_swift()? If it exists, show the signature and a doc link.
> If it does NOT exist, say so plainly and outline the real approach (a parser
> or an ISO 20022 / MT library).

> **Note:** Lesson: give Claude explicit permission to say 'this does not exist'. Always verify library functions, financial formulas, and regulatory codes against authoritative docs - never against the model's confidence.

**Part D - Correct and lock it in**

> ▶ Prompt to try (copy & paste into Claude Code):
> Fix simple_interest to treat rate_pct as a percentage, correct the comment,
> and add a pytest asserting simple_interest(10000, 8.5, 2) == 1700.0. Run it.

**Checkpoint**

- You identified the rate/100 bug before it was fixed.

- You confirmed whether the SWIFT function was real.

- The corrected function passes its test.

# Lab 10: Analyse an Existing Codebase & Generate Docs (No New Code)

*Module 11 - Code Analysis*

| **Domain focus**  | Healthcare    |
|-------------------|---------------|
| **Level**         | Foundation    |
| **Duration**      | 35-40 minutes |
| **Prerequisites** | Labs 1-2      |

**Objective**

- Get an accurate explanation and dependency map of unfamiliar code.

- Generate documentation from source.

- Run a read-only review - the safest, highest-value use when inheriting a client codebase.

**Part A - Produce a small multi-file service to analyse**

So the room analyses the same thing, first have Claude scaffold a small service, then analyse it as if you just inherited it.

> ▶ Prompt to try (copy & paste into Claude Code):
> Scaffold a tiny appointments service across 3 files: models.py (Appointment
> dataclass), store.py (in-memory repository with book/cancel/list), and
> service.py (book_slot, cancel_slot). Use synthetic data. Plant one subtle
> concurrency-style bug that allows double-booking the same clinician slot.

**Part B - Understand it**

> ▶ Prompt to try (copy & paste into Claude Code):
> You've just inherited this service. Explain each module and the data flow
> to a new team member, and flag risks you notice. Do not modify code.

**Part C - Dependency map & docs**

> ▶ Prompt to try (copy & paste into Claude Code):
> Produce a call/dependency graph across the three files as a simple text tree,
> and generate a README section documenting the public functions with params
> and return types.

**Part D - Read-only reviewer scenario**

The deliverable here is judgement, not code - like vetting a subcontractor's module before you approve it.

> ▶ Prompt to try (copy & paste into Claude Code):
> Act as reviewer. In under 200 words: summarise what the service does, rate
> production-readiness 1-5, and list the top 3 fixes (include the double-booking
> risk). Do NOT write or modify any code.

> **Note:** Read-only analysis carries no risk of unintended edits and is excellent for onboarding onto a client's legacy repo. Pair it with Lab 6's file-scoping to keep large repos cheap to analyse.

**Checkpoint**

- Claude's explanation matches the actual behaviour.

- It surfaced the double-booking risk.

- The read-only review produced a summary and fix-list without editing files.

# Lab 11: Specification-Based Development: Requirement to Implementation

*Module 12 - Specification Based Development*

| **Domain focus**  | Banking & Finance |
|-------------------|-------------------|
| **Level**         | Foundation        |
| **Duration**      | 40 minutes        |
| **Prerequisites** | Labs 3-4          |

**Objective**

- Turn a written spec into an implementation, an API surface, and a test suite.

**Scenario**

Compliance handed you a spec for KYC field validation used at account onboarding. You will implement it with proper structure and tests, using only synthetic identifiers.

**Step 1 - Implement from the spec**

> ▶ Prompt to try (copy & paste into Claude Code):
> Implement this spec in kyc.py.
> validate_pan(pan): Indian PAN = 5 letters, 4 digits, 1 letter (10 chars),
> case-insensitive (normalise to upper). Return {'valid':True} or
> {'valid':False,'reason':<specific text>}.
> validate_ifsc(ifsc): 11 chars, first 4 letters, 5th char '0', last 6
> alphanumeric. Same return shape.
> Use only synthetic values; never fetch or persist real identifiers.

**Step 2 - Generate tests from the spec**

> ▶ Prompt to try (copy & paste into Claude Code):
> Generate pytest tests covering, per validator: a valid value, wrong length,
> wrong character class in a positional slot, lowercase that must still pass
> after normalising, and empty input. Use obviously fake values like ABCDE1234F
> and HDFC0ABC123.

**Step 3 - Expose it as an API**

> ▶ Prompt to try (copy & paste into Claude Code):
> Add a FastAPI router: POST /kyc/pan and POST /kyc/ifsc, each taking a JSON
> body and returning the validator result. Include pydantic request models,
> structured errors, and an example request/response in the docstring.

**Step 4 - Run it**

> Terminal:
> $ pytest -q

> **Note:** Spec-driven generation is only as good as the spec. Explicit inputs, outputs, and edge cases yield precise code; vague specs yield vague code. If Claude asks a clarifying question, your spec had a real gap - answer it and fold the answer back into the spec.

**Checkpoint**

- kyc.py implements both validators with specific rejection reasons.

- Tests exist for every spec case and pass.

- Both endpoints return the validation result as JSON.

# Lab 12: Defect Analysis, Fixing & Resolution

*Module 13 - Defect Analysis, Fixing & Resolution*

| **Domain focus**  | Healthcare   |
|-------------------|--------------|
| **Level**         | Foundation   |
| **Duration**      | 35 minutes   |
| **Prerequisites** | Labs 3 and 9 |

**Objective**

- Reproduce a defect, do root-cause analysis, fix it, and validate.

- Catch a second, deeper bug hiding behind the obvious one.

**Scenario**

An appointment-reminder scheduler sends reminders at the wrong local time for some patients and occasionally crashes. Timezone bugs are a classic in healthcare scheduling - you will reproduce, diagnose, and fix it.

**Step 1 - Seed the buggy code**

> ▶ Prompt to try (copy & paste into Claude Code):
> Create reminder.py with reminder_time(appt_iso, minutes_before) that parses
> appt_iso with datetime.fromisoformat, subtracts minutes_before, and returns
> the result. Introduce TWO bugs: (a) it assumes naive local time and ignores
> any timezone offset, and (b) it crashes when appt_iso has a trailing 'Z'.
> Add a main block calling reminder_time('2026-03-14T09:00:00Z', 30).

**Step 2 - Reproduce and capture the failure**

> Terminal:
> $ python reminder.py

Copy the traceback and give it to Claude for root-cause analysis:

> ▶ Prompt to try (copy & paste into Claude Code):
> Here is the traceback:
> <paste the exact error>
> Do a root-cause analysis: the immediate crash AND the deeper correctness bug
> around timezones. Explain both before changing anything.

**Step 3 - Fix**

> ▶ Prompt to try (copy & paste into Claude Code):
> Fix reminder.py: accept 'Z' and explicit offsets, work in timezone-aware UTC
> internally, and return an aware datetime. Show me the diff of the changes.

**Step 4 - Validate**

> ▶ Prompt to try (copy & paste into Claude Code):
> Add pytest: '...T09:00:00Z' minus 30 == 08:30 UTC; a +05:30 offset input is
> normalised correctly; and a naive input raises a clear error instead of
> silently guessing. Run the tests.

> **Note:** Discipline that prevents shipping a second bug: reproduce first, confirm root cause before editing, then validate with tests that would fail on the old behaviour. In clinical scheduling, 'looks fixed' is not fixed.

**Checkpoint**

- You captured the real traceback and handed it to Claude.

- Both the crash and the timezone correctness bug were identified.

- Tests pass and would have failed against the original code.

# Lab 13: Refactor a Legacy Module Safely

*Module 14 - Refactoring Codebase*

| **Domain focus**  | Banking & Finance |
|-------------------|-------------------|
| **Level**         | Foundation        |
| **Duration**      | 35 minutes        |
| **Prerequisites** | Labs 3 and 9      |

**Objective**

- Detect code smells and refactor without changing behaviour.

- Use a characterisation-test safety net to prove behaviour is preserved.

**Scenario**

You inherited a fee-calculation function nobody wants to touch. It works in production, so you must refactor without altering results - the classic legacy constraint.

**Step 1 - Seed the legacy code**

> ▶ Prompt to try (copy & paste into Claude Code):
> Create fees.py with one long function calc(x, t): if t=='wire' fee is 0.02*x
> with a minimum of 25; if t=='card' fee is 0.015*x; if t=='ach' fee is 5 flat.
> Make it deliberately smelly: single-letter names, magic numbers repeated
> inline, nested if/else, no docstring, no validation, using float for money.

**Step 2 - Detect smells**

> ▶ Prompt to try (copy & paste into Claude Code):
> List the code smells in fees.py, worst first (naming, magic numbers, float
> money, structure, missing docs/validation). Do not refactor yet.

**Step 3 - Pin current behaviour FIRST**

> ▶ Prompt to try (copy & paste into Claude Code):
> Write pytest characterisation tests that capture calc's CURRENT outputs for
> wire (above and below the 25 minimum), card, and ach. Run them so we have a
> green baseline to protect during the refactor.

**Step 4 - Refactor behind the net**

> ▶ Prompt to try (copy & paste into Claude Code):
> Refactor fees.py: descriptive names, named constants, clear structure, a
> docstring, validation for unknown types, and Decimal for money. Behaviour must
> stay identical - the characterisation tests must still pass. Show the diff.

> Terminal:
> $ pytest -q

> **Note:** The characterisation tests from Step 3 are the whole point: refactoring changes structure, not behaviour, and a green baseline is your proof. Never refactor regulated financial logic without that net.

**Checkpoint**

- You have a severity-ordered smell list.

- Baseline tests existed before the refactor and still pass after.

- The refactored code uses named constants and Decimal, with behaviour unchanged.

# Lab 14: Orchestrate a Multi-Step Feature End-to-End

*Module 15 - Basics of Agentic Workflows & Automation*

| **Domain focus**  | Healthcare     |
|-------------------|----------------|
| **Level**         | Foundation     |
| **Duration**      | 40-45 minutes  |
| **Prerequisites** | Labs 3, 10, 12 |

**Objective**

- Drive Claude through plan -\> implement -\> test -\> run across multiple files.

- Apply guardrails and decide where a human approval gate belongs.

**What 'agentic' means here**

An agentic workflow is Claude planning several steps then carrying them out - reading files, editing code, running tests - pausing at checkpoints you define. You set guardrails; Claude does the legwork. Plan mode (Shift+Tab) is your first guardrail.

**Steps**

**Step 1.** Enter plan mode and ask for a PLAN ONLY - no edits. Review it before approving.

> ▶ Prompt to try (copy & paste into Claude Code):
> (Press Shift+Tab until you see plan mode.)
> Add a 'cancel appointment' capability to the appointments service from Lab 10:
> a cancel_slot(appt_id, reason) that frees the slot, writes an audit log entry,
> and refuses to cancel an already-cancelled appointment. Include unit tests.
> Give me a step-by-step PLAN only. Do not edit files yet.

**Step 2.** Approve the plan and let Claude execute, pausing after each file.

> ▶ Prompt to try (copy & paste into Claude Code):
> The plan is good. Implement it step by step. After each file you change,
> stop and tell me what you did and why before continuing.

**Step 3.** Have Claude run the suite and report, then you decide whether to accept.

> ▶ Prompt to try (copy & paste into Claude Code):
> Run pytest and show the results. Summarise what changed across files and
> confirm the audit log entry is written on cancel.

**Guardrails to discuss**

- Plan-first: never let an agent make sweeping edits before you have seen and approved the plan.

- Reversibility: work under version control so any step can be undone (Lab 15).

- Approval gates: automate the mechanical steps, but a human approves anything that writes to real patient data, migrates a schema, or deploys.

> **Note:** When to automate vs keep a human in the loop: automate repetitive, low-risk, reversible steps; gate anything irreversible, regulated, or safety-critical. In Healthcare and Finance that line is drawn conservatively - and your client's auditors will ask where it is.

**Checkpoint**

- You reviewed and approved a plan before any edits.

- The feature spans multiple files, tests pass, and cancel writes an audit entry.

- You can name two steps you would automate and one you would gate behind human approval.

# Lab 15: Explore, Compare & Roll Back Changes

*Module 16 - Explore and Rollback*

| **Domain focus**  | Banking & Finance     |
|-------------------|-----------------------|
| **Level**         | Foundation            |
| **Duration**      | 30 minutes            |
| **Prerequisites** | Git installed; Lab 14 |

**Objective**

- Experiment safely with two independent undo mechanisms.

- Review an AI-generated diff before accepting it.

- Roll back cleanly with Git and with Claude Code's own checkpoints.

**Why this matters**

In regulated systems, 'undo' is a control, not a convenience. You already know Git; the new skill is combining your Git workflow with Claude Code's in-session checkpoints, and reviewing AI diffs as carefully as you would a colleague's PR.

**Step 1 - Baseline with Git**

> Terminal:
> $ git init && git add . && git commit -m "baseline before experiment"

**Step 2 - Make an experimental change**

> ▶ Prompt to try (copy & paste into Claude Code):
> In money.py, add an optional processing_fee (Decimal, default 0) applied on
> the first transaction of a batch. Treat this as a spike I may keep or discard.

**Step 3 - Review the diff before deciding**

> Terminal:
> $ git diff

Read the diff as you would review a PR: are the changes scoped, is money still Decimal, are tests updated? This review step is where you catch problems the model introduced.

**Step 4 - Roll back (two ways)**

- Git - discard uncommitted changes and return to baseline:

> Terminal:
> $ git restore .

- Claude Code checkpoints - inside the session, press Esc twice to open the rewind menu, or type the command below, to step the conversation and edits back to an earlier point:

> Inside the session:
> /rewind

> **Note:** Git and Claude checkpoints are complementary: checkpoints are fast in-session undos while you work; Git is the durable, auditable history your client and auditors rely on. Commit at every known-good state.

**Step 5 - Keep it, if you want it**

> Terminal (only if you decided to keep the change):
> $ git add money.py && git commit -m "spike: optional processing fee"

**Checkpoint**

- You created a Git baseline commit.

- You reviewed an AI-generated diff and could explain what changed.

- You rolled a change back and returned to the baseline cleanly.
