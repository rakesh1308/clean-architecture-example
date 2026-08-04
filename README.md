# Clean Architecture Android Example

A reference Android application demonstrating **Uncle Bob's Clean
Architecture** with **Dagger 2** dependency injection and **Retrofit 2** for
remote data — separated into three Gradle modules with independent test
sources.

The project was built as a teaching template for separating concerns in
non-trivial Android apps and for showing where unit tests belong at each
layer of the architecture.

---

## Architecture

```
┌────────────────────────────────────────────────────────┐
│  :app   — UI layer (Activities, Adapters, Presenters)  │
│          depends on → domain (interface only)          │
│                    → data   (Dagger graph wiring)      │
└────────────────────────────────────────────────────────┘
        ▲                                 ▲
        │ exposes Presenters              │ implements domain interfaces
        │ (MVP)                           │ (Repository, DataSource)
┌────────────────────────────────────────────────────────┐
│  :data  — Repository implementations, Retrofit APIs,  │
│          entity ↔ domain model mappers, Dagger modules│
│          depends on → domain                           │
└────────────────────────────────────────────────────────┘
        ▲
        │ pure Kotlin / Java business rules
┌────────────────────────────────────────────────────────┐
│  :domain — Entities, Use Cases (Interactors),         │
│           Repository interfaces, no Android imports   │
└────────────────────────────────────────────────────────┘
```

The **dependency rule** is strictly enforced: dependencies only point
inward (`app` → `data` → `domain`). The `domain` module is pure Java and
has no Android dependencies, so its business rules can be exercised with
plain JUnit on the JVM.

---

## Tech stack

| Concern | Choice |
|---|---|
| Language | Java 8 |
| Architecture | Clean Architecture + MVP |
| DI | Dagger 2 (per-Activity / per-Application components) |
| Networking | Retrofit 2 |
| Async | Custom `Executor` injection (testable, no `AsyncTask`) |
| Build | Gradle (Groovy DSL) |
| CI | Jenkins (`jenkinsfile` included) |

---

## Module layout

```
.
├── app/        # UI layer — Activities, Presenters, Adapters, Dagger wiring
├── data/       # Repository implementations, Retrofit services, model mappers
├── domain/     # Pure Java — Entities, Use Cases, Repository interfaces
├── gradle/
├── build.gradle
├── settings.gradle
└── jenkinsfile
```

### `domain` — what lives here
- `bean/` — plain entities (`Comic`, …)
- `interactor/` — use cases (`GetComicsUseCase`, `GetComicDetailUseCase`)
- `repository/` — `ComicsRepository` interface (implemented in `data`)
- `executor/` — `Executor` interface (testable alternative to `AsyncTask`)
- `exception/` — domain-level errors

### `data` — what lives here
- `repository/` — `ComicsRepositoryImpl`
- `net/` — Retrofit `ApiService` + `RestApi` adapter
- `entity/mapper/` — DTO ↔ domain entity mappers
- `di/` — `DataModule` for Dagger

### `app` — what lives here
- `presentation/` — `ComicsPresenter`, `ComicDetailPresenter` (MVP)
- `view/` — `MainActivity`, `ComicDetailActivity`, `ComicsAdapter`
- `di/` — `ApplicationComponent`, `ComicsComponent`

---

## Testing strategy

Unit tests live next to the code they exercise:

| Module | Test source set | What it tests |
|---|---|---|
| `domain` | `domain/src/test/` | Interactors with a fake `Executor` and fake repository |
| `data`   | `data/src/test/`   | Repository against a fake `ApiService` |
| `app`    | `app/src/test/`    | Presenters with fake interactors |

The Espresso (`androidTest`) source sets in `app/` and `data/` are present
but minimal — the focus is on **fast JVM unit tests** at the Presenter and
Interactor levels, which is where Clean Architecture pays off most.

Run the JVM test suite:

```bash
./gradlew test
```

---

## Notable patterns demonstrated

- **Per-Activity Dagger component** (`ComicsComponent`) for scoped
  presenter injection, with `@PerActivity` scope
- **Interface-based Executor injection** — presentable from any thread pool
  for production, single-thread for tests
- **Mapper layer** between Retrofit DTOs and domain entities — domain
  never sees transport-level types
- **Navigator abstraction** — present an intent without knowing about
  `Activity` (improves Presenter testability)

---

## License

MIT — see `LICENSE` if present, otherwise standard MIT terms apply.
