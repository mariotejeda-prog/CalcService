# SAP ABAP Archaeology — Analysis Standards

> Context pack for the `sap-abap-archaeology` capability. Defines the **source-export-first** posture, the
> read-only / no-secrets rules, the ABAP object vocabulary, and the Clean Core signal definitions the stream
> must follow. Traceable to `./input/PROPOSED-CAPABILITIES.md` (Capability 1 + MCP selection guidance).

## Access Mode — Source-Export-First (default)

This capability defaults to **`access_mode = source-export-only`** (OQ-3): it analyzes **only provided exports**
and requires **no live system, no `vsp`/ADT MCP connection**. This makes it runnable today without resolving
the ADT MCP licensing (OQ-1) or `vsp`-acceptability (OQ-3) questions.

| Mode | Behavior | Gating |
|---|---|---|
| `source-export-only` (default) | Parse provided source export / abapGit + ATC/Unit/where-used/DDIC/transport/TARS files. Optional **vsp OFFLINE** parser/linter only. | None — runnable now |
| `live-adt` | Read-only live access via official SAP ADT MCP or `vsp` connected to the client system | **Gated** on OQ-1 + OQ-3 |
| `hybrid` | Provided exports + targeted live read-only lookups | **Gated** on OQ-1 + OQ-3 |

Default to provided exports. The live branch is an enhancement, not a prerequisite.

## Read-Only / No-Secrets Posture

- Strictly **read-only**: never modify ABAP source, transports, or any system.
- **Never** read, store, or emit passwords, cookies, tokens, or service keys. `adt_connection_profile` references
  a credential source only — it must not contain secrets.
- Prefer source-export-only for archaeology; editing/live access is out of scope for this capability.

## Expected Evidence (per source analysis)

ABAP source export / abapGit repository, package name, transport list, ATC report, ABAP Unit output, where-used
export, DDIC export, optional Globant ABAP TARS export, system/profile descriptor. Optional live ADT connection
profile (live branch only).

- Every analyzed item must cite its evidence source recorded in the evidence manifest.
- Absent evidence is recorded as "not provided" — never invented or inferred.
- TARS is **optional**; its export contract is unconfirmed (OQ-2) — record shape/version and treat unknown fields conservatively.

## ABAP Object Vocabulary

Catalog objects by type: classes/interfaces, function groups/modules, programs/reports, DDIC (tables, structures,
data elements, domains), CDS views, RAP artifacts (behavior definitions/projections), enhancements (BAdIs, user
exits, enhancement spots), and external interfaces (RFC/BAPI, OData, IDoc). Record released/unreleased status and
package where evidenced.

## Clean Core Signal Definitions

Derive these upgrade-risk signals strictly from evidence:

- **Core modification** — changes to SAP standard objects (modifications/repairs).
- **Unreleased-API usage** — use of objects/APIs not released for the relevant extensibility tier.
- **Enhancement points** — BAdIs/exits/enhancement spots in use (key-user vs developer extensibility signal).
- **Side-by-side candidates** — logic decoupled enough to move to BTP/CAP.

Mark uncertainty explicitly; these signals feed `sap-clean-core-analysis` and modernization decisions.

## Relationship to `legacy-insights`

This stream **owns its own documentation output** (`sap_abap_archaeology_package`) and does **not** depend on the
generic `legacy-insights` stream. Per the source proposal's "Playlist Updates", `sap-abap-archaeology` may run
"before or **in place of** `legacy-insights`" — which is why this ABAP playlist is unaffected by the broken
`code-interviewer` tool.

## Skill / Tool Dependency Note

`sap-abap`, `sap-abap-cds`, and `sap-btp-developer-guide` come from the external `secondsky/sap-skills` pack.
The optional `vsp` offline parser comes from `vibing-steampunk`. Confirm availability to the Stepwise runtime
before execution — these are extension prerequisites, not runtime guarantees.
