> **Execution Protocol section file — §6 Dual-Format Input Detection.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 6. Dual-Format Input Detection

All skills support two input formats. Detection is automatic.

| Format | Signal | Handling |
|--------|--------|---------|
| Agent-native | Single `*-SPEC-*.md` in input folder | Parse structured sections directly |
| Legacy | Multiple numbered files (00-XX) in input folder | Read index (00-*), then load files on demand |

Agent-native is preferred. Legacy is for backward compatibility.

---

