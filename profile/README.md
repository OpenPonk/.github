# OpenPonk Architecture, Repository, and Package Reference

This document provides a comprehensive reference of the runtime architecture, repositories, packages, and dependency graph of the **OpenPonk** modeling platform (`OpenPonk` GitHub organization: 39 repositories total, comprising 28 active Pharo repositories, 4 non-Pharo infrastructure/web repositories, and 7 deprecated/archived repositories).

## 1. Global Architectural Rules & LLM Guidelines

1. **Target Environment & Pharo Idioms:**

   * **Environment:** Pharo Smalltalk.

   * **Class Definitions:** Use fluid class definition syntax with separate `tag:` and `package:` setters (never merged into a single string), terminated with `install`.

   * **Package Queries:** Always use `self packageOrganizer` (never `RPackageOrganizer default`).

2. **Repository-to-Package Tree Invariant (**$1 : N$ **Exclusive Ownership):**

   * Every Pharo package in OpenPonk belongs to **exactly one** GitHub repository. This invariant holds 100% across all active repositories.

3. **Two-Level Package & Test Organization:**

   * Pharo organizes code into **Packages** and **Package Tags** (2 levels).

   * **No Standalone `-Tests` Packages:** Contrary to standard Pharo convention, OpenPonk does **not** use separate `*-Tests` packages. All unit and integration tests reside directly inside their main package under the `Tests` tag (or specialized tags such as `*-Tests` or `Tests - *`).

4. **Metacello Baselines (`BaselineOf*`):**

   * Every active Pharo repository defines a corresponding `BaselineOf*` package. These packages are omitted from the explicit package lists below, **except** for the `simulation` repository, which intentionally defines two separate baselines (`BaselineOfOpenPonkModelSimulation` and `BaselineOfOpenPonkSimulation`) to decouple pure domain model simulation from GUI controllers.

   * Most notation plugin baselines define two standard groups:

     * `plugin`: Loads only the plugin's own packages (and domain/utility prerequisites) without pulling the `openponk` core. Used by `BaselineOfOpenPonkPlugins` when composing all-in-one builds.

     * `default`: Loads `plugin` plus the `openponk` core platform for standalone use.

5. **Flexible Metamodel Foundation:**

   * **Minimal Requirement (`#uuid`):** OpenPonk supports completely independent domain metamodels. Any Smalltalk object can serve as a domain model or model element; the only mandatory requirement imposed by the platform is implementing the `#uuid` method.

   * **Optional Bootstrap Metamodel (`openponk-model`):** Provides a lightweight optional base (`OPModelObject`, `OPDirectedAssociation`, etc.) offering `#uuid`, announcements, and common model functionality out of the box. Used directly or transitively (e.g., via `simulation` in `petrinets`) by most non-UML notations (BORM, BPMN, Petri Nets, FSM, Markov Chains, ERD, DEMO).

   * **UML & OntoUML Ecosystem (`uml-metamodel`):** Uses an independent, full OMG UML 2.5 implementation (`OpenPonk-UML-Meta` and `OpenPonk-UML-Metamodel`). **Crucial Rule:** `OpenPonk-UML-Metamodel` is **generated code** produced by `uml-bootstrap-generator` and must **never** be modified manually without corresponding changes in the generator.

6. **Status Marker Definitions:**

   * *(unmarked)* – **Active:** Loaded in the standard all-in-one build.

   * `[outdated]` – **Outdated / Unloaded:** Temporarily excluded from the default build, but retained in its baseline and potentially functional or slated for revival.

   * `[deprecated]` – **Deprecated:** Permanently abandoned or superseded. **LLMs must never import, instantiate, subclass, or reference classes from `[deprecated]` packages or repositories.**

   * `[legacy / unmaintained]` – **Legacy / Unmaintained:** Historical unloaded extension packages whose current functionality and compatibility are unverified. **Do not use in new code.**

## 2. Core Runtime Architecture & Component Layers

OpenPonk separates project/domain data, model-level UI abstraction, visual diagram state, interactive canvas coordination, and Spec2 workbench GUI into distinct layers with strict responsibilities and lifecycles.

### 2.1 Project & Plugin Entry Points

* **Plugin (`OPPlugin`):** Each notation defines a subclass of `OPPlugin`, which registers the notation in OpenPonk and binds together its root model class, `OPDiagramController` class,ViewModel factory/navigation, and icon.

* **Project (`OPProject`):** Holds the project's root models and their associated diagrams via a dictionary mapping `{ rootModel -> { diagrams } }`.

  * A **root model** (e.g., a UML Package, an FSM, or a BPMN process) contains its own internal **model elements** (e.g., a UML Class inside a UML Package).

  * One root model can have multiple diagrams, whereas each diagram is associated with **exactly one** root model.

  * **Persistence:** `OPProject` is **persistent** (serialized indirectly: its metadata, root models, and diagrams are saved into the project archive/directory and `OPProject` is reconstructed upon loading).

* **Project Controller (`OPProjectController`):** Created when an `OPProject` is opened inside an `OPWorkbench`. Manages the open project's lifecycle and holds the active `OPDiagramController` instances (one for each currently open diagram). Purely **transient**.

### 2.2 The Five Architectural Layers

1. **Domain Model (`Models` tag):**

   * **Role:** Represents pure domain concepts (root models and their contained model elements). Can be any arbitrary class implementing `#uuid` (such as `OpenPonk-UML-Metamodel` classes or subclasses of `OPModelObject` from `OpenPonk-Model`).

   * **Dependencies:** Completely decoupled from diagrams, Roassal shapes, controllers, and Spec2 GUI.

   * **Persistence:** **Persistent** (serialized when the project is saved).

2. **ViewModel (`ViewModels` tag):**

   * **Role:** Sits directly above the Domain Model. Because OpenPonk supports arbitrary domain metamodels, ViewModels provide a **unified API** over heterogeneous domain models.

   * **Responsibilities:**

     * Exposes structural relationships of the model (containment hierarchy, associations, parent/child elements).

     * Handles **creation and deletion** of domain model elements.

     * Defines diagram-independent GUI metadata, such as the **icon** and **unified display name/label** used in trees, lists, and forms.

   * **Primary GUI Consumers:** Diagram-independent GUI components—specifically the **Project Tree** and **Properties Form (Magritte)**—operate primarily on **ViewModels**, as they require a uniform representation of the model and its structure rather than diagram visuals.

   * **Persistence:** **Transient** (runtime-only). Created when a project is opened in an `OPWorkbench` and destroyed when the project is closed.

3. **DiagramElement / Shape (`DiagramElements` tag):**

   * **Role:** Represents visual elements belonging to a diagram (nodes, edges, labels) as well as the diagram itself.

   * **Hierarchy & Naming Convention:**

     * The base class `OPDiagramElement` forms a composite tree via `owningElement` ($0..1$) and `ownedElements` ($0..N$).

     * Concrete classes for individual elements typically end with the **`Shape` suffix** rather than `DiagramElement` (e.g., `OPUmlClassShape`).
       
     * Notable core `OPDiagramElement` subclasses are `OPEdge` and `OPNode`. `OPNode` has core subclass `OPBoundNode`. Notation-specific shape classes are usually subclasses of these 4. `OPBoundNode` has core subclasses `OPDiagram`. 

     * The root element representing the entire diagram is `OPDiagram` (specifically its subclass `OPOpenPonkDiagram` that includes OpenPonk-specific information like `isOpen`, `isSelected` and `camera`).

   * **Roassal3 Encapsulation:**

     * While `OPDiagram` wraps a Roassal `RSCanvas` (`roassalShape` $0..1$), individual `OPDiagramElement` subclasses wrap an `RSShape` (`roassalShape` $0..1$).

     * Although Roassal3 supports interactivity and provides methods for adding and removing shapes, continuous incremental diagram editing is not its primary use case. Therefore, `OPDiagramElement` subclasses—supported by extensions in `OpenPonk-Roassal`—encapsulate almost all Roassal-related complexity, holding persistent visual state (positions, bounds, polyline waypoints, styling) and translating model/diagram updates into incremental Roassal shape mutations.

   * **Lifecycle & Persistence:** **Persistent** (serialized as part of the diagram when saving a project). `DiagramElement` instances exist in memory for all diagrams in an open `OPProject` **regardless of whether a given diagram is currently open in an `OPEditor` or rendered on a Roassal canvas**.

4. **Controller (`Controllers` tag):**

   * **Role:** Sits above **ViewModels** and **DiagramElements**. Both `OPDiagramController` (for the diagram root) and `OPElementController` (for individual model/diagram elements) inherit from `OPController`.

   * **Responsibilities:**

     * An `OPDiagramController` is created for each diagram opened in an `OPEditor` and creates/manages the `OPElementController` instances for the elements displayed in that diagram.

     * Mediates interactive user operations on the Roassal canvas (palette tool actions, drag-and-drop, element creation/connection, selection) using announcement subscriptions and Commander commands (e.g., `OPDiagramRemoveFigureCommand`), coordinating between the rendered `DiagramElement` (`Shape`) and its `ViewModel` / Domain Model.

   * **Persistence:** **Transient** (runtime-only). Controllers exist **only** for diagrams and diagram elements that are currently loaded and displayed on a Roassal canvas inside an active `OPEditor`.

5. **Workbench & Editor GUI (`OpenPonk-Spec`):**

   * **`OPWorkbench`:** The main Spec2 window representing a single open project (backed by `OPProjectController`). Hosts project-wide views (**Project Tree**, **Properties Form**) that interact with **ViewModels**, and a tabbed area hosting $0..N$ **`OPEditor`** instances.

   * **`OPEditor`:** A single tab inside `OPWorkbench` dedicated to one open diagram (`OPOpenPonkDiagram`). Holds a reference to the diagram's `OPDiagramController`, the notation tool palette, diagram toolbar, and an **`OPCanvasPresenter`** (a subclass of `SpRoassalPresenter` that embeds the Roassal3 `RSCanvas` into Spec2). Operates on **Controllers** and **DiagramElements**.

### 2.3 Instance Multiplicities (Within One Open `OPWorkbench`)

**Key Single-Representation Invariant:** OpenPonk does **not** allow a single domain model element to be represented by multiple diagram elements within the **same** diagram. For any given pair of `(Domain Model element, Diagram)`—or `(Domain Model element, OPDiagramController)`—there exists at most **$1$ `DiagramElement` (`Shape`)** ($0..1$) and at most **$1$ `OPElementController`** ($0..1$).

| Source Entity | Target Entity | Multiplicity | Architectural Explanation | 
| ----- | ----- | ----- | ----- | 
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

## 3. Three-Level Repository & Package Inventory

*(Note: Class and extension counts below are approximate order-of-magnitude indicators rounded upward to account for ongoing growth).*

### CATEGORY 1: Core & Platform (5 Repositories)

#### 1. `openponk`

* **Purpose:** Core of the OpenPonk modeling platform (ViewModels, Controllers, DiagramElements, Roassal3 diagram rendering, Spec2 workbench GUI, persistence, Undo/Redo).

* **Baseline (`BaselineOfOpenPonk`):**

  * **External dependencies:** `Roassal` (loads `#Full`), `RoassalExporters`, `Magritte` (loads `Core`, `Magritte-Morph`), `Mocketry`, `XPath`, `NeoJSON`.

  * **Internal dependencies:** `synchronized-links` (`OpenPonkSynchronizedLinks`), `zoomer` (`OpenPonkZoomer`).

  * **Groups:** `default` (`OpenPonk-Core`, `OpenPonk-Spec`), `dev-plugin` (`OpenPonk-DevTools`), `dev` (`default`, `dev-plugin`).

* **Packages:**

  * **`OpenPonk-Roassal`** `(~75 classes, ~75 ext)`

    * **Tags:** `[AnchorConstraint, AttachPoints, CreationFeedback, Exports, Interactions, Manifest, Markers, Meta, Polyline, Tests]`

    * **Internal baseline requires:** `Roassal`, `RoassalExporters`

    * **Role:** Extension layer over Roassal3 for incremental diagram rendering, polylines/edges, anchor constraints, attach points, creation feedback, markers, and canvas exports.

  * **`OpenPonk-Core`** `(~250 classes, ~15 ext)`

    * **Tags:** `[Announcements, Controllers, Controllers-Tests, DiagramElements, DiagramOnlyElements, Icons, Layouting, LegacySupport-Diagrams, Manifest, Models, Persistence, Plugin, Templates, Tests, UICommands, UndoRedo, UndoRedo-Tests, Version, ViewModels]`

    * **Internal baseline requires:** `Magritte`, `Mocketry`, `XPath`, `NeoJSON`, `OpenPonk-Roassal`

    * **Role:** Core platform architecture: **Projects** (`OPProject`, `OPProjectController`), **ViewModels** (unified API over domain models, model structure & lifecycle, GUI labels/icons), **Controllers** (`OPController`, `OPDiagramController`, `OPElementController`), **DiagramElements** (`OPDiagram`, `OPOpenPonkDiagram`, `OPDiagramElement`, visual shapes), plugin registration (`OPPlugin`), project & diagram serialization, UI commands, and Undo/Redo.

  * **`OpenPonk-Morphic`** `(<10 classes, <5 ext)`

    * **Tags:** `[Manifest, Morphic, Tests]`

    * **Internal baseline requires:** *(none)*

    * **Role:** Low-level Morphic components and layout adjustments.

  * **`OpenPonk-Spec`** `(~220 classes, ~25 ext)`

    * **Tags:** `[Application, Commands, Commands-CanvasPresenter, Commands-ProjectTree, Commands-RoassalShape, Commands-Workbench, Dialogs, Export, Form, Layouting, Manifest, Palettes, Tests, Windows]`

    * **Internal baseline requires:** `Magritte`, `OpenPonkSynchronizedLinks`, `OpenPonkZoomer`, `OpenPonk-Morphic`, `OpenPonk-Core`

    * **Role:** Main Spec2 GUI (`OPWorkbench`, `OPEditor`, `OPCanvasPresenter`, Project Tree, tool palettes, Magritte property forms, dialogs).

  * **`OpenPonk-DevTools`** `(~15 classes, ~10 ext)`

    * **Tags:** `[Export, Manifest, Releases, Watchdog]`

    * **Internal baseline requires:** *(loaded via `dev-plugin` group)*

    * **Role:** Developer utilities, release builders, and runtime watchdog tools.

#### 2. `openponk-model`

* **Purpose:** Optional foundational domain metamodel providing a quick bootstrap with common model features (`#uuid`, announcements, associations) for notations that do not require an independent custom metamodel.

* **Baseline (`BaselineOfOpenPonkModel`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `openponk` (`OpenPonk`, required in `default` group only; `plugin` group loads only `OpenPonk-Model`).

* **Packages:**

  * **`OpenPonk-Model`** `(~20 classes, 0 ext)`

    * **Tags:** `[Manifest, ModelObjects, Tests, ViewModels]`

    * **Internal baseline requires:** None in `plugin` group (`OpenPonk` in `default` group).

    * **Role:** Reusable base classes for domain models (`OPModelObject`, `OPDirectedAssociation`, etc.) and their base view models.

#### 3. `plugins`

* **Purpose:** Umbrella repository for assembling all-in-one OpenPonk builds and distributions.

* **Baseline (`BaselineOfOpenPonkPlugins`):**

  * **External dependencies:** None directly.

  * **Internal dependencies:**

    * `class-editor` (group `complete`)

    * `borm-editor` (group `plugin`)

    * `petrinets` (group `plugin`)

    * `fsm-editor` (group `plugin`)

    * `BPMN` (group `plugin`)

    * `markov-chains` (group `plugin`)

    * `ERD` (group `plugin`)

    * `DEMO` (group `plugin`)

    * `openponk` (group `dev-plugin`, loaded in `dev` group)

* **Packages:**

  * *(No application packages; contains only `BaselineOfOpenPonkPlugins`.)*

#### 4. `simulation`

* **Purpose:** Discrete-event and step-by-step simulation framework for OpenPonk models and diagrams.

* **Baselines (Dual-Baseline Architecture):**

  1. **`BaselineOfOpenPonkModelSimulation`:**

     * **Internal dependencies:** `openponk-model` (`OpenPonkModel`)

     * **Loads:** `OpenPonk-ModelSimulation`

  2. **`BaselineOfOpenPonkSimulation`:**

     * **Internal dependencies:** `openponk` (`OpenPonk`), `OpenPonkModelSimulation`

     * **Loads:** `OpenPonk-Simulation`

* **Packages:**

  * **`OpenPonk-ModelSimulation`** `(~40 classes, 0 ext)`

    * **Tags:** `[Events, Manifest, Simulators, Tests]`

    * **Internal baseline requires:** `OpenPonkModel`

    * **Role:** GUI-independent model simulation framework (event queues, step simulators).

  * **`OpenPonk-Simulation`** `(<10 classes, 0 ext)`

    * **Tags:** `[Manifest, Uncategorized]`

    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModelSimulation`

    * **Role:** Bridges model simulators with OpenPonk GUI and diagram controllers.

#### 5. `openponk-git`

* **Purpose:** Git integration for uploading and downloading a diagram and its underlying model to/from remote Git repositories (GitHub, GitLab.com) directly from the OpenPonk workbench.

* **Architectural & Verification Note:** Loaded into the all-in-one build via `class-editor` (`complete-plugin`). While toolbar submenu population (`Model -> Git model repository`) is covered by a unit test (`OPGitModelMenuTest`), end-to-end upload/download functionality was historically tested only with UML diagrams and has not been actively used or verified in recent practice.

* **Baseline (`BaselineOfOpenPonkGit`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `openponk` (`OpenPonk`).

* **Packages:**

  * **`OpenPonk-Git`** `(~20 classes, 0 ext)`

    * **Tags:** `[Commands, Manifest, Tests, Uncategorized]`

    * **Internal baseline requires:** `OpenPonk`

    * **Role:** Toolbar commands and dialogs for pushing/fetching a project (diagram + model) to/from GitHub and GitLab.

### CATEGORY 2: Notation Plugins – Editors & Models (9 Repositories)

#### 6. `class-editor`

* **Purpose:** UML Class Diagram editor plugin (with generic UML profile support).

* **Architectural Note:** `OpenPonk-ClassEditor` itself is a pure UML class diagram editor and contains no OntoUML-specific logic. OntoUML is implemented in separate repositories that extend `class-editor`. While `OpenPonk-ClassEditor` can theoretically be loaded standalone without OntoUML (via the `plugin` or `default` baseline groups), standard builds always load the `complete` group, which uses `OpenPonk-ClassEditor-Plugins` to pull in the OntoUML ecosystem and `openponk-git`.

* **Baseline (`BaselineOfOpenPonkClassEditor`):**

  * **External dependencies:** None directly (transitively pulls `PetitParser`, `XMLWriter`, `XMLParser` via dependencies).

  * **Internal dependencies:**

    * Group `plugin` (loads `OpenPonk-ClassEditor`): requires `uml-xmi` (`OpenPonkUMLXMI`), `uml-bootstrap-generator` (`OpenPonkUMLBootstrapGenerator`), `uml-profiles` (`OpenPonkUMLProfiles`).

    * Group `complete-plugin` (loads `OpenPonk-ClassEditor-Plugins`): additionally requires `ontouml-verifications` (`OpenPonkOntoUMLVerifications`), `ontouml-patterns` (`OpenPonkOntoUMLPatterns`), `openponk-ontouml-to-uml-transformation` (`OpenPonkOntoUMLTransformations`), `openponk-git` (`OpenPonkGit`).

    * Group `default`: `openponk` + `plugin`.

    * Group `complete`: `default` + `complete-plugin`.

* **Active Packages:**

  * **`OpenPonk-ClassEditor`** `(~180 classes, ~25 ext)`

    * **Tags:** `[Announcements, Commands, Commands-ProjectTree, Commands-RoassalShape, Controllers, DiagramElements, Examples, Forms, Manifest, Persistence, Plugin, Profiles, Tests, Tests - Controllers, Tests - DiagramElements, Tests - UI, UI, ViewModels]`

    * **Internal baseline requires:** `OpenPonkUMLXMI`, `OpenPonkUMLBootstrapGenerator`, `OpenPonkUMLProfiles` (and `OpenPonk`)

    * **Role:** UML class diagram editor, view models, controllers, visual shapes (`*Shape`), stereotype/profile application UI, and profile generator wizard (`OPUmlProfileGeneratorWizard`).

  * **`OpenPonk-ClassEditor-Plugins`** `(<10 classes, 0 ext)`

    * **Tags:** `[Manifest, Uncategorized]`

    * **Internal baseline requires:** `OpenPonk-ClassEditor`, `OpenPonkOntoUMLVerifications`, `OpenPonkOntoUMLPatterns`, `OpenPonkOntoUMLTransformations`, `OpenPonkGit`

    * **Role:** Integration glue package that pulls the OntoUML extensions and Git plugin into complete Class Editor builds.

* **Unloaded Packages (`[legacy / unmaintained]`):**

  * **`DynaCASE-Cormas`** `[legacy / unmaintained]` – Legacy Cormas agent-based simulation integration.

  * **`UMLClassEditor-Cormas`** `[legacy / unmaintained]` – Legacy Cormas extension for the UML class editor.

  * **`LiveClassEditor`** `[legacy / unmaintained]` – Experimental live Pharo class synchronization with class diagrams.

  * **`LiveGenerator`** `[legacy / unmaintained]` – Experimental live code generation utility.

  * **`LiveInstanceVisualization`** `[legacy / unmaintained]` – Experimental live object graph visualization.

  * **`OpenPonk-UML-DSL`** `[legacy / unmaintained]` – Unmaintained textual DSL for building UML class diagrams.

  * **`UML-CodeGenerator`** `[legacy / unmaintained]` – Unmaintained Smalltalk code generator from UML models.

#### 7. `borm-model`

* **Purpose:** Domain metamodel for the BORM (Business Object Relation Modeling) notation.

* **Baseline (`BaselineOfOpenPonkBormModel`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `openponk-model` (`OpenPonkModel`), `synchronized-links` (`OpenPonkSynchronizedLinks`).

* **Packages:**

  * **`OpenPonk-BormModel`** `(~40 classes, 0 ext)`

    * **Tags:** `[Conditions, Factory, Manifest, Models, Tests]`

    * **Internal baseline requires:** `OpenPonkModel`, `OpenPonkSynchronizedLinks`

    * **Role:** BORM domain model entities (participants, states, activities, transitions, communications, constraints/conditions).

#### 8. `borm-editor`

* **Purpose:** Diagram editor and simulator for BORM Object Relation Diagrams (ORD).

* **Baseline (`BaselineOfOpenPonkBormEditor`):**

  * **External dependencies:** `PetitParser`.

  * **Internal dependencies:** `borm-model` (`OpenPonkBormModel`), `simulation` (`OpenPonkSimulation`), `openponk` (`OpenPonk`).

* **Packages:**

  * **`OpenPonk-BormEditor`** `(~100 classes, <10 ext)`

    * **Tags:** `[Commands, Controllers, DiagramElements, Dsl, Examples, Extensions, Icons, Manifest, Tests, UI, Utility, ViewModels]`

    * **Internal baseline requires:** `PetitParser`, `OpenPonkBormModel`

    * **Role:** BORM diagram view models, controllers, shapes, textual condition/DSL parser, and UI commands.

  * **`OpenPonk-BormSimulation`** `(~25 classes, 0 ext)`

    * **Tags:** `[Manifest, Simulation, Tests]`

    * **Internal baseline requires:** `OpenPonk-BormEditor`, `OpenPonkSimulation`

    * **Role:** Token-based process simulator and visualization for BORM diagrams.

  * **`Borm-OpenCabe-Importer`** `[deprecated]`

    * **Role:** Unloaded importer for the defunct Eclipse-based OpenCABE tool. Do not use.

#### 9. `BPMN`

* **Purpose:** Editor, validator, simulator, and XML serializer for BPMN 2.0 diagrams.

* **Baseline (`BaselineOfOpenPonkBPMN`):**

  * **External dependencies:** None directly (transitively pulls `XMLParser` via `xml-dom-visitor`).

  * **Internal dependencies:** `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`), `simulation` (`OpenPonkSimulation`), `xml-dom-visitor` (`OpenPonkXMLDOMVisitor`).

* **Packages:**

  * **`OpenPonk-BPMN`** `(~150 classes, <5 ext)`

    * **Tags:** `[Announcements, Commands, Controllers, DiagramElements, Examples, Factory, Icons, Manifest, Models, Plugin, Tests, ViewModels]`

    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`

    * **Role:** Core BPMN 2.0 domain model, view models, diagram elements, controllers, and workbench plugin.

  * **`OpenPonk-BPMN-XML`** `(~15 classes, 0 ext)`

    * **Tags:** `[DOM, Manifest, Reader, Tests, Writer]`

    * **Internal baseline requires:** `OpenPonk-BPMN`, `OpenPonkXMLDOMVisitor`

    * **Role:** BPMN 2.0 XML/XSD DOM import and export.

  * **`OpenPonk-BPMN-Validation`** `(~80 classes, 0 ext)`

    * **Tags:** `[Controllers, Manifest, Messages, Results, Settings, Tests, UI, Validation-Classes]`

    * **Internal baseline requires:** `OpenPonk-BPMN`

    * **Role:** BPMN 2.0 structural/semantic validation rules and diagnostic UI.

  * **`OpenPonk-BPMN-Simulation`** `(~40 classes, 0 ext)`

    * **Tags:** `[Manifest, Simulation, Tests]`

    * **Internal baseline requires:** `OpenPonk-BPMN`, `OpenPonkSimulation`

    * **Role:** BPMN token-flow simulation engine and UI integration.

#### 10. `petrinets`

* **Purpose:** Petri Nets diagram editor and simulator (Place/Transition nets and optional Boolean Colored Petri Nets).

* **Baseline (`BaselineOfOpenPonkPetriNets`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `openponk` (`OpenPonk`), `simulation` (`OpenPonkSimulation`, which transitively provides `openponk-model`). Defines a separate `bcpn` group for Boolean Colored Petri Nets.

* **Packages:**

  * **`OpenPonk-PetriNets`** `(~100 classes, 0 ext)`

    * **Tags:** `[Announcements, Controllers, DiagramElements, Examples, Manifest, Models, Simulation, Tests, ViewModels]`

    * **Internal baseline requires:** `OpenPonkSimulation` (and `OpenPonk`)

    * **Role:** Place/Transition Petri net domain model, view models, visual shapes, controllers, and token-firing simulation.

  * **`OpenPonk-BooleanColoredPetriNets`** `[outdated]`

    * **Internal baseline requires:** `OpenPonk-PetriNets` (loaded only via explicit `bcpn` baseline group)

    * **Role:** Extension for Boolean Colored Petri Nets; excluded from the default build, but retained in the baseline and potentially functional.

#### 11. `fsm-editor`

* **Purpose:** Finite-State Machine (FSM) diagram editor, simulator, and GraphML serializer.

* **Baseline (`BaselineOfOpenPonkFsmEditor`):**

  * **External dependencies:** `Mustache`.

  * **Internal dependencies:** `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`).

* **Packages:**

  * **`OpenPonk-FSM`** `(~70 classes, 0 ext)`

    * **Tags:** `[Controllers, DiagramElements, Examples, Icons, Manifest, Models, Plugin, Serialization, Simulations, Tests, ViewModels]`

    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`, `Mustache`

    * **Role:** FSM metamodel, view models, diagram editor, state simulation, and GraphML serialization (`OPFsmGraphML`).

#### 12. `markov-chains`

* **Purpose:** Markov Chains diagram editor, simulator, and solver integration.

* **Baseline (`BaselineOfOpenPonkMarkovChains`):**

  * **External dependencies:** `Mustache`.

  * **Internal dependencies:** `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`).

* **Packages:**

  * **`OpenPonk-MarkovChains`** `(~60 classes, 0 ext)`

    * **Tags:** `[Commands, Controllers, DiagramElements, Examples, Icons, Manifest, Models, Plugin, Serialization, Simulations, Tests, ViewModels, Zinc]`

    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`, `Mustache`

    * **Role:** Markov chain model, view models, visual editor, serialization, simulation, and Zinc HTTP client integration.

#### 13. `ERD`

* **Purpose:** Entity-Relationship Diagram (ERD) editor and SQL/DDL generator.

* **Baseline (`BaselineOfOpenPonkERD`):**

  * **External dependencies:** `Mustache`.

  * **Internal dependencies:** `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`).

* **Packages:**

  * **`OpenPonk-ERD`** `(~75 classes, 0 ext)`

    * **Tags:** `[Commands, Controllers, DiagramElements, Examples, Icons, Manifest, Models, Plugin, Serialization, Tests, ViewModels]`

    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`, `Mustache`

    * **Role:** ER domain model (entities, attributes, keys, relationships, cardinalities), view models, diagram editor, and Mustache-based serialization/export.

#### 14. `DEMO`

* **Purpose:** DEMO (Design & Engineering Methodology for Organizations) notation plugin.

* **Baseline (`BaselineOfOpenPonkDEMO`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`). *(Supersedes the deprecated `demo-editor` repository).*

* **Packages:**

  * **`OpenPonk-DEMO`** `(~100 classes, 0 ext)`

    * **Tags:** `[Controllers, DiagramElements, Examples, Icons, Manifest, Models, Plugin, Tests, ViewModels]`

    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`

    * **Role:** DEMO enterprise ontology diagrams, models, view models, controllers, and shapes.

### CATEGORY 3: UML & OntoUML Ecosystem (9 Repositories)

#### 15. `uml-metamodel`

* **Purpose:** Complete OMG UML 2.5 metamodel implementation in Pharo.

* **Architectural Warning:** `OpenPonk-UML-Metamodel` consists of **generated code** (produced by `uml-bootstrap-generator`). Never modify classes or methods in `OpenPonk-UML-Metamodel` manually without updating the generator in `uml-bootstrap-generator`.

* **Baseline (`BaselineOfOpenPonkUMLMetamodel`):**

  * **External dependencies:** None.

  * **Internal dependencies:** None.

* **Packages:**

  * **`OpenPonk-UML-Meta`** `(~15 classes, 0 ext)`

    * **Tags:** `[Manifest, Models, Tests]`

    * **Internal baseline requires:** None.

    * **Role:** Hand-written meta-layer supporting UML metamodel reflection, opposite properties, and slot descriptors.

  * **`OpenPonk-UML-Metamodel`** `(~260 classes, 0 ext)`

    * **Tags:** `[Actions, Activities, Classification, CommonBehavior, CommonStructure, Deployments, InformationFlows, Interactions, Manifest, Meta, Packages, SimpleClassifiers, StateMachines, StructuredClassifiers, UseCases, Values]`

    * **Internal baseline requires:** `OpenPonk-UML-Meta`

    * **Role:** Generated implementation of the full UML 2.5 specification.

  * **`OP-UML-GTExtensions`** `[deprecated]`

    * **Role:** Unloaded extensions for legacy GTInspector and Roassal2 (replaced by OpenPonk's Spec2 Project Tree). Do not use.

#### 16. `uml-bootstrap-generator`

* **Purpose:** Code generator that bootstraps `OpenPonk-UML-Metamodel` from OMG XMI specifications and generates Pharo packages for UML/OntoUML profiles.

* **Architectural Note:** Intentionally loaded into production builds (via `class-editor`) both to keep its test suite continuously verified against the active Pharo/UML environment and because `OPUmlProfileGeneratorWizard` in `OpenPonk-ClassEditor` uses its profile generator at runtime.

* **Baseline (`BaselineOfOpenPonkUMLBootstrapGenerator`):**

  * **External dependencies:** None directly.

  * **Internal dependencies:** `xmi` (`OpenPonkXMI`), `pharo-changes-builder` (`ChangesBuilder`).

* **Packages:**

  * **`OpenPonk-UML-Bootstrap`** `(~50 classes, 0 ext)`

    * **Tags:** `[Classification, CommonStructure, Deployments, Manifest, Packages, SimpleClassifiers, StructuredClassifiers, Super, Values]`

    * **Internal baseline requires:** None.

    * **Role:** Minimal bootstrap metamodel used to parse raw OMG UML XMI specification files.

  * **`OpenPonk-UML-BootstrapGenerator`** `(~30 classes, 0 ext)`

    * **Tags:** `[Bootstrap-Generator, Manifest, Metamodel-Generator, Profile-Generator, Reader, Templates, Tests, Utilities]`

    * **Internal baseline requires:** `OpenPonkXMI`, `ChangesBuilder`, `OpenPonk-UML-Bootstrap`

    * **Role:** Code generators producing the bootstrap model, `OpenPonk-UML-Metamodel`, and custom UML profile packages.

#### 17. `uml-profiles`

* **Purpose:** Core UML profile application mechanism and small example/test profiles.

* **Baseline (`BaselineOfOpenPonkUMLProfiles`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `uml-metamodel` (`OpenPonkUMLMetamodel`).

* **Packages:**

  * **`OpenPonk-UML-Profiles`** `(<10 classes, <10 ext)`

    * **Tags:** `[Manifest, Profile]`

    * **Internal baseline requires:** `OpenPonkUMLMetamodel`

    * **Role:** Core profile registration and stereotype application mechanism extending UML metamodel elements.

  * **`OpenPonk-ClockProfile`** `(<10 classes, <5 ext)`

    * **Tags:** `[Manifest, Metamodel, Model]`

    * **Internal baseline requires:** `OpenPonk-UML-Profiles`

    * **Role:** Minimal example/test UML profile demonstrating how a profile is implemented and generated.

  * **`OpenPonk-IssuesProfile`** `(<10 classes, <5 ext)`

    * **Tags:** `[Manifest, Metamodel, Model]`

    * **Internal baseline requires:** `OpenPonk-UML-Profiles`

    * **Role:** Minimal example/test UML profile demonstrating how a profile is implemented and generated.

#### 18. `uml-xmi`

* **Purpose:** XMI serialization and deserialization for `OpenPonk-UML-Metamodel` and applied UML profiles.

* **Baseline (`BaselineOfOpenPonkUMLXMI`):**

  * **External dependencies:** None directly.

  * **Internal dependencies:** `xmi` (`OpenPonkXMI`), `uml-metamodel` (`OpenPonkUMLMetamodel`).

* **Packages:**

  * **`OpenPonk-UML-XMI`** `(~10 classes, 0 ext)`

    * **Tags:** `[Manifest, Reader, Tests, Utility, Writer]`

    * **Internal baseline requires:** `OpenPonkXMI`, `OpenPonkUMLMetamodel`

    * **Role:** Converts between generic `OpenPonk-XMI` graphs and `OpenPonk-UML-Metamodel` instances (including stereotypes).

#### 19. `ontouml-profile`

* **Purpose:** OntoUML stereotype profile and UI extension plugin for the UML Class Editor.

* **Baseline (`BaselineOfOpenPonkOntoUMLProfile`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `class-editor` (`OpenPonkClassEditor`).

* **Packages:**

  * **`OpenPonk-OntoUML-Profile`** `(~50 classes, ~10 ext)`

    * **Tags:** `[Examples, Manifest, Model, Tests]`

    * **Internal baseline requires:** `OpenPonkClassEditor`

    * **Role:** OntoUML profile implementation (stereotypes such as `Kind`, `SubKind`, `Role`, `Phase`, `Relator`, `Mode`, etc.; code-generated via `OntoUMLProfileFactory class>>#generate` using `ChangesBuilder`).

  * **`OpenPonk-OntoUML-Plugin`** `(~15 classes, <10 ext)`

    * **Tags:** `[Icons, Manifest, Menu, Plugin, Tests]`

    * **Internal baseline requires:** `OpenPonk-OntoUML-Profile`

    * **Role:** Extends `OpenPonk-ClassEditor` with OntoUML stereotype menus, icons, and visual conventions.

#### 20. `ontouml-modelquery`

* **Purpose:** Query layer for inspecting and filtering OntoUML-stereotyped UML models.

* **Baseline (`BaselineOfOpenPonkOntoUMLModelQuery`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `ontouml-profile` (`OpenPonkOntoUMLProfile`).

* **Packages:**

  * **`OpenPonk-OntoUML-ModelQuery`** `(<10 classes, 0 ext)`

    * **Tags:** `[Manifest, Uncategorized]`

    * **Internal baseline requires:** `OpenPonkOntoUMLProfile`

    * **Role:** Query utilities over OntoUML classes, associations, generalizations, and stereotypes.

#### 21. `ontouml-patterns`

* **Purpose:** Ontology-driven design patterns catalog and instantiation UI for OntoUML.

* **Baseline (`BaselineOfOpenPonkOntoUMLPatterns`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `ontouml-profile` (`OpenPonkOntoUMLProfile`).

* **Packages:**

  * **`OpenPonk-OntoUML-Patterns`** `(~40 classes, 0 ext)`

    * **Tags:** `[Commands, Manifest, Patterns, Tests, UI]`

    * **Internal baseline requires:** `OpenPonkOntoUMLProfile`

    * **Role:** Pattern templates, wizards, and commands for applying OntoUML design patterns in the Class Editor.

#### 22. `ontouml-verifications`

* **Purpose:** OntoUML model verification and ontological anti-pattern detection.

* **Baseline (`BaselineOfOpenPonkOntoUMLVerifications`):**

  * **External dependencies:** `PetitParser`.

  * **Internal dependencies:** `ontouml-modelquery` (`OpenPonkOntoUMLModelQuery`).

* **Packages:**

  * **`OpenPonk-OntoUML-Verifications`** `(~110 classes, 0 ext)`

    * **Tags:** `[Antipattern-Classes, Antipattern-Tests, Commands, Controllers, DocumentationReferences, Exceptions, Manifest, Messages, Results, Settings, Traits, UI, Updating, Verification-Classes, Verification-Tests]`

    * **Internal baseline requires:** `OpenPonkOntoUMLModelQuery`, `PetitParser`

    * **Role:** OntoUML syntax/semantics verification rules, anti-pattern detectors, and diagnostic UI.

#### 23. `openponk-ontouml-to-uml-transformation`

* **Purpose:** Model-to-model transformation from conceptual OntoUML models to implementation-level UML models.

* **Baseline (`BaselineOfOpenPonkOntoUMLTransformations`):**

  * **External dependencies:** None.

  * **Internal dependencies:** `ontouml-modelquery` (`OpenPonkOntoUMLModelQuery`).

* **Packages:**

  * **`OpenPonk-OntoUML-Transformations`** `(~20 classes, 0 ext)`

    * **Tags:** `[Commands, Controllers, Manifest, Transformations, Transformers, UI]`

    * **Internal baseline requires:** `OpenPonkOntoUMLModelQuery`

    * **Role:** Transforms OntoUML constructs into standard implementation UML class models.

### CATEGORY 4: Shared Libraries & Utilities (5 Repositories)

#### 24. `synchronized-links`

* **Purpose:** Self-synchronizing bidirectional relations for Smalltalk objects.

* **Baseline (`BaselineOfOpenPonkSynchronizedLinks`):**

  * **Dependencies:** None. *(Consumed by `openponk` and `borm-model`.)*

* **Packages:**

  * **`OpenPonk-SynchronizedLinks`** `(~15 classes, 0 ext)`

    * **Tags:** `[Links, Manifest, Tests]`

    * **Internal baseline requires:** None.

    * **Role:** Maintains bidirectional consistency across $1:1$, $1:N$, and $M:N$ object references.

#### 25. `zoomer`

* **Purpose:** Tool for setting sharp, device-appropriate UI zoom levels.

* **Baseline (`BaselineOfOpenPonkZoomer`):**

  * **Dependencies:** None. *(Consumed by `openponk` in `OpenPonk-Spec`.)*

* **Packages:**

  * **`OpenPonk-Zoomer`** `(<10 classes, 0 ext)`

    * **Tags:** `[Model, Tests, UI]`

    * **Internal baseline requires:** None.

    * **Role:** World/canvas scaling widget and zoom factor management.

#### 26. `xmi`

* **Purpose:** Generic XMI file to XMI object-graph materialization and serialization.

* **Architectural Note:** Written as a generic XMI library, but consumed exclusively by the UML ecosystem (`uml-xmi`, `uml-bootstrap-generator`).

* **Baseline (`BaselineOfOpenPonkXMI`):**

  * **External dependencies:** `XMLWriter`.

  * **Internal dependencies:** `xml-dom-visitor` (`OpenPonkXMLDOMVisitor`).

* **Packages:**

  * **`OpenPonk-XMI`** `(~25 classes, 0 ext)`

    * **Tags:** `[DOM, Manifest, Reader, Tests, Writer]`

    * **Internal baseline requires:** `OpenPonkXMLDOMVisitor`, `XMLWriter`

    * **Role:** Reads and writes XMI files into/from intermediate XMI element graphs.

  * **`OpenPonk-XMI-Inspection`** `(<5 classes, ~10 ext)`

    * **Tags:** `[Manifest]`

    * **Internal baseline requires:** `OpenPonk-XMI`

    * **Role:** Inspector extensions for inspecting XMI graph nodes.

#### 27. `xml-dom-visitor`

* **Purpose:** Visitor pattern implementation for `XMLParser` DOM trees.

* **Architectural Note:** Required by `BaselineOfOpenPonkXMI` and `BaselineOfOpenPonkBPMN`. Adds `#accept:` extension methods to `XMLCData`, `XMLDocument`, `XMLElement`, and `XMLString`, though its own classes (`XMLDOMVisitor`, `XMLDOMElementVisitor`, `XMLDOMTypedElement`) currently have no direct references from other OpenPonk packages.

* **Baseline (`BaselineOfOpenPonkXMLDOMVisitor`):**

  * **External dependencies:** `XMLParser`.

  * **Internal dependencies:** None.

* **Packages:**

  * **`OpenPonk-XMLDOMVisitor`** `(~10 classes, <10 ext)`

    * **Tags:** `[DOM, Manifest, Tests, Visitor]`

    * **Internal baseline requires:** `XMLParser`

    * **Role:** Visitor double-dispatch and typed element wrappers over XML DOM nodes.

#### 28. `pharo-changes-builder`

* **Purpose:** Lightweight wrapper simplifying common `RBRefactoring` operations during programmatic code generation.

* **Architectural Note:** Actively used for code generation in `uml-bootstrap-generator`, `ontouml-profile` (`OntoUMLProfileFactory`), and `class-editor` (`OPUmlProfileGeneratorWizard`).

* **Baseline (`BaselineOfChangesBuilder`):**

  * **Dependencies:** None.

* **Packages:**

  * **`ChangesBuilder`** `(~20 classes, ~10 ext)`

    * **Tags:** `[Model, Tests, Utilities]`

    * **Internal baseline requires:** None.

    * **Role:** Programmatic creation of packages, classes, instance variables, and methods via refactoring changesets.

### CATEGORY 5: Infrastructure, Web & Specifications – Non-Pharo Repositories (4 Repositories)

* **29. `ci-scripts`** – Shared GitHub Actions workflows and CI scripts used across OpenPonk repositories. *(Note: Individual Pharo repositories often define `.github/dependent-repositories.txt` listing downstream repositories whose CI builds are automatically triggered on push).*

* **30. `releaser`** – Python scripts for building and publishing numbered OpenPonk releases.

* **31. `specs-repository`** – Storage for formal specification files (such as OMG UML/XMI specifications consumed by `uml-bootstrap-generator`) and published XMI profile definitions referenced by exported OpenPonk/OntoUML XMI files and saved projects (e.g., `OntoUML/1.0/OntoUML.profile.xmi`, referenced via `https://openponk.org/OntoUML/1.0/`).

* **32. `openponk.github.io`** – Static HTML website for OpenPonk (edited directly in this repository).

### CATEGORY 6: Deprecated & Archived Repositories – Outside Active Builds (7 Repositories)

* **33. `openponk.github.io-sources`** `[deprecated]` – Legacy JavaScript static-site generator for the website; superseded by direct editing in `openponk.github.io`.

* **34. `demo-editor`** `[deprecated -> DEMO]` – Original non-functional implementation of the DEMO editor; completely replaced by `DEMO`.

* **35. `uml-shapes`** `[deprecated -> OpenPonk-Core]` – Early partial implementation of OMG UML Diagram Definition / Diagram Interchange. Generalized for all notations and merged into the `DiagramElements` (Shapes) architecture in `OpenPonk-Core`.

* **36. `file-dialog`** `[deprecated -> NewTools]` – Custom replacement for Pharo's native file dialog; superseded once its concepts were integrated into standard Pharo `NewTools`.

* **37. `uml-playground`** `[deprecated]` – Historical experimental repository from early UML metamodel development.

* **38. `xsd`** `[deprecated]` – Unmaintained XML Schema experiments (never loaded into OpenPonk).

* **39. `xsd-generator`** `[deprecated]` – Unmaintained XSD generator experiments (never loaded into OpenPonk).

## 4. Dependency Graph & Layered Architecture

### 4.1 Third-Party External Dependencies

| External Library | Role in OpenPonk | Consumed By (OpenPonk Baselines) | 
| ----- | ----- | ----- | 
| **`Roassal`** (Roassal3) | Canvas and vector graphics rendering engine | `openponk` (`OpenPonk-Roassal`) | 
| **`RoassalExporters`** | Diagram export to image/vector formats | `openponk` (`OpenPonk-Roassal`) | 
| **`Magritte`** | Meta-descriptions and dynamic property forms | `openponk` (`OpenPonk-Core`, `OpenPonk-Spec`) | 
| **`Mocketry`** | Mocking framework for unit tests | `openponk` (`OpenPonk-Core`) | 
| **`XPath`** | XML path querying | `openponk` (`OpenPonk-Core`) | 
| **`NeoJSON`** | JSON serialization for projects and diagrams | `openponk` (`OpenPonk-Core`) | 
| **`XMLParser`** | DOM XML parsing | `xml-dom-visitor` (`OpenPonk-XMLDOMVisitor`) | 
| **`XMLWriter`** | XML stream writing | `xmi` (`OpenPonk-XMI`) | 
| **`PetitParser`** | Parser combinators for DSLs and expressions | `borm-editor`, `ontouml-verifications` | 
| **`Mustache`** | Template engine for code/file export | `fsm-editor`, `markov-chains`, `ERD` | 

### 4.2 Architectural Layers (Bottom-Up Dependency Flow)

#### Layer 0: Standalone Low-Level Utilities & Base Metamodels (Zero Internal Dependencies)

* `synchronized-links` (`OpenPonk-SynchronizedLinks`)

* `zoomer` (`OpenPonk-Zoomer`)

* `pharo-changes-builder` (`ChangesBuilder`)

* `xml-dom-visitor` (`OpenPonk-XMLDOMVisitor`) $\rightarrow$ *depends on external `XMLParser`*

* `uml-metamodel` (`OpenPonk-UML-Meta` $\rightarrow$ `OpenPonk-UML-Metamodel`)

#### Layer 1: Core Infrastructure, Optional Base Metamodel & Serialization Utilities

* `openponk` (`OpenPonk-Roassal` $\rightarrow$ `OpenPonk-Core`; `OpenPonk-Morphic` $\rightarrow$ `OpenPonk-Spec`; `OpenPonk-DevTools`)

  * *Depends on:* `synchronized-links`, `zoomer`, plus external `Roassal`, `RoassalExporters`, `Magritte`, `Mocketry`, `XPath`, `NeoJSON`.

* `openponk-model` (`OpenPonk-Model`)

  * *Depends on:* none in `plugin` group (`openponk` in `default` group).

* `xmi` (`OpenPonk-XMI` $\rightarrow$ `OpenPonk-XMI-Inspection`)

  * *Depends on:* `xml-dom-visitor`, external `XMLWriter`.

* `uml-profiles` (`OpenPonk-UML-Profiles` $\rightarrow$ `OpenPonk-ClockProfile`, `OpenPonk-IssuesProfile`)

  * *Depends on:* `uml-metamodel`.

#### Layer 2: Domain Extensions, Simulation Framework & UML Tooling

* `simulation`:

  * `BaselineOfOpenPonkModelSimulation` (`OpenPonk-ModelSimulation`) $\rightarrow$ *depends on:* `openponk-model`.

  * `BaselineOfOpenPonkSimulation` (`OpenPonk-Simulation`) $\rightarrow$ *depends on:* `openponk`, `OpenPonkModelSimulation`.

* `openponk-git` (`OpenPonk-Git`) $\rightarrow$ *depends on:* `openponk`.

* `borm-model` (`OpenPonk-BormModel`) $\rightarrow$ *depends on:* `openponk-model`, `synchronized-links`.

* `uml-xmi` (`OpenPonk-UML-XMI`) $\rightarrow$ *depends on:* `xmi`, `uml-metamodel`.

* `uml-bootstrap-generator` (`OpenPonk-UML-Bootstrap` $\rightarrow$ `OpenPonk-UML-BootstrapGenerator`) $\rightarrow$ *depends on:* `xmi`, `pharo-changes-builder`.

#### Layer 3: Notation Editors (Core Plugins)

* **UML Class Editor (built on `openponk` + `uml-metamodel` ecosystem):**

  * `class-editor` (`OpenPonk-ClassEditor`, group `plugin`) $\rightarrow$ *depends on:* `openponk`, `uml-xmi`, `uml-bootstrap-generator`, `uml-profiles`.

* **General Notation Editors (built on `openponk` + `openponk-model`):**

  * `DEMO` (`OpenPonk-DEMO`) $\rightarrow$ *depends on:* `openponk`, `openponk-model`.

  * `fsm-editor` (`OpenPonk-FSM`) $\rightarrow$ *depends on:* `openponk`, `openponk-model`, external `Mustache`.

  * `markov-chains` (`OpenPonk-MarkovChains`) $\rightarrow$ *depends on:* `openponk`, `openponk-model`, external `Mustache`.

  * `ERD` (`OpenPonk-ERD`) $\rightarrow$ *depends on:* `openponk`, `openponk-model`, external `Mustache`.

* **Simulated / Validated Notation Editors (built on `openponk` + `simulation` + `openponk-model`):**

  * `borm-editor` (`OpenPonk-BormEditor` $\rightarrow$ `OpenPonk-BormSimulation`) $\rightarrow$ *depends on:* `openponk`, `borm-model` (which depends on `openponk-model`), `simulation`, external `PetitParser`.

  * `BPMN` (`OpenPonk-BPMN` $\rightarrow$ `OpenPonk-BPMN-XML`, `OpenPonk-BPMN-Validation`, `OpenPonk-BPMN-Simulation`) $\rightarrow$ *depends on:* `openponk`, `openponk-model`, `simulation`, `xml-dom-visitor`.

  * `petrinets` (`OpenPonk-PetriNets` \[and optional `OpenPonk-BooleanColoredPetriNets`\]) $\rightarrow$ *depends on:* `openponk`, `simulation` (and through `simulation` transitively on `openponk-model`).

#### Layer 4: OntoUML Ecosystem & Class Editor Extensions

* `ontouml-profile` (`OpenPonk-OntoUML-Profile` $\rightarrow$ `OpenPonk-OntoUML-Plugin`) $\rightarrow$ *depends on:* `class-editor`.

* `ontouml-modelquery` (`OpenPonk-OntoUML-ModelQuery`) $\rightarrow$ *depends on:* `ontouml-profile`.

* `ontouml-patterns` (`OpenPonk-OntoUML-Patterns`) $\rightarrow$ *depends on:* `ontouml-profile`.

* `ontouml-verifications` (`OpenPonk-OntoUML-Verifications`) $\rightarrow$ *depends on:* `ontouml-modelquery`, external `PetitParser`.

* `openponk-ontouml-to-uml-transformation` (`OpenPonk-OntoUML-Transformations`) $\rightarrow$ *depends on:* `ontouml-modelquery`.

* `class-editor` (`OpenPonk-ClassEditor-Plugins`, group `complete-plugin`) $\rightarrow$ *aggregates:* `OpenPonk-ClassEditor`, `ontouml-verifications`, `ontouml-patterns`, `openponk-ontouml-to-uml-transformation`, `openponk-git`.

#### Layer 5: All-in-One Distribution Assembly

* `plugins` (`BaselineOfOpenPonkPlugins`) $\rightarrow$ *aggregates:* `class-editor` (`complete`), `borm-editor` (`plugin`), `petrinets` (`plugin`), `fsm-editor` (`plugin`), `BPMN` (`plugin`), `markov-chains` (`plugin`), `ERD` (`plugin`), `DEMO` (`plugin`), plus `openponk` (`dev-plugin` in `dev` group).

### 4.3 Architectural Observations & Maintenance Notes

1. **Transitive Loading of `openponk-git` via `class-editor`:** `OpenPonk-Git` depends only on `openponk` core and provides workbench-level Git commands (`Model -> Git model repository`), but in the all-in-one build (`BaselineOfOpenPonkPlugins`) it is loaded transitively via `BaselineOfOpenPonkClassEditor` (`complete-plugin` $\rightarrow$ `OpenPonk-ClassEditor-Plugins`) rather than as a direct top-level dependency in `BaselineOfOpenPonkPlugins`.

2. **Unreferenced Visitor Classes in `xml-dom-visitor`:** While `BaselineOfOpenPonkXMI` and `BaselineOfOpenPonkBPMN` depend on `OpenPonkXMLDOMVisitor`, neither `OpenPonk-XMI` nor `OpenPonk-BPMN-XML` currently subclasses or references `XMLDOMVisitor`, `XMLDOMElementVisitor`, or `XMLDOMTypedElement`.

3. **Tag Naming Inconsistency in `OpenPonk-ClassEditor`:** While `OpenPonk-Core` and `OpenPonk-OntoUML-Verifications` use hyphenated test tags without spaces (`Controllers-Tests`, `UndoRedo-Tests`, `Verification-Tests`, `Antipattern-Tests`), `OpenPonk-ClassEditor` uses spaces around hyphens (`Tests - Controllers`, `Tests - DiagramElements`, `Tests - UI`).

4. **Remaining Dead Packages in Active Repositories:** `Borm-OpenCabe-Importer` (in `borm-editor`), `OP-UML-GTExtensions` (in `uml-metamodel`), and the 7 `[legacy / unmaintained]` packages in `class-editor` still reside in their respective Git repositories as unloaded code.
