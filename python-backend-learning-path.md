# Python Backend Learning & Practice Guide (WorkIndia Reference)

A structured roadmap to learn and practice Python backend development using **WorkIndia's conventions, repos, and microservice patterns** as the reference standard.

> **Primary references**
> - Org: [WorkIndia-Private on GitHub](https://github.com/WorkIndia-Private)
> - Backend guidelines: `wi-backend-guidelines` (internal skill / `.claude/skills/wi-backend-guidelines/`)
> - Sample service: [wi-sample-ms](https://github.com/WorkIndia-Private/wi-sample-ms)
> - Shared libs: [wi-djangoutils-py](https://github.com/WorkIndia-Private/wi-djangoutils-py), [wi-fastapiutils-py](https://github.com/WorkIndia-Private/wi-fastapiutils-py)
> - FastAPI scaffold: [cookiecutter-fastapi-template](https://github.com/WorkIndia-Private/cookiecutter-fastapi-template)

---

## What WorkIndia Backend Looks Like

WorkIndia runs **Python microservices** (`*-ms`) built on:

| Layer | Django services | FastAPI services |
|-------|-----------------|------------------|
| HTTP | `BaseAPIView` (DRF) | `wifastapiutils.views.APIView` |
| Orchestration | Controller | Controller |
| Business + DB | Helper | Handler (+ Helper) |
| Reusable logic | Utils | Utils |
| Validation | Pydantic (not DRF serializers) | Pydantic |
| Logging | `wilogkit` (structured) | `wilogkit` (structured) |
| Messaging | `wikafkaadapter` (Kafka) | Kafka adapters |
| Scheduled jobs | Django management commands | — |
| Config | `settings/` + Consul + env vars | `pydantic_settings` + env vars |

**Typical repo layout** (from `wi-sample-ms`, `wiemployerjobpost-ms`):

```text
.
├── app/
│   ├── api/
│   │   ├── views/
│   │   ├── controllers/
│   │   ├── helpers/          # or handlers/ (FastAPI)
│   │   ├── pydantic_models/
│   │   ├── pubsub/           # producers, consumers, payloads
│   │   └── management/commands/
│   ├── <service-name>/
│   │   └── settings/         # base, development, staging, production
│   ├── manage.py
│   └── requirements/
├── build/                    # Docker, K8s, Helm (prod_values.yml)
├── test/                     # Postman collections
├── Makefile
└── pyproject.toml
```

---

## How to Use This Guide

1. **Learn** the concept in each phase (theory + read WorkIndia code).
2. **Practice** the exercise in your own mini-project before moving on.
3. **Review** your code against the checklist at the end of each phase.
4. **Clone and read** real services — don't just read docs.

**Suggested pace:** 8–12 weeks part-time (5–10 hrs/week). Adjust based on your Python baseline.

---

## Phase 0 — Python Foundations (Week 1)

### What to learn

- Python 3.10+ syntax: functions, classes, modules, packages
- Type hints: `Optional`, `List`, `Dict`, `Union`, return types
- OOP: classes, inheritance, `@staticmethod`, `__init__`
- Enums (`enum.Enum`) and constants (`UPPER_SNAKE_CASE`)
- PEP 8: import order (stdlib → third-party → local), naming
- Virtual environments: `venv`, `pip`, `requirements.txt`
- Git basics: branch, commit, PR workflow

### What to practice

| # | Exercise | Success criteria |
|---|----------|------------------|
| 1 | Build a CLI `Job` class with `title`, `status` (enum), `created_at` | Type hints on all methods; no magic strings for status |
| 2 | Write `JobRepository` with `create`, `get_by_id`, `list_active` using in-memory list | Class-based; specific exceptions (`JobNotFoundError`) |
| 3 | Add structured logging with a logger (use `logging` module first; switch to `wilogkit` in Phase 1) | Log with key=value fields, not f-strings in log messages |
| 4 | Refactor a 80-line script into 3 modules: `models`, `repository`, `main` | Clear separation of concerns |

### WorkIndia reference

- Read: `01-python-fundamentals.md` in `wi-backend-guidelines`
- Skim any `api/helpers/*.py` in [wiemployerjobpost-ms](https://github.com/WorkIndia-Private/wiemployerjobpost-ms) — notice class-based helpers and type hints

---

## Phase 1 — Pydantic, Logging & Error Handling (Week 2)

### What to learn

- **Pydantic v2**: `BaseModel`, `Field`, `@field_validator`, `model_config = {"from_attributes": True}`
- Separate **request** and **response** models
- **wilogkit**: structured logging (`logger.info("event_name", job_id=123)`)
- Specific exception handling — never `except Exception:` or bare `except:`
- Custom error types (WorkIndia uses `wierrors`)

### What to practice

| # | Exercise | Success criteria |
|---|----------|------------------|
| 1 | Define `JobCreateRequest`, `JobResponse`, `JobListRequest` Pydantic models | Validation rules via `Field()`; separate req/res models |
| 2 | Build a fake API layer that validates JSON → Pydantic → returns `model_dump()` | Invalid input returns structured validation errors |
| 3 | Replace all `print()` with wilogkit-style structured logs | Every log includes relevant IDs (`job_id`, `employer_id`) |
| 4 | Implement `JobNotFoundError`, `InvalidJobStateError` with clear messages | Callers catch specific types, not broad exceptions |

### WorkIndia reference

- Guidelines: `01-python-fundamentals.md`, `02-architecture.md` (Error Handling section)
- Search in any `*-ms` repo: `api/pydantic_models/` or `api/models/pydantic/`

---

## Phase 2 — Django REST API Basics (Weeks 3–4)

WorkIndia's majority stack is **Django + DRF views + Pydantic validation** (not DRF serializers).

### What to learn

- Django project structure: `settings/`, `urls.py`, `manage.py`
- Django ORM basics: `Model`, `ForeignKey`, `QuerySet`, migrations
- DRF: `Response`, HTTP status codes
- **View → Controller → Helper → Utils** layering
- `BaseAPIView` from `wierrors` (not plain `APIView`)
- Settings by flavour: `development`, `staging`, `production`

### What to practice

Build a mini **Job Posting Service** locally (no need to match WorkIndia infra yet):

| # | Exercise | Files to create |
|---|----------|-----------------|
| 1 | Scaffold Django project with split settings | `settings/base.py`, `development.py` |
| 2 | Create `Job` and `Employer` models with indexes, `db_table`, timestamps | `api/models.py` |
| 3 | Implement `CreateJobView` → `JobController` → `JobHelper` | `views/`, `controllers/`, `helpers/` |
| 4 | Add `GET /jobs/` with pagination and `GET /jobs/{id}/` | Use Pydantic for query params and response |
| 5 | Add `PATCH /jobs/{id}/close/` with state validation | Enum for status; helper owns business rules |
| 6 | Write 3 unit tests: create job, validation failure, not found | `pytest` or Django `TestCase` |

**Layer rules to enforce:**

```text
View       → HTTP only (validate input, call controller, format response)
Controller → Orchestration only (no DB, no HTTP)
Helper     → Business logic + ORM
Utils      → Stateless helpers (date formatting, string normalize)
```

### WorkIndia reference

- Clone [wi-sample-ms](https://github.com/WorkIndia-Private/wi-sample-ms) — run `make init` and `make runserver`
- Read: `04-django-endpoints.md`
- Explore: [wiemployerjobpost-ms](https://github.com/WorkIndia-Private/wiemployerjobpost-ms) — `app/api/controllers/`, `app/api/helpers/`, `app/api/views/`
- Shared lib: [wi-djangoutils-py](https://github.com/WorkIndia-Private/wi-djangoutils-py)

---

## Phase 3 — Database Patterns (Week 5)

### What to learn

- Query optimization: `select_related`, `prefetch_related`, `only`, `values`, `defer`
- N+1 problem and how to spot it
- Bulk ops: `bulk_create`, `.update()` vs row-by-row loops
- Transactions: `transaction.atomic()`, `select_for_update()`
- **Rule:** no external calls (email, Kafka) inside transactions
- Indexing strategy for WHERE / ORDER BY / JOIN fields
- Cache-before-query pattern (Redis via `wiredisadapter` at WorkIndia)

### What to practice

| # | Exercise | Success criteria |
|---|----------|------------------|
| 1 | Fix an intentional N+1 in a list endpoint | Use `select_related`; verify query count with Django Debug Toolbar or `assertNumQueries` |
| 2 | Bulk import 1000 jobs via `bulk_create(batch_size=500)` | No loop with `.create()` |
| 3 | Implement "publish job" with `atomic()` + audit log row | Kafka/email simulated calls happen **outside** the transaction |
| 4 | Add composite index on `(employer_id, status)` | Migration created; explain why in a comment |
| 5 | Add Redis-style cache wrapper (or Django cache) for `get_employer()` | Cache key pattern: `employer:{id}`, TTL 300s |

### WorkIndia reference

- Read: `06-database.md`
- Search repos for: `.select_related(`, `.bulk_create(`, `transaction.atomic`

---

## Phase 4 — Architecture & Design Patterns (Week 6)

### What to learn

- SOLID principles (especially Single Responsibility, Dependency Injection)
- Class-based design — no module-level business functions
- Factory and Strategy patterns
- Behavior-based naming (`JobHelperWithGeoFilter`) — not `V2`, `V3` class names
- Size limits: class < 300 lines, method < 50 lines
- Dependency injection via `__init__(self, helper=None)`

### What to practice

| # | Exercise | Success criteria |
|---|----------|------------------|
| 1 | Refactor a fat view that does DB + logic into proper layers | View has zero ORM calls |
| 2 | Add a `PricingStrategy` interface with two implementations | Controller accepts strategy via DI |
| 3 | Add a new job sort behavior via **new helper subclass**, not editing existing helper | Open/Closed principle |
| 4 | Write a `JobController` with `isinstance` guard in `__init__` | Raises `TypeError` if wrong type passed |
| 5 | Code review your own project using the WorkIndia review template | Fill ✅ / ⚠️ / 🔴 sections |

### WorkIndia reference

- Read: `02-architecture.md`
- Notice in `wiemployerjobpost-ms`: multiple controllers like `editcontrollerv2.py` vs behavior-named classes — understand what **not** to do for new code

---

## Phase 5 — Kafka / Pub-Sub (Week 7)

WorkIndia uses **Kafka** for async event-driven workflows via `wikafkaadapter`.

### What to learn

- Producer: `@Singleton`, `BaseProducer`, `BasePydanticProducerAdapter`
- Consumer: `BasePydanticConsumer`, thin `handle_message()` → delegate to helper
- Topic constants in `pubsub/kafkaconfig.py` — no magic strings
- Pydantic payloads in `pubsub/payloads/`
- Management command to start consumer
- Never produce inside `transaction.atomic()`

### What to practice

| # | Exercise | Success criteria |
|---|----------|------------------|
| 1 | Define `JobEventPayload` with `JobAction` enum | Pydantic + enum for action field |
| 2 | On job create, produce a `job.created` event (mock producer if no Kafka) | Producer adapter pattern; log before/after |
| 3 | Build `JobEventConsumer` that delegates to `JobEventHelper.process()` | No business logic in consumer |
| 4 | Add `python manage.py run_job_event_consumer` command | Command's `handle()` is one line |
| 5 | Draw a sequence diagram: API → DB commit → produce event → consumer → helper | Understand ordering and failure modes |

### WorkIndia reference

- Read: `07-kafka.md`
- Skills: `create-consumer`, `create-producer`
- Explore: `app/api/pubsub/` and `app/api/management/commands/*consumer*` in production services

---

## Phase 6 — Cron Jobs & Background Processing (Week 8)

### What to learn

- Django management commands: `BaseCommand`, `add_arguments`, `CommandError`
- Idempotent cron design (`update_or_create`, dedup keys)
- `.iterator(chunk_size=500)` for large datasets
- K8s CronJob config in `build/prod_values.yml`
- When to use cron vs Kafka consumer

### What to practice

| # | Exercise | Success criteria |
|---|----------|------------------|
| 1 | Create `sync_expired_jobs_command` that closes jobs past expiry | Logic in helper; command delegates only |
| 2 | Add `--dry-run` flag | Validates args; raises `CommandError` on bad input |
| 3 | Make the sync idempotent — safe to run twice | No duplicate side effects |
| 4 | Draft a `prod_values.yml` cron entry with resources limits | Includes `DJANGO_SETTINGS_MODULE`, schedule, CPU/memory |
| 5 | Document: when would you pick cron vs consumer for a new feature? | 1-page decision note |

### WorkIndia reference

- Read: `08-cron-jobs.md`
- Explore: `app/api/management/commands/` in [wiemployerjobpost-ms](https://github.com/WorkIndia-Private/wiemployerjobpost-ms)

---

## Phase 7 — FastAPI Track (Week 9, optional but valuable)

Many newer WorkIndia services use **FastAPI** with `wifastapiutils`.

### What to learn

- FastAPI + `wifastapiutils.views.APIView`
- `@APIView.configure(...)` decorator pattern
- View → Controller → **Handler** (instead of Helper for DB)
- Pydantic models in `api/pydantic_models/` (shared, no circular imports)
- Route registration via `View.register_route(router, path, dependencies=[Depends(auth)])`
- `pydantic_settings.BaseSettings` for config

### What to practice

| # | Exercise | Success criteria |
|---|----------|------------------|
| 1 | Scaffold from [cookiecutter-fastapi-template](https://github.com/WorkIndia-Private/cookiecutter-fastapi-template) | Project runs locally |
| 2 | Implement `POST /jobs/` with full layer stack | Models in `pydantic_models/`, not in view |
| 3 | Add API key auth via `Depends` at route registration | Reuse existing auth pattern |
| 4 | Port one endpoint from your Django practice project to FastAPI | Same Pydantic contracts, different framework |

### WorkIndia reference

- Read: `05-fastapi-endpoints.md`, `03-settings-configs.md` (FastAPI section)
- Shared lib: [wi-fastapiutils-py](https://github.com/WorkIndia-Private/wi-fastapiutils-py)

---

## Phase 8 — Settings, Config & Production Readiness (Week 10)

### What to learn

- Environment-based settings (`FLAVOUR=development|staging|production`)
- No hardcoded secrets — env vars + startup validation
- Consul for runtime feature flags (conceptual)
- Docker + Makefile commands (`make init`, `make runserver`)
- Pre-commit hooks (`.pre-commit-config.yaml`)
- Postman collections for API testing

### What to practice

| # | Exercise | Success criteria |
|---|----------|------------------|
| 1 | Split settings into `base/development/staging/production` | Fail fast if required env vars missing |
| 2 | Dockerize your practice service | `Dockerfile` + `docker compose up` works |
| 3 | Add `.pre-commit-config.yaml` with black/flake8/isort | Hooks pass on commit |
| 4 | Write Postman collection for all endpoints | Environment variables for base URL |
| 5 | Add a `/health/` endpoint | Returns 200 with service name and flavour |

### WorkIndia reference

- Read: `03-settings-configs.md`
- Compare: `app/wi-sample-ms/settings/` and `build/prod_values.yml`

---

## Phase 9 — Capstone Project (Weeks 11–12)

Build one end-to-end feature as if shipping at WorkIndia.

### Suggested capstone: **Employer Job Post Lifecycle**

**Features:**
1. `POST /api/jobs/` — create draft job
2. `PATCH /api/jobs/{id}/publish/` — publish with validation
3. `GET /api/jobs/` — list with filters + pagination
4. Kafka event on publish (`job.published`)
5. Consumer updates a read model or sends notification (simulated)
6. Cron: auto-close jobs expired after 30 days
7. Tests for happy path + validation + not found
8. Structured logging throughout

**Definition of done (WorkIndia checklist):**

- [ ] `wilogkit` logging — no `print()`
- [ ] Pydantic for all I/O — no DRF serializers
- [ ] Type hints on all signatures
- [ ] Class-based views, controllers, helpers
- [ ] `BaseAPIView` (Django) or `APIView` (FastAPI)
- [ ] No business logic or ORM in views/controllers
- [ ] No broad `except Exception`
- [ ] Enums/constants for status values
- [ ] Kafka produce outside transactions
- [ ] Cron command delegates to helper; idempotent
- [ ] Settings split by environment; no hardcoded secrets

---

## Repos to Study (by priority)

| Priority | Repo | Why |
|----------|------|-----|
| 1 | [wi-sample-ms](https://github.com/WorkIndia-Private/wi-sample-ms) | Minimal Django service skeleton |
| 2 | [wi-djangoutils-py](https://github.com/WorkIndia-Private/wi-djangoutils-py) | Shared Django utilities |
| 3 | [wi-fastapiutils-py](https://github.com/WorkIndia-Private/wi-fastapiutils-py) | FastAPI base classes |
| 4 | [cookiecutter-fastapi-template](https://github.com/WorkIndia-Private/cookiecutter-fastapi-template) | New service scaffold |
| 5 | [wiemployerjobpost-ms](https://github.com/WorkIndia-Private/wiemployerjobpost-ms) | Real controllers, helpers, crons, consumers |
| 6 | [wiredisadapter](https://github.com/WorkIndia-Private/wiredisadapter) | Redis caching patterns |
| 7 | [wi-consul-python-adapter](https://github.com/WorkIndia-Private/wi-consul-python-adapter) | Runtime config |

**Other `*-ms` repos worth browsing:** `wi-user-ms`, `wi-chat-ms`, `wi-referral-ms`, `wi-document-ms`

---

## Weekly Study Routine

| Day | Activity | Time |
|-----|----------|------|
| Mon | Read guideline doc + explore 1 WorkIndia repo file | 1 hr |
| Tue–Wed | Build practice exercise | 2–3 hrs |
| Thu | Write tests for what you built | 1 hr |
| Fri | Self-review against WorkIndia checklist; note gaps | 30 min |
| Sat | Read one PR or diff in a real `*-ms` repo (if access) | 1 hr |

---

## Common Mistakes to Avoid

| Mistake | WorkIndia standard |
|---------|-------------------|
| Business logic in views | Views delegate to controller only |
| DRF serializers for validation | Pydantic models only |
| `print()` for debugging | `wilogkit` structured logging |
| Raw topic strings `"job-events"` | `KafkaTopic.JOB_EVENTS` constant |
| ORM calls in controllers | DB access in helpers/handlers only |
| `except Exception:` | Catch specific exceptions |
| Magic strings for status | `JobStatus.ACTIVE.value` enum |
| Kafka produce inside transaction | Produce after commit |
| Class named `HelperV2` | Behavior name: `HelperWithGeoFilter` |
| Loading 100k rows into memory | `.iterator(chunk_size=500)` |

---

## External Resources (fill gaps)

| Topic | Resource |
|-------|----------|
| Python typing | [docs.python.org/3/library/typing.html](https://docs.python.org/3/library/typing.html) |
| Pydantic v2 | [docs.pydantic.dev](https://docs.pydantic.dev) |
| Django | [docs.djangoproject.com](https://docs.djangoproject.com) |
| Django ORM perf | [docs.djangoproject.com/en/stable/topics/db/optimization/](https://docs.djangoproject.com/en/stable/topics/db/optimization/) |
| FastAPI | [fastapi.tiangolo.com](https://fastapi.tiangolo.com) |
| Kafka concepts | [kafka.apache.org/documentation/](https://kafka.apache.org/documentation/) |
| SOLID | Clean Architecture / Fowler's patterns (conceptual) |

---

## Quick Self-Assessment

Rate yourself 1–5 after each phase:

| Skill | 1 (new) | 3 (comfortable) | 5 (production-ready) |
|-------|---------|-----------------|----------------------|
| Python + type hints | | | |
| Pydantic validation | | | |
| Django layered architecture | | | |
| ORM + query optimization | | | |
| Kafka producer/consumer | | | |
| Cron / management commands | | | |
| FastAPI (optional) | | | |
| Settings & deployment basics | | | |

**Target:** Score ≥ 3 on all rows before contributing to a WorkIndia backend repo.

---

## Next Steps

1. Clone `wi-sample-ms` and get it running locally (`make init && make runserver`).
2. Start **Phase 0** if Python basics need refresh; otherwise jump to **Phase 1**.
3. Keep this file updated — tick off exercises as you complete them.
4. When ready, ask for a small backend ticket in a real service and compare your capstone structure to production code.

---

*Generated from WorkIndia backend guidelines (`wi-backend-guidelines`) and org repo structure. Update as internal standards evolve.*
