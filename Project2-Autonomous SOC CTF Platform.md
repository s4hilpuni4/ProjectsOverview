# Autonomous SOC CTF Platform

## Technical Documentation, System Architecture, Security Controls, and Implementation Plan

> **Project type:** Cybersecurity training and defensive
> incident-response simulation\
> **Document status:** Proposed technical design based on the project
> specification\
> **Primary objective:** Provide an isolated, repeatable environment in
> which AI-driven adversary simulations generate security telemetry for
> human blue-team participants to detect, investigate, and respond to.

------------------------------------------------------------------------

## Table of Contents

1.  [Project Overview](#1-project-overview)
2.  [Objectives and Scope](#2-objectives-and-scope)
3.  [Core System Components](#3-core-system-components)
4.  [Recommended Technology Stack](#4-recommended-technology-stack)
5.  [High-Level Architecture](#5-high-level-architecture)
6.  [Adversary Engine](#6-adversary-engine)
7.  [Target Sandbox and Isolation](#7-target-sandbox-and-isolation)
8.  [Telemetry Pipeline](#8-telemetry-pipeline)
9.  [SOC and SIEM Layer](#9-soc-and-siem-layer)
10. [Game Engine and Scoring](#10-game-engine-and-scoring)
11. [Data Model](#11-data-model)
12. [API Design](#12-api-design)
13. [Infrastructure as Code and Lifecycle
    Automation](#13-infrastructure-as-code-and-lifecycle-automation)
14. [Security Architecture and
    Guardrails](#14-security-architecture-and-guardrails)
15. [Observability and Reliability](#15-observability-and-reliability)
16. [Testing and Validation](#16-testing-and-validation)
17. [Implementation Roadmap](#17-implementation-roadmap)
18. [Team and Resources](#18-team-and-resources)
19. [Risks and Mitigations](#19-risks-and-mitigations)
20. [Open Decisions](#20-open-decisions)
21. [Definition of Done](#21-definition-of-done)
22. [References](#22-references)

------------------------------------------------------------------------

## 1. Project Overview

The Autonomous Security Operations Center (SOC) Capture The Flag (CTF)
Platform is a cybersecurity training environment that simulates
multi-stage adversary activity against isolated virtual targets. Human
participants act as blue-team analysts. They use security monitoring and
SIEM tools to identify suspicious behavior, investigate alerts, contain
incidents, collect flags, and submit incident reports.

The platform combines:

-   An AI-assisted adversary engine that selects from approved
    simulation actions.
-   Isolated, temporary target environments that host deliberately
    vulnerable systems or predefined scenarios.
-   Host and network telemetry collection.
-   A SOC/SIEM environment for event search, detection, triage, and
    investigation.
-   A game engine that manages challenge state, flag validation, timers,
    and scoring.
-   Infrastructure automation to create, reset, and destroy environments
    consistently.

**Safety boundary:** This platform is intended for authorized training
inside explicitly controlled environments. Offensive tools must be
restricted to dedicated lab targets. The architecture must not permit
the agent to scan or attack arbitrary public or corporate systems.

This document expands the supplied four-page project specification into
a repository-ready technical plan. The original specification defines
the principal components and technology directions; additional data
models, API examples, testing criteria, and implementation details below
are proposed engineering recommendations and must be validated before
development.

## 2. Objectives and Scope

### 2.1 Strategic objectives from the specification

-   **Operational skill validation:** Measure defensive incident
    response against adaptive scenarios rather than only static puzzles.
-   **Autonomous threat generation:** Use an LLM-driven decision process
    to choose among approved scenario actions based on observed target
    telemetry.
-   **Objective scoring:** Correlate flag capture, alert accuracy, Mean
    Time to Detect (MTTD), and incident report completeness.

### 2.2 Initial MVP scope

The first release should demonstrate one complete, safe training
scenario:

1.  Create a competition or practice session.
2.  Provision a dedicated, isolated target environment.
3.  Start a bounded adversary simulation using an allowlisted scenario.
4.  Generate host and/or network telemetry.
5.  Ingest events into the SOC/SIEM environment.
6.  Allow a participant to investigate alerts and submit findings.
7.  Validate a challenge flag.
8.  Calculate a transparent score.
9.  Reset and destroy the environment.

### 2.3 Explicit boundaries

The platform should not execute unrestricted autonomous attacks. Every
action must be restricted to a dedicated CTF target range, validated by
deterministic controls, logged, and subject to a kill switch. Scenarios
should be designed for training and should not provide a route to
external systems.

## 3. Core System Components

The supplied specification identifies the following main subsystems.

  -----------------------------------------------------------------------
  Component               Technologies named in   Main responsibility
                          the specification       
  ----------------------- ----------------------- -----------------------
  Adversary engine        Custom ReAct agent,     Iterative scenario
                          Metasploit RPC, Nuclei, decision-making and
                          LangChain               approved
                                                  security-module
                                                  orchestration

  Target sandbox          Docker, Firecracker     Temporary targets
                          MicroVMs, Linux/Windows containing deliberately
                          guest OS                vulnerable
                                                  configurations and
                                                  workloads

  Telemetry pipeline      Sysmon, Auditd, eBPF    Collect host, process,
                          probes, Zeek, Vector,   file-integrity, and
                          Fluent Bit              network metadata

  SOC/SIEM layer          OpenSearch, Wazuh,      Event ingestion,
                          Elastic Stack           searches, dashboards,
                                                  and detection
                                                  correlation

  Game engine             FastAPI, Redis,         Flag validation, health
                          PostgreSQL              checks, challenge
                                                  state, timers, and
                                                  scoring

  Infrastructure          Terraform, cloud-init   Repeatable provisioning
  automation                                      and lifecycle
                                                  management

  Network containment     VXLAN/Geneve,           Isolated networking and
                          Calico/Cilium policies  restricted connectivity
  -----------------------------------------------------------------------

### Component responsibilities

-   **Adversary engine:** Selects from a constrained set of actions and
    records the reason, inputs, result, and scenario state for each
    decision.
-   **Target sandbox:** Hosts intentionally vulnerable lab machines or
    scenario services and is disposable between rounds.
-   **Telemetry pipeline:** Transports structured events to a separate
    ingestion path so security monitoring is not dependent on the
    target's availability.
-   **SOC/SIEM layer:** Gives participants searchable events,
    dashboards, and detection alerts.
-   **Game engine:** Is the authoritative source for challenge state,
    deadlines, flag validity, and scoring.
-   **Infrastructure automation:** Creates environments from reviewed
    templates and removes them after use or when a safety limit is
    reached.

## 4. Recommended Technology Stack

The technologies below are based on the supplied specification. The
exact versions, deployment modes, and hosting choices need a proof of
concept.

  -----------------------------------------------------------------------
  Layer                   Technology              Purpose
  ----------------------- ----------------------- -----------------------
  Agent orchestration     Python, custom          Select approved
                          ReAct-style controller, scenario actions and
                          LangChain if useful     interpret bounded
                                                  telemetry

  Security simulation     Metasploit RPC, Nuclei, Execute pre-approved
                          vetted scenario modules lab exercises only

  Game API                FastAPI                 REST endpoints and
                                                  challenge orchestration

  Game state              PostgreSQL              Persistent users,
                                                  sessions, challenges,
                                                  events, and scores

  Short-lived state /     Redis                   Timers, ephemeral
  queues                                          state, and queue
                                                  coordination where
                                                  needed

  Lab targets             Linux/Windows guests;   Run challenge services
                          Docker for suitable     
                          workloads               

  Strong isolation        Firecracker MicroVMs    Separate guest kernels
                          where supported         and reduce
                                                  shared-kernel risk

  Host telemetry          Sysmon on Windows;      Record relevant host
                          Auditd/eBPF on Linux    activity

  Network telemetry       Zeek                    Protocol metadata such
                                                  as DNS, HTTP, and TLS
                                                  observations

  Log forwarding          Vector or Fluent Bit    Collect and forward
                                                  structured events

  SIEM                    Wazuh + OpenSearch, as  Endpoint monitoring,
                          recommended by the      indexing, search, and
                          specification           dashboards

  Networking              VPC, isolated virtual   Restrict traffic to the
                          networks, firewall      approved lab range
                          rules, Calico/Cilium    
                          where applicable        

  Infrastructure as code  Terraform               Repeatable environment
                                                  provisioning

  Guest initialization    cloud-init or           Apply deterministic
                          platform-specific       target configuration
                          initialization          

  Containers              Docker                  Package supporting
                                                  services and suitable
                                                  workloads

  Monitoring              Cloud/provider metrics  Monitor platform health
                          plus SIEM health checks and telemetry
                                                  completeness

  CI/CD                   GitHub Actions or       Automated checks and
                          equivalent              controlled releases
  -----------------------------------------------------------------------

### Important implementation note

Firecracker requires a compatible host environment and operational
setup. Validate it with a small proof of concept before designing around
it. Docker containers alone are not equivalent to a separate-kernel
MicroVM security boundary. Do not claim strong isolation until the
actual deployment configuration has been tested.

## 5. High-Level Architecture

``` mermaid
flowchart TD
    A[Participant / Blue-Team Analyst] --> B[Training UI / SOC Dashboard]
    B --> C[Game API - FastAPI]
    C --> D[(PostgreSQL)]
    C --> E[Redis - Timers / Ephemeral State]
    C --> F[Session and Scoring Service]

    F --> G[Scenario Controller]
    G --> H[AI Decision Layer]
    H --> I[Policy and Target Validation Proxy]
    I --> J[Approved Simulation Modules]

    J --> K[Isolated Target Sandbox]
    K --> L[Host Telemetry]
    K --> M[Network Telemetry - Zeek]
    L --> N[Vector / Fluent Bit]
    M --> N
    N --> O[Isolated Ingestion Buffer]
    O --> P[Wazuh / OpenSearch SIEM]
    P --> B

    F --> Q[Flag Validation and Score]
    Q --> D
    R[Lifecycle Scheduler] --> S[Provision / Reset / Destroy]
    S --> K
```

### Architecture principles

1.  **Separate control and data planes:** Game/session control should
    not depend on a participant's ability to query SIEM data.
2.  **Constrain adversary actions:** The AI proposes an action, but a
    deterministic policy layer decides whether it is permitted.
3.  **Keep targets disposable:** Each session uses known templates and
    can be reset or destroyed.
4.  **Protect telemetry independently:** Logs should be forwarded to a
    separate ingestion path and should not be lost merely because a
    target crashes.
5.  **Make scoring auditable:** Every score change should be traceable
    to a recorded event and scoring rule.
6.  **Default to deny:** Network access is denied unless explicitly
    required for the scenario.

## 6. Adversary Engine

The supplied specification describes an iterative ReAct-style agent. In
this design, the model may help choose among approved scenario actions,
but it does not receive unrestricted shell or network access.

### Conceptual decision loop

1.  Read the current scenario state.
2.  Receive sanitized observations from the authorized target range.
3.  Produce a structured proposal for the next approved action.
4.  Validate the proposal against a deterministic policy and target
    whitelist.
5.  Reject the action if the schema, scenario state, destination, or
    time budget is invalid.
6.  Execute only an approved action through a controlled adapter.
7.  Record the result and relevant telemetry correlation identifiers.
8.  Update the scenario state and repeat until completion or a stopping
    condition.

### Action contract

Every proposed action should include fields such as:

-   `scenario_id`
-   `session_id`
-   `action_type` from a fixed allowlist
-   `target_id` from the session's registered target inventory
-   `parameters` validated against a per-action schema
-   `reason_code` or short rationale for auditability
-   `timeout_seconds`
-   `idempotency_key`

The execution layer must reject arbitrary target IPs, arbitrary command
strings, unknown tools, unapproved payloads, and out-of-range
parameters. A prompt instruction alone is not a security boundary.

### Stopping conditions

-   Scenario time limit reached.
-   Action budget exhausted.
-   Target health check fails repeatedly.
-   An action violates policy.
-   A participant or authorized operator stops the round.
-   Telemetry ingestion falls below a safety threshold.
-   The environment appears to be reaching a host or external network
    boundary.

### Safety note

Use only lab-scoped, preconfigured scenarios and security modules. This
document intentionally does not provide exploit commands, payload
recipes, or instructions for attacking real systems.

## 7. Target Sandbox and Isolation

The target sandbox hosts the vulnerable services and workloads used by a
challenge.

### Target lifecycle

1.  Select a reviewed target template.
2.  Create a new session-specific sandbox.
3.  Apply network policies and outbound restrictions before starting the
    workload.
4.  Initialize target services and telemetry agents.
5.  Verify health and isolation.
6.  Run the exercise.
7.  Collect final telemetry and score-relevant events.
8.  Destroy or reset the environment.
9.  Confirm temporary credentials, disks, and state have been removed or
    sanitized.

### Isolation requirements

-   Each session must have a unique target inventory and network
    boundary.
-   Deny external WAN egress by default.
-   Allow only required traffic between the scenario controller, target,
    telemetry forwarders, and SIEM ingestion endpoints.
-   Do not expose target management interfaces to participants or public
    networks.
-   Restrict access to host control sockets and privileged APIs.
-   Apply CPU, memory, disk, process, and time limits.
-   Use MicroVMs for scenarios that need a separate guest-kernel
    boundary.
-   Keep secrets and cloud credentials out of target guests.
-   Destroy resources when sessions expire or a safety stop occurs.

### Docker versus MicroVMs

Docker is useful for packaging services and running many low-risk
workloads, but standard containers share the host kernel. Firecracker
MicroVMs provide a different isolation boundary and are the preferred
direction in the supplied specification for kernel-exploitation
scenarios. The team must test boot time, resource use, compatibility,
and operational recovery before committing.

## 8. Telemetry Pipeline

Telemetry is the evidence defenders use to identify what happened during
an exercise.

### Host telemetry

-   **Windows:** Sysmon events such as process creation, network
    connections, and relevant file activity.
-   **Linux:** Auditd and/or carefully selected eBPF probes for process
    execution, socket activity, file writes, and privilege changes.

### Network telemetry

-   **Zeek:** Protocol metadata and logs for supported traffic,
    including DNS, HTTP, and TLS metadata.
-   Network visibility should be limited to the isolated CTF range and
    should avoid collecting unrelated traffic.

### Forwarding and ingestion

The specification names Vector and Fluent Bit as lightweight forwarders.
They can collect, transform, and ship structured JSON events into an
isolated ingestion buffer before events are indexed in the SIEM.

### Telemetry pipeline requirements

-   Use a documented event schema and timestamp standard.
-   Attach session, scenario, and target identifiers where possible.
-   Track dropped events, queue depth, ingestion delay, and parsing
    failures.
-   Apply buffering and bounded backpressure behavior.
-   Protect ingestion endpoints with authentication and network
    controls.
-   Prevent untrusted target workloads from directly modifying stored
    SIEM events.
-   Define retention and access policies for telemetry.
-   Standardize fields across Sysmon, Auditd/eBPF, and Zeek inputs.

The supplied specification proposes Elastic Common Schema (ECS)
mappings. Confirm how the selected Wazuh/OpenSearch deployment will
represent and query the normalized fields, and document any mappings or
transformations.

## 9. SOC and SIEM Layer

The specification recommends Wazuh with OpenSearch as the preferred
path, with Elastic Stack as an alternative.

### Responsibilities

-   Ingest and index security events.
-   Display host and network telemetry.
-   Provide searches and filtering.
-   Surface alerts and detection rules.
-   Allow analysts to investigate a timeline.
-   Preserve relevant evidence for scoring and incident reports.

### Defensive workflow

1.  Participant opens the assigned SIEM workspace.
2.  Participant reviews new alerts and searches relevant telemetry.
3.  Participant identifies suspicious behavior and builds an incident
    timeline.
4.  Participant records findings and submits an alert or incident
    report.
5.  Participant applies allowed containment or mitigation actions if the
    scenario supports them.
6.  The game engine evaluates the submission against the scenario's
    scoring rubric.

### Per-participant separation

Where each participant receives a separate SIEM workspace, enforce
access boundaries between workspaces. The team should evaluate whether
logical separation is sufficient or whether dedicated instances are
required for particular competitions. Consider resource usage and
operational complexity when choosing.

## 10. Game Engine and Scoring

The specification identifies FastAPI, Redis, and PostgreSQL for the game
engine. PostgreSQL should remain the authoritative persistent record;
Redis can be used for timers and ephemeral coordination.

### Responsibilities

-   Create and manage challenge sessions.
-   Track scenario state and round deadlines.
-   Validate flags.
-   Record submissions and participant actions.
-   Run health checks.
-   Calculate scores.
-   Produce participant and competition summaries.
-   Trigger reset and cleanup workflows.

### Proposed scoring dimensions

The specification explicitly names flag capture, detection-alert
accuracy, MTTD, and incident-report completeness.

  -----------------------------------------------------------------------
  Dimension                           Proposed evaluation
  ----------------------------------- -----------------------------------
  Flag capture                        Valid flag submitted for the
                                      correct session and challenge

  Alert accuracy                      Correctness of submitted alert
                                      classification against the rubric

  MTTD                                Time from the scenario's defined
                                      detection start to valid detection

  Report completeness                 Presence and correctness of
                                      required timeline, evidence,
                                      impact, and remediation fields
  -----------------------------------------------------------------------

The exact weights and formulas are not specified in the source and must
be agreed with the product owner. Publish the rubric to participants
where appropriate and make score calculations reproducible.

### Flag validation

-   Generate flags with sufficient randomness.
-   Associate each flag with a challenge and session.
-   Store a secure verifier or hash where practical instead of exposing
    expected values.
-   Rate-limit submissions.
-   Make successful submission idempotent.
-   Log attempts without logging sensitive secrets unnecessarily.
-   Never accept a flag solely because the client reports success.

## 11. Data Model

The following schema is a proposed starting point, not a schema defined
in the original specification.

  -----------------------------------------------------------------------
  Entity                  Example fields          Purpose
  ----------------------- ----------------------- -----------------------
  `users`                 `id`, `display_name`,   Participant identity
                          `created_at`            reference

  `competitions`          `id`, `name`,           Competition lifecycle
                          `starts_at`, `ends_at`, 
                          `status`                

  `scenarios`             `id`, `version`,        Reviewed scenario
                          `difficulty`,           definitions
                          `template_id`, `status` 

  `sessions`              `id`, `competition_id`, Participant exercise
                          `scenario_id`,          instance
                          `user_id`, `status`,    
                          `started_at`,           
                          `ended_at`              

  `targets`               `id`, `session_id`,     Session-scoped target
                          `template_id`,          inventory
                          `health_status`         

  `telemetry_refs`        `id`, `session_id`,     References to SIEM
                          `source`, `event_time`, evidence
                          `correlation_id`        

  `flags`                 `id`, `session_id`,     Flag validation state
                          `flag_hash`, `points`,  
                          `claimed_at`            

  `submissions`           `id`, `session_id`,     Flags, alerts, and
                          `user_id`,              incident reports
                          `submission_type`,      
                          `payload`, `created_at` 

  `scores`                `id`, `session_id`,     Auditable score records
                          `user_id`, `score`,     
                          `calculated_at`,        
                          `rubric_version`        

  `audit_logs`            `id`, `actor_id`,       Administrative and
                          `action`,               security actions
                          `resource_id`,          
                          `created_at`            
  -----------------------------------------------------------------------

### Data design rules

-   Use stable identifiers and foreign-key constraints.
-   Store timestamps consistently in UTC.
-   Index session, scenario, user, and time fields used by common
    queries.
-   Use transactions for related state and scoring updates.
-   Version scenarios and scoring rubrics so old scores remain
    explainable.
-   Avoid placing raw high-volume telemetry in the transactional
    database; store it in the SIEM or an appropriate event store and
    retain references.
-   Define retention and deletion policies for accounts, submissions,
    and logs.

## 12. API Design

The following API endpoints are illustrative and should be formalized in
an OpenAPI document.

  ----------------------------------------------------------------------------------------
  Method                  Endpoint                                 Purpose
  ----------------------- ---------------------------------------- -----------------------
  `POST`                  `/api/v1/sessions`                       Create a practice or
                                                                   competition session

  `GET`                   `/api/v1/sessions/{sessionId}`           Retrieve authorized
                                                                   session state

  `POST`                  `/api/v1/sessions/{sessionId}/start`     Start a prepared
                                                                   scenario

  `POST`                  `/api/v1/sessions/{sessionId}/stop`      Stop a session or
                                                                   request cleanup

  `GET`                   `/api/v1/sessions/{sessionId}/events`    Retrieve permitted
                                                                   event references or
                                                                   summaries

  `POST`                  `/api/v1/sessions/{sessionId}/flags`     Submit a flag for
                                                                   validation

  `POST`                  `/api/v1/sessions/{sessionId}/reports`   Submit an incident
                                                                   report

  `GET`                   `/api/v1/sessions/{sessionId}/score`     Retrieve score and
                                                                   rubric breakdown

  `GET`                   `/health/live`                           Liveness check

  `GET`                   `/health/ready`                          Readiness check
  ----------------------------------------------------------------------------------------

### API requirements

Each endpoint should define request/response schemas, authentication,
authorization, session ownership, rate limits, idempotency behavior,
error codes, and audit requirements. Never expose sandbox management
credentials or unrestricted target control through participant-facing
endpoints.

## 13. Infrastructure as Code and Lifecycle Automation

The source specification names Terraform and cloud-init for declarative
provisioning and initialization.

### Provisioning flow

1.  Validate the requested scenario against an approved catalog.
2.  Allocate a session and resource budget.
3.  Apply network containment before starting targets.
4.  Provision target MicroVMs or other approved workloads.
5.  Initialize the guest OS and telemetry agents.
6.  Run health and isolation checks.
7.  Mark the session ready only after all mandatory checks pass.
8.  Track resource ownership and expiration.
9.  Reset or destroy the environment at the end of the session.
10. Verify cleanup and record the result.

### Lifecycle automation requirements

-   Use version-controlled Terraform modules.
-   Store state securely and restrict write access.
-   Separate environments and cloud accounts where appropriate.
-   Apply quotas and resource budgets.
-   Tag resources with session, owner, expiry, and scenario identifiers.
-   Use automatic expiration and cleanup jobs.
-   Make cleanup idempotent so repeated requests are safe.
-   Alert on orphaned or unexpectedly long-running resources.
-   Test recovery after interrupted provisioning.

## 14. Security Architecture and Guardrails

This is the highest-priority area because the project intentionally
emulates adversarial activity.

### Dual-tier containment

**Tier 1 --- Deterministic application policy**

-   Maintain an authoritative allowlist of target IDs and
    session-specific addresses.
-   Validate every proposed action against a strict schema.
-   Reject unknown tools, action types, parameters, and destinations.
-   Bind every action to an active session and scenario.
-   Enforce action budgets, timeouts, and rate limits.
-   Record allow/deny decisions.

**Tier 2 --- Infrastructure enforcement**

-   Default-deny network policies and firewall rules.
-   Deny external WAN egress from target sandboxes.
-   Allow only the specific control and telemetry paths needed.
-   Restrict access to host management interfaces and cloud metadata
    services.
-   Separate control-plane credentials from guest workloads.
-   Use MicroVM boundaries for scenarios that require stronger kernel
    isolation.
-   Provide an independent emergency stop to terminate actions and
    sessions.

Prompt engineering alone is not a security boundary. The AI model must
not be able to bypass the policy proxy or change the infrastructure
firewall.

### Target whitelist proxy

The source specification explicitly calls for a proxy interceptor that
validates AI-agent target IP parameters against a dynamic CTF range
whitelist before execution.

The proxy should:

1.  Resolve the requested target ID to the session's registered
    inventory.
2.  Reject arbitrary IPs and any destination outside the authorized
    range.
3.  Validate the action type and parameter schema.
4.  Check session state, permissions, timeout, and action budget.
5.  Log the decision and issue a short-lived, scoped execution request.
6.  Re-check authorization at execution time to avoid stale-state
    issues.

### Operational safety

-   Run exercises only against explicitly authorized lab assets.
-   Do not provide the adversary agent with unrestricted internet
    access.
-   Avoid shared privileged credentials.
-   Keep target workloads disposable.
-   Provide a documented stop/kill mechanism.
-   Test isolation from the host and from other participants.
-   Conduct security review before enabling new scenario modules.

## 15. Observability and Reliability

Monitor the training platform, the attack simulation controller,
telemetry pipeline, SIEM, and game engine independently.

  -----------------------------------------------------------------------
  Area                                Signals
  ----------------------------------- -----------------------------------
  Session lifecycle                   Provisioning time, startup
                                      failures, cleanup failures, active
                                      sessions

  Agent control                       Allowed/denied actions, action
                                      duration, policy violations, stop
                                      events

  Target health                       Guest availability, resource
                                      limits, failed health checks

  Telemetry                           Events per source, ingestion delay,
                                      dropped events, parser errors

  SIEM                                Indexing failures, query latency,
                                      storage capacity, alert pipeline
                                      health

  Game engine                         API latency, flag submissions,
                                      scoring failures, session state
                                      errors

  Infrastructure                      CPU, memory, disk, network, budget,
                                      orphaned resources
  -----------------------------------------------------------------------

### Operational practices

-   Use structured logs and correlation IDs across session, target,
    action, and telemetry records.
-   Synchronize clocks and store timestamps in UTC.
-   Alert on egress policy violations, failed cleanup, telemetry loss,
    and agent policy violations.
-   Keep audit logs separate from participant-editable data.
-   Test backup and recovery procedures.
-   Document incident response and platform shutdown steps.
-   Define SLOs and recovery objectives based on the competition's
    requirements.

## 16. Testing and Validation

### Unit tests

Test flag validation, scoring formulas, state transitions, action
schemas, target authorization, timeout handling, and cleanup
idempotency.

### Integration tests

Test the game API with PostgreSQL and Redis, the scenario controller
with mocked simulation modules, telemetry forwarding, SIEM ingestion,
and lifecycle automation.

### End-to-end tests

Run a safe, predefined scenario in a disposable lab and verify:

-   Session provisioning completes.
-   The target is reachable only through approved paths.
-   Approved simulation activity generates expected telemetry.
-   Events arrive in the SIEM with correct session identifiers.
-   The participant can investigate and submit evidence.
-   Flags and reports are scored according to the published rubric.
-   The environment is reset or destroyed.
-   Cleanup and audit records are complete.

### Security and containment tests

-   Attempt to submit an arbitrary target address and verify rejection.
-   Verify external egress is blocked from sandbox networks.
-   Verify one session cannot reach another session's targets or data.
-   Verify participants cannot access privileged control-plane APIs.
-   Verify the kill switch stops further execution.
-   Verify that target crashes do not compromise the host or other
    sessions.
-   Verify quotas prevent unbounded resource creation.

### Performance tests

Measure sandbox cold-start time, concurrent session capacity, telemetry
throughput, indexing lag, dashboard query latency, and cleanup
throughput. Use test targets and approved traffic generators inside the
lab only.

## 17. Implementation Roadmap

The durations below are rough planning estimates and require validation
through a proof of concept.

  -----------------------------------------------------------------------
  Phase                   Indicative duration     Deliverable
  ----------------------- ----------------------- -----------------------
  Requirements and threat 1--2 weeks              Scope, threat model,
  model                                           safe-use boundaries,
                                                  component contracts

  Sandbox proof of        1--3 weeks              Tested target
  concept                                         provisioning and
                                                  containment

  Telemetry and SIEM      2--3 weeks              Host/network telemetry
                                                  visible in SIEM

  Game engine             2--3 weeks              Sessions, flags,
                                                  reports, state
                                                  transitions, scoring

  Constrained agent       2--4 weeks              Policy-gated scenario
  integration                                     controller using
                                                  approved actions

  Lifecycle automation    1--3 weeks              Terraform provisioning,
                                                  reset, expiration,
                                                  cleanup

  End-to-end hardening    1--3 weeks              Security tests,
                                                  operational dashboards,
                                                  recovery runbooks
  -----------------------------------------------------------------------

Some work can happen in parallel. A safe first milestone should be a
deterministic, non-autonomous scenario. Add LLM-assisted action
selection only after containment, telemetry, and cleanup are proven.

### Recommended order of implementation

1.  Define the threat model and authorization boundary.
2.  Build a single disposable target and validate network isolation.
3.  Configure one telemetry source and confirm events arrive in the
    SIEM.
4.  Implement sessions, flags, reports, and deterministic scoring.
5.  Implement a controlled scenario runner with an allowlisted action
    schema.
6.  Add the LLM decision layer behind the policy proxy.
7.  Automate provisioning and cleanup.
8.  Test concurrency, failure handling, and containment.
9.  Run a controlled pilot before increasing scenario complexity.

## 18. Team and Resources

Roles can be combined depending on team size, but each responsibility
needs an owner.

  -----------------------------------------------------------------------
  Role                                Responsibility
  ----------------------------------- -----------------------------------
  Backend/platform engineer           FastAPI, session lifecycle,
                                      scoring, database

  Security engineer                   Threat model, scenario safety,
                                      detection logic, containment
                                      testing

  AI/agent engineer                   Constrained decision loop,
                                      structured output, evaluation

  Cloud/DevOps engineer               Terraform, networking, MicroVM
                                      infrastructure, CI/CD

  Detection/SIEM engineer             Wazuh/OpenSearch, telemetry
                                      parsing, detection rules,
                                      dashboards

  Frontend engineer                   Participant UI, incident workflow,
                                      leaderboard and reports

  QA engineer                         Functional, integration,
                                      containment, performance and
                                      regression testing
  -----------------------------------------------------------------------

### Resources to provision

-   Version-controlled repository and issue tracker.
-   Isolated cloud account or lab environment.
-   Approved target templates and scenario catalog.
-   Terraform state storage and access controls.
-   SIEM and telemetry test environment.
-   AI provider account, with strict usage limits if an external model
    is used.
-   CI runners and image/artifact registry.
-   Monitoring, alerting, and cost controls.
-   Documented emergency-stop and incident-response contacts.

## 19. Risks and Mitigations

  -----------------------------------------------------------------------
  Risk                    Impact                  Mitigation
  ----------------------- ----------------------- -----------------------
  Agent targets an        External damage or      Target whitelist proxy,
  unauthorized host       legal exposure          network egress denial,
                                                  action allowlist

  Sandbox escape          Host or neighboring     MicroVM isolation where
                          session compromise      needed, patching, least
                                                  privilege, containment
                                                  tests

  Telemetry loss          Incomplete defensive    Buffering, health
                          exercise and unfair     metrics, ingestion
                          scoring                 alerts, recovery
                                                  handling

  SIEM resource           Poor visibility and     Resource quotas,
  exhaustion              degraded participant    ingestion limits,
                          experience              retention, load tests

  Inconsistent scoring    Unfair competition      Versioned scoring
                                                  rubric, deterministic
                                                  calculations, audit
                                                  trail

  Orphaned infrastructure Unexpected costs and    TTLs, automated
                          residual attack surface cleanup, ownership
                                                  tags, orphan alerts

  AI hallucination or     Failed or unsafe        Structured outputs,
  invalid action          scenario execution      schema validation,
                                                  policy proxy, safe
                                                  fallback

  Cross-session access    Participant data or     Per-session network and
                          target compromise       identity boundaries,
                                                  authorization tests

  Provisioning failure    Session cannot start or Idempotent workflows,
                          leaves partial          health checks,
                          resources               compensating cleanup

  High operating cost     Unsustainable training  Quotas, concurrency
                          environment             limits, budgets,
                                                  auto-expiration
  -----------------------------------------------------------------------

## 20. Open Decisions

The original specification provides recommended directions but does not
settle every implementation detail. Confirm:

-   Where will the lab run: cloud, on-premises, or dedicated bare-metal
    hosts?
-   Which guest operating systems and target templates are required?
-   What level of isolation is required for each scenario?
-   How many concurrent participants and targets are expected?
-   Will each participant have a separate SIEM instance or a logically
    isolated workspace?
-   Which Wazuh/OpenSearch versions and deployment model will be used?
-   How will ECS-compatible event normalization be implemented and
    validated?
-   Which scenario actions are approved for the first release?
-   What is the scoring formula and how are disputed results reviewed?
-   What are the maximum session duration and resource budget?
-   What data-retention and participant privacy rules apply?
-   Who can stop a running exercise and under what conditions?

## 21. Definition of Done

The first controlled release is ready only when:

-   [ ] The threat model and authorized lab boundary are documented.
-   [ ] A session can provision an approved disposable target.
-   [ ] External WAN egress is blocked or explicitly controlled and
    verified.
-   [ ] Session-to-session isolation tests pass.
-   [ ] At least one host or network telemetry source is collected.
-   [ ] Telemetry is visible in the configured SIEM with session
    identifiers.
-   [ ] The participant can submit a flag and incident report.
-   [ ] Scoring is deterministic, versioned, and auditable.
-   [ ] Every agent action is validated by a deterministic policy layer.
-   [ ] Arbitrary target addresses and unapproved actions are rejected.
-   [ ] The emergency stop works.
-   [ ] Failed provisioning and cleanup paths are tested.
-   [ ] Automated tests and CI checks pass.
-   [ ] Logs, alerts, quotas, and resource expiration are configured.
-   [ ] A session can be reset or destroyed and cleanup verified.
-   [ ] Security review and controlled pilot approval are complete.

------------------------------------------------------------------------

## 22. References

-   [Wazuh documentation](https://documentation.wazuh.com/)
-   [OpenSearch documentation](https://opensearch.org/docs/latest/)
-   [Zeek documentation](https://docs.zeek.org/)
-   [Sysmon
    documentation](https://learn.microsoft.com/sysinternals/downloads/sysmon)
-   [Linux Audit documentation](https://linux.die.net/man/8/auditd)
-   [Firecracker documentation](https://firecracker-microvm.github.io/)
-   [Terraform
    documentation](https://developer.hashicorp.com/terraform/docs)
-   [FastAPI documentation](https://fastapi.tiangolo.com/)
-   [OWASP Top 10 for LLM
    Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/)

## Document Notes

This document is based on the supplied *Project Specification:
Autonomous SOC CTF Platform* (four pages). The specification explicitly
identifies the ReAct-style adversary engine, constrained security-module
execution, Docker/Firecracker targets, Sysmon/Auditd/eBPF/Zeek
telemetry, Vector/Fluent Bit forwarding, Wazuh/OpenSearch as the
preferred SIEM path, FastAPI/Redis/PostgreSQL game engine, network
containment, Terraform/cloud-init, the architecture decision matrix, and
three initial implementation steps: sandbox proof of concept, telemetry
standardization, and target-whitelist guardrails.

The detailed API endpoints, data entities, testing plan, scoring model
structure, roadmap estimates, and acceptance checklist are proposed
implementation guidance. They are not represented as requirements
explicitly specified in the source PDF.
