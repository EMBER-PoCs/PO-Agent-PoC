# Specification Review Report

**Spec Reviewed:** specs/eu-vtp-functional-spec.md  
**Requirements Baseline:** requirements/structured-requirements-eu-vtp.md (93 requirements, REQ-001 – REQ-093)  
**Technical Reference:** requirements/technical-analysis-eu-vtp.md  
**Reviewer:** spec-reviewer (automated)  
**Date:** 2026-03-22  
**Verdict:** APPROVED WITH COMMENTS

---

## Coverage Analysis

All 93 requirements (REQ-001 through REQ-093) are referenced in the Section 3 traceability matrix and have corresponding spec sections with acceptance criteria. Full coverage table below.

| Requirement ID | Covered? | Spec Section | Notes |
|---------------|----------|--------------|-------|
| REQ-001 | ✅ Yes | §4.1.1 | All acceptance criteria present |
| REQ-002 | ✅ Yes | §4.1.2 | All acceptance criteria present |
| REQ-003 | ✅ Yes | §4.1.3 | Spec adds localStorage detail beyond requirement |
| REQ-004 | ✅ Yes | §4.1.4 | All acceptance criteria present |
| REQ-005 | ✅ Yes | §4.1.5 | All acceptance criteria present |
| REQ-006 | ✅ Yes | §4.1.6 | All acceptance criteria present; **also listed in §4.22** (duplicate — see Issue #3) |
| REQ-007 | ✅ Yes | §4.1.7 | All acceptance criteria present |
| REQ-008 | ✅ Yes | §4.1.8 | All acceptance criteria present |
| REQ-009 | ✅ Yes | §4.2.1 | Complete with filter type table and data model |
| REQ-010 | ✅ Yes | §4.2.2 | All acceptance criteria present including layout width |
| REQ-011 | ✅ Yes | §4.2.3 | All acceptance criteria present |
| REQ-012 | ✅ Yes | §4.2.4 | Carline list, responsive breakpoints, and toggle covered |
| REQ-013 | ✅ Yes | §4.2.5 | All acceptance criteria present |
| REQ-014 | ✅ Yes | §4.2.6 | Full list of range filter dimensions included |
| REQ-015 | ✅ Yes | §4.2.7 | All acceptance criteria present |
| REQ-016 | ✅ Yes | §4.2.8 | All acceptance criteria present |
| REQ-017 | ✅ Yes | §4.2.9 | All acceptance criteria present |
| REQ-018 | ✅ Yes | §4.2.10 | `deactivateSelectAllDealers` in business rules but absent from acceptance criteria (see Issue #7) |
| REQ-019 | ✅ Yes | §4.3.1 | All acceptance criteria present |
| REQ-020 | ✅ Yes | §4.3.2 | Includes ES market exception |
| REQ-021 | ✅ Yes | §4.3.3 | All acceptance criteria present |
| REQ-022 | ✅ Yes | §4.4.1 | Full layout table with all sections |
| REQ-023 | ✅ Yes | §4.4.2 | All acceptance criteria present |
| REQ-024 | ✅ Yes | §4.4.3 | All acceptance criteria present including ARIA roles |
| REQ-025 | ✅ Yes | §4.4.4 | Equipment split, tyre labels, video all covered |
| REQ-026 | ✅ Yes | §4.4.5 | 65+ fields, initial/extended views covered |
| REQ-027 | ✅ Yes | §4.4.6 | Template substitution, warranty types, programmatic navigation |
| REQ-028 | ✅ Yes | §4.4.7 | NWS/BEV Agency exception documented |
| REQ-029 | ✅ Yes | §4.4.8 | All acceptance criteria present |
| REQ-030 | ✅ Yes | §4.4.9 | `isCampaignActive()` date range logic covered |
| REQ-031 | ✅ Yes | §4.4.10 | Liquid products, sessionStorage persistence covered |
| REQ-032 | ✅ Yes | §4.4.11 | `<Spawn>` pattern, NC/UC conditions covered |
| REQ-033 | ✅ Yes | §4.4.12 | All acceptance criteria present |
| REQ-034 | ✅ Yes | §4.4.13 | NBA types and content-configurable order covered |
| REQ-035 | ✅ Yes | §4.5.1 | All acceptance criteria present |
| REQ-036 | ✅ Yes | §4.5.2 | Image selection matrix by vehicle type covered |
| REQ-037 | ✅ Yes | §4.5.3 | Responsive sizes, dynamic alt text, fullscreen covered |
| REQ-038 | ✅ Yes | §4.5.4 | All four conditions, z-index management documented |
| REQ-039 | ✅ Yes | §4.6.1 | Full user flow with confirmation overlay logic |
| REQ-040 | ✅ Yes | §4.6.2 | Auth flow, auto-add, toast, toolbar hiding covered |
| REQ-041 | ✅ Yes | §4.6.3 | All acceptance criteria present |
| REQ-042 | ✅ Yes | §4.7.1 | Currency formatting, symbol position, footnotes covered |
| REQ-043 | ✅ Yes | §4.7.2 | Finance toggle, change rate, e-commerce hiding covered |
| REQ-044 | ✅ Yes | §4.7.3 | Business model, rate inclusion, product options covered |
| REQ-045 | ✅ Yes | §4.7.4 | Three finance modes with table and disclaimer toggles |
| REQ-046 | ✅ Yes | §4.7.5 | CRS API data flow, session persistence, caching covered |
| REQ-047 | ✅ Yes | §4.7.6 | ES and JP market templates documented |
| REQ-048 | ✅ Yes | §4.8.1 | WLTP/NEDC, blacklist, PHEV multi-fuel covered |
| REQ-049 | ✅ Yes | §4.8.2 | EnVKV 2024 SVG labels, PLP+PDP scope covered |
| REQ-050 | ✅ Yes | §4.8.3 | French `eecLabel` override covered |
| REQ-051 | ✅ Yes | §4.9.1 | All acceptance criteria present |
| REQ-052 | ✅ Yes | §4.9.2 | Partner Business Card FA, config props, display toggles |
| REQ-053 | ✅ Yes | §4.9.3 | Chain dealer fetch, graceful degradation covered |
| REQ-054 | ✅ Yes | §4.10.1 | CTA types, phone options, fallback button covered |
| REQ-055 | ✅ Yes | §4.10.2 | Sticky bar, portal, popover overflow, mobile layout |
| REQ-056 | ✅ Yes | §4.10.3 | All 22 CTA types catalogued with data model table |
| REQ-057 | ✅ Yes | §4.10.4 | Include/exclude dealer filtering covered |
| REQ-058 | ✅ Yes | §4.10.5 | POST data profiles, URL placeholders, targets covered |
| REQ-059 | ✅ Yes | §4.10.6 | GLC checkout via e-commerce service, payload covered |
| REQ-060 | ✅ Yes | §4.10.7 | Lite reservation conditions covered |
| REQ-061 | ✅ Yes | §4.11.1 | 404 status, page-info-service, i18n messages covered |
| REQ-062 | ✅ Yes | §4.11.2 | 404 redirect and silent hiding for non-404 errors |
| REQ-063 | ✅ Yes | §4.12.1 | Data flow diagram, field mapping, headless pattern |
| REQ-064 | ✅ Yes | §4.12.2 | Pub/sub API table, SSR serialization, singleton |
| REQ-065 | ✅ Yes | §4.13.1 | Comprehensive field table by category |
| REQ-066 | ✅ Yes | §4.13.2 | Comprehensive field table by category |
| REQ-067 | ✅ Yes | §4.13.3 | All shared config categories documented |
| REQ-068 | ✅ Yes | §4.13.4 | Business model × availability matrix, price fragments |
| REQ-069 | ✅ Yes | §5.1.1 | Full SSR pipeline, per-package strategy table, safety patterns (see Issue #1) |
| REQ-070 | ✅ Yes | §5.1.2 | Skeleton component list, display conditions covered |
| REQ-071 | ✅ Yes | §5.1.3 | Optimization technique table covered |
| REQ-072 | ✅ Yes | §5.1.4 | Thresholds documented with severity note (see Issue #2) |
| REQ-073 | ✅ Yes | §5.2 | ARIA, focus, keyboard, heading hierarchy covered |
| REQ-074 | ✅ Yes | §5.3.1 | Full event table with data fields |
| REQ-075 | ✅ Yes | §5.3.2 | Full event table with product tracking data |
| REQ-076 | ✅ Yes | §5.3.3 | Filter tracking events and data fields covered |
| REQ-077 | ✅ Yes | §4.14.1 | Autocomplete, radius, 30-day localStorage covered |
| REQ-078 | ✅ Yes | §4.14.2 | Geolocation, error handling covered |
| REQ-079 | ✅ Yes | §4.14.3 | Two-click consent, persistent/per-session, toggle |
| REQ-080 | ✅ Yes | §4.15.1 | EnVKV component, German fallback text covered |
| REQ-081 | ✅ Yes | §4.15.2 | French eecLabel override covered |
| REQ-082 | ✅ Yes | §4.15.3 | Distance sort exception, price template covered |
| REQ-083 | ✅ Yes | §4.15.4 | Power, mileage, currency, date pattern options |
| REQ-084 | ✅ Yes | §4.15.5 | All four business models with detection and effects |
| REQ-085 | ✅ Yes | §4.16.1 | Four bundles, webpack aliases, dynamic helpers |
| REQ-086 | ✅ Yes | §4.17.1 | Order status ranges and badge types |
| REQ-087 | ✅ Yes | §4.17.2 | Availability computation factors listed |
| REQ-088 | ✅ Yes | §4.18.1 | i18n services, key patterns, locale detection |
| REQ-089 | ✅ Yes | §4.19.1 | Filter button, favorites link, back button, SSR guard |
| REQ-090 | ✅ Yes | §4.20.1 | Four ID types, three URL extraction methods |
| REQ-091 | ✅ Yes | §4.21.1 | NBA button, popover, analytics event |
| REQ-092 | ✅ Yes | §4.22.1 | Share action, clipboard, tracking |
| REQ-093 | ✅ Yes | §4.23.1 | Footnote services, three disclaimer types, toggle |

**Coverage Summary:** 93/93 requirements covered (100%)

---

## Issues Found

### Critical (Must Fix)

_No critical issues identified._

### Major (Should Fix)

**1. REQ-069 — SSR strategy discrepancy between requirement and spec**
- **Location:** §5.1.1 vs REQ-069 acceptance criteria
- **Problem:** REQ-069 states "PLP and PDP render skeleton placeholders during SSR (not real vehicle data)" — implying both PLP and PDP use skeleton-only SSR. The spec correctly documents divergent strategies: PLP renders skeletons only while PDP performs full SSR with real vehicle data (three-path initialization). The spec is more accurate than the requirement, but the discrepancy is not explicitly noted.
- **Recommendation:** Add a note in §5.1.1 acknowledging the divergence from REQ-069's wording, or flag REQ-069 for revision. The spec's documentation is correct per the technical analysis.

**2. Lighthouse accessibility threshold vs WCAG 2.0 AA claim**
- **Location:** §5.1.4 (REQ-072) and §5.2 (REQ-073)
- **Problem:** §5.2 states the WCAG target is "WCAG 2.0 AA (per Audi platform requirements)" but §5.1.4 documents a Lighthouse accessibility threshold of 0.3 (30%), which is far below what would enforce WCAG 2.0 AA compliance. The spec notes the threshold is "low" but does not flag the contradiction or recommend a target. The technical analysis classifies this as a **high-severity risk**.
- **Recommendation:** Add a subsection or callout in §5.2 explicitly flagging the gap between the stated WCAG 2.0 AA target and the enforced 0.3 Lighthouse threshold. Recommend a minimum actionable threshold (e.g., ≥ 0.7) or reference the technical analysis risk item.

**3. REQ-006 duplicate entry in traceability matrix**
- **Location:** Section 3 traceability table
- **Problem:** REQ-006 (Share Results URL) appears in both §4.1 ("Vehicle Search & Browsing") and §4.22 ("Share"). §4.1.6 covers PLP share; §4.22 is a "Share" meta-section that lists both REQ-006 and REQ-092. This makes it unclear which section is authoritative for REQ-006. A requirement should map to one primary spec section.
- **Recommendation:** Remove REQ-006 from the §4.22 row. §4.22 should only contain REQ-092 (PDP share). Add a cross-reference note in §4.22 pointing to §4.1.6 for PLP share.

**4. PLP SSR SEO gap not elevated as a concern**
- **Location:** §5.1.1 and §5.4
- **Problem:** §5.4 mentions "PLP renders skeletons during SSR — vehicle listing content not available for crawlers in initial HTML" as a table row, but doesn't call this out as a risk. The technical analysis identifies this as a medium-severity risk that limits the SEO value of vehicle listing pages. For a spec that documents existing behavior, this should be flagged more prominently.
- **Recommendation:** Add a note or risk callout in §5.4 explicitly identifying this as a known limitation with potential SEO impact on vehicle listing pages.

### Minor (Nice to Fix)

**5. Japan (JP) market not included in Market-Specific Behaviors section**
- **Location:** §4.15 (Market-Specific Behaviors) vs §4.7.6 (REQ-047)
- **Problem:** §4.15 only documents DE, FR, and ES market variations. Japan (JP) is referenced in §4.7.6 as having a dedicated price template (`PriceInformationJapan`), but is not listed in §4.15. The spec's Open Issues (§7, item #7) asks whether the EU VTP is deployed in Japan, but the market-specific section should still reference JP for completeness.
- **Recommendation:** Either add a §4.15.6 stub for JP market variations (even if marked as "scope TBD") or add a note in §4.15 acknowledging that JP price template logic exists in code and is pending scope confirmation.

**6. Partner Business Card version pinning not flagged**
- **Location:** §4.9.2 (REQ-052)
- **Problem:** The technical analysis notes that the dealer-info FA defaults to `v3.2.0-rc.4` of the Partner Business Card — a release candidate, not a stable version. The spec does not mention this version or flag the risk.
- **Recommendation:** Add a note in §4.9.2 or §7 (Open Issues) about the release candidate version dependency.

**7. `deactivateSelectAllDealers` missing from REQ-018 acceptance criteria**
- **Location:** §4.2.10 (REQ-018)
- **Problem:** REQ-018 acceptance criteria include the ability to deactivate the "Select all dealers" checkbox via `deactivateSelectAllDealers`. The spec's acceptance criteria for §4.2.10 mention the checkbox but not the deactivation configuration. The business rules section does mention it.
- **Recommendation:** Add "Select all dealers can be deactivated via `deactivateSelectAllDealers`" to the §4.2.10 acceptance criteria for completeness.

**8. No explicit WCAG 2.0 AA compliance declaration**
- **Location:** §5.2
- **Problem:** While §5.2 references WCAG 2.0 AA in the implementation details table, there is no first-class declarative statement that the VTP targets WCAG 2.0 AA. This should be a prominent, unambiguous statement since it's a mandatory Audi platform requirement.
- **Recommendation:** Add a clear statement at the top of §5.2: "The EU VTP targets WCAG 2.0 AA compliance as mandated by Audi platform requirements."

**9. Requirement ID gap in spec section numbering not explained**
- **Location:** §4.13 → §4.14
- **Problem:** Functional sections jump from REQ-068 (§4.13.4) to REQ-077 (§4.14.1). Requirements REQ-069–REQ-076 are in §5 (Non-Functional Requirements). The reorganization is logical but may confuse readers tracing requirement IDs sequentially through Section 4.
- **Recommendation:** Add a brief note after §4.13 or at the start of §4.14 stating that REQ-069 through REQ-076 are covered in Section 5 (Non-Functional Requirements).

**10. Hardcoded German fallback text not flagged as technical debt**
- **Location:** §4.15.1 (REQ-080)
- **Problem:** REQ-080 mentions "Fallback sorting explanation text in German if i18n key is empty" and the technical analysis flags this as a concern (hardcoded German text in `CountAndSort.tsx`). The spec documents it as a business rule but doesn't flag it as technical debt or an i18n violation.
- **Recommendation:** Add a note that the hardcoded German fallback should be moved to i18n keys.

---

## Cross-Reference: Technical Feasibility

The spec's technical design notes (§6) and appendices align well with the technical analysis:

| Technical Analysis Item | Spec Coverage |
|------------------------|---------------|
| Architecture: monorepo structure | ✅ §1 Overview, §6, Appendix B |
| Package relationships: dependency graph | ✅ §6 Architecture Decisions |
| Configuration data flow: headless FA + pub/sub | ✅ §4.12 with data flow diagram |
| Vehicle data flow: SCS API → Redux/Context | ✅ §4.7.5 data flow, §4.4.1 user flow |
| SSR pipeline: Renderman → Feature Hub → bundles → hydration | ✅ §5.1.1 with pipeline diagram |
| Dual design system: 4 bundles, webpack aliases | ✅ §4.16.1 with detailed tables |
| Content Fragment hierarchy | ✅ Appendix D |
| Feature Service dependency matrix | ✅ Appendix A |
| External service dependencies | ✅ §6 External Service Dependencies table |
| Technology stack | ✅ Appendix B |
| Inter-app communication | ✅ Appendix C |
| Technical risks (high/medium/low severity) | ⚠️ Partially — risks not surfaced in spec (see Issue #2, #4) |

---

## Domain-Specific Completeness

| Domain Check | Status | Notes |
|-------------|--------|-------|
| Market-specific variations (DE, FR, ES) | ✅ Covered | §4.15 with dedicated subsections; JP partially addressed in §4.7.6 |
| Content author configuration points | ✅ Covered | §4.13 with comprehensive field tables for PLP, PDP, and shared VTP config |
| SSR compatibility | ✅ Covered | §5.1.1 with per-package strategy table, safety patterns, and SSR service list |
| WCAG 2.0 AA accessibility | ⚠️ Partially | Implementation details present in §5.2 but no strong declarative compliance statement; Lighthouse threshold contradicts WCAG target |
| Feature service dependencies | ✅ Covered | Appendix A with full matrix (22 services × 5 packages) |
| Content Fragment Model hierarchy | ✅ Covered | Appendix D with full tree structure |
| Data models / API contracts | ✅ Covered | Data models inline throughout §4; SCS/CRS/OneGraph/Omnigraph listed in §6 |
| Non-functional requirements | ✅ Covered | §5 covers SSR, performance, accessibility, tracking, SEO, security |

---

## Strengths

- **Complete requirement coverage**: All 93 requirements are traceable in the spec with matching acceptance criteria — no gaps in coverage.
- **Consistent document structure**: Every spec section follows a uniform pattern (Description → Business Rules → Data Model → Acceptance Criteria), making the document predictable and easy to navigate.
- **Strong data model documentation**: Filter types, CTA types, warranty types, business models, and price configurations are documented with clear tables and enumerations.
- **Effective use of diagrams**: Data flow diagrams (configuration flow, SSR pipeline, finance flow) add clarity beyond textual descriptions.
- **Comprehensive appendices**: Feature service matrix, technology stack, inter-app communication, and content fragment hierarchy are all documented in structured appendices.
- **Well-scoped Open Issues section**: §7 lists 15 specific unresolved questions with impact context, showing intellectual honesty about documentation gaps.
- **Cross-references between sections**: Market-specific behaviors reference back to their primary spec sections (e.g., §4.15.1 → §4.8.2), reducing duplication.
- **SSR strategy documented per package**: The per-package SSR strategy table in §5.1.1 is a particularly valuable artifact for developers.

---

## Summary

The EU VTP functional specification is a comprehensive, well-structured document that achieves 100% traceability against the 93 structured requirements. The document is internally consistent, follows spec standards (ISO 8601 dates, version number, Draft status), and provides thorough coverage of domain-specific concerns including SSR compatibility, content author configuration, market variations, and feature service dependencies.

The primary concerns are:

1. The Lighthouse accessibility threshold (0.3/30%) contradicts the stated WCAG 2.0 AA compliance target — this tension should be explicitly flagged rather than merely noted in passing.
2. A minor SSR strategy discrepancy between REQ-069's wording and the spec's (correct) documentation should be acknowledged.
3. REQ-006 appears in two traceability matrix rows, which should be cleaned up.
4. The PLP SSR SEO limitation deserves more prominent treatment as a known risk.

None of these issues are blockers. The spec accurately represents the system's behavior as reverse-engineered from the codebase, and the Open Issues section appropriately captures unresolved questions. The verdict is **APPROVED WITH COMMENTS** — the identified issues should be addressed in the next revision but do not prevent the spec from serving as a reliable reference.
