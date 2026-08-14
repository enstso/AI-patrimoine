# AI Patrimoine

# Target Architecture for the Agentic Platform

## 1. Feedback from the First Version

A first version of the project was built mainly with **n8n**.

This approach made it possible to quickly build a prototype, but several limitations appeared as the workflow became more complex:

* poor visibility into agent behavior;
* lack of detailed decision traceability;
* difficulty understanding errors;
* excessive token consumption;
* repeated calls to AI models;
* complex workflows that were difficult to maintain;
* limited control over the overall state of a case;
* difficulty measuring performance;
* insufficiently centralized security and access management;
* operating costs that were difficult to control.

n8n was used simultaneously for integrations, business logic, and agent orchestration. This combination quickly made the solution difficult to optimize and evolve.

The new architecture must therefore clearly separate responsibilities.

# 2. General Principle of the New Architecture

The platform is based on three main layers:

* **n8n** for integrations and simple automations;
* **LangChain** for building agents that access the information system;
* **LangGraph** for orchestrating the entire agentic process.

These three components are complemented by:

* an observability layer;
* a security layer;
* a business database;
* an access-rights management system;
* a human-validation interface.

The overall logic can be summarized as follows:

```text
Enterprise Applications
Teams, DMS, calendar, assets, finance, accounting
                         │
                         ▼
                  Security Layer
          Authentication, permissions, middleware
                         │
                         ▼
                     LangGraph
              Orchestration and case state
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
         Simple Agents         Business Agents
             n8n                LangChain
              │                     │
              └──────────┬──────────┘
                         ▼
              Enterprise Systems
                         │
                         ▼
                   Observability
          Traces, costs, errors, statistics
```

# 3. Role of n8n

n8n remains part of the architecture, but its role is limited to simple and clearly defined integration tasks.

An agent or workflow can be developed with n8n when its responsibility is limited, low-risk, and mainly deterministic.

Examples:

* retrieve a Teams transcript;
* move a document to a storage space;
* send a notification;
* forward an approval request;
* generate a file from a template;
* create a calendar event;
* send an email;
* trigger a webhook;
* call an external API;
* update a status in a database.

n8n therefore acts as an **automation and integration layer** between the different services.

It should not be solely responsible for:

* the complete memory of a case;
* complex decisions;
* access rules for asset-related data;
* long reasoning processes;
* control loops;
* multi-agent strategies;
* complex business error management.

# 4. Role of LangChain

LangChain is used to build business agents that need to interact with company data and tools.

It can notably be used when an agent needs to:

* query the asset-management system;
* search for information in the DMS;
* analyze multiple documents;
* access financial data;
* compare a transcript with a history of decisions;
* call several tools successively;
* produce a structured response;
* select the appropriate tool for a situation;
* use a vector database;
* apply specific business rules.

Examples of LangChain agents:

* transcript analysis agent;
* renovation/acquisition classification agent;
* asset-analysis agent;
* renovation specialist agent;
* acquisition specialist agent;
* document-analysis agent;
* decision-control agent;
* meeting-minutes preparation agent;
* authorized-person search agent.

LangChain agents must be built as **independent services with strictly defined tools**.

Each agent should only have access to the functions required for its mission.

For example, the document-analysis agent may read documents, but it should not be able to modify asset data or create a financial commitment.

# 5. Role of LangGraph

LangGraph is the **brain and main orchestrator** of the platform.

It controls:

* process steps;
* agent sequencing;
* conditions for moving from one step to another;
* case memory;
* human approvals;
* errors;
* recovery after interruptions;
* renovation and acquisition branches;
* interactions with n8n;
* interactions with LangChain agents.

LangGraph must maintain a centralized state for each case.

Example:

```text
Case
├── Identifier
├── Operation type
├── Related assets
├── Transcripts
├── Documents
├── Detected decisions
├── Validated decisions
├── Actions to be performed
├── Owners
├── Required participants
├── Pending approvals
├── Scheduled meetings
├── Confidence level
├── Current step
└── Event history
```

LangGraph then decides which component should intervene.

Example:

```text
Receipt of transcript
             │
             ▼
Transcript analysis
             │
             ▼
Has a decision been made?
        ┌────┴────┐
        │         │
       No        Yes
        │         │
        ▼         ▼
     Human    Case classification
   validation        │
                ┌────┴─────┐
                ▼          ▼
          Renovation   Acquisition
                │          │
                ▼          ▼
      Renovation agent  Acquisition agent
                │          │
                └────┬─────┘
                     ▼
              Result verification
                     │
                     ▼
          Meeting-minutes generation
                     │
                     ▼
             Availability search
                     │
                     ▼
               User validation
                     │
                     ▼
            Next meeting creation
```

# 6. Rule for Choosing Between n8n and LangChain

The technology choice depends on the responsibility of the agent.

## Use n8n when:

* the task is simple;
* the workflow is deterministic;
* the number of steps is limited;
* the agent does not handle critical data;
* no complex memory is required;
* actions are easy to verify;
* the main objective is to connect applications.

## Use LangChain when:

* the agent must reason across multiple sources;
* the agent must dynamically select tools;
* the agent interacts with the company's asset-management system;
* the agent handles sensitive information;
* the agent must analyze multiple documents;
* the agent must comply with complex business rules;
* structured and controlled output is required;
* permissions must be managed precisely.

## Use LangGraph when:

* several agents must collaborate;
* the process contains multiple branches;
* the case evolves across several meetings;
* execution can be interrupted and resumed;
* human validation is required;
* persistent state must be maintained;
* decisions depend on the results of previous steps;
* the system must handle errors and recovery.

# 7. Observability Layer

Observability is a central component of the new architecture.

It must make it possible to trace every step executed by the system.

For each operation, the platform should be able to record:

* case identifier;
* agent called;
* model used;
* prompt used or prompt version;
* tools called;
* documents consulted;
* processing duration;
* number of tokens consumed;
* estimated cost;
* result produced;
* confidence level;
* errors encountered;
* retry attempts;
* human approvals;
* person who approved the action;
* date and time of each action.

## Example Trace

```json
{
  "trace_id": "TRACE-2026-00482",
  "case_id": "PAT-2026-0017",
  "workflow": "meeting_analysis",
  "agent": "classification_agent",
  "model": "llm_model",
  "start_date": "2026-07-23T10:02:15",
  "duration_ms": 2350,
  "input_tokens": 4120,
  "output_tokens": 620,
  "estimated_cost": 0.034,
  "result": "RENOVATION",
  "confidence": 0.93,
  "status": "SUCCESS"
}
```

# 8. Statistics and Monitoring

The collected traces will make it possible to build dashboards.

The platform will notably be able to measure:

## Technical Performance

* average analysis duration;
* error rate per agent;
* number of retries;
* service availability;
* tool response time;
* workflow failure rate.

## Economic Performance

* average cost per case;
* average cost per meeting;
* token consumption per agent;
* cost per document type;
* cost per model;
* cost of failed processing;
* savings achieved through caching.

## Business Performance

* number of decisions extracted;
* number of decisions validated;
* number of human corrections;
* rate of ambiguous decisions;
* average time between two meetings;
* average validation time;
* number of renovation cases;
* number of acquisition cases;
* number of actions completed on time.

## Agent Quality

* average confidence level;
* correct classification rate;
* false-positive rate;
* false-negative rate;
* number of corrected meeting minutes;
* error frequency by document type;
* most frequently used tools;
* steps most frequently requiring human intervention.

This data can then be used to implement optimization strategies.

# 9. Future Optimization Strategies

Thanks to observability, several optimizations can be implemented.

## Cost Optimization

* use a smaller model for simple tasks;
* reserve advanced models for complex decisions;
* avoid resending entire transcripts;
* summarize older meetings;
* use caching for previously calculated results;
* limit the number of successive calls;
* combine certain analyses;
* stop a workflow when essential information is missing.

## Performance Optimization

* parallelize independent analyses;
* preload required documents;
* index documents in a vector database;
* use structured JSON outputs;
* reduce prompt size;
* select only relevant passages;
* apply programmatic rules before calling a model.

## Quality Optimization

* compare results from multiple agents;
* implement a verification agent;
* validate each decision against an excerpt from the transcript;
* measure human corrections;
* improve prompts based on observed errors;
* create business test datasets;
* version agents and prompts.

# 10. Enhanced Access Management

Access to tools and data must be controlled through a middleware layer.

The middleware intervenes before every call to a tool or service.

It checks in particular:

* user identity;
* agent identity;
* user role;
* agent permissions;
* case type;
* scope of the relevant assets;
* data sensitivity level;
* requested action;
* whether human approval is required.

## Example Control

```text
Acquisition agent
        │
        ▼
Request access to financial data
        │
        ▼
Security middleware
        │
        ├── Is the agent authorized?
        ├── Is the user authorized?
        ├── Is the case within their scope?
        ├── Is the data necessary?
        └── Is validation required?
                │
           ┌────┴────┐
           ▼         ▼
       Authorized   Denied
           │         │
           ▼         ▼
       Tool call    Logging
```

# 11. Principle of Least Privilege

Each agent is granted only the permissions required for its role.

Examples:

| Agent                 | Permissions                                                            |
| --------------------- | ---------------------------------------------------------------------- |
| Transcript Agent      | Read and structure a transcript                                        |
| Document Agent        | Access documents associated with the case                              |
| Renovation Agent      | Access technical asset data                                            |
| Acquisition Agent     | Access authorized acquisition data                                     |
| Calendar Agent        | Read availability and suggest time slots                               |
| Meeting-Minutes Agent | Generate a document without being able to publish it autonomously      |
| Financial Agent       | Access certain financial data without being able to commit expenditure |

An agent should never be given unrestricted access to the entire information system.

# 12. Additional Middleware

Several middleware components can be placed around the agents.

## Authentication Middleware

It verifies the identity of the user and the application.

## Authorization Middleware

It controls roles and permissions.

## Data-Filtering Middleware

It masks or removes data that the agent is not authorized to access.

## Human-Validation Middleware

It blocks sensitive operations until approval is received from an authorized person.

## Limiting Middleware

It limits:

* number of calls;
* token consumption;
* maximum cost per case;
* maximum execution time;
* number of attempts.

## Audit Middleware

It records all sensitive actions.

## Tool-Protection Middleware

It validates parameters before allowing an agent to use a tool.

# 13. Actions Requiring Human Validation

The following actions should not be executed automatically:

* final approval of an acquisition;
* budget approval;
* financial commitment;
* official launch of renovation work;
* modification of critical asset data;
* sending official meeting minutes;
* inviting external participants;
* transmitting a confidential document;
* closing a case;
* deleting information.

The agent may prepare the action, but the **final decision remains with a human**.

# 14. High-Level Technical Architecture

```text
┌──────────────────────────────────────────────┐
│                 Interfaces                   │
│ Business app, portal, Teams, emails          │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│          API Gateway and Middleware          │
│ Authentication, roles, filtering, quotas    │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│                  LangGraph                   │
│ Orchestration, state, branches, validations │
└──────────────┬─────────────────┬─────────────┘
               │                 │
               ▼                 ▼
┌──────────────────────┐ ┌─────────────────────┐
│ LangChain Agents     │ │ n8n Workflows       │
│ Business reasoning   │ │ Simple integrations │
└───────────┬──────────┘ └──────────┬──────────┘
            │                       │
            └───────────┬───────────┘
                        ▼
┌──────────────────────────────────────────────┐
│ Tools and Information Systems                │
│ Assets, DMS, finance, calendars, Teams       │
└──────────────────────┬───────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────┐
│ Observability and Audit                      │
│ Traces, costs, quality, errors, statistics  │
└──────────────────────────────────────────────┘
```

# 15. Strategy Summary

The new architecture is based on a clear separation of responsibilities:

```text
n8n
Automation and integration of simple tasks

LangChain
Business-agent development and controlled access to tools

LangGraph
Orchestration, memory, decision-making and case lifecycle

Middleware
Security, permissions, validation and action control

Observability
Traceability, costs, performance, quality and optimization
```

This architecture addresses the limitations encountered with the initial prototype.

It makes **AI Patrimoine**:

* more maintainable;
* more secure;
* more observable;
* more cost-efficient;
* easier to optimize;
* better suited to complex business processes;
* more reliable when interacting with the company's asset-management environment.
