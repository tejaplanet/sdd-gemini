<!--
Sync Impact Report
- Version change: Initial (1.0.0)
- Modified principles: Inferred from sdd-fm-demo-001 (IOM FM Service standards)
- Added sections: Core Principles (I-V), Technical Constraints, Development Workflow
- Templates requiring updates:
  - .specify/templates/plan-template.md: ✅ updated (checked)
  - .specify/templates/spec-template.md: ✅ updated (checked)
  - .specify/templates/tasks-template.md: ✅ updated (checked)
- Follow-up TODOs: 
  - [ ] Initialize documentation content for the 5 pillars (Architecture, Service Pattern, Config, Payload, Testing) in docs/
-->

# sdd-fm-demo-002 Constitution

## Core Principles

### I. Modular Specification-Driven Development
This project strictly follows a **Modular Specification-Driven Development** approach. The AI agent MUST always refer to the specific documentation files in the `docs/` directory before writing or modifying any code. Documentation pillars (Architecture, Service Pattern, Configuration, Payload, Testing) are the non-negotiable source of truth.

### II. Mandatory Foundational Deliverables
Before generating business logic, foundational files (`go.mod`, `main.go`, `service/service.go`, `service/cons.go`) MUST exist. If missing, they MUST be generated from standard templates. A Service file is NEVER complete without its companion Table-Driven unit test (`[FeatureName]Service_test.go`).

### III. Zero External Library Policy
NO external libraries for HTTP, Logging, or Messaging. Only approved project-specific frameworks (e.g., `request.CallWithETCDConfigTimeoutHttpStatus`, `logger.InitZFlow`) are permitted. This ensures strict adherence to the IOM core framework and simplifies dependency management.

### IV. Standardized SequenceService Anatomy
Every Use Case must follow a fixed signature: `func (s SequenceService) [FeatureName]Service(ctx rabbitmq.Context, info order.Info) (stepStatus string, results []orderflow.Result, payload string, err error)`. Implementation MUST follow the 7-step standard defined in `docs/02_core_service_pattern.md`: Init ZFlow, Panic Handler, Pre-Exec Check, Request/Response Mapping, and Structured Return Sequence.

### V. Payload & Configuration Integrity
All data mapping MUST use the `docs/04_payload_dictionary.md`. Configuration (Endpoints, Timeouts) MUST be managed via ETCD/Redis. Hardcoding values is strictly prohibited; use the `AppConfig` pattern and `service/cons.go` for retrieval.

## Technical Constraints

**Go 1.22.2+**: Maintain compatibility with `gitlab.com/ft25/iom/framework`.  
**Redis & ETCD**: Primary dependencies for state and configuration. All private dependency rules in `go.mod` (using `replace` block) must be strictly followed.

## Development Workflow

**TDD Mandatory**: Unit tests in `tests/` using `testify` and Table-Driven patterns.  
**Semantic Versioning**: Track changes in `version/version.go`.  
**Documentation First**: Specifications (`specs/`) must be updated and approved before implementation begins.

## Governance
This constitution supersedes all informal practices. Amendments require a `Sync Impact Report` and version bump. All pull requests and AI interactions must verify compliance with these principles.

**Version**: 1.0.0 | **Ratified**: 2026-04-02 | **Last Amended**: 2026-04-02
