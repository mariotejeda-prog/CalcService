---
name: cortex-navigator
description: Ground endpoint, cross-service, and cross-repo facts in the Cortex MCP servers (platform-cortex knowledge graph + cortex-code search) — full tool surface, mandatory contract verification, worked examples.
precedence: 60
scope: architecture
---
## Cortex grounding (MCP)

Two Cortex MCP servers may exist in your tool list — use whichever are present:
- `platform-cortex` — architecture graph derived from code: services, endpoints, contracts, deps, Kafka. Source of truth for cross-service + API-surface facts.
- `cortex-code` (optional) — federated code search. ONLY for repos NOT checked out in workspace; local files → normal read/grep tools.

MUST call platform-cortex BEFORE writing diagnosis/fix when:
1. Bug/task touches an HTTP endpoint (route, request/response shape, persistence behind it) → `get_endpoint_contract` first. Wrong contract assumptions = common root-cause class.
2. About to claim what another service exposes/consumes/depends on → ground it first.
3. Unsure which service/repo owns a behavior → `find_relevant_services`.

Record EVERY Cortex call in the artifact's audit/evidence section next to source reads (`Contract check:` line or `mcp_reads:` entries): tool + args + one-line finding. Unrecorded = unverified.

Parallel delegation: subagent workers may lack MCP tools and never see this contract — the COORDINATOR owns the mandatory checks. Run them yourself (before dispatch, passing findings to workers, or on collected results BEFORE finalizing artifacts) and record the `mcp_reads` evidence in each output artifact. Never assume a worker grounded anything.

platform-cortex tools:
- `find_relevant_services(task_description, max_results=5, filters?)` — ranked discovery. `filters` = exact case-insensitive pre-scoring narrow on `type|domain|owner|tier|database_type|cache_type`. Vague description → unranked full inventory.
- `get_service_context(name, include?)` — orientation. Defaults: manifest, deps, contracts, notes, communication (Kafka publishes/subscribes + HTTP calls/called_by). Extra on request: `"entry_points"`, `"agent_context"`, `"domain_context"`, `"context_pack"` — the service's harvested AI/domain docs; ask when you need intent, not just structure.
- `list_endpoints(service)` — authoritative method+path existence index.
- `get_endpoint_contract(service, method, path)` — params + request/response DTOs + transitively resolved schemas WITH source file paths (jump straight to the right files). Mobile services answer "no API spec" (consumers, not providers).

cortex-code tools (compose: graph first → take `repo_name` → scope search):
- `list_repos()` — valid `repo_name` join keys + doc counts.
- `search_code(query, repo_name?, lang?, max_files=50, context_lines=2)` — zoekt syntax: terms, regex, boolean ops, `sym:Name` (symbol lookup), `file:pattern`. `repo_name` scopes exactly.
- `read_file(repo_name, path, start_line?, end_line?)` — paths come from search_code.

Examples — illustrative ONLY. Always substitute YOUR task's real names; NEVER query these literals:
- `find_relevant_services` `{"task_description": "order total not recalculated after item removal", "filters": {"domain": "orders"}}`
- `get_endpoint_contract` `{"service": "orders-service", "method": "PUT", "path": "/v1/orders/{orderId}"}`
- `get_service_context` `{"name": "orders-service", "include": ["manifest", "communication", "context_pack"]}`
- `search_code` `{"query": "sym:recalculateTotal", "repo_name": "orders-service"}` → `read_file` `{"repo_name": "orders-service", "path": "src/.../OrderService.java", "start_line": 40, "end_line": 90}`

Rules:
- Existence check = `list_endpoints`. Soft "no API spec" from `get_endpoint_contract` proves NOTHING about existence.
- NEVER invent an endpoint, topic, DTO, or dependency. `Service not found` = graph doesn't know it — say so.
- Tools absent this session → proceed; mark cross-service/contract claims unverified.
- Budget: 1–3 graph calls; code-search only for repos not in workspace. Grounding never replaces source reads.
