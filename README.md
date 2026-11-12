# Actix Web SQLx Starter

A small, opinionated starting point for a versioned HTTP API built with [Actix Web](https://actix.rs/). It includes YAML and environment-based configuration, structured request logging, a PostgreSQL connection pool through [SQLx](https://github.com/launchbadge/sqlx), database migrations, a health endpoint, and integration tests.

## Get started

You'll need [Rust and Cargo](https://rustup.rs/), Docker, and the SQLx CLI. The project uses Rust edition 2024, which requires Rust 1.85 or later; its dependencies may require a newer compiler.

Install the SQLx CLI with PostgreSQL and Rustls support:

```sh
cargo install --version='~0.8' sqlx-cli --no-default-features --features rustls,postgres
```

Clone the repository or create a project from the template, then initialize PostgreSQL and run the migrations from the project root:

```sh
./scripts/init_db.sh
```

On Windows PowerShell, use:

```powershell
./scripts/init_db.ps1
```

The setup script starts a temporary PostgreSQL container, creates the application database, and runs the migrations. Set `SKIP_DOCKER=1` when PostgreSQL is already running. The database setup values can be customized with `DB_PORT`, `SUPERUSER`, `SUPERUSER_PWD`, `APP_USER`, `APP_USER_PWD`, and `APP_DB_NAME`.

Start the application:

```sh
cargo run --locked
```

In another terminal, try the health endpoint:

```sh
curl -i http://127.0.0.1:8000/api/v1/health_check
```

In Windows PowerShell, use `curl.exe` instead of `curl` if `curl` is an alias for `Invoke-WebRequest`.

You should see `HTTP/1.1 200 OK` with an empty body. This is a simple liveness response; it does not query the database. Press Ctrl+C in the server terminal to stop it.

## Configuration

Configuration is loaded in this order, with later sources overriding earlier ones:

1. `configuration/base.yaml`
2. `configuration/<environment>.yaml`
3. Environment variables prefixed with `APP_`, using `__` to separate nested keys

Set `APP_ENVIRONMENT` to `local` (the default) or `production`. These values are case-insensitive.

| Setting | Environment variable | Local default | Production default |
| --- | --- | --- | --- |
| Listen host | `APP_APPLICATION__HOST` | `127.0.0.1` | `0.0.0.0` |
| Listen port | `APP_APPLICATION__PORT` | `8000` | `8000` |
| Application base URL | `APP_APPLICATION__BASE_URL` | `http://127.0.0.1` | **Required; not supplied** |
| Database host | `APP_DATABASE__HOST` | `127.0.0.1` | `127.0.0.1` |
| Database port | `APP_DATABASE__PORT` | `5432` | `5432` |
| Database username | `APP_DATABASE__USERNAME` | `postgres` | `postgres` |
| Database password | `APP_DATABASE__PASSWORD` | `password` | `password` |
| Database name | `APP_DATABASE__DATABASE_NAME` | `default` | `default` |
| Require SSL | `APP_DATABASE__REQUIRE_SSL` | `false` | `true` |

For example, to run locally on a different port:

```sh
APP_APPLICATION__PORT=8080 cargo run --locked
```

In PowerShell, set the variable before running Cargo instead:

```powershell
$env:APP_APPLICATION__PORT = "8080"
cargo run --locked
```

> **Production:** `base_url` is required, but the supplied production configuration does not set it. Provide it through `APP_APPLICATION__BASE_URL` or add it to the YAML before starting the server. Production listens on all IPv4 interfaces by default and requires SSL for the database connection.

For example, in a POSIX shell:

```sh
APP_ENVIRONMENT=production \
APP_APPLICATION__BASE_URL=https://api.example.com \
cargo run --locked --release
```

The base URL is made available to handlers as application data; it does not set the listening address or enable HTTPS. The server currently serves plain HTTP, so terminate TLS at a reverse proxy or add TLS support before exposing it over HTTPS. Set environment variables in your shell or deployment environment; the application does not load `.env` files.

Configuration files are resolved relative to the **current working directory**, not the executable. Run from the project root, or deploy the `configuration/` directory beneath the server's working directory.

## Database and migrations

Migration files live in `migrations/`. To create a migration:

```sh
sqlx migrate add describe_the_change
```

Set `DATABASE_URL` before running SQLx commands directly:

```sh
export DATABASE_URL=postgres://postgres:password@localhost:5432/default
sqlx migrate run
```

The integration tests create a unique test database for each application instance, run the migrations, and require a reachable PostgreSQL server using the local configuration defaults.

## API routes

| Method | Path | Response |
| --- | --- | --- |
| `GET` | `/api/v1/health_check` | `200 OK`, empty body |

Routes are registered in the `/api/v1` scope in `src/startup.rs`. Unversioned paths and versions that are not registered return `404 Not Found`.

### Add an endpoint

1. Add a handler under `src/routes/`, declare and re-export its module in `src/routes.rs`.
2. Import the handler in `src/startup.rs` and register it in the `/api/v1` scope. For example, for a handler named `hello`:

   ```rust
   .service(
       web::scope("/api/v1")
           .route("/health_check", web::get().to(health_check))
           .route("/hello", web::get().to(hello)),
   )
   ```

3. Add an integration test under `tests/api/` and declare its module in `tests/api/main.rs`. Use `spawn_app()` from `tests/api/helpers.rs` to start a test server with an isolated database.

When turning the template into your own service, rename the package and binary in `Cargo.toml` and update references to the `actix_web_sqlx_starter` library in the source and tests.

## Development

```sh
cargo test --locked                       # run all tests; PostgreSQL must be running
cargo test --locked --test api            # run API integration tests
cargo fmt --all -- --check                # check formatting
cargo clippy --locked --all-targets       # run Clippy
cargo build --locked --release            # build a release binary
```

The integration tests start real HTTP servers on OS-assigned ports and send requests using Reqwest. They read the normal configuration before overriding the port and database name, so PostgreSQL must be available with valid settings.

The application writes structured Bunyan-style JSON logs to stdout. Set `RUST_LOG` to change the log filter (the default is `info`). Test logs are suppressed by default; set `TEST_LOG` to any value and disable output capture to see them:

```sh
TEST_LOG=1 cargo test --locked -- --nocapture
```

In PowerShell:

```powershell
$env:TEST_LOG = "1"
cargo test --locked -- --nocapture
```

## Project layout

```text
configuration/          Base, local, and production YAML settings
migrations/             SQLx database migrations
scripts/init_db.sh      PostgreSQL setup and migration script
scripts/init_db.ps1     PostgreSQL setup and migration script for PowerShell
src/bin/main.rs         Application entry point
src/configuration.rs    Application and database configuration
src/startup.rs          Server setup, pool, middleware, and versioned routes
src/routes/             HTTP handlers
src/telemetry.rs        Structured logging and tracing setup
tests/api/              Integration tests and test-server helper
```

## License

This project is licensed under the [MIT License](License.txt).
