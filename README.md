# lokryn-compliance-log-schema

Protocol Buffers schema for compliance-grade audit logging. Built for SOC2, HIPAA, and PCI environments. Works everywhere, optimized for AI/agent systems.

## The Problem

There's no standard for compliance-grade audit logging. Everyone rolls their own. When you need to prove what happened, when, and why—especially with AI agents making autonomous decisions—you're stuck stitching together ad-hoc logs that weren't designed for auditors.

## The Solution

A single, opinionated schema that covers:

- Traditional audit events (login, file access, config changes)
- AI/agent-specific events (tool calls, model inference, autonomous decisions)
- Sensitivity classification for data handling policies
- Outcome tracking for success/failure analysis

One format. Drop-in ready for any logging system. Built for compliance teams to actually use.

---

## Installation

### Buf (Recommended)

Add to your `buf.yaml`:

```yaml
deps:
  - buf.build/lokryn/compliance-log-schema
```

Then run:

```bash
buf mod update
```

### Generate Python Code

```bash
buf generate buf.build/lokryn/compliance-log-schema
```

Or with `buf.gen.yaml`:

```yaml
version: v2
plugins:
  - remote: buf.build/protocolbuffers/python
    out: gen
```

---

## Schema Overview

### EventType

What happened.

| Event | Description |
|-------|-------------|
| `EVENT_LOGIN` | User authentication |
| `EVENT_LOGOUT` | Session termination |
| `EVENT_FILE_ACCESS` | File read/write/delete |
| `EVENT_POLICY_CHANGE` | Security policy modification |
| `EVENT_PRIVILEGE_USE` | Elevated permission usage |
| `EVENT_CONFIG_CHANGE` | System configuration change |
| `EVENT_DATA_EXPORT` | Data leaving the system |
| `EVENT_NETWORK_CONNECTION` | Network activity |
| `EVENT_PROCESS_START` | Process execution |
| `EVENT_PROCESS_STOP` | Process termination |
| `EVENT_USER_MANAGEMENT` | User account changes |
| `EVENT_RESOURCE_ACCESS` | Generic resource access |

#### AI/Agent Events

| Event | Description |
|-------|-------------|
| `EVENT_TOOL_INVOCATION` | MCP tool call, function call, API invocation |
| `EVENT_MODEL_INFERENCE` | LLM API call (completion, embedding, etc.) |
| `EVENT_AGENT_DECISION` | Autonomous decision point (branching, action selection) |
| `EVENT_DELEGATION` | Agent delegating to sub-agent or another system |
| `EVENT_CONTEXT_ACCESS` | RAG retrieval, memory access, context window operations |
| `EVENT_PROMPT_EXECUTION` | Prompt template execution |
| `EVENT_GUARDRAIL_CHECK` | Safety/guardrail evaluation (pass/fail) |

### Severity

How important is this event. Follows [RFC 5424](https://datatracker.ietf.org/doc/html/rfc5424) syslog levels.

| Level | Value | When to Use |
|-------|-------|-------------|
| `SEVERITY_DEBUG` | 1 | Detailed debugging |
| `SEVERITY_INFO` | 2 | Normal operations |
| `SEVERITY_NOTICE` | 3 | Significant but normal |
| `SEVERITY_WARNING` | 4 | Something's off |
| `SEVERITY_ERROR` | 5 | Operation failed |
| `SEVERITY_CRITICAL` | 6 | System component failing |
| `SEVERITY_ALERT` | 7 | Immediate action needed |
| `SEVERITY_EMERGENCY` | 8 | System unusable |

### Outcome

Did it work?

| Outcome | Description |
|---------|-------------|
| `OUTCOME_SUCCESS` | Operation completed |
| `OUTCOME_FAILURE_UNAUTHORIZED` | Authentication failed |
| `OUTCOME_FAILURE_DENIED` | Authorization failed |
| `OUTCOME_FAILURE_ERROR` | System/application error |
| `OUTCOME_PARTIAL` | Partially completed |

### Sensitivity

How sensitive is the data involved?

| Level | Description |
|-------|-------------|
| `SENSITIVITY_PUBLIC` | No restrictions |
| `SENSITIVITY_INTERNAL` | Internal use only |
| `SENSITIVITY_CONFIDENTIAL` | Need-to-know basis |
| `SENSITIVITY_RESTRICTED` | Regulatory/contractual restrictions |
| `SENSITIVITY_HIGHLY_RESTRICTED` | Maximum protection (PII, PHI, PCI) |

---

## LogRequest Message

The core message structure:

```protobuf
message LogRequest {
  EventType event_type   = 1;
  Outcome outcome        = 2;
  Severity severity      = 3;
  string actor_id        = 4;   // Who did it (user, service, agent)
  string component       = 5;   // What system component
  string environment     = 6;   // prod, staging, dev
  string resource        = 7;   // What was accessed/modified
  string message         = 8;   // Human-readable description
  bytes payload          = 9;   // Structured data (JSON/CBOR)
  repeated string policy_tags = 10;  // Compliance tags (PCI, HIPAA, etc.)
  Sensitivity sensitivity = 11;
}
```

---

## Usage Examples

### Python

```python
from lokryn.compliance.v1 import log_pb2

# Log an MCP tool call
log = log_pb2.LogRequest(
    event_type=log_pb2.EVENT_TOOL_INVOCATION,
    outcome=log_pb2.OUTCOME_SUCCESS,
    severity=log_pb2.SEVERITY_INFO,
    actor_id="agent-001",
    component="mcp-client",
    environment="production",
    resource="tools/database_query",
    message="Executed database query tool",
    payload=b'{"query": "SELECT * FROM users", "rows_returned": 42}',
    policy_tags=["SOC2", "data-access"],
    sensitivity=log_pb2.SENSITIVITY_CONFIDENTIAL,
)

# Serialize
data = log.SerializeToString()
```

### Logging a Guardrail Check

```python
log = log_pb2.LogRequest(
    event_type=log_pb2.EVENT_GUARDRAIL_CHECK,
    outcome=log_pb2.OUTCOME_FAILURE_DENIED,
    severity=log_pb2.SEVERITY_WARNING,
    actor_id="agent-001",
    component="safety-filter",
    environment="production",
    resource="guardrails/pii-detection",
    message="PII detected in agent output, blocked",
    payload=b'{"rule": "ssn-pattern", "action": "block"}',
    policy_tags=["PII", "HIPAA"],
    sensitivity=log_pb2.SENSITIVITY_HIGHLY_RESTRICTED,
)
```

### Logging an Agent Decision

```python
log = log_pb2.LogRequest(
    event_type=log_pb2.EVENT_AGENT_DECISION,
    outcome=log_pb2.OUTCOME_SUCCESS,
    severity=log_pb2.SEVERITY_INFO,
    actor_id="agent-001",
    component="decision-engine",
    environment="production",
    resource="workflows/customer-support",
    message="Agent chose to escalate to human",
    payload=b'{"options": ["respond", "escalate", "defer"], "selected": "escalate", "confidence": 0.92}',
    policy_tags=["audit-trail"],
    sensitivity=log_pb2.SENSITIVITY_INTERNAL,
)
```

---

## Why This Schema?

### For Compliance Teams

- Maps directly to SOC2, HIPAA, PCI audit requirements
- Sensitivity levels match data classification policies
- Policy tags let you filter by compliance framework
- Outcome tracking shows attempted vs. successful actions

### For AI/Agent Systems

- First-class support for tool calls, model inference, and autonomous decisions
- Tracks delegation chains (agent → sub-agent → tool)
- Guardrail events create audit trail for safety checks
- Context access events show what information agents used

### For Developers

- Single schema, multiple languages (Python, Go, TypeScript, etc.)
- Protobuf means type safety and compact serialization
- Drop-in ready for any logging backend
- Works with Field Notes or your own system

---

## Ecosystem

This schema is the foundation for:

- **[lokryn-mcp](https://github.com/lokryn-llc/mcp-log)** - MCP client wrapper that auto-generates compliant logs
- **[Field Notes](https://docs.lokryn.com/docs/field-notes)** - Tamper-evident audit logging with sub-second queries

---

## License

**AGPL-3.0** - Free to use, modify, and distribute. If you run a modified version as a network service, you must open source your changes.

Full text: https://www.gnu.org/licenses/agpl-3.0.txt

**Commercial License** - Need to use this without AGPL obligations? Contact us at [license@lokryn.com](mailto:license@lokryn.com).

---

## Contributing

Issues and PRs welcome. For significant changes, open an issue first to discuss.

---

## About Lokryn

Blue collar data tools for SMBs. Compliance-ready. No bloat. Fair pricing.

[lokryn.com](https://lokryn.com)
