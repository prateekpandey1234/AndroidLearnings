# GitHub Actions & Workflows — What, Why, and How
> This document explains GitHub Actions from the ground up, using **this repository's own workflow** — [`.github/workflows/maestro-test-suit.yml`](../.github/workflows/maestro-test-suit.yml) — as the running example. That workflow builds the WorkIndia Android app and runs Maestro end-to-end (E2E) tests on an Android emulator.
---
## Table of Contents
1. [What is GitHub Actions?](#1-what-is-github-actions)
2. [Why do we use it?](#2-why-do-we-use-it)
3. [The building blocks (concepts & vocabulary)](#3-the-building-blocks-concepts--vocabulary)
4. [How a workflow runs — the big picture](#4-how-a-workflow-runs--the-big-picture)
5. [Our workflow, dissected line by line](#5-our-workflow-dissected-line-by-line)
6. [The full journey of our Maestro workflow (diagram)](#6-the-full-journey-of-our-maestro-workflow)
7. [Expressions, contexts, and variables](#7-expressions-contexts-and-variables)
8. [Caching — why our builds are fast](#8-caching--why-our-builds-are-fast)
9. [Artifacts — getting results out](#9-artifacts--getting-results-out)
10. [Triggers you could add next](#10-triggers-you-could-add-next)
11. [GitHub Actions vs. our Jenkins pipelines](#11-github-actions-vs-our-jenkins-pipelines)
12. [Cheat sheet](#12-cheat-sheet)
---
## 1. What is GitHub Actions?
**GitHub Actions is GitHub's built-in CI/CD (Continuous Integration / Continuous Delivery) platform.** It lets you run automated tasks — building code, running tests, releasing apps — directly from your repository, triggered by events that happen on GitHub (a push, a pull request, a button click, a schedule, …).
Everything is described in **YAML files** that live inside the repository under:
```text
.github/
└── workflows/
    └── maestro-test-suit.yml   ← our only workflow (for now)
```
Because the automation is *versioned alongside the code*, a change to the build process is reviewed in a pull request just like any other code change.
```mermaid
flowchart LR
    A["Event happens on GitHub<br/>(push, PR, manual click, cron)"] --> B["GitHub reads YAML files in<br/>.github/workflows/"]
    B --> C["Matching workflows start"]
    C --> D["Jobs run on runner machines<br/>(fresh VMs)"]
    D --> E["Results reported back:<br/>checks, logs, artifacts"]
```
---
## 2. Why do we use it?
| Problem without CI | How GitHub Actions solves it |
|---|---|
| "It works on my machine" — builds behave differently per developer | Every run starts on a **fresh, identical virtual machine**, so results are reproducible |
| Manual testing is slow and easy to forget | Tests run **automatically** on every trigger; nobody has to remember |
| Broken code gets merged unnoticed | A failing workflow shows a red ❌ on the commit/PR, blocking merges if you want |
| Release steps are tribal knowledge in someone's head | The whole process is **codified in YAML**, reviewable and versioned |
| Long feedback loops | Runs happen in parallel on cloud machines the moment code changes |
For this repository specifically: building an Android APK, booting an emulator, and clicking through the entire app with Maestro takes a long time and a beefy machine. GitHub Actions gives us an **8-vCPU cloud machine on demand** (`blacksmith-8vcpu-ubuntu-2204`) that does all of it unattended and uploads the test results at the end — even when the tests fail.
---
## 3. The building blocks (concepts & vocabulary)
GitHub Actions has a small hierarchy of concepts. From the outside in:
```mermaid
flowchart TD
    subgraph W["Workflow — one YAML file (maestro-test-suit.yml)"]
        subgraph J["Job — run_maestro_tests (runs on one runner VM)"]
            S1["Step: Checkout code"] --> S2["Step: Setup Java"]
            S2 --> S3["Step: Setup Gradle"]
            S3 --> S4["Step: … more steps …"]
            S4 --> S5["Step: Upload artifacts"]
        end
    end
    E["Event / Trigger<br/>(workflow_dispatch)"] -->|starts| W
    R["Runner<br/>blacksmith-8vcpu-ubuntu-2204"] -.->|executes| J
```
| Term | What it is | In our workflow |
|---|---|---|
| **Event / trigger** | The thing that starts a workflow (`on:`) | `workflow_dispatch` — a manual "Run workflow" button in the Actions tab |
| **Workflow** | One YAML file = one automated process | `Run Maestro Tests` |
| **Job** | A group of steps that runs on **one machine**. Jobs can run in parallel or depend on each other | `run_maestro_tests` (we have a single job) |
| **Runner** | The machine (VM) a job executes on. GitHub-hosted, self-hosted, or third-party | `blacksmith-8vcpu-ubuntu-2204` — a third-party (Blacksmith) high-performance runner |
| **Step** | A single task inside a job. Runs sequentially, shares the same filesystem | "Checkout code", "Build Debug APK", … (10 steps total) |
| **Action** | A **reusable, packaged step** published by GitHub or the community, referenced with `uses:` | `actions/checkout@v4`, `actions/setup-java@v4`, `reactivecircus/android-emulator-runner@v2` |
| **`run:` step** | A step that executes raw shell commands instead of a packaged action | The `sed` commands, `./gradlew assemblePreproductionDebug`, … |
| **Artifact** | Files a job saves so you can download them after the run | `maestro-results` (test reports, kept 7 days) |
| **Cache** | Files persisted **between** runs to speed things up | Gradle build cache + the `~/.maestro` install |
> **Key distinction — `uses:` vs `run:`**
> `uses:` pulls in a pre-built action someone already wrote (like importing a library).
> `run:` executes shell commands you write yourself (like writing your own code).
> A good workflow mixes both: reuse where possible, script where necessary.
---
## 4. How a workflow runs — the big picture
The lifecycle of any GitHub Actions run:
```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant Runner as Runner VM
    participant Store as Artifact/Cache storage
    Dev->>GH: Trigger event (here: click "Run workflow")
    GH->>GH: Parse .github/workflows/*.yml, find matching workflows
    GH->>Runner: Provision a fresh VM, send the job definition
    Runner->>GH: Clone the repository (actions/checkout)
    Store-->>Runner: Restore caches (Gradle, Maestro)
    Runner->>Runner: Execute steps one by one
    Runner->>Store: Save caches + upload artifacts
    Runner->>GH: Report status (success ✅ / failure ❌) + stream logs
    GH->>Dev: Show result in the Actions tab / commit checks
```
Important properties to internalize:
- **Every run starts from a clean slate.** The runner VM has nothing from previous runs except what you explicitly restore from *cache*.
- **Steps in a job run sequentially and share a filesystem** — that's why our "Build Debug APK" step can install the APK produced two steps earlier.
- **If any step fails, later steps are skipped by default** — unless a step opts out with `if: always()` (ours does, for uploading test results).
- **Logs stream live** — you can watch the emulator boot and tests run in real time from the Actions tab.
---
## 5. Our workflow, dissected line by line
Below is [`maestro-test-suit.yml`](../.github/workflows/maestro-test-suit.yml) explained section by section.
### 5.1 Name and trigger
```yaml
name: Run Maestro Tests
on:
  workflow_dispatch:
```
- `name:` is what shows in the Actions tab sidebar.
- `on: workflow_dispatch:` means this workflow **only runs when a human presses the "Run workflow" button** (GitHub → *Actions* tab → *Run Maestro Tests* → *Run workflow*). Nothing happens automatically on push or PR. This makes sense here because a full E2E suite is expensive (~up to 90 minutes) — you run it deliberately, not on every commit.
### 5.2 Workflow-level environment variables
```yaml
env:
  GRADLE_OPTS: "-Dorg.gradle.daemon=false -Dorg.gradle.parallel=true -Dorg.gradle.caching=true"
```
`env:` at the top level injects environment variables into **every step of every job**. Here we tune Gradle: no long-lived daemon (pointless on a throwaway VM), parallel module builds, and the build cache switched on.
### 5.3 The job definition
```yaml
jobs:
  run_maestro_tests:
    timeout-minutes: 90
    runs-on: blacksmith-8vcpu-ubuntu-2204
```
- `run_maestro_tests` is the job's ID (one job in this workflow).
- `timeout-minutes: 90` — a safety net: if the emulator hangs, the job is killed instead of burning compute forever.
- `runs-on:` selects the runner. Instead of GitHub's stock `ubuntu-latest`, we use a **Blacksmith** 8-vCPU Ubuntu 22.04 machine — Android builds and emulators are CPU/RAM hungry, and this label routes the job to faster third-party hardware.
### 5.4 The steps — three phases
The 10 steps fall into three logical phases:
```mermaid
flowchart TD
    subgraph P1["Phase 1 — Prepare the machine"]
        A1["1. Checkout code<br/><code>actions/checkout@v4</code>"] --> A2["2. Setup Java 17 (Temurin)<br/><code>actions/setup-java@v4</code>"]
        A2 --> A3["3. Setup Gradle + build cache<br/><code>gradle/actions/setup-gradle@v4</code>"]
        A3 --> A4["4. Restore Maestro cache<br/><code>actions/cache@v3</code>"]
        A4 --> A5["5. Install Maestro 1.30.4<br/>(only if cache missed)"]
    end
    subgraph P2["Phase 2 — Build the app"]
        B1["6. Patch gradle.properties<br/>(prod API URLs, version 650 / 7.1.6.2)"] --> B2["7. ./gradlew assemblePreproductionDebug<br/>→ app-preproduction-debug.apk"]
    end
    subgraph P3["Phase 3 — Test & report"]
        C1["8. Enable KVM<br/>(hardware acceleration for emulator)"] --> C2["9. Boot Pixel 4 emulator (API 33),<br/>install APK, run Maestro flows<br/><code>reactivecircus/android-emulator-runner@v2</code>"]
        C2 --> C3["10. Upload test results as artifact<br/><code>if: always()</code>"]
    end
    P1 --> P2 --> P3
```
#### Phase 1 — Prepare the machine
| Step | What it does | Why |
|---|---|---|
| **Enable KVM** (`run:` + `udev` rules) | Grants all users access to `/dev/kvm` | KVM = hardware virtualization. Without it the Android emulator runs in pure software emulation and is unusably slow |
| **Run Maestro E2E tests** (`reactivecircus/android-emulator-runner@v2`) | Boots a headless **Pixel 4, API 33, x86_64** emulator (4 cores, 4 GB RAM, no window/audio/animations), then runs the `script:` inside it: `adb install` the APK, verify it's installed, and execute `Development/.maestro/run_maestro.sh` with tags `onboarding,home,job_detail,main_list,my_activity_page,profile_page,profile_edit,resume` | This is the heart of the workflow. One community action encapsulates the very fiddly work of creating an AVD, booting it, and waiting for it to be ready. The tags select which Maestro flow suites to run |
| **Upload Maestro Test Results** (`actions/upload-artifact@v4`) | Zips everything under `Development/.maestro/tests/**` into an artifact named `maestro-results`, kept for 7 days. **`if: always()`** | `if: always()` is crucial: by default steps are skipped once something fails — but a *failed* test run is exactly when you most need the reports and screenshots. This guarantees you can always download the evidence |
---
## 6. The full journey of our Maestro workflow
End-to-end, including the failure path:
```mermaid
flowchart TD
    Start(["Developer clicks 'Run workflow'<br/>in the Actions tab"]) --> Provision["GitHub provisions a Blacksmith<br/>8-vCPU Ubuntu 22.04 VM"]
    Provision --> Checkout["Shallow-clone the repo"]
    Checkout --> Toolchain["Install JDK 17 + configure Gradle"]
    Toolchain --> CacheCheck{"Maestro in cache?"}
    CacheCheck -- "hit" --> PathOnly["Just add ~/.maestro/bin to PATH"]
    CacheCheck -- "miss" --> Download["Download Maestro 1.30.4,<br/>then add to PATH"]
    PathOnly --> Patch["sed-patch gradle.properties<br/>(API URLs + version)"]
    Download --> Patch
    Patch --> Build["./gradlew assemblePreproductionDebug"]
    Build --> KVM["Enable KVM acceleration"]
    KVM --> Emu["Boot headless Pixel 4 emulator (API 33)"]
    Emu --> Install["adb install the freshly built APK"]
    Install --> Tests["run_maestro.sh — Maestro clicks through<br/>onboarding, home, job detail, profile, resume…"]
    Tests -- "all flows pass" --> Green["✅ Job succeeds"]
    Tests -- "a flow fails" --> Red["❌ Job fails"]
    Green --> Upload["Upload test results artifact<br/>(if: always — runs on BOTH paths)"]
    Red --> Upload
    Upload --> Done(["Logs + 'maestro-results' artifact<br/>available for 7 days"])
```
---
## 7. Expressions, contexts, and variables
Workflows aren't static text — GitHub evaluates **expressions** written as `${{ … }}` before/while running. Our file uses several:
| Expression in our workflow | Context it reads | Meaning |
|---|---|---|
| `${{ github.ref != 'refs/heads/main' }}` | `github` — info about the event/repo | "Is this run *not* on the `main` branch?" → used to make the Gradle cache read-only off-`main` |
| `${{ runner.os }}` | `runner` — info about the VM | Resolves to `Linux`; part of the Maestro cache key so an OS change invalidates the cache |
Other commonly used contexts you'll meet:
- `${{ secrets.MY_TOKEN }}` — encrypted repository/organization secrets (API keys, signing keys). Never printed in logs.
- `${{ github.event_name }}`, `${{ github.actor }}`, `${{ github.sha }}` — what triggered the run, who, and which commit.
- `${{ env.SOMETHING }}` — environment variables defined with `env:`.
There are also **special files** for talking to the runner from shell scripts:
- `echo "some/dir" >> "$GITHUB_PATH"` — prepend to `PATH` for subsequent steps (our Maestro install step does this).
- `echo "KEY=value" >> "$GITHUB_ENV"` — set an env var for subsequent steps.
---
## 8. Caching — why our builds are fast
A fresh VM means nothing survives between runs — downloads and compilation would repeat every time. Caching fixes that. We use **two independent caches**:
```mermaid
flowchart LR
    subgraph Run1["Run #1 (cold)"]
        D1["Download all Gradle deps<br/>+ compile everything<br/>+ download Maestro"] --> S1["Save caches"]
    end
    subgraph Storage["GitHub cache storage"]
        K1[("Gradle cache<br/>(deps + build cache)")]
        K2[("maestro-Linux-1.30.4<br/>(~/.maestro)")]
    end
    subgraph Run2["Run #2 (warm)"]
        R2["Restore caches"] --> F2["Only changed code recompiles,<br/>Maestro install skipped entirely"]
    end
    S1 --> K1 & K2
    K1 & K2 --> R2
```
1. **Gradle cache** — managed automatically by `gradle/actions/setup-gradle@v4`. Combined with `--build-cache` and `GRADLE_OPTS`, unchanged modules aren't recompiled. The `cache-read-only` guard means only `main` runs update the shared cache.
2. **Maestro cache** — `actions/cache@v3` with key `maestro-${{ runner.os }}-1.30.4`. The version number is baked into the key, so bumping Maestro to a new version automatically misses the old cache and triggers a fresh install.
> **Cache vs. artifact:** a *cache* is an optimization between runs (may be evicted, keyed lookup). An *artifact* is an output of a run you want to keep and download (test reports, APKs).
---
## 9. Artifacts — getting results out
The runner VM is destroyed after the job — anything not uploaded is gone. Our last step:
```yaml
- name: Upload Maestro Test Results
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: maestro-results
    path: |
      Development/.maestro/tests/**
    retention-days: 7
    if-no-files-found: warn
```
- Downloadable from the run's **Summary page** in the Actions tab (a zip named `maestro-results`).
- `retention-days: 7` — auto-deleted after a week to save storage.
- `if-no-files-found: warn` — don't fail the whole run just because no reports were produced (e.g., the build failed before tests started); just print a warning.
- `if: always()` — upload even when earlier steps failed (see §5.4).
---
## 10. Triggers you could add next
Today the suite is manual-only. The `on:` block supports many other events — some natural next steps for this repo:
```yaml
on:
  workflow_dispatch:          # keep the manual button
  pull_request:               # run automatically on every PR into develop
    branches: [develop]
  schedule:                   # nightly run at 02:00 UTC (cron syntax)
    - cron: "0 2 * * *"
  push:
    tags: ["v*"]              # run when a release tag like v7.1.7 is pushed
```
You can also add **inputs** to `workflow_dispatch` (like Jenkins parameters — see next section), e.g. letting the person triggering the run choose which Maestro tags to execute:
```yaml
on:
  workflow_dispatch:
    inputs:
      tags:
        description: "Comma-separated Maestro tags to run"
        default: "onboarding,home"
        required: true
# …then reference it as ${{ inputs.tags }} in the test step
```
Other useful features not yet used here: multiple parallel jobs with `needs:` dependencies, `matrix:` builds (e.g., test on API 30 *and* 33 simultaneously), `secrets` for signing keys, and `concurrency:` to cancel superseded runs.
---
## 11. GitHub Actions vs. our Jenkins pipelines
This repo also contains `Development/JenkinsfileProd` and `Development/JenkinsfileStag` — Jenkins pipelines used for Play Store releases. Same idea (pipeline-as-code), different platform:
| Aspect | GitHub Actions (`maestro-test-suit.yml`) | Jenkins (`JenkinsfileProd`) |
|---|---|---|
| Where it runs | Cloud runners provisioned per run | A Jenkins server/agents we host and maintain |
| Definition language | YAML | Groovy DSL |
| Trigger | GitHub events (here: manual dispatch) | Jenkins UI / webhooks |
| Parameters | `workflow_dispatch` `inputs:` | `parameters { string(...) choice(...) }` |
| Reuse mechanism | Marketplace actions (`uses:`) | Jenkins plugins + shared libraries |
| Infrastructure upkeep | None (managed) | Ours to patch, scale, secure |
| Integration with PRs | Native (checks, required status) | Via plugins/webhooks |
Interestingly, both do the exact same `sed` trick on `gradle.properties` before building — the same release-engineering logic, expressed in two systems.
---
## 12. Cheat sheet
```yaml
name: <workflow name>            # shown in the Actions tab
on: <events>                     # what starts it: push / pull_request /
                                 # workflow_dispatch / schedule / …
env:                             # variables for all jobs & steps
  KEY: value
jobs:
  <job_id>:
    runs-on: <runner label>      # which machine
    timeout-minutes: <n>         # kill switch
    steps:
      - name: <human label>
        uses: <owner/action@vN>  # reusable action …
        with: { key: value }     #   … configured via inputs
      - name: <human label>
        run: <shell commands>    # or raw shell
        working-directory: <dir>
        if: always()             # condition (always / success / failure / expression)
```
**Where to look when something fails:**
1. Repo → **Actions** tab → click the red run.
2. Click the failing job → expand the failing step → read the log.
3. Download the `maestro-results` artifact from the run summary for screenshots/reports of the failed flows.
**Official docs:** [docs.github.com/actions](https://docs.github.com/en/actions) · Workflow syntax reference: [docs.github.com/actions/reference/workflow-syntax-for-github-actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
