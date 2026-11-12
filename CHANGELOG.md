# Changelog

All notable changes to the Actix Web SQLx Starter project will be documented in this file.

## [0.3.0] - 2026-11-12

### General cleanup and dependency update

- Align the common application structure with `actix-web-starter`.
- Version the health endpoint under `/api/v1` and test unsupported API paths.
- Refresh the README with setup, configuration, migration, and development guidance.
- Normalize dependency version requirements and disable unnecessary Actix Web default features.
- Correct the repository URL and application log name.

## [0.2.1] - 2026-09-20

### Fixed

- **Route/test mismatch**
  - register the health check at `/api/health_check` to match the integration test
    (the v0.2.0 tidy updated the test but not the route)

## [0.2.0] - 2026-02-22

### Finish Zero2Prod style template

- **Split into bin/lib Format**
  - create modules for configuration, telemetry, startup
  - add integration tests

## [0.1.0] - 2026-02-22

### Initial Commit

- **Scaffold Based on Zero2Prod**
  - use intro example from Actix Web docs as starter
  - add `/health_check` route
