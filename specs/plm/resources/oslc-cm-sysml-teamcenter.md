# OSLC Configuration Management, SysML v2 as Versioned `oslc_am` Resources, and Teamcenter PLM

*A layered analysis of how three systems relate — and why the disagreement about where Teamcenter "overlaps" is an artifact of viewing one integrated stack from different aspects.*

## Thesis

There is a recurring disagreement or confusion about how Teamcenter PLM relates to the rest of the systems-engineering lifecycle. One viewpoint sees Teamcenter as overlapping with **OSLC Configuration Management** — it plays the role of *selecting* things. Another viewpoint sees Teamcenter as overlapping with **SysML v2** — it *defines* the things that get selected. Both observations are correct, and they are not in conflict. They are two slices of the same fact: OSLC Configuration Management and SysML each deliberately occupy a single aspect of a layered model, while Teamcenter is vertically integrated and spans the entire model. Which system Teamcenter "looks like" depends only on which aspect you happen to be looking at.

The cost of Teamcenter's integration — and the reason interoperability with an OSLC-based lifecycle is genuinely hard — is that it does not draw the line that OSLC draws between the *selector* and the *selected*. The deeper issue is not merely *where* the selector lives, but *what kind* of selector it is: OSLC's is extensional and explicit, Teamcenter's is intensional and computed. Reconciling those two notions of selection is the real subject of the comparison, and it is precisely what an effectivity-based resolution mechanism for OSLC PLM is designed to bridge.

## A shared conceptual frame

To compare the three systems without talking past each other, it helps to fix a small, neutral vocabulary that all of them can be mapped onto.

A **Thing** has a conceptual identity — a near-empty peg that persists over time. It is almost propertyless on its own; properties and relationships do not float on the timeless identity. Instead they are borne by **versions**: time-indexed states that capture the thing's temporal lifecycle. A thing also exhibits **variability** — a choice-indexed dimension orthogonal to time. Properties and relationships are therefore indexed not by the bare identity but by a *point in the version × variant space*. (A variant may be modeled as a distinct identity with its own part number, or as an option-loaded generic carrying variant conditions; the two systems below make different choices here.)

Three structural layers sit on top of this:

1. **Definition** — what *kind* of thing this is.
2. **Usage / occurrence** — the use of a kind in a role within a containing structure. This is still specification, not realization, and it is the layer where a SysML part usage and a PLM BOM line meet.
3. **Instance** — the realized, identifiable individual in a delivered unit, often serialized.

**Effectivity** is a predicate that selects region(s) of the version × variant space. It attaches at two levels: to a version (release validity — when is this revision valid at all) and, more fundamentally, to a *relationship/usage* (when is this child-in-parent valid). A part is never "effective" in the abstract; a *usage* of a part in a parent is effective over a range.

Finally, a **configuration** is a named, often baselined selection, and it is dual. It has an *intensional* side — the selection rules: a revision rule, a variant configuration, an effectivity context — and an *extensional* side: the resolved member set. The function that maps the intensional selector to a concrete instance or baseline is **resolution**. In this frame, effectivity and variability are *inputs* to resolution, version is the *axis* resolution traverses, configuration is the *specification* of a resolution, and an instance is its *output*.

This vocabulary gives us the aspects against which to map each system.

## SysML v2 as versioned `oslc_am` resources

In a pure SysML v2 setting, the language separates **definition** (`PartDefinition`), **usage** (`PartUsage`), and **individual** (snapshot/occurrence). A definition is a KerML classifier describing a *kind*; a usage is a feature representing that kind playing a role inside a containing definition or usage; an individual is a realized occurrence. SysML v2's `variation`/`variant` mechanism covers the variability axis genuinely, expressing the option space a definition supports. The Systems Modeling API adds a Git-like commit/branch/tag model, giving SysML a version axis of its own — though that is *version of the model*, not effectivity-driven product configuration.

When SysML content is brought into an OSLC lifecycle, those model elements surface as **`oslc_am:Resource`** instances in the Architecture Management domain. Each element has a concept resource (identity) and version resources (state), and the elements are placed under OSLC configurations like any other domain resource. This is the integration reality in tools such as model management within IBM ELM: SysML/UML models can be exposed as versioned AM resources that participate in component-level and global configurations.

The crucial point for the comparison: as `oslc_am` resources, SysML model elements are **defined content — the things being selected.** They occupy the *definition* and *usage/structure* bands. A root part's internal structure, populated with part usages, is structurally BOM-view-shaped — which is exactly why the "Teamcenter defines the selected things" intuition is reasonable. But SysML, even with variation modeling, does not own the *selection* band: it can *define* variability, but it has no native, managed runtime resolution of that variability against effectivity. It is the disciplined definition/structure layer, and nothing more.

## OSLC Configuration Management

OSLC Configuration Management occupies the **selection band** and almost nothing else. Its `Configuration` resource — specialized as `Baseline` (immutable), `Stream` (mutable), or `ChangeSet` — together with its `Selections` is a *selector*: it picks specific versions of versioned domain resources. The concept-resource/version-resource split gives it a version model; Global Configurations compose component-level configurations into a system-wide selector across tools and vendors.

What OSLC Configuration Management deliberately does *not* do is define the semantics of what is selected. Definition is pushed entirely into the domain specifications — Requirements Management, Quality Management, Architecture Management, Change Management. A SysML model element is content (an `oslc_am:Resource`); a requirement is content; a test case is content. OSLC Configuration Management only says *which versions of that content are in this configuration*. It is a pure selector layer deliberately kept empty of definitional and structural meaning.

This separation is the source of OSLC's central value: one Global Configuration can compose a DOORS Next requirement, an ETM test case, and a SysML model — each a versioned domain resource owned by a different tool — into a single coherent, linkable, navigable system baseline. That cross-tool composition is only possible because the selector is a *separate, explicit, externally referenceable* resource, distinct from the things it selects.

## Teamcenter PLM

Teamcenter spans every aspect of the model in one integrated object graph, which is precisely why it resembles whichever neighbor you compare it against.

- **Item** carries conceptual identity (the master, the part number) — the definition band.
- **Item Revision** carries the version axis, properties, lifecycle state, and attached CAD/datasets — definition + version.
- **BOM line / BOM View Revision (BVR)** is the usage/structure band: the parent-revision uses a child with quantity, find number, position — *and* it embeds variant conditions and effectivity ranges. This cell is the hinge.
- **Revision Rule + Variant Rule + Effectivity context** is the selector: a predicate, evaluated at traversal/expansion time, that resolves which Item Revisions a configured structure contains. This is the selection band.
- **Configured / serialized structure** is the instance band: the materialized as-built or as-maintained unit.

The conflation the two viewpoints argue about is real, but it is a three-way entanglement, not two-way. The BOM line is *not* the selector; it is structure that *carries* selection predicates. The actual selecting is done by the rule layer (revision rule + variant rule + effectivity), which is easy to overlook because Teamcenter applies it as a session/context overlay rather than persisting it as a first-class, linkable selection resource. The "Teamcenter ≈ OSLC Configuration Management" viewpoint is looking at the rule layer; the "Teamcenter ≈ SysML resource" viewpoint is looking at the Item and structure; the Teamcenter system itself does both in one place.

## Aspect coverage at a glance

| Aspect (concern) | SysML v2 (as `oslc_am`) | OSLC Configuration Management | Teamcenter | CDCM |
|---|---|---|---|---|
| Definition (what kind) | Part definition — defined content | pushed to domain specs | Item | Configuration Item Types (server-internal: properties, editor view, lifecycle, authorization) |
| Version (temporal) | model commits (via API) | concept / version split | Item revision | Configurations are versioned (Stream / Baseline / ChangeSet); Configuration Items themselves are not, but their work products may be versioned |
| Structure / usage (occurrence) | part usage tree | (domain's job) | BOM line / BVR (+ effectivity) — *hinge* | Contribution to a Configuration (the only structural relationship between Configuration Items) |
| Selection (the selector) | not managed | Configuration + Selections | Revision rule + variant + effectivity | Configuration + Contribution list (purely extensional, structural; `orderId` + depth-first first-match-wins) |
| Instance (realized unit) | individual / snapshot | (domain's job) | configured BOM / serial unit | n/a |

SysML fills the definition and structure rows; OSLC Configuration Management fills the version-model and selection rows; Teamcenter fills the entire column. The single cell that is *both* defined content and a carrier of selection input — the BOM line — is the conflation point.

CDCM is a fourth point of reference along this axis. It is already an OSLC Configuration Management provider (it registers with JTS as an ELM Global Configuration provider), and it extends OSLC Configuration Management with typed Configuration Items that carry definition-style properties (lifecycle, authorization, editor view) without being cross-tool linkable domain resources in the OSLC sense. CDCM's selector is purely extensional — the contribution list *is* the resolved set — which is the inverse of Teamcenter's intensional rule + context. The Mapping 1 conflation that this document and the companion mapping doc identify as fatal for Teamcenter (because it loses parts as referenceable resources) is *benign* for CDCM, because CDCM's own content was never expected to participate in cross-tool resource linking. The conflation that matters depends on what gets externalized, not just on whether the system embeds definition in the configuration layer.

## The core mismatch: extensional vs. intensional selection

Locating the selector in the right band is only half the story. The harder difference is *what kind* of selector each system uses.

**OSLC's selector is extensional and explicit.** A `Configuration`/`Selections` resource enumerates — or references via contribution — the specific versions that are in. Selection is, in essence, a set of version pins. `oslc_config:UnboundSelections` is the escape hatch for "not yet pinned." or how the pinning is done is custom implementation in the server. 

**Teamcenter's selector is intensional and computed.** A revision rule plus an effectivity context is a *predicate*, evaluated lazily against effectivity data embedded in the structure at traversal time. Selection is a *rule*, not a stored set; the configured BOM is materialized on demand rather than enumerated in advance.

This is why neither "overlap" framing resolves the integration problem. You cannot hand an OSLC Global Configuration "just the Teamcenter selector," because that selector is not a referenceable resource — it is a rule plus effectivity scattered across BOM lines and resolved in-session. Nor can you hand it "just the selected content," because the content and the selection predicates are entangled in the same BOM line. OSLC's entire cross-tool value proposition depends on the selector being separate, explicit, and linkable; Teamcenter's value proposition is the opposite — total integration with no impedance mismatch between structure and configuration — purchased at the price of not being able to externalize either half.

## An alternate proposal: collapse the selector into the selected (and why it fails)

A recurring counter-proposal in OSLC-OP discussions of the PLM ↔ OSLC mapping deserves explicit treatment because it appears, at first glance, to recapture Teamcenter's vertical integration cleanly: rather than separate the selector and the selected at all, **subclass the PLM domain resources directly under the OSLC Configuration Management resources**. Declare `oslc_plm:Part rdfs:subClassOf oslc_config:Configuration` and `oslc_plm:PartUsage rdfs:subClassOf oslc_config:Contribution`. Now a Part *is* a Configuration; a PartUsage *is* a Contribution; the BOM hierarchy and the configuration hierarchy are literally the same hierarchy. The intuition is that this fuses the two layers the way Teamcenter does, while staying within OSLC's vocabulary by extending it through subclassing rather than by adding new selection types alongside it.

Even on its own terms the proposal does not eliminate the need for selection machinery: variability and effectivity criteria still have to be captured and computed somewhere. The natural place, under this proposal, is on the PartUsage-as-Contribution, which would then carry domain-specific properties — option conditions, effectivity ranges — that contributions in OSLC Configuration Management do not. That observation alone is a sign that the unification is incomplete: the proposal already has to introduce a new selection surface to do the work an `oslc_config:Contribution` does not do, which is the same surface the additive selection-subclasses in the reconciliation (next section) introduce. But the more serious problem is structural, and worth working through because the way it fails surfaces a property of OSLC Configuration Management that is easy to overlook.

### Where the proposal breaks: a versioning regress

OSLC Configuration Management Configurations are not versioned the way domain resources are. A Component has multiple Configurations — each Configuration is a Stream, a Baseline, or a ChangeSet — but those are not "versions of one Configuration." They are distinct Configurations within the same Component, identified by distinct URLs. The Configuration itself is the unit of selection, not a thing being selected. There is no Configuration-Context that applies *to* a Configuration; Configurations are what *provide* the Configuration-Context to other resources.

An `oslc_plm:Part`, by contrast, is unambiguously a versioned domain resource. It has revisions; a configuration selects a specific revision as a version pin. The concept-resource / version-resource split that OSLC Configuration Management uses for all selected content applies directly to Parts. Parts are the *selected*.

If `oslc_plm:Part` subclasses `oslc_config:Configuration`, the Part inherits two incompatible versioning postures at once. As a Configuration it has no version axis — only its Stream / Baseline / ChangeSet variants, none of which is "a version of the Part." As a versioned domain resource it has a proper revision sequence indexed by version. These are different shapes of versioning living on the same resource.

The conflict shows up exactly when the Part is contributed to a parent Configuration through a PartUsage-as-Contribution. The protocol question is: *which version of the Part-as-Configuration is being contributed?* In ordinary OSLC Configuration Management a Contribution names a specific Configuration — a specific Stream or Baseline — and the URL itself disambiguates. But a versioned domain-resource Part has no Stream or Baseline of its own; it has revisions, and revisions are resolved against the parent Configuration's Configuration-Context. To pick the right revision of a Part-as-Configuration, a resolver would have to apply a Configuration-Context *to the Part-as-Configuration itself.*

That Configuration-Context cannot come from the Part. A Configuration *provides* a context; it does not receive one. It can only come from a containing Configuration — but in this scheme the containing Configuration is itself a Part-as-Configuration, which itself needs a Configuration-Context to be resolved, and so on up the tree. The recursion only terminates at a root Configuration with no parent, where there is no Configuration-Context to apply. Every Part-as-Configuration in the middle of the tree is left without a definite revision until the entire ancestry is resolved, and the root has no machinery to break the regress.

### What the regress is really telling us

The regress is the symptom of a deeper structural property: **the layer separation in OSLC Configuration Management is what makes versioning composable in the first place.** Configurations are not versioned because they are the apparatus that *defines* what versioning means for everything else. A Configuration says "in this context, the selected version of every concept resource is X." For a Configuration to itself be a versioned resource, a higher Configuration would have to specify which version of *it* is applicable in the current context. That higher Configuration would need a higher one in turn. The recursion can only terminate at a root with no parent — where, by construction, there is no Configuration-Context to disambiguate the root's own versioning.

OSLC Configuration Management cuts this knot by making Configurations *not* versioned in the version-resource sense. A Configuration is identified by its URL. Different Configurations of the same Component coexist as peer resources, not as versions of one Configuration. This is the property that lets a Global Configuration compose contributions from multiple Components without each Component needing an outer Configuration to pick which version of itself is the active one.

The subclassing proposal recaptures Teamcenter's unified meta-model at the vocabulary level but loses the property that made versioning composable in the first place. Teamcenter avoids the regress because its selectors (Revision Rule + Variant Rule + Effectivity context) are *not* themselves versioned, persistent, navigable resources — they are session/context overlays applied at traversal time. The proposal cannot replicate that escape hatch while also being an OSLC Configuration Management extension, because OSLC Configuration Mmanagement's selectors are persistent, navigable Configurations by definition.

### Relation to the mappings in the companion document

In the framing of `oslc-teamcenter-plm-mapping.md`, the subclassing proposal is a *more formal version of Mapping 1* — "PLM as configuration." Where Mapping 1 informally folds Items and Item Revisions into the configuration layer, the subclassing proposal makes the conflation explicit in the vocabulary by declaring it as an `rdfs:subClassOf` relationship. Subclassing makes the conflation look principled, but the structural problems are the same: the versioning regress surfaces the moment a consumer tries to pick a specific revision.

CDCM is sometimes raised at this point as a counter-example: it also extends OSLC Configuration Management with typed content that lives in the configuration layer, and works fine. The relevant difference is that CDCM Configuration Items are not versioned as separate domain resources and are not expected to be linked-to as cross-tool referenceable resources — they live entirely inside the configuration layer by deliberate scope (see the corresponding section in `oslc-teamcenter-plm-mapping.md`). The subclassing proposal does not have that scope guarantee: an `oslc_plm:Part` is *exactly* a versioned, externally referenceable domain resource — that is what makes it a Part rather than an organizational container. Forcing it to also be a Configuration is the source of the regress.

### Why this is not the reconciliation

The subclassing proposal is appealing because it appears to fuse the two layers the way Teamcenter does, while staying inside OSLC vocabulary. It does not survive contact with versioning. The fix is not to find a cleverer way to make the subclassing work; it is to accept the layer separation OSLC already enforces — selectors and selected things are distinct kinds of resource — and to add the missing PLM-specific selection criteria (effectivity, variability) to the *selector* layer, alongside `oslc_config:Selections` rather than under it. That is what the reconciliation in the next section does.

## Reconciliation: effectivity-based resolution as the bridge

The reconciliation is not to choose a camp but to give OSLC Configuration Management a way to express the *intensional*, effectivity-driven selection that Teamcenter performs natively, while keeping the selector an explicit, addressable resource.

A two-stage resolution mechanism does exactly this, modeled as additive subclasses of `oslc_config:Selections`: `oslc_plm:EffectivitySelections` (in the PLM domain, since only PLM resolves date/unit/serial effectivity) and `oslc:VariabilitySelections` (in OSLC core, since variant selection is a universal resource concern). With an `oslc_config.effectivity` query parameter that supplies an effectivity context and resolution of unbound selections to *effective* versions at query time (returning 404 for selections that are not effective in the requested context), an OSLC configuration gains the ability to carry a Teamcenter-style effectivity context and variant configuration and resolve late — without abandoning the requirement that the configuration remain an explicit, externally linkable resource. Each selection type is owned by the specification responsible for resolving it, which formalizes what each domain contributes to a configuration while leaving the core OSLC Configuration Management spec untouched.

That mechanism sits squarely in the selection band. It teaches OSLC's extensional model to behave intensionally when the domain demands it, which is precisely the capability the PLM integration was buying. Predicate language for effectivity itself — the runtime grammar for evaluating whether a usage is effective — is best deferred to a domain specification (with PLCS / ISO 10303-239 reference data as the vocabulary layer), since no common runtime predicate language exists across PLM vendors today.

## A caveat on the SysML side

Because SysML v2 sits in the definition and structure bands, it is tempting to treat it as a drop-in for Teamcenter's "defined things." It is not a perfect substitute. SysML's `variation`/`variant` covers the variability *axis* genuinely, but SysML has no native managed *effectivity* — there is no first-class dated, lot, or serial effectivity type of the kind PLCS/AP242 and Teamcenter provide. Effectivity conditions can be modeled as metadata or constraints, but the *managed runtime resolution* of variability against effectivity is absent. That gap is, once again, the selection band — which is why selection keeps surfacing as the true subject of the entire comparison.

## Conclusion: the relationship, stated cleanly

Teamcenter does not *overlap with* OSLC Configuration Management or SysML v2 so much as it *subsumes the layer each of them isolates*:

- **SysML v2 (as versioned `oslc_am` resources)** is the disciplined definition/structure layer — Teamcenter's Item and BOM, minus the lifecycle and configuration machinery. It is the *selected* content.
- **OSLC Configuration Management** is the disciplined selection layer — Teamcenter's revision/variant/effectivity rules, made explicit, externalized, and linkable across tools. It is the *selector*.
- **Teamcenter PLM** is the vertically integrated system that does all of it in one model, paying for that integration in interoperability, with the BOM line as the cell that is simultaneously defined content and selection input.

The two aspects are both valid because they are each describing the same integrated stack from a different perspective. And an effectivity-based resolution mechanism for OSLC Configuration Management is the construct that lets an OSLC-based, multi-tool lifecycle re-import the one capability that the integration was buying: intensional, effectivity-driven selection — expressed, this time, as an explicit and governable resource.
