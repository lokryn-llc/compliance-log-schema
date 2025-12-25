# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2025-12-25

### Added

- **MCP (Model Context Protocol) Events** - First-class support for MCP lifecycle and operations:
  - `EVENT_MCP_INITIALIZE`, `EVENT_MCP_INITIALIZED`, `EVENT_MCP_PING`, `EVENT_MCP_SHUTDOWN`
  - `EVENT_TOOL_LIST`, `EVENT_RESOURCE_LIST`, `EVENT_PROMPT_LIST`, `EVENT_RESOURCE_READ`
  - `EVENT_SAMPLING_REQUEST`, `EVENT_SAMPLING_RESPONSE`
  - `EVENT_TRANSPORT_CONNECT`, `EVENT_TRANSPORT_DISCONNECT`, `EVENT_TRANSPORT_ERROR`

- **GenAI Payload** - Structured payload for LLM operations:
  - Model identification (model, provider)
  - Token usage tracking (input, output, total)
  - Request parameters (temperature, max_tokens, top_p)
  - Response metadata (finish_reason, time_to_first_token_ms)
  - Message and tool call logging

- **MCP Payload** - Structured payload for MCP operations:
  - Protocol version and request tracking
  - Tool invocation with arguments and results
  - Resource operations with URI and MIME type
  - Sampling support (nested GenAI payload)
  - Capability negotiation

- **Pre-translated Export Fields** - Zero-cost export to OCSF and OTel:
  - `ocsf_class_uid` and `ocsf_activity_id` for OCSF export
  - `otel_operation_name` for OTel GenAI semantic conventions

- **Correlation Fields** - Distributed tracing support:
  - `trace_id`, `span_id`, `parent_span_id`
  - `session_id` for conversation tracking

- **Python Mapping Helpers**:
  - `get_mapping()` - Get OCSF/OTel values for an EventType
  - `get_ocsf_values()` - Get class_uid and activity_id
  - `get_otel_operation_name()` - Get OTel GenAI operation name
  - `EVENT_TYPE_MAPPINGS` - Complete mapping dictionary

- **Standard Mappings Documentation**:
  - OCSF v1.3.0 field and event type mappings
  - OTel GenAI v1.37.0 semantic conventions
  - Elastic ECS v8.17 mappings
  - Splunk CIM v6.1 mappings

- **Platform Integration Guides**:
  - AWS Security Lake
  - Datadog
  - Elasticsearch
  - Splunk Enterprise
  - Google Chronicle
  - Microsoft Sentinel

### Changed

- Renamed `LogEntry` to `LogRequest` for clarity
- Reorganized EventType enum values:
  - Traditional events: 1-12
  - AI/Agent events: 20-26
  - MCP events: 30-42
- Updated Sensitivity enum to align with OCSF confidentiality_id (0-4)
- Updated Outcome enum: replaced `OUTCOME_PARTIAL` with `OUTCOME_OTHER` (99)

## [0.1.0] - 2024-12-24

### Added

- Initial release
- Core `LogEntry` message with 11 fields
- EventType enum with traditional audit events (LOGIN, LOGOUT, FILE_ACCESS, etc.)
- AI/Agent event types (TOOL_INVOCATION, MODEL_INFERENCE, AGENT_DECISION, etc.)
- Severity levels following RFC 5424 syslog
- Outcome tracking (SUCCESS, FAILURE_UNAUTHORIZED, FAILURE_DENIED, FAILURE_ERROR)
- Sensitivity classification (PUBLIC, INTERNAL, CONFIDENTIAL, RESTRICTED, HIGHLY_RESTRICTED)
- Policy tags for compliance framework tagging
- PyPI package: `lokryn-compliance-log`
- Buf module: `buf.build/lokryn/compliance-log-schema`
