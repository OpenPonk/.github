# OpenPonk Core Runtime Architecture & Repository Overview

This document defines the global architectural rules, runtime component layers, instance multiplicities, element lifecycles, and active repository map of the **OpenPonk** modeling platform (`OpenPonk` GitHub organization: 39 repositories total, comprising 28 active Pharo repositories, 4 non-Pharo infrastructure/web repositories, and 7 deprecated/archived repositories).

For the detailed three-level package/tag inventory, baseline groups, and bottom-up dependency layers across all 39 repositories, see [`repository_catalog.md`](./repository_catalog.md).

## 1. Global Architectural Rules

1. **Two-Level Package & Standalone `*-Tests` Package Organization:**
   * Pharo organizes code into **Packages** and **Package Tags** (2 levels).
   * **Standalone `*-Tests` Packages:** Every package that has unit or integration tests defines a dedicated companion `*-Tests` package (e.g., `OpenPonk-Core-Tests`, `OpenPonk-Spec-Tests`, `OpenPonk-Roassal-Tests`, `OpenPonk-Morphic-Tests`, `OpenPonk-DevTools-Tests`). Production packages must never contain `TestCase` subclasses, test-only helper classes, or tags containing the word `Test`.
   * **Mirrored Tag Naming Inside `*-Tests` Packages:** Inside `*-Tests` packages, the word `Tests` is **never** used in tag names. Instead, tags inside `*-Tests` mirror the tags of the corresponding production package (e.g., `Controllers`, `DiagramElements`, `UndoRedo`, `Persistence`, `Windows`, `Dialogs`, `Form`).
   * **SmalltalkCI Coverage Configuration (`.smalltalk.ston`):** While `#testing` runs all `OpenPonk*` (and `ChangesBuilder*`) packages, the `#coverage` section explicitly lists only production package names without wildcards (`*`), ensuring that test classes and abstract plugin test suites (`OPAbstractPluginExampleRoundtripTest`, `OPLegacyProjectLoadTest`, etc.) do not skew test coverage metrics.

2. **Repository-to-Package Tree Invariant ($1 : N$ Exclusive Ownership):**
   * Every Pharo package in OpenPonk belongs to **exactly one** GitHub repository. This invariant holds 100% across all active repositories and is continuously verified by `OPPackageBoundaryTest`.

3. **Metacello Baselines (`BaselineOf*`):**
   * Every active Pharo repository defines a corresponding `BaselineOf*` package, **except** for the `simulation` repository, which intentionally defines two separate baselines (`BaselineOfOpenPonkModelSimulation` and `BaselineOfOpenPonkSimulation`) to decouple pure domain model simulation from GUI controllers.
   * Every baseline loads its `*-Tests` packages automatically alongside its production packages in its standard groups (`default`, `plugin`, `dev-plugin`).
   * Most notation plugin baselines define two standard groups:
     * `plugin`: Loads only the plugin's own packages and test packages (plus domain/utility prerequisites) without pulling the `openponk` core. Used by `BaselineOfOpenPonkPlugins` when composing all-in-one builds.
     * `default`: Loads `plugin` plus the `openponk` core platform for standalone use.

4. **Flexible Metamodel Foundation:**
   * **Minimal Requirement (`#uuid`):** OpenPonk supports completely independent domain metamodels. Any Smalltalk object can serve as a domain model or model element; the only mandatory requirement imposed by the platform is implementing the `#uuid` method.
   * **Optional Bootstrap Metamodel (`openponk-model`):** Provides a lightweight optional base (`OPModelObject`, `OPDirectedAssociation`, etc.) offering `#uuid`, announcements, and common model functionality out of the box. Used directly or transitively (e.g., via `simulation` in `petrinets`) by most non-UML notations (BORM, BPMN, Petri Nets, FSM, Markov Chains, ERD, DEMO).
   * **UML & OntoUML Ecosystem (`uml-metamodel`):** Uses an independent, full OMG UML 2.5 implementation (`OpenPonk-UML-Meta` and `OpenPonk-UML-Metamodel`). **Crucial Rule:** Only `OpenPonk-UML-Metamodel` (and `OpenPonk-UML-Bootstrap`) is **generated code** produced by `uml-bootstrap-generator` and must **never** be modified manually without corresponding changes in the generator. Unlike `OpenPonk-UML-Metamodel`, `OpenPonk-OntoUML-Profile` is maintained as a regular package rather than locked generated code.

5. **Status Marker Definitions:**
   * *(unmarked)* – **Active:** Loaded in the standard all-in-one build.
   * `[outdated]` – **Outdated / Unloaded:** Temporarily excluded from the default build (`OpenPonk-BooleanColoredPetriNets` in `petrinets`), but retained in its baseline and potentially functional or slated for revival.
   * `[deprecated]` – **Deprecated:** Permanently abandoned or superseded (`Borm-OpenCabe-Importer` in `borm-editor`, `OP-UML-GTExtensions` in `uml-metamodel`, plus the 7 archived repositories). Classes from `[deprecated]` packages or repositories must never be imported, instantiated, subclassed, or referenced.
   * `[legacy / unmaintained]` – **Legacy / Unmaintained:** Historical unloaded extension packages in `class-editor` (`DynaCASE-Cormas`, `UMLClassEditor-Cormas`, `LiveClassEditor`, `LiveGenerator`, `LiveInstanceVisualization`, `OpenPonk-UML-DSL`, `UML-CodeGenerator`) whose current functionality and compatibility are unverified.

## 2. Core Runtime Architecture & Component Layers

OpenPonk separates project/domain data, model-level UI abstraction, visual diagram state, interactive canvas coordination, and Spec2 workbench GUI into distinct layers with strict responsibilities and lifecycles.

### 2.1 Project & Plugin Entry Points

* **Plugin (`OPPlugin`):** Each notation defines a subclass of `OPPlugin`, which registers the notation in OpenPonk and binds together its root model class, `OPDiagramController` class, ViewModel factory/navigation, and icon.
* **Project (`OPProject`):** Holds the project's root models and their associated diagrams via a dictionary mapping `{ rootModel -> { diagrams } }`.
  * A **root model** (e.g., a UML Package, an FSM, or a BPMN process) contains its own internal **model elements** (e.g., a UML Class inside a UML Package).
  * One root model can have multiple diagrams, whereas each diagram is associated with **exactly one** root model.
  * **Persistence:** `OPProject` is **persistent** (serialized indirectly: its metadata, root models, and diagrams are saved into the project archive/directory and `OPProject` is reconstructed upon loading).
* **Project Controller (`OPProjectController`):** Created when an `OPProject` is opened inside an `OPWorkbench` (or via `OPProjectController forProject: aProject` in headless tests). Manages the open project's lifecycle, `OPViewModelRegistry`, `OPUndoManager`, and holds the active `OPDiagramController` instances (one for each currently open diagram). Purely **transient**.

### 2.2 The Five Architectural Layers

1. **Domain Model (`Models` tag):**
   * **Role:** Represents pure domain concepts (root models and their contained model elements). Can be any arbitrary class implementing `#uuid` (such as `OpenPonk-UML-Metamodel` classes or subclasses of `OPModelObject` from `OpenPonk-Model`).
   * **Dependencies:** Completely decoupled from diagrams, Roassal shapes, controllers, and Spec2 GUI.
   * **Persistence:** **Persistent** (serialized when the project is saved).

2. **ViewModel (`ViewModels` tag):**
   * **Role:** Sits directly above the Domain Model. Because OpenPonk supports arbitrary domain metamodels, ViewModels provide a **unified API** over heterogeneous domain models.
   * **Responsibilities:**
     * Exposes structural relationships of the model (containment hierarchy, associations, parent/child elements) and property accessors/mutators (`#propertyNamed:`, `#propertyNamed:put:`, `#structuralPropertySelectors`).
     * Handles **creation and deletion** of domain model elements.
     * Defines diagram-independent GUI metadata, such as the **icon** and **unified display name/label** used in trees, lists, and forms.
   * **Primary GUI Consumers:** Diagram-independent GUI components—specifically the **Project Tree** and **Properties Form (`OPPropertiesForm` / `OPDynamicForm` declarative bindings)**—operate primarily on **ViewModels**, as they require a uniform representation of the model and its structure rather than diagram visuals.
   * **Persistence:** **Transient** (runtime-only). Created when a project is opened in an `OPWorkbench` and destroyed when the project is closed.

3. **DiagramElement / Shape (`DiagramElements` tag):**
   * **Role:** Represents visual elements belonging to a diagram (nodes, edges, labels) as well as the diagram itself.
   * **Hierarchy & Naming Convention:**
     * The base class `OPDiagramElement` forms a composite tree via `owningElement` ($0..1$) and `ownedElements` ($0..N$).
     * Concrete classes for individual elements typically end with the **`Shape` suffix** rather than `DiagramElement` (e.g., `OPUmlClassShape`).
     * Notable core `OPDiagramElement` subclasses are `OPEdge` and `OPNode`. `OPNode` has core subclass `OPBoundNode`. Notation-specific shape classes are usually subclasses of these 4. `OPBoundNode` has core subclass `OPDiagram`.
     * The root element representing the entire diagram is `OPDiagram` (specifically its subclass `OPOpenPonkDiagram` that includes OpenPonk-specific information like `isOpen`, `isSelected`, and `camera`).
   * **Roassal3 Encapsulation:**
     * While `OPDiagram` wraps a Roassal `RSCanvas` (`roassalShape` $0..1$), individual `OPDiagramElement` subclasses wrap an `RSShape` (`roassalShape` $0..1$).
     * `OPDiagramElement` subclasses—supported by extensions in `OpenPonk-Roassal`—encapsulate almost all Roassal-related complexity, holding persistent visual state (positions, bounds, polyline waypoints, styling) and translating model/diagram updates into incremental Roassal shape mutations.
   * **Lifecycle & Persistence:** **Persistent** (serialized as part of the diagram when saving a project). `DiagramElement` instances exist in memory for all diagrams in an open `OPProject` **regardless of whether a given diagram is currently open in an `OPEditor` or rendered on a Roassal canvas**.

4. **Controller (`Controllers` tag):**
   * **Role:** Sits above **ViewModels** and **DiagramElements**. Both `OPDiagramController` (for the diagram root) and `OPElementController` (for individual model/diagram elements) inherit from `OPController`.
   * **Responsibilities:**
     * An `OPDiagramController` is created for each diagram opened in an `OPEditor` and creates/manages the `OPElementController` instances for the elements displayed in that diagram.
     * Mediates interactive user operations on the Roassal canvas (palette tool actions, drag-and-drop, element creation/connection, selection) using announcement subscriptions and transactional `OPCommand`s via `OPUndoManager`, coordinating between the rendered `DiagramElement` (`Shape`) and its `ViewModel` / Domain Model.
   * **Persistence:** **Transient** (runtime-only). Controllers exist **only** for diagrams and diagram elements that are currently loaded and displayed on a Roassal canvas inside an active `OPEditor` (or headless test).

5. **Workbench & Editor GUI (`OpenPonk-Spec`):**
   * **`OPWorkbench`:** The main Spec2 window representing a single open project (backed by `OPProjectController`). Hosts project-wide views (**Project Tree**, **Properties Form** built on `OPDynamicForm` declarative bindings) that interact with **ViewModels**, and a tabbed area hosting $0..N$ **`OPEditor`** instances.
   * **`OPEditor`:** A single tab inside `OPWorkbench` dedicated to one open diagram (`OPOpenPonkDiagram`). Holds a reference to the diagram's `OPDiagramController`, the notation tool palette, diagram toolbar, and an **`OPCanvasPresenter`** (a subclass of `SpRoassalPresenter` that embeds the Roassal3 `RSCanvas` into Spec2). Operates on **Controllers** and **DiagramElements**.

### 2.3 Instance Multiplicities (Within One Open `OPWorkbench`)

**Key Single-Representation Invariant:** OpenPonk does **not** allow a single domain model element to be represented by multiple diagram elements within the **same** diagram. For any given pair of `(Domain Model element, Diagram)`—or `(Domain Model element, OPDiagramController)`—there exists at most **$1$ `DiagramElement` (`Shape`)** ($0..1$) and at most **$1$ `OPElementController`** ($0..1$).

| Source Entity | Target Entity | Multiplicity | Architectural Explanation |
| :--- | :--- | :--- | :--- |
| **`OPProject`** | **`OPProjectController`** | $1 : 0..1$ | $1$ `OPProjectController` when the project is open in an `OPWorkbench`; $0$ when closed/serialized. |
| **Root Domain Model** | **Diagram (`OPOpenPonkDiagram`)** | $1 : 0..N$ | In `OPProject`, one root model maps to $0..N$ diagrams; each diagram is associated with exactly $1$ root model. |
| **Domain Model** | **ViewModel** | $1 : 1$ | Exactly $1$ `ViewModel` exists per `Domain Model` element while the project is open in an `OPWorkbench` ($1 : 0$ when the project is closed). |
| **(Domain Model, Diagram)** | **DiagramElement (`Shape`)** | $1 : 0..1$ | Within a single diagram, a domain model element is represented by at most $1$ `DiagramElement` (either shown once in that diagram or not present in it). |
| **(Domain Model, `OPDiagramController`)** | **`OPElementController`** | $1 : 0..1$ | Within a single open diagram controller, a domain model element has at most $1$ `OPElementController`. |
| **Domain Model** | **DiagramElement (`Shape`)** *(across project)* | $1 : 0..N$ | Across all diagrams in a project, a single model element can appear in $0..N$ different diagrams (at most $1$ `Shape` per diagram). Conversely, a model-backed `DiagramElement` references $1$ model element. |
| **Diagram (`OPOpenPonkDiagram`)** | **`OPEditor` / `OPDiagramController`** | $1 : 0..1$ | A diagram (root `DiagramElement`) is either currently open in one editor tab with one `OPDiagramController` ($1$) or closed ($0$). |
| **`OPWorkbench`** | **`OPEditor`** | $1 : 0..N$ | A workbench displays $0..N$ editor tabs, bounded by the total number of diagrams in the project (at most $1$ `OPEditor` per diagram). |
| **DiagramElement (`Shape`)** | **Controller (`OPController`)** | $1 : 0..1$ | A controller exists ($1$) if and only if the diagram containing the `DiagramElement` is currently open in an `OPEditor`; otherwise $0$. |
| **ViewModel** | **Controller (`OPElementController`)** *(across project)* | $1 : 0..N$ | Follows transitively: $1$ `ViewModel` per `Model` $\rightarrow$ $0..N$ `DiagramElement`s across different diagrams $\rightarrow$ $0..1$ `Controller` per `DiagramElement` in open editors. |

### 2.4 Element Lifecycle: Closing, Hiding, and Removing

* **Closing an `OPEditor` Tab or Saving & Closing a Project:**
  * When an `OPEditor` tab is closed, its `OPDiagramController`, all its `OPElementController`s, and the Roassal canvas shapes (`RSCanvas` / `RSShape`) are destroyed, while the **DiagramElements**, **ViewModels**, and **Domain Models** remain in memory inside the project.
  * When the project is saved and the `OPWorkbench` is closed, **Domain Models** and **DiagramElements** persist in serialized form (`OPProject`), while `OPProjectController`, all **ViewModels**, and all **Controllers** are destroyed.
* **"Hide" (Removing an element from a diagram only):**
  * In OpenPonk terminology, removing a visual shape from a diagram without deleting the underlying domain concept is called **hiding**.
  * **Destroyed:** The `OPElementController` and the `DiagramElement` (`Shape`) in that specific diagram.
  * **Preserved:** The `ViewModel` and `Domain Model` (as well as any `DiagramElement`s and `OPElementController`s representing that same model element in other diagrams).
* **"Remove" / "Remove from Model" (Deleting the domain concept completely):**
  * Deletes the element from the domain model and cleans up all dependent representations across the entire project, regardless of the element's current visual state:
    1. *Element is displayed in one or more open diagrams:* Destroys the active `OPElementController`(s), all associated `DiagramElement`(s) (`Shape`s), the `ViewModel`, and the `Domain Model`.
    2. *Element is present in one or more closed (unloaded) diagrams:* No `OPElementController` exists for those closed diagrams; destroys all associated `DiagramElement`(s) in those diagrams, the `ViewModel`, and the `Domain Model`.
    3. *Element is not present in any diagram (exists only in the model / Project Tree):* Neither `OPElementController` nor `DiagramElement` exists; destroys the `ViewModel` and the `Domain Model`.

## 3. Compact Active Repository & Dependency Map (28 Pharo Repositories)

*(Every production package listed below has a companion `*-Tests` package unless noted in `repository_catalog.md`.)*

| Repository | Category | Production Package(s) | Internal Prerequisites | Role & Scope |
| :--- | :--- | :--- | :--- | :--- |
| **`openponk`** | Core & Platform | `OpenPonk-Roassal`, `OpenPonk-Core`, `OpenPonk-Morphic`, `OpenPonk-Spec`, `OpenPonk-DevTools` | `synchronized-links`, `zoomer` | Core platform, ViewModels, Controllers, Shapes, Spec2 GUI, Undo/Redo, persistence |
| **`openponk-model`** | Core & Platform | `OpenPonk-Model` | *(none in `plugin` group)* | Optional base domain metamodel (`OPModelObject`, `#uuid`, announcements) |
| **`plugins`** | Core & Platform | *(only `BaselineOfOpenPonkPlugins`)* | All notation plugins | Umbrella baseline assembling the all-in-one build |
| **`simulation`** | Core & Platform | `OpenPonk-ModelSimulation`, `OpenPonk-Simulation` | `openponk-model`, `openponk` | Discrete-event/step model simulator and GUI simulation bridge |
| **`openponk-git`** | Core & Platform | `OpenPonk-Git` | `openponk` | GitHub/GitLab project upload/download integration |
| **`class-editor`** | Notation Plugin | `OpenPonk-ClassEditor`, `OpenPonk-ClassEditor-Plugins` | `openponk`, `uml-xmi`, `uml-bootstrap-generator`, `uml-profiles` | UML Class Diagram editor, profile application UI, and profile generator wizard |
| **`borm-model`** | Notation Plugin | `OpenPonk-BormModel` | `openponk-model`, `synchronized-links` | BORM domain metamodel |
| **`borm-editor`** | Notation Plugin | `OpenPonk-BormEditor`, `OpenPonk-BormSimulation` | `borm-model`, `simulation`, `openponk` | BORM diagram editor, BormLang DSL (`PetitParser`), token simulator |
| **`BPMN`** | Notation Plugin | `OpenPonk-BPMN`, `OpenPonk-BPMN-XML`, `OpenPonk-BPMN-Validation`, `OpenPonk-BPMN-Simulation` | `openponk`, `openponk-model`, `simulation`, `xml-dom-visitor` | BPMN 2.0 editor, XML import/export, validation, and simulation |
| **`petrinets`** | Notation Plugin | `OpenPonk-PetriNets` | `openponk`, `simulation` | Place/Transition Petri nets editor and simulator |
| **`fsm-editor`** | Notation Plugin | `OpenPonk-FSM` | `openponk`, `openponk-model` | Finite-State Machine editor, recognizer, and GraphML serializer (`Mustache`) |
| **`markov-chains`** | Notation Plugin | `OpenPonk-MarkovChains` | `openponk`, `openponk-model` | Markov chains editor, simulator, and Zinc solver client (`Mustache`) |
| **`ERD`** | Notation Plugin | `OpenPonk-ERD` | `openponk`, `openponk-model` | Entity-Relationship Diagram editor and SQL/DDL generator (`Mustache`) |
| **`DEMO`** | Notation Plugin | `OpenPonk-DEMO` | `openponk`, `openponk-model` | DEMO enterprise ontology notation plugin |
| **`uml-metamodel`** | UML & OntoUML | `OpenPonk-UML-Meta`, `OpenPonk-UML-Metamodel` *(generated — locked)* | *(none)* | Meta-layer and generated OMG UML 2.5 metamodel |
| **`uml-bootstrap-generator`** | UML & OntoUML | `OpenPonk-UML-Bootstrap` *(locked)*, `OpenPonk-UML-BootstrapGenerator` | `xmi`, `pharo-changes-builder` | Generator for `OpenPonk-UML-Metamodel` and custom UML profile packages |
| **`uml-profiles`** | UML & OntoUML | `OpenPonk-UML-Profiles`, `OpenPonk-ClockProfile`, `OpenPonk-IssuesProfile` | `uml-metamodel` | UML profile registration, stereotype mechanism, and example profiles |
| **`uml-xmi`** | UML & OntoUML | `OpenPonk-UML-XMI` | `xmi`, `uml-metamodel` | XMI serialization/deserialization for UML models and profiles |
| **`ontouml-profile`** | UML & OntoUML | `OpenPonk-OntoUML-Profile`, `OpenPonk-OntoUML-Plugin` | `class-editor` | OntoUML stereotype profile and Class Editor UI plugin |
| **`ontouml-modelquery`** | UML & OntoUML | `OpenPonk-OntoUML-ModelQuery` | `ontouml-profile` | Query utilities for inspecting/filtering OntoUML models |
| **`ontouml-patterns`** | UML & OntoUML | `OpenPonk-OntoUML-Patterns` | `ontouml-profile` | OntoUML ontology design patterns catalog and wizards |
| **`ontouml-verifications`** | UML & OntoUML | `OpenPonk-OntoUML-Verifications` | `ontouml-modelquery` | OntoUML verification rules and anti-pattern detectors (`PetitParser`) |
| **`openponk-ontouml-to-uml-transformation`** | UML & OntoUML | `OpenPonk-OntoUML-Transformations` | `ontouml-modelquery` | Model-to-model transformations from OntoUML to implementation UML |
| **`synchronized-links`** | Shared Library | `OpenPonk-SynchronizedLinks` | *(none)* | Self-synchronizing bidirectional object references |
| **`zoomer`** | Shared Library | `OpenPonk-Zoomer` | *(none)* | World and canvas zoom scaling widget |
| **`xmi`** | Shared Library | `OpenPonk-XMI`, `OpenPonk-XMI-Inspection` | `xml-dom-visitor` | Generic XMI file to object-graph reader/writer (`XMLWriter`) |
| **`xml-dom-visitor`** | Shared Library | `OpenPonk-XMLDOMVisitor` | *(none; `XMLParser`)* | Visitor double-dispatch and typed wrappers over `XMLParser` DOM trees |
| **`pharo-changes-builder`** | Shared Library | `ChangesBuilder` | *(none)* | Wrapper over `RBRefactoring` for programmatic code generation |

### 3.1 Third-Party External Dependencies

| External Library | Role in OpenPonk | Consumed By (OpenPonk Baselines) |
| :--- | :--- | :--- |
| **`Roassal`** (Roassal3) | Canvas and vector graphics rendering engine | `openponk` (`OpenPonk-Roassal`) |
| **`RoassalExporters`** | Diagram export to image/vector formats | `openponk` (`OpenPonk-Roassal`) |
| **`Magritte`** | Legacy baseline dependency (Workbench property forms now use `OPDynamicForm` declarative bindings instead of Magritte descriptors) | `openponk` (`OpenPonk-Core`, `OpenPonk-Spec`) |
| **`Mocketry`** | Mocking framework for unit tests | `openponk` (`OpenPonk-Core`) |
| **`XPath`** | XML path querying | `openponk` (`OpenPonk-Core`) |
| **`NeoJSON`** | JSON serialization for projects and diagrams | `openponk` (`OpenPonk-Core`) |
| **`XMLParser`** | DOM XML parsing | `xml-dom-visitor` (`OpenPonk-XMLDOMVisitor`) |
| **`XMLWriter`** | XML stream writing | `xmi` (`OpenPonk-XMI`) |
| **`PetitParser`** | Parser combinators for DSLs and expressions | `borm-editor`, `ontouml-verifications` |
| **`Mustache`** | Template engine for code/file export | `fsm-editor`, `markov-chains`, `ERD` |