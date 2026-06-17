# Mapping Teamcenter PLM onto OSLC for the Digital Thread

*Three candidate mappings — PLM as configuration, PLM as resource definition, and the split that conforms to OSLC — with core and PLM selection extensions (`oslc:VariabilitySelections`, `oslc_plm:EffectivitySelections`) that complete the coverage.*

## Purpose

The immediate goal is to expose Teamcenter PLM resources as OSLC resources so that they can contribute to a broader **digital thread** for lifecycle management, configured and managed alongside other tools — IBM ELM, OpenText Octane, Jira, PTC Windchill, Aras Innovator, and others. In that thread, the OSLC representations of PLM parts, UML models, and SysML v2 models are all *definition/specification* content that must be versioned, linked, and selected by shared configurations.

Teamcenter integrates definition and selection into a single object graph (Item, Item Revision, BOM line/BVR, revision rule, variant rule, effectivity). For the digital thread we do **not** need to preserve that integration. It is acceptable — preferable — to refactor Teamcenter's integrated concepts into distinct OSLC concepts, provided the overall **intensional** aspects (the selection rules and criteria) and **extensional** aspects (the resolved set of specific versions) are covered in a reasonable, understandable way. The question is which refactoring does that while conforming to the existing OSLC specifications.

This document evaluates three mappings and recommends the third.

## What must be covered: PLM's three selection axes

A configured PLM structure is resolved along three orthogonal axes, not one:

1. **Version** — which revision of a part is in play. OSLC Configuration Management already covers this axis natively, through the concept-resource / version-resource split and `oslc_config:Selections`.
2. **Effectivity** — which usages and versions are valid in a given effectivity context (date, unit/serial, lot, end-item, block). Stock OSLC Configuration Management has no concept of this.
3. **Variability** — which usages are included given a chosen variant configuration (option values satisfying variant conditions on BOM lines). Stock OSLC Configuration Management has no concept of this either.

Each axis has an intensional form (a criterion or rule — "effective on 2026-06-01", "variant = European left-hand-drive") and an extensional form (the resolved set of specific version pins the criterion produces). Any correct mapping must place all three axes somewhere, preserve both forms, and keep the result composable with non-PLM tools that only know the version axis.

## Mapping 1 — PLM as configuration

The first candidate treats Teamcenter as essentially a configuration-management system and maps its constructs onto OSLC Configuration Management: Items and Item Revisions become configuration content, BOM views become configurations, revision and variant rules become the selection logic of `oslc_config:Configuration`.

This mapping fails in a characteristic way. OSLC Configuration Management defines a configuration as a *selector over versioned domain resources* — it is not itself the resource being described. An Item Revision is not a selection; it is a defined thing with properties, structure, and relationships. Folding Items and Item Revisions into the configuration layer overloads `oslc_config:Configuration` with definitional content it does not and should not model, and it gives the digital thread nowhere to *point* when another tool wants to link to "the part" as a resource. A requirement in ELM cannot trace to a configuration; it traces to a resource. Mapping 1 corresponds to the "Teamcenter ≈ OSLC Configuration Management" intuition, and it is wrong for the same reason that intuition is incomplete: it sees only the selector band and loses the definition the parts actually carry.

**A scope qualifier on Mapping 1's failure.** Mapping 1 is not universally wrong — it is wrong *when the content needs to be a referenceable domain resource for cross-tool linking*. MID Cross Domain Configuration Management (CDCM) extends OSLC Configuration Management in a Mapping-1-shaped way: it lets administrators define Configuration Item types with work product storage locations that describe as well as select items, similar to Teamcenter's conflation. However, CDCM Configuration Items are not themselves versioned resources, and other than contributions to configurations, they have no links to other OSLC resources. The Mapping 1 conflation is therefore benign for CDCM, because what was conflated was never expected to be externalized as a cross-tool resource in the first place. See *CDCM — a scoped Mapping 1 that works* below for the full treatment.

## Mapping 2 — PLM as resource definition

The second candidate treats Teamcenter as a domain of definable resources and maps everything onto OSLC domain resources: Item → a Part resource, Item Revision → a Part version resource, BOM line → a PartUsage resource, BVR → the assembly structure. Effectivity ranges and variant conditions become properties on the PartUsage. This is the "PLM as resources" approach used to define the proposed [OSLC PLM specification](https://github.com/oslc-op/oslc-specs/blob/master/specs/plm/plm-spec.html). 

This mapping captures the definition and structure beautifully — it is exactly what the digital thread needs for the *content* of PLM. But it has nowhere to put the selection machinery except *inside the resources themselves*, as domain-specific properties on PartUsage. That reproduces Teamcenter's own conflation inside OSLC: the selection predicates ride on the structure, so the selector is no longer a separate, addressable resource. A Global Configuration composing PLM with ELM, Octane, Jira, Windchill, and Aras Innovator cannot reason about effectivity or variant resolution, because those concerns are buried in domain properties the configuration protocol does not understand. Mapping 2 corresponds to the "Teamcenter ≈ SysML / defined things" intuition, and it is wrong for the symmetric reason: it sees only the definition aspects and cannot externalize selection.

The two failed mappings are the two camps. Each isolates one band of an integrated system and tries to make it the whole picture.

## Mapping 3 — the split, extended for PLM (recommended)

The correct factoring separates the concerns the way OSLC already does, and then extends the selection layer to cover the two PLM axes OSLC does not yet have.

### Definition / specification: `oslc_plm:Part` and `oslc_plm:PartUsage` as AM resources

PLM definition content is modeled as architecture-management resources — an `oslc_plm:Part` and `oslc_plm:PartUsage` profiled on `oslc_am:Resource`. (OSLC AM is deliberately generic; SysML and UML elements are already represented as `oslc_am:Resource`, so PLM parts join the same family and link naturally to them.)

- `oslc_plm:Part` ≈ a part definition (Teamcenter Item, with its revisions exposed as `oslc_am` resource versions).
- `oslc_plm:PartUsage` ≈ an occurrence — the BOM line as a relationship from a parent part version to a child part, carrying quantity, find number, and position.

These are the **selected** resources. They are versioned, linkable, and they participate in the digital thread exactly like an ELM requirement, an Octane/Jira work item, or a Rhapsody UML class. Critically, the variant conditions and effectivity ranges that Teamcenter stamps on a BOM line are *not* modeled here as opaque properties that only Teamcenter understands; they are surfaced to the selection layer (below) so a configuration can reason about them.

### Selection: OSLC Configurations, extended with `oslc:VariabilitySelections` and `oslc_plm:EffectivitySelections`

Selection stays in the configuration layer, where OSLC consumers expect it. The version axis is handled by stock OSLC Configuration Management: an `oslc_config:Configuration` (Baseline / Stream / ChangeSet) referencing `oslc_config:Selections` resolves each concept resource to a specific version. The two additional PLM axes are added as two further subclasses of `oslc_config:Selections`, and they are placed according to a single principle: **a selection type is owned by the specification responsible for resolving it.** The set of these subclasses across the specs becomes the vocabulary of what each domain can contribute to a local configuration.

- **`oslc:VariabilitySelections`** (OSLC **core**) — a selection along the variant axis. It carries a variant *configuration* (the intensional criterion: a set of option values, or an expression that satisfied variant conditions must match). Applied to a configuration, it includes or excludes usages whose variant conditions match the chosen options, resolving to extensional pins. Variability is placed in core because it is a *universal* resource concern: optional and alternative content occurs in requirements, test cases, models, and parts alike, so any OSLC provider can compute a variant selection.
- **`oslc_plm:EffectivitySelections`** (OSLC **PLM**) — a selection along the effectivity axis. It carries an effectivity *context* (the intensional criterion: a date, a unit/serial number, an end-item plus range). Applied to a configuration, it binds the effectivity-bearing usages to those effective in the context and resolves them to extensional version pins; usages not effective in the context are excluded (a fetch for a non-effective selection returns 404). Effectivity is placed in PLM because only a PLM-capable provider can resolve its date/unit/serial/lot/end-item semantics.

Both are plural, matching the `oslc_config:Selections` convention they specialize — a class name that implies multiplicity is slightly odd, but consistency with the established OSLC Configuration Management naming wins. Both slot into the existing configuration model as additive selection criteria. This placement is itself a cohesion and conformance decision (see below): the core OSLC Change Management specification is left untouched — neither new type is added to `oslc_config:` — at the cost of a single vocabulary reference from core into config (`oslc:VariabilitySelections rdfs:subClassOf oslc_config:Selections`). That reference makes core and config mutually dependent in the vocabulary, a deliberate tradeoff that avoids perturbing the configuration standard; in practice it is benign, since config is rarely adopted without core. (If strictly acyclic layering were preferred instead, an abstract selection base could be hoisted into core and all three subclasses reparented under it — at the cost of a small, backward-compatible change to the config spec.)

### Vocabulary sketch

```turtle
@prefix oslc:        <http://open-services.net/ns/core#> .
@prefix oslc_plm:    <http://open-services.net/ns/plm#> .
@prefix oslc_config: <http://open-services.net/ns/config#> .
@prefix oslc_am:     <http://open-services.net/ns/am#> .
@prefix rdfs:        <http://www.w3.org/2000/rdf-schema#> .

# --- Definition / specification: the SELECTED content ---
oslc_plm:Part       rdfs:subClassOf oslc_am:Resource .   # ~ Teamcenter Item
oslc_plm:PartUsage  rdfs:subClassOf oslc_am:Resource .   # ~ Teamcenter BOM line / occurrence
# PartUsage relates a parent part version to a child part, with quantity,
# and exposes (rather than hides) its variant condition and effectivity range
# so the selection layer can resolve them.

# --- Selection: the SELECTOR, extending OSLC Configuration Management ---
# Each selection type is owned by the spec responsible for resolving it.
oslc:VariabilitySelections     rdfs:subClassOf oslc_config:Selections .  # variant axis  (core: universal)
oslc_plm:EffectivitySelections rdfs:subClassOf oslc_config:Selections .  # effectivity axis (PLM-only)
```

```turtle
# A PLM-aware configuration aggregates version selections (stock OSLC Configuration Management)
# plus the two extension selections, all via the standard oslc_config:selections
# property — so a tool that only understands version selection still sees a
# valid configuration, and the rdf:type distinguishes the richer kinds.
<#engineConfig> a oslc_config:Stream ;
    oslc_config:selections <#versionPins> ,        # version axis  (oslc_config:Selections)
                           <#euLeftHandDrive> ,     # variant axis  (oslc:VariabilitySelections)
                           <#effOn2026-06-01> .     # effectivity   (oslc_plm:EffectivitySelections)

<#euLeftHandDrive> a oslc:VariabilitySelections ;
    oslc:variantConfig [ oslc:option "region=EU" , "drive=LHD" ] .

<#effOn2026-06-01> a oslc_plm:EffectivitySelections ;
    oslc_plm:effectivityContext [ oslc_plm:onDate "2026-06-01"^^xsd:date ] .
```

### Resolution: composing the three axes

A fully resolved PLM configuration is the composition of the three selections applied to the part/usage structure:

```
configured BOM  =  VariabilitySelections ∘ EffectivitySelections ∘ version Selections  applied to the PartUsage tree
```

In protocol terms, a consumer resolves the structure by supplying the configuration context plus the extension criteria as query parameters — e.g. `oslc_config.context=<gc>` for the version-selecting configuration, `oslc_config.effectivity=<date|unit>` to pick the applicable `oslc_plm:EffectivitySelections`, and a corresponding variant parameter to pick the `oslc:VariabilitySelections`. The server returns the extensional set: the effective, variant-correct versions — the configured BOM. The predicate language that actually *evaluates* whether a usage is effective or whether a variant condition is satisfied is deferred to the PLM domain specification, with PLCS / ISO 10303-239 reference data recommended as the vocabulary layer, since no common runtime predicate language exists across PLM vendors.

### Precise and imprecise BOMView Revisions

Teamcenter distinguishes **precise** from **imprecise** BOMView Revisions, and the distinction maps cleanly onto the Stream / Baseline pair that the selection extensions live inside. The mapping is worth recording explicitly, because it shows that the extensions formalize a behaviour Teamcenter already supports rather than introducing a new one.

In Teamcenter:

- An **imprecise BVR** has BOM lines that reference Items (master records, not specific ItemRevisions). At navigation time a Revision Rule + Effectivity Cursor + Variant Rule resolves which specific ItemRevision applies for each Item. The BVR is **late-bound**: different rules and cursors yield different concrete revisions.
- A **precise BVR** has BOM lines that reference specific ItemRevisions directly. No rule or cursor is applied at navigation; the revisions are pinned. The BVR is **early-bound**: it represents a single, frozen configuration of the structure. Promotion from imprecise to precise typically happens at release-to-manufacturing.

In the Mapping 3 framing:

- **Imprecise BVR ⇔ `oslc_config:Stream`** carrying `oslc_plm:effectivityContext`, optionally `oslc:variabilityContext`, and `oslc_plm:EffectivitySelections` (plus `oslc:VariabilitySelections` if variability is in play) whose `oslc_config:selects` is *server-maintained*. The Stream's selects names the currently-resolved Part / PartUsage version pins. If the effectivity context reference is changed (or the referenced context resource is edited), the selects is recomputed. The Stream is the late-binding configuration.
- **Precise BVR ⇔ `oslc_config:Baseline`** that freezes both the `EffectivitySelections.selects` (specific Part / PartUsage version URIs) and the `effectivityContext` reference that produced them. The Baseline is the early-binding configuration. Asking the Baseline "which revision of this Item is in?" returns the frozen version pin directly — no rule, no cursor, no resolution.

Teamcenter's promotion of an imprecise BVR to a precise BVR corresponds exactly to baselining a Stream: the Stream's currently-resolved `EffectivitySelections.selects` is frozen into a Baseline along with the effectivity context reference that produced it, and the result is a navigable, externally-referenceable snapshot. That is precisely what release-to-manufacturing BVRs are used for.

Three clarifying points:

1. **Late binding is not the same as "no effectivity applied yet."** An imprecise BVR with no Revision Rule and no Effectivity Cursor still has *all* its lines effective — what changes when a rule and cursor are supplied is *which specific revision* is chosen for each Item. The OSLC analogue is that the Stream's `EffectivitySelections.selects` is always defined (it is server-maintained), and its content reflects the configuration's currently-attached effectivity context. Changing the context produces a different selects; deleting the context produces a 400 on the configuration's POST/PATCH if it contains an `EffectivitySelections` (consistent with the rule that an `EffectivitySelections`-bearing configuration must carry an `effectivityContext`).
2. **Precise BVRs do not necessarily come from baselining.** An engineer can author a precise BVR directly by pinning each BOM line to a specific revision. The OSLC equivalent is creating a Baseline directly with manually-specified `EffectivitySelections.selects`, rather than by snapshotting a Stream. Both are admissible. The `effectivityContext` on such a Baseline reflects the context the engineer asserted (or used to compute the pins manually), not necessarily a previous Stream state.
3. **A Baseline retains its effectivity-context reference even after pinning.** The frozen `oslc_plm:effectivityContext` on a Baseline is the audit trail of what context produced the pins. A downstream consumer can reason about the Baseline ("this is the configuration as effective on 2026-06-01 for end-item EW2027-001") without re-running any resolution; the context plus the selects together fully describe the configuration.

This subsection thus does not introduce a new concept — it makes explicit that the existing Stream / Baseline distinction in OSLC Configuration Management, extended with the new `oslc_plm:EffectivitySelections` (and, where relevant, `oslc:VariabilitySelections`) subclasses on the configuration, is exactly the imprecise / precise distinction Teamcenter has carried since the inception of its BVR model. A Teamcenter adapter can map imprecise BVRs to Streams and precise BVRs to Baselines without semantic loss, and a consumer that understands `oslc_config:Stream` / `oslc_config:Baseline` understands the late-binding / early-binding character of the BVR it is looking at.

## Why the concerns are separated

The separation is not stylistic; it is required for the digital thread to work and for the result to conform to OSLC.

**To conform to existing OSLC specifications.** OSLC Configuration Management defines configurations as selectors over versioned domain resources, and the domain specifications (AM, RM, QM, CM) define resources that carry no selection logic of their own. A mapping that embeds selection inside resources (Mapping 2) or that promotes resources into the configuration layer (Mapping 1) violates that contract and breaks interoperation with any consumer that speaks the standard configuration protocol. Keeping `oslc_plm:Part`/`PartUsage` as plain domain resources and keeping selection in the configuration layer — extended by the core and PLM `oslc_config:Selections` subclasses — is what makes a Teamcenter adapter a conformant OSLC provider rather than a bespoke bridge.

**To assign each selection to the spec that resolves it.** Beyond conformance, the placement of the two extension types follows functional cohesion: the specification responsible for *computing* a kind of selection owns its type. Core owns variability because every provider can compute a variant selection; PLM owns effectivity because only a PLM provider can resolve date/unit/serial/lot/end-item semantics; the configuration layer owns version selection. The union of these subclasses formalizes exactly what each domain can contribute to a local configuration, and keeps coupling between the specs minimal — the configuration standard need not learn anything PLM-specific, and each domain adds only the selection it can actually resolve.

**To enable a heterogeneous digital thread.** The entire objective is composing PLM with ELM, Octane, Jira, Windchill, and Aras under Global Configurations. That is only possible when the selector is a standalone, addressable resource (the configuration) and the content is standard domain resources. The split is the precondition for cross-tool composition; the conflated mappings cannot participate.

**To keep selection explicit and governable.** A digital thread needs provenance: baselines, change sets, auditability. An explicit configuration resource — rather than a session-time rule overlay computed inside Teamcenter — is what carries that governance and what an external tool can link to, baseline, and reason about.

**To degrade gracefully across mixed tools.** Because `oslc:VariabilitySelections` and `oslc_plm:EffectivitySelections` are *additive* subclasses of `oslc_config:Selections` — referenced through the same `oslc_config:selections` property — a tool that does not understand them still receives a valid, version-selected configuration. PLM-aware consumers get the additional effectivity and variant resolution. The thread does not require every tool to understand PLM or variant semantics in order to function — a hard requirement when ELM, Jira, and Aras have very different native models.

**To allow reuse of a part across many resolutions.** Separating definition from selection lets the same `oslc_plm:Part` participate in many configurations, each resolving it differently by version, effectivity, and variant — the same way the concept/version split lets one concept resource appear in many configurations. Conflated models lose this.

## How the extensions complete the coverage

With the split in place, every aspect of Teamcenter PLM maps to a conformant OSLC construct, and all three selection axes are covered without overloading any one layer:

| Teamcenter concept | OSLC mapping | Band / role |
|---|---|---|
| Item | `oslc_plm:Part` (concept resource) | definition — selected content |
| Item Revision | `oslc_plm:Part` version (`oslc_am` resource version) | version — selected content |
| BOM line / BVR | `oslc_plm:PartUsage` | structure/usage — selected content |
| Revision rule (version pick) | `oslc_config:Configuration` + `oslc_config:Selections` | selection — version axis |
| Effectivity (date/unit/lot/end-item) | `oslc_plm:EffectivitySelections` | selection — effectivity axis (PLM extension) |
| Variant rule / variant conditions | `oslc:VariabilitySelections` | selection — variant axis (core extension) |
| Imprecise BVR (late-bound; resolves at navigation) | `oslc_config:Stream` (with server-maintained `EffectivitySelections.selects` against the stream's attached `oslc_plm:effectivityContext`) | selection — binding posture (late) |
| Precise BVR (early-bound; revisions pinned) | `oslc_config:Baseline` (with frozen `EffectivitySelections.selects` and frozen `oslc_plm:effectivityContext`) | selection — binding posture (early) |
| Configured / serialized structure | resolved extensional set (configured BOM) | instance — resolution output |

The intensional/extensional duality is preserved throughout. Each selection resource holds an intensional criterion — a version rule, an effectivity context, a variant configuration — and the configuration *resolves* to an extensional set of specific version pins. `oslc_plm:EffectivitySelections` and `oslc:VariabilitySelections` are precisely the constructs that let an OSLC configuration carry the *additional* selection that Teamcenter performs beyond version selection, expressed as explicit, governable, linkable resources rather than as an in-session rule overlay. With the version axis from stock OSLC Configuration Management and these two subclasses, an OSLC configuration covers all of the selection aspects of PLM — and does so in a way the rest of the digital thread can consume.

The placement principle ("a selection type is owned by the spec responsible for resolving it") also tells us what an OSLC Configuration Management provider that does *not* resolve any of these axes must produce: nothing new. A pure aggregator that only composes other providers' configurations adds no selection subclasses — it simply carries them through. CDCM (next section) is the canonical example.

## CDCM — a scoped Mapping 1 that works

MID Cross Domain Configuration Management (CDCM) is already an OSLC Configuration Management provider — it registers with JTS as an ELM Global Configuration provider and speaks the standard `/gc` protocol. It is not a foreign system being mapped *onto* OSLC; it is an OSLC Configuration Management implementation that extends the standard with a typed Configuration Item / Work Product layer. Understanding where CDCM sits in the three-mapping frame sharpens what Mapping 1 actually fails at, and shows how a Mapping 3 PLM provider and CDCM are complementary roles in a digital thread.

### What CDCM is, in the layered model

In the vocabulary of the companion document (`oslc-cm-sysml-teamcenter.md`):

| Aspect | CDCM |
|---|---|
| Definition (what kind) | Configuration Item Types (CDCM-owned; carry properties, editor view, lifecycle, authorization) |
| Version (temporal) | Configurations are versioned (Stream / Baseline / ChangeSet); Configuration Items themselves are not |
| Structure / usage | Contribution to a Configuration — the only structural relationship between Configuration Items |
| Selection (the selector) | Configuration + ordered Contribution list — purely extensional |
| Instance | n/a |

CDCM does Mapping 1: it embeds typed content in the configuration layer. But unlike Teamcenter:

- CDCM has no rule layer to externalize. Its selector is purely extensional — the contribution list (ordered by `orderId`, walked depth-first with first-match-wins on the resolved flat list) *is* the resolved set. This is the inverse of Teamcenter's intensional, in-session rule + context overlay.
- CDCM's Configuration Items are not versioned. Versioning exists only at the configuration level (Stream / Baseline / ChangeSet on the configuration itself).
- CDCM's Configuration Items have no cross-domain relationships beyond contribution. Other OSLC tools never link to a CDCM Configuration Item *as a domain resource*; they link to the Configuration that contains it (or to the work-product URL it points at).
- The CDCM extensions for Configuration Item and Work Product definitions are not exposed through its OSLC API. That is, the only configuration contributions that are included in the OSLC resource representations are the configuration items whose work products are references to other CDCM configurations in the same configuration area, or whose work products are OSLC local configurations of OSLC domain resources. 

### Why Mapping 1 is fatal for Teamcenter but not for CDCM

The Mapping 1 critique is "Mapping 1 conflates definition into the configuration layer, leaving the digital thread with nowhere to point when another tool wants to link to the part as a resource." That critique still bites — but only when there *is* a resource that needs to be pointed at. The condition for Mapping 1 to fail is not embedding definition; it is failing to externalize a referenceable resource that other tools need to link to.

Teamcenter parts are exactly that kind of resource: ELM requirements, change requests, test cases, and SysML model elements all need to trace to a versioned part. Mapping 1 makes that impossible. CDCM Configuration Items are the opposite kind of content: organizational metadata that other tools were never going to link to as resources. CDCM was introduced as a means of collecting and organizing a large number of documents and other resources that were formerly managed manually; the linkable resources in question are the *work products themselves* (DOORS Next modules, ETM test plans, model artifacts at URLs), not CDCM's organizational wrappers around them. The Mapping 1 conflation in CDCM therefore conflates only the bookkeeping with the configuration layer that already does the bookkeeping anyway.

This sharpens the Mapping 1 critique in the companion document: *Mapping 1 is wrong for content that other tools link to as cross-domain resources; it is a legitimate, scoped design choice for content that is purely internal to the configuration system.* CDCM is the worked example of the latter.

### CDCM is a configuration provider, not a selected-content provider

There is a second asymmetry between CDCM and Teamcenter, and the three-mapping frame as written doesn't surface it. The three mappings implicitly assume the system being mapped *both* provides definition content *and* is configured by OSLC for the resolution of its own content. CDCM is different: it is an OSLC Configuration Management **provider**, not a selected-content provider.

- CDCM produces Global Configurations that other OSLC applications (DOORS Next, ETM, etc.) use as contexts to resolve their *own* version pins (see `docs/OslcGcIntegration.md`).
- CDCM itself does not consume any configuration context to select its own resources. Its content isn't versioned at the Configuration Item level, so the question doesn't arise.
- CDCM's flat-list endpoint produces an ordered list of local configurations; the *owning* providers do the concept-URI-to-version-resource resolution against their own selections. CDCM is the aggregator, not the resolver.

So in the role taxonomy the three mappings implicitly use, CDCM is a third kind of OSLC Configuration Management player: not a domain-resource provider (RM, QM, AM), and not a vertically integrated tool that needs to be mapped to one (Teamcenter, Windchill, Aras), but a *Global Configuration aggregator* that composes both kinds upward.

### CDCM and Mapping 3 are complementary, not competing

If Teamcenter is exposed via Mapping 3 (`oslc_plm:Part` / `oslc_plm:PartUsage` as `oslc_am` resources, plus `oslc_plm:EffectivitySelections` / `oslc:VariabilitySelections` in the configuration layer), CDCM aggregates the resulting local configurations into Global Configurations alongside ELM, Octane, Jira, Windchill, Aras, and other tool configurations. CDCM does not need to know anything about the PLM extensions to perform this aggregation: the new selection subclasses are additive `oslc_config:Selections` specializations referenced through the same `oslc_config:selections` property, so CDCM's flat-list pipeline already carries them transparently. The same is true for any future selection subclasses adopted by other domains.

The placement principle works in CDCM's favour: *CDCM owns no resolution, so CDCM adds no selection types.* It is the worked example of a CM provider that legitimately produces zero extension subclasses while still being a first-class participant in the digital thread.

### What this changes in the recommendation

The recommendation in this document — Mapping 3 for vertically integrated PLM systems being exposed to OSLC — is unchanged. CDCM sits at a different layer and does not compete with it. Two clarifications are now visible that should be carried into the conclusion:

1. The Mapping 1 critique is scope-dependent. It is fatal for systems whose content needs to be cross-tool linkable as domain resources; it is a legitimate choice for systems whose content is purely organizational and never expected to be pointed at by other tools.
2. The digital-thread story has three CM-side roles, not one: a *domain-resource provider* (RM, QM, AM, and now PLM via Mapping 3), a *PLM provider* (Teamcenter / Windchill / Aras via Mapping 3, supplying the PLM-specific selection subclasses), and a *Global Configuration aggregator* (CDCM, supplying composition and organizational typing without producing any new selection subclasses). The three roles compose; the digital thread requires all three but does not require any one tool to play all three.

## Conclusion

The first two mappings each collapse Teamcenter's integrated model onto a single OSLC band and fail in mirror-image ways: as configuration, it loses the parts as resources; as resource definition, it cannot externalize selection. The third mapping conforms to OSLC by keeping definition in `oslc_plm:Part`/`PartUsage` architecture-management resources and selection in OSLC configurations, and it completes PLM coverage by extending the configuration layer with two `oslc_config:Selections` subclasses — `oslc:VariabilitySelections` in core (variability is universal) and `oslc_plm:EffectivitySelections` in PLM (only PLM resolves it). That split is what lets Teamcenter become a first-class participant in a configured digital thread spanning ELM, Octane, Jira, Windchill, Aras, and the SysML/UML models it traces to — with the full intensional and extensional character of PLM selection preserved, and with the core OSLC specifications left intact.

CDCM rounds out the picture as the canonical aggregator role: an OSLC Configuration Management provider that extends the standard with a typed Configuration Item layer for organization, deliberately embeds that typing in the configuration layer because its content was never expected to be a cross-tool referenceable resource, and aggregates external local configurations (including those from Mapping-3 PLM providers carrying the new selection subclasses) into Global Configurations. The Mapping 1 critique is scope-dependent: it is fatal for content that other tools need to link to, and a legitimate design choice for content that they don't. The three CM-side roles — domain-resource provider, PLM provider, Global Configuration aggregator — compose, and the digital thread requires all three. Mapping 3 fits Teamcenter; CDCM fits the aggregator role; OSLC Configuration Management with the two new selection subclasses fits both into one configured lifecycle.
