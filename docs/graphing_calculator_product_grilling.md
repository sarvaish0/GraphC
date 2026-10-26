# Graphing Calculator Website --- Product Grilling & Decision Log

**Document purpose:** Consolidate the product idea, decisions made
during the grilling session, unresolved questions, contradictions, and
technical risks into one working reference.

**Status:** Working specification --- not yet a final PRD or
implementation plan.

------------------------------------------------------------------------

## 1. Product at a glance

### 1.1 Product concept

A browser-based mathematical graphing calculator for college students.
Users should be able to enter a mathematical expression and see its
graph quickly, then optionally open advanced controls to define the
function more formally and inspect mathematical properties.

The product aims to combine a low-friction graphing workflow with
explicit control over mathematical definitions and honest reporting of
what the engine can and cannot determine.

### 1.2 Core user workflow

1.  Enter an expression, for example `f(x) = x^2`.
2.  See the graph immediately.
3.  Optionally open advanced settings to define the domain, codomain,
    parameters, and other constraints.
4.  Inspect points and, where mathematically defined and supported,
    tangent information.
5.  Compare multiple graphs.
6.  Save a graph or share a link that reconstructs its state.

### 1.3 Target audience

-   **Primary audience:** College students.
-   **Primary need:** Explore functions, compare graphs, and understand
    how constraints affect a function's graph.
-   **Important product quality:** Mathematical correctness takes
    priority over adding many features.

### 1.4 Differentiation hypothesis

The initial hypothesis is that the product can offer more explicit
control over mathematical definitions than familiar graphing
calculators, especially by treating **domain and codomain as first-class
properties** rather than merely plotting controls.

This is a hypothesis to validate, not an established competitive
advantage. Existing tools such as Desmos and GeoGebra already support
substantial graphing, restrictions, and 3D capabilities. The product
must identify a more specific unmet need and test it with actual
students.

------------------------------------------------------------------------

## 2. Decisions made so far

The labels below distinguish decisions from recommendations and open
questions.

### 2.1 Product scope

  -----------------------------------------------------------------------
  Area                    Current decision        Notes
  ----------------------- ----------------------- -----------------------
  Target audience         College students        The initial audience;
                                                  narrower personas still
                                                  need validation.

  Product priority        Mathematical            Prefer a smaller
                          correctness             trustworthy engine to a
                                                  broad but misleading
                                                  one.

  Interaction model       Immediate graphing with Fast path first; formal
                          optional advanced       controls should not
                          settings                obstruct ordinary use.

  2D graph types for v1   Explicit functions      Mandatory initial graph
                          `y = f(x)`              type.

  3D graph types for v1   Explicit surfaces       Mandatory initial 3D
                          `z = f(x,y)`            type.

  Parametric curves       Defer                   Not part of the initial
                                                  mandatory scope.

  Implicit                Defer                   Not part of the initial
  equations/curves                                mandatory scope.

  Multiple graphs         Support                 General intersections
                          overlay/comparison of   and cross-type
                          explicit 2D functions   comparisons are
                                                  deferred.

  Domain controls         Automatic analysis plus Inferred mathematical
                          user-declared           validity and the user's
                          constraints             declared domain must
                                                  remain distinct.

  Codomain                Represent explicitly    Do not conflate
                                                  codomain with range.

  Range                   Calculate exactly when  Estimates must be
                          supported; otherwise    labelled and scoped.
                          estimate                

  2D discrete functions   Infer point rendering   Integer-domain
                          from a discrete         functions should render
                          declared domain         as isolated points by
                                                  default, not as a
                                                  connected curve.

  Rendering override      Recommended: `auto`,    Visualization style
                          `curve`, `points`,      must not silently
                          `both`                  change the mathematical
                                                  domain.

  3D domain               Start with rectangular  Arbitrary regions and
                          bounds for `(x,y)`      region editors are
                                                  deferred.

  Persistence             Backend for accounts,   Ordinary plotting
                          saving, and sharing     should run in the
                                                  browser without a
                                                  per-plot backend
                                                  request.

  Sharing                 Shareable state/URL     Exact encoding and
                                                  persistence strategy
                                                  remain open.

  3D motion               Optional idle           Pause during user
                          auto-rotation           interaction; allow
                                                  speed control and
                                                  resume.

  Expression evaluation   Use a parsed expression Do not use JavaScript
                          grammar/AST             `eval()` or
                                                  `Function()` to
                                                  evaluate user
                                                  expressions.

  Numerical rendering     Adaptive sampling with  Balanced
                          explicit uncertainty    performance/accuracy
                                                  trade-off.

  Unresolved regions      Render valid regions    Do not draw a
                          and indicate unresolved connection when the
                          regions                 engine cannot justify
                                                  it.
  -----------------------------------------------------------------------

### 2.2 Function-definition semantics

The interface and engine should keep these concepts separate:

-   **Domain:** Inputs the function is declared to accept.
-   **Expression-validity domain:** Inputs for which the expression can
    be evaluated under the engine's supported real-valued semantics.
-   **Codomain:** The declared set of allowed output values.
-   **Range/image:** Values actually attained by the function over its
    declared domain.
-   **Viewport:** The portion of the graph currently visible.
-   **Rendering mode:** How valid mathematical data is displayed, such
    as a continuous curve or isolated points.

These are related, but they are not interchangeable.

**Example:** If a user declares the domain of `f(x) = sqrt(x)` as all
real numbers, the engine should not silently replace the declared domain
with `[0, ∞)`. It should report that the expression is not real-valued
for part of the declared domain, then plot only valid portions and make
the incompatibility visible.

### 2.3 Domain and discrete rendering

-   Domains must support more than numeric minimum and maximum values.
-   The conceptual model includes continuous intervals, integer domains,
    natural-number domains, unions, and expression-defined restrictions.
-   Interval endpoints need inclusive/exclusive semantics.
-   Natural-number semantics must be explicitly defined: does `N`
    include zero or begin at one?
-   For an integer-domain function such as `f(x)=x^2` with `x ∈ Z`, the
    default display should be isolated points.
-   A separate rendering override may show sampled points for a
    continuous-domain function without changing its mathematical domain.

### 2.4 Mathematical analysis

-   Derivatives and tangents are initially focused on supported explicit
    2D functions.
-   Tangent information should only be presented where the
    derivative/tangent is defined and the method is supported.
-   Corners, discontinuities, and vertical tangents must not be
    presented as ordinary finite-slope tangents.
-   Symbolic differentiation or automatic differentiation may be used
    where supported; numerical differentiation is a fallback with known
    limitations.
-   Intersections should initially focus on explicit 2D functions, with
    a defined search interval and numerical tolerances.
-   Root-finding must account for the fact that sign-change methods can
    miss tangential roots and may find duplicate or multiple roots.
-   Exact range or intersection claims should only be labelled exact
    when a supported method establishes them.

### 2.5 Persistence and architecture

The plotting engine should be browser-first. A backend is primarily
needed for accounts, saved graphs, and sharing---not for every
expression evaluation or redraw.

A graph's saved state should preserve at least:

-   Original expression and function name.
-   Declared domain and codomain.
-   Parameter values and slider configuration.
-   Viewport and visualization settings.
-   Relevant 3D camera/rotation settings.
-   Schema/version information so saved graphs can be migrated as the
    product evolves.

Analysis results such as derivatives, range estimates, and intersections
should be versioned or cached carefully and invalidated when relevant
inputs change. Saved analysis should not be assumed valid after an
expression, parameter, or domain changes.

------------------------------------------------------------------------

## 3. Numerical rendering decisions and proposed rules

### 3.1 Chosen direction

The current direction for rendering is **adaptive sampling with explicit
uncertainty**, balancing accuracy and responsiveness. If the engine
cannot resolve a region within its limits, it should preserve valid
regions and indicate the unresolved part rather than draw a misleading
connection.

### 3.2 Proposed 2D sampling pipeline

1.  Establish the viewport and convert between data coordinates and
    screen coordinates.
2.  Create initial sample intervals across the visible x-range.
3.  Evaluate samples and classify results, including finite values,
    invalid real-domain evaluations, and evaluation errors.
4.  Estimate local rendering error in screen/pixel space.
5.  Refine intervals selectively where the estimated error or risk is
    high.
6.  Stop when the error criterion is met or a configured computation
    budget is exhausted.
7.  Render connected segments only when the connection is sufficiently
    justified.
8.  Render gaps or uncertainty indicators where the interval is invalid,
    suspected discontinuous, or under-resolved.

A starting pixel-error target around **0.75--1.5 pixels** was discussed
as a benchmark to test, not as a universal correctness guarantee.

### 3.3 Important limitation

Midpoint deviation or curvature checks alone cannot prove that a curve
is fully resolved. A highly oscillatory function can vary substantially
between sampled points while appearing deceptively smooth at the sample
locations.

Potential refinement triggers include:

-   Large screen-space deviation from a straight segment.
-   Rapid changes in slope or curvature.
-   Non-finite results or evaluation errors.
-   Large screen-space jumps.
-   Expression-structure hazards, such as a denominator that may become
    zero.
-   Suspected domain boundaries or singularities.
-   A sudden change in validity or local behavior.

These checks reduce risk but cannot guarantee detection of every
discontinuity or feature for arbitrary expressions.

### 3.4 Hard resource limits

The engine needs explicit limits for:

-   Function evaluations.
-   Maximum refinement depth.
-   Memory use.
-   Time spent on a graph update.
-   Root-finding and subexpression boundary searches.

If a limit is reached before a region is resolved, the engine should
mark that region **under-resolved**. It must not label the region
accurate merely because the computation stopped.

### 3.5 Examples that the renderer must handle

  -----------------------------------------------------------------------
  Expression or domain                Expected behavior
  ----------------------------------- -----------------------------------
  `x^2`                               Ordinary continuous curve.

  `1/x`                               Separate branches; never bridge
                                      across the singularity at `x=0`
                                      when identified.

  `sqrt(x)`                           No real-valued curve for `x<0`.

  `log(x)`                            No real-valued curve for `x<=0`.

  `(x^2-1)/(x-1)`                     Preserve the hole at `x=1`; do not
                                      automatically fill it merely
                                      because the expression simplifies
                                      elsewhere.

  `abs(x)` or `|x|`                   Continuous curve with a corner at
                                      zero; derivative is not defined
                                      there.

  `sin(1000x)`                        Refine in response to
                                      high-frequency variation or
                                      explicitly report under-resolution.

  `sin(1/x)` for `x>0`                Do not claim the oscillations near
                                      zero are fully rendered; expose
                                      unresolved regions when finite
                                      sampling cannot resolve them.

  `x^2`, with `x ∈ Z`                 Isolated points by default, not a
                                      connected curve.
  -----------------------------------------------------------------------

### 3.6 3D surface safeguards

3D surfaces need a separate mesh/patch algorithm; the 2D sampling
algorithm should not simply be reused without considering topology.

The engine must account for:

-   Invalid vertices.
-   Faces or triangles that cross invalid regions.
-   Extremely large or distorted triangles.
-   Under-resolved patches.
-   Surface clipping to valid regions.
-   Viewport-dependent detail and resource limits.

Removing invalid vertices alone is insufficient: remaining mesh faces
can still bridge a hole or invalid region.

Representative tests:

  -----------------------------------------------------------------------
  Surface                             Expected behavior
  ----------------------------------- -----------------------------------
  `z=x^2+y^2`                         Smooth, correctly shaped surface.

  `z=1/(x^2+y^2)`                     Do not bridge the singular origin.

  `z=sqrt(x^2+y^2-1)`                 Exclude the region where the
                                      radicand is negative.

  `z=sin(10x)sin(10y)`                Increase sampling where surface
                                      variation demands it, or mark
                                      unresolved patches.

  `z=sqrt(1-x^2-y^2)`                 Clip the surface to the valid disk.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 4. Contradictions, tensions, and unresolved decisions

This section is the most important output of the grilling: these items
need explicit decisions or validation before implementation.

### 4.1 Product and scope questions

#### A. Is the differentiator sufficiently distinct?

**Tension:** The product wants to differentiate through formal
domain/codomain controls, but established graphing tools already support
restrictions, sliders, and 3D plotting.

**Decision still needed:** - Which specific task do students struggle to
complete in existing tools? - Is codomain validation a frequent enough
problem to motivate adoption? - Is the target user learning formal
function theory, exploring graphs, or doing engineering/scientific
plotting?

**Recommended next step:** Interview a small set of students and ask
them to complete real tasks in existing tools. Observe friction instead
of asking only whether they like the idea.

#### B. What does "all common expressions" mean?

**Tension:** Broad expression support conflicts with a correctness-first
engine and finite implementation resources.

**Decision still needed:** Publish a supported grammar for v1, including
operators, precedence, constants, functions, implicit multiplication,
function composition, absolute value, piecewise expressions, and error
behavior.

**Recommended principle:** Never advertise universal or arbitrary
expression support unless the actual parser and semantics justify it.

#### C. How much of the analysis belongs in v1?

**Tension:** Derivatives, tangents, range analysis, intersections,
domain inference, 3D rendering, accounts, sharing, and uncertainty
reporting are all substantial systems.

**Decision still needed:** Which analysis features are required for the
first usable release, and which can be added after core plotting is
trustworthy?

**Recommended principle:** Make parsing, domain-aware plotting, discrete
rendering, and safe adaptive rendering the foundation. Treat advanced
analysis as separate capabilities with explicit support levels.

### 4.2 Mathematical semantics questions

#### D. What exactly happens when the declared domain conflicts with expression validity?

A declared domain and an expression's real-valued validity are distinct.
The engine should not silently rewrite the user's mathematical
declaration.

**Decision still needed:** Choose the UI behavior when the expression is
invalid on part of the declared domain. Options include a warning plus
partial plot, an explicit incompatible-domain state, or a prompt to
revise the declaration.

**Current direction:** Keep the declaration, report the incompatibility,
plot only valid portions, and clearly show the omitted portions. The
exact UI remains undecided.

#### E. What is the convention for natural numbers?

**Decision still needed:** Does `N` mean `{0,1,2,...}` or `{1,2,3,...}`?

**Recommendation:** Make the convention visible in settings/help and
avoid silently relying on a universal definition.

#### F. How should codomain violations be reported?

A numerical sample outside the declared codomain may demonstrate a
violation, but a finite set of samples inside the codomain cannot prove
that all outputs comply.

**Current direction:** Distinguish: - **Proven violation:** A supported
analysis establishes an output outside the declared codomain. -
**Validated within supported analysis:** The supported method checked
the relevant scope, with its limitations disclosed. - **Unknown:** The
engine cannot establish compliance.

**Decision still needed:** Define which methods qualify for each status
and how those statuses are worded in the UI.

#### G. How should range results be represented?

**Current direction:** Return exact results only when supported analytic
methods establish them; otherwise return a numerical estimate.

**Decision still needed:** Define: - Whether the estimate applies to the
full declared domain or only the visible viewport. - How
confidence/limitations are communicated without suggesting a proof. -
How to handle unbounded ranges and ranges with excluded values. -
Whether the UI displays a set/interval or just estimated extrema.

**Critical distinction:** A range estimated from visible samples is not
the mathematical range over the entire domain.

#### H. Can a declared continuous domain be rendered as points?

This is a display question, not a domain question.

**Current direction:** Yes, through a separate rendering override. A
point-style display must not turn a continuous function into a discrete
function mathematically.

**Decision still needed:** Define point sampling density, whether
endpoints are included, and how the point display responds to zooming.

### 4.3 Numerical correctness questions

#### I. What does "resolved" guarantee?

**Tension:** Adaptive sampling can estimate geometric error, but it
cannot prove that arbitrary functions contain no hidden oscillations or
singularities between samples.

**Decision still needed:** Specify a bounded operational guarantee
rather than claiming perfect graph reconstruction.

**Proposed conservative rule:** A segment may be connected only when: -
Its endpoints and required interior probes are valid and inside the
declared domain. - The estimated screen-space error is within the
configured tolerance. - No known expression boundary or singularity lies
across it. - No risk trigger remains unresolved. - The computation
budget has not forced an unresolved result.

If any condition remains unknown when the budget is exhausted, do not
connect across that interval.

This rule is conservative but still does not guarantee that all hazards
will be detected.

#### J. How should uncertainty be represented internally?

A single mutually exclusive status enum is probably insufficient because
domain membership, evaluation validity, sampling resolution, and
connection eligibility are different dimensions.

**Recommended model:** Track these independently:

-   **Domain membership:** inside / outside / unknown.
-   **Evaluation validity:** valid / invalid / unknown.
-   **Sampling resolution:** resolved / under-resolved.
-   **Topology decision:** connect / break / unknown.

The UI can combine these into understandable labels such as "valid,"
"domain excluded," "possible discontinuity," or "under-resolved."

**Decision still needed:** Finalize internal types, transition rules,
and UI labels.

#### K. What is the acceptable performance budget?

**Tension:** More refinement improves visible fidelity but can increase
latency and CPU usage, especially with expensive expressions or multiple
graphs.

**Decision still needed:** Establish evaluation/time budgets and
benchmark them on target devices. The budget should be measured rather
than selected by intuition alone.

#### L. What is the policy for zooming and resizing?

**Current direction:** Zooming in should trigger more local sampling;
zooming out should avoid wasting computation. Expression or parameter
changes should invalidate relevant cached results.

**Decision still needed:** Decide whether resize preserves the data
viewport exactly, preserves scale, or fits the graph automatically.
Define cache keys and cancellation behavior for stale graph updates.

### 4.4 3D questions

#### M. What does "3D graphing" guarantee in v1?

**Current scope:** Surfaces of the form `z=f(x,y)`, initially over
rectangular bounds.

**Decision still needed:** Define the mesh resolution strategy, clipping
behavior, camera controls, touch interaction, rotation speed, and the
visual representation of unresolved patches.

**Boundary:** Parametric surfaces, implicit 3D surfaces, arbitrary
domain-region editors, and general 3D intersections remain out of the
initial scope unless deliberately re-added.

### 4.5 Architecture and persistence questions

#### N. What state goes into a shareable link?

**Current direction:** Share links should reconstruct graph state.

**Decision still needed:** Choose between: - Encoding state in the
URL. - Saving a server-side graph and linking to an identifier. - A
hybrid approach.

Also decide what happens with very long expressions, private graphs,
deleted graphs, schema migrations, and incompatible future versions.

#### O. What is the minimal account/backend requirement?

**Current direction:** Browser-first plotting; backend for saving,
accounts, and sharing.

**Decision still needed:** Determine whether guest/local-only use is
supported, whether accounts are required to save, and what access
control applies to shared graphs.

#### P. How should analysis caching work?

**Current direction:** Cache or version analysis results and invalidate
them when relevant inputs change.

**Decision still needed:** Define cache dependencies. At minimum,
results may depend on expression/AST version, parameters, declared
domain/codomain, analysis method/version, and possibly viewport when the
result is viewport-scoped.

------------------------------------------------------------------------

## 5. Proposed status model for graph regions

This is a recommendation to resolve a design problem, not yet a
finalized implementation contract.

Instead of a single status enum, represent each interval (2D) or
patch/cell (3D) using orthogonal facts:

``` text
RegionAssessment
  domainMembership: inside | outside | unknown
  evaluationValidity: valid | invalid | unknown
  samplingResolution: resolved | under-resolved
  connectionPolicy: connect | break | unknown
  estimatedScreenError?: number
  reasons: Diagnostic[]
```

A diagnostic should have a stable code, severity, and user-facing
message. Examples include `DOMAIN_EXCLUDED`, `NON_REAL_RESULT`,
`POSSIBLE_POLE`, `SAMPLE_BUDGET_EXHAUSTED`, and
`GEOMETRY_ERROR_TOO_HIGH`.

### Rendering policy

-   **Connect:** Only when validity, domain membership, resolution, and
    hazard checks satisfy the rendering rules.
-   **Break:** When the region is outside the declared domain, known
    invalid, or crosses a known boundary that must not be bridged.
-   **Unknown:** When evidence is insufficient. Refine if budget
    remains; otherwise show a gap or uncertainty indicator.

This model prevents a common design mistake: treating "invalid,"
"excluded," and "not sufficiently sampled" as if they meant the same
thing.

------------------------------------------------------------------------

## 6. Proposed test and acceptance checklist

These are suggested acceptance criteria to refine, not a claim that they
are already implemented.

### 6.1 Expression and domain handling

-   [ ] Supported syntax and precedence are documented.
-   [ ] Unsafe JavaScript evaluation mechanisms are not used.
-   [ ] Invalid syntax produces a useful diagnostic.
-   [ ] Real-domain restrictions are identified for supported
    operations.
-   [ ] Declared domain and expression-validity domain remain distinct.
-   [ ] Integer/natural domains render as isolated points by default.
-   [ ] Natural-number convention is explicit.
-   [ ] Viewport limits are never confused with domain limits.

### 6.2 2D rendering

-   [ ] Valid continuous regions are rendered.
-   [ ] Known-invalid regions are not plotted as real curves.
-   [ ] Known singularities and holes are not bridged.
-   [ ] Adaptive sampling responds to screen-space error.
-   [ ] High-frequency examples either resolve within budget or show
    uncertainty.
-   [ ] Computation budgets are enforced.
-   [ ] Budget exhaustion cannot silently produce a false "resolved"
    status.
-   [ ] Zooming, resizing, and parameter changes have defined behavior.
-   [ ] Results are reproducible under the same expression, settings,
    and engine version.

### 6.3 3D rendering

-   [ ] Invalid regions are clipped.
-   [ ] Mesh faces do not knowingly bridge invalid regions.
-   [ ] High-variation regions trigger refinement or uncertainty.
-   [ ] Camera and auto-rotation controls behave predictably.
-   [ ] Touch interaction is supported.
-   [ ] Resource limits apply to mesh generation and updates.

### 6.4 Analysis and claims

-   [ ] Exact results are only labelled exact when established by a
    supported method.
-   [ ] Numerical estimates state their scope.
-   [ ] Derivative/tangent failures at corners or discontinuities are
    handled explicitly.
-   [ ] Intersection search intervals and tolerances are documented.
-   [ ] Analysis results are invalidated when their dependencies change.

### 6.5 Persistence

-   [ ] Saved graphs preserve expressions and declared mathematical
    settings.
-   [ ] Saved graphs preserve parameters and visualization state.
-   [ ] Shared state can be reconstructed.
-   [ ] Saved data is versioned.
-   [ ] Stale analysis is recomputed or clearly marked stale.

------------------------------------------------------------------------

## 7. Recommended implementation order

This sequence minimizes the risk of building advanced features on an
unreliable foundation.

1.  **Specify expression grammar and semantics.** Define operators,
    functions, constants, precedence, implicit multiplication, error
    handling, and supported real-valued behavior.
2.  **Build the parser and safe evaluator.** Use an AST and explicit
    evaluation outcomes; do not use `eval()` or `Function()`.
3.  **Implement domain representation.** Support continuous intervals,
    discrete domains, unions, and explicit constraints needed for v1.
4.  **Build a basic 2D renderer.** Start with fixed sampling, segment
    topology, invalid-value handling, and point rendering for discrete
    domains.
5.  **Add adaptive sampling and region assessment.** Introduce error
    estimates, hazards, budgets, and unresolved-region display.
6.  **Create a numerical regression corpus.** Include all examples in
    this document and test after every renderer change.
7.  **Add domain/codomain diagnostics and range reporting.** Expose only
    the analysis levels the engine can justify.
8.  **Build the 3D surface renderer separately.** Use mesh/patch
    classification and topology safeguards.
9.  **Add derivatives/tangents and 2D intersections.** Keep support
    boundaries and tolerances explicit.
10. **Add persistence and sharing.** Save versioned graph definitions
    and visualization state.
11. **Polish interaction and accessibility.** Keyboard controls,
    touch-friendly sliders/camera controls, reduced-motion behavior, and
    clear uncertainty explanations.

This is a proposed sequence. If the product's validated differentiator
depends on codomain validation, that analysis may need to move
earlier---but only after its scope and correctness criteria are defined.

------------------------------------------------------------------------

## 8. Honest product guarantees

The product should not promise more than its mathematical and numerical
methods support.

### Reasonable guarantees to aim for

-   The engine will not intentionally connect regions it has explicitly
    classified as invalid, excluded, or unresolved.
-   Known expression constraints will be surfaced rather than silently
    hidden.
-   Exact and estimated analysis results will be distinguished.
-   Computation budgets will be enforced and budget exhaustion will be
    visible.
-   Declared domain, codomain, range, viewport, and rendering mode will
    be kept conceptually separate.

### Guarantees not justified for arbitrary expressions

-   Detecting every discontinuity.
-   Rendering every feature of every curve.
-   Proving a graph is complete based on finite samples.
-   Computing the exact range for every function.
-   Computing every derivative or intersection.
-   Guaranteeing 3D surface accuracy merely because a mesh looks smooth.

The distinction is important: a visually plausible graph is not
automatically a mathematically complete graph.

------------------------------------------------------------------------

## 9. Immediate decisions to make next

The following are the highest-value unresolved decisions. Resolve these
before turning this document into a final PRD.

1.  **Differentiator validation:** Which concrete student task is
    meaningfully better than using existing tools?
2.  **V1 expression grammar:** Exactly which syntax and functions are
    supported?
3.  **V1 analysis scope:** Which of domain diagnostics, codomain
    validation, range estimation, derivatives, tangents, and
    intersections ship in the first release?
4.  **Natural-number convention:** Does `N` include zero?
5.  **Conflict UI:** How does the interface present a declared domain
    that includes inputs where the expression is invalid?
6.  **Uncertainty contract:** Finalize the region-assessment dimensions
    and connection policy.
7.  **Performance targets:** Set and benchmark evaluation/time budgets
    on representative devices.
8.  **Share model:** URL-encoded state, server-side saved graphs, or
    hybrid?
9.  **3D behavior:** Define mesh-quality limits, invalid-patch display,
    camera controls, and auto-rotation behavior.
10. **User validation:** Test the proposed workflow with students before
    committing to the full feature set.

------------------------------------------------------------------------

## 10. Decision log conventions for future grilling

As the product is challenged further, record each item as one of:

-   **DECIDED:** Explicitly accepted by the product owner.
-   **PROPOSED:** Recommended but not yet accepted.
-   **OPEN:** Requires a decision or evidence.
-   **RISK:** Could invalidate the design or create a
    correctness/performance problem.
-   **DEFERRED:** Intentionally excluded from the current scope.

Do not promote a recommendation to a decision merely because it appears
in this document. Revisit open items after user testing and technical
prototypes.
