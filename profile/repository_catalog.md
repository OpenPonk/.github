# OpenPonk Complete Repository, Package & Dependency Catalog

This document provides the complete three-level inventory of all 41 repositories in the **`OpenPonk`** GitHub organization (29 active Pharo repositories, 5 non-Pharo infrastructure/web repositories, and 7 deprecated/archived repositories), including `BaselineOf*` groups, package tags, class/extension counts, and bottom-up dependency layers.

For global architectural rules, the five runtime component layers, instance multiplicities, and element lifecycles (`Close`, `Hide`, `Remove`), see [`architecture_core.md`](./architecture_core.md).

## 1. Three-Level Repository & Package Inventory

*(Note: Class and extension counts below are approximate order-of-magnitude indicators rounded upward to account for ongoing growth. Every test package `*-Tests` mirrors the tags of its production package without using the word `Tests`.)*

### CATEGORY 1: Core & Distribution (2 Repositories)

#### 1. `openponk`
* **Purpose:** Core of the OpenPonk modeling platform (ViewModels, Controllers, DiagramElements, Roassal3 diagram rendering, Spec2 workbench GUI, persistence, Undo/Redo).
* **Baseline (`BaselineOfOpenPonk`):**
  * **External dependencies:** `Roassal` (loads `#Full`), `RoassalExporters`, `Magritte` (loads `Core`, `Magritte-Morph`), `Mocketry`, `XPath`, `NeoJSON`.
  * **Internal dependencies:** `synchronized-links` (`OpenPonkSynchronizedLinks`), `zoomer` (`OpenPonkZoomer`).
  * **Groups:** `default` (`OpenPonk-Roassal`, `OpenPonk-Roassal-Tests`, `OpenPonk-Core`, `OpenPonk-Core-Tests`, `OpenPonk-Morphic`, `OpenPonk-Morphic-Tests`, `OpenPonk-Spec`, `OpenPonk-Spec-Tests`), `dev-plugin` (`OpenPonk-DevTools`, `OpenPonk-DevTools-Tests`), `dev` (`default`, `dev-plugin`).
* **Packages:**
  * **`OpenPonk-Roassal`** `(~65 classes, ~65 ext)` & **`OpenPonk-Roassal-Tests`** `(~15 classes)`
    * **Tags:** `[AnchorConstraint, AttachPoints, CreationFeedback, Exports, Interactions, Manifest, Markers, Meta, Polyline]`
    * **Internal baseline requires:** `Roassal`, `RoassalExporters`
    * **Role:** Extension layer over Roassal3 for incremental diagram rendering, polylines/edges, anchor constraints, attach points, creation feedback, markers, and canvas exports.
  * **`OpenPonk-Core`** `(~165 classes, ~15 ext)` & **`OpenPonk-Core-Tests`** `(~85 classes, 1 ext)`
    * **Tags:** `[Announcements, Controllers, DiagramElements, DiagramOnlyElements, Icons, Layouting, LegacySupport-Diagrams, Manifest, Models, Persistence, Plugin, Templates, UndoRedo, Version, ViewModels]`
    * **Internal baseline requires:** `Magritte`, `Mocketry`, `XPath`, `NeoJSON`, `OpenPonk-Roassal`
    * **Role:** Core platform architecture: **Projects** (`OPProject`, `OPProjectController`), **ViewModels** (unified API over domain models, model structure & lifecycle, GUI labels/icons), **Controllers** (`OPController`, `OPDiagramController`, `OPElementController`, `OPHeadlessCanvasPresenter`), **DiagramElements** (`OPDiagram`, `OPOpenPonkDiagram`, `OPDiagramElement`, visual shapes), plugin registration (`OPPlugin`), project & diagram serialization, UI commands, and Undo/Redo. Never references `OpenPonk-Spec`.
  * **`OpenPonk-Morphic`** `(<10 classes, <5 ext)` & **`OpenPonk-Morphic-Tests`** `(1 class)`
    * **Tags:** `[Manifest, Morphic]`
    * **Internal baseline requires:** *(none)*
    * **Role:** Low-level Morphic components and layout adjustments.
  * **`OpenPonk-Spec`** `(~170 classes, ~25 ext)` & **`OpenPonk-Spec-Tests`** `(~55 classes, 5 ext)`
    * **Tags:** `[Application, Commands, Commands-CanvasPresenter, Commands-ProjectTree, Commands-RoassalShape, Commands-Workbench, Dialogs, Export, Form, Layouting, Manifest, Palettes, Windows]`
    * **Internal baseline requires:** `Magritte`, `OpenPonkSynchronizedLinks`, `OpenPonkZoomer`, `OpenPonk-Morphic`, `OpenPonk-Core` (`OpenPonk-Spec-Tests` requires `OpenPonk-Spec`, `OpenPonk-Core-Tests`, and `OpenPonk-Morphic-Tests`)
    * **Role:** Main Spec2 GUI (`OPWorkbench`, `OPEditor`, `OPCanvasPresenter`, Project Tree, tool palettes, `OPPropertiesForm` / `OPDynamicForm` declarative property forms and `OPPropertiesFormMagritteVisitor` Magritte alternative, dialogs). `OpenPonk-Spec-Tests` also hosts reusable abstract plugin test suites (`OPAbstractPluginExampleViewModelTest`, `OPAbstractPluginExampleLifecycleTest`, `OPAbstractPluginExampleRoundtripTest`, `OPExamplePersistenceTest`, `OPLegacyProjectLoadTest`, `OPLegacyProjectLoadTestGenerator`).
  * **`OpenPonk-DevTools`** `(~15 classes, <5 ext)` & **`OpenPonk-DevTools-Tests`** `(2 classes)`
    * **Tags:** `[Export, Manifest, Releases, Watchdog]`
    * **Internal baseline requires:** `OpenPonk-Core`, `OpenPonk-Spec` (loaded via `dev-plugin` group; `OpenPonk-DevTools-Tests` requires `OpenPonk-DevTools` and `OpenPonk-Core-Tests`)
    * **Role:** Developer utilities, release download statistics, test failure exporters, and runtime mutation watchdog tools.

#### 2. `plugins`
* **Purpose:** Umbrella repository for assembling all-in-one OpenPonk builds and distributions.
* **Baseline (`BaselineOfOpenPonkPlugins`):**
  * **External dependencies:** None directly.
  * **Internal dependencies:** `class-editor` (group `complete`), `borm-editor` (group `plugin`), `petrinets` (group `plugin`), `fsm-editor` (group `plugin`), `BPMN` (group `plugin`), `markov-chains` (group `plugin`), `ERD` (group `plugin`), `DEMO` (group `plugin`), `openponk` (group `dev-plugin`, loaded in `dev` group), `uml-bootstrap-generator` (group `dev-plugin`, loaded in `dev` group).
* **Packages:** *(No application packages; contains only `BaselineOfOpenPonkPlugins`.)*

### CATEGORY 2: Optional Platform Extensions (4 Repositories)

#### 3. `openponk-model`
* **Purpose:** Optional foundational domain metamodel providing a quick bootstrap with common model features (`#uuid`, announcements, associations) for notations that do not require an independent custom metamodel.
* **Baseline (`BaselineOfOpenPonkModel`):** External dependencies: None. Internal dependencies: `openponk` (`OpenPonk`, declared in `default` group only; note that `OpenPonk-Model` tag `ViewModels` statically references `OpenPonk-Core`, so `openponk` must load before `openponk-model`).
* **Packages:**
  * **`OpenPonk-Model`** `(~15 classes, 0 ext)` & **`OpenPonk-Model-Tests`** `(5 classes)`
    * **Tags:** `[Manifest, ModelObjects, ViewModels]`
    * **Internal baseline requires:** None declared in `plugin` group (`OpenPonk` and `OpenPonk-Model-Tests` in `default` group; statically depends on `OpenPonk-Core` via `ViewModels`).
    * **Role:** Reusable base classes for domain models (`OPModelObject`, `OPDirectedAssociation`, etc.) and their base view models.

#### 4. `simulation`
* **Purpose:** Discrete-event and step-by-step simulation framework for OpenPonk models and diagrams.
* **Baselines (Dual-Baseline Architecture):**
  1. **`BaselineOfOpenPonkModelSimulation`:** Internal dependencies: `openponk-model` (`OpenPonkModel`). Loads: `OpenPonk-ModelSimulation`, `OpenPonk-ModelSimulation-Tests`.
  2. **`BaselineOfOpenPonkSimulation`:** Internal dependencies: `openponk` (`OpenPonk`), `OpenPonkModelSimulation`. Loads: `OpenPonk-Simulation`, `OpenPonk-Simulation-Tests`.
* **Packages:**
  * **`OpenPonk-ModelSimulation`** `(~20 classes, 0 ext)` & **`OpenPonk-ModelSimulation-Tests`** `(~20 classes)`
    * **Tags:** `[Events, Manifest, Simulators]`
    * **Internal baseline requires:** `OpenPonkModel`
    * **Role:** GUI-independent model simulation framework (event queues, step simulators).
  * **`OpenPonk-Simulation`** `(<10 classes, 0 ext)` & **`OpenPonk-Simulation-Tests`** `(<10 classes)`
    * **Tags:** `[Manifest, Simulation]`
    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModelSimulation`
    * **Role:** Bridges model simulators with OpenPonk GUI and diagram controllers.

#### 5. `validation`
* **Purpose:** Model validation framework, transient diagram highlighting, and Spec2 diagnostic UI for OpenPonk models and diagrams.
* **Baseline (`BaselineOfOpenPonkValidation`):** External dependencies: None. Internal dependencies: `openponk` (`OpenPonk`, loaded in `default` group; `model` group loads headless `OpenPonk-ModelValidation` + `-Tests` with zero `OpenPonk` dependencies; `plugin` group loads `model` plus `OpenPonk-Validation` + `-Tests`).
* **Packages:**
  * **`OpenPonk-ModelValidation`** `(<10 classes, 0 ext)` & **`OpenPonk-ModelValidation-Tests`** `(~15 classes)`
    * **Tags:** `[Messages, Results, Rules, Settings, Validators]`
    * **Internal baseline requires:** *(none)*
    * **Role:** GUI-independent pure-Pharo model validation engine (declarative rules, value-equal diagnostics, result aggregation, hierarchy-based validators, and Pharo Settings integration).
  * **`OpenPonk-Validation`** `(<10 classes, 0 ext)` & **`OpenPonk-Validation-Tests`** `(~15 classes)`
    * **Tags:** `[Commands, Validation]`
    * **Internal baseline requires:** `OpenPonk-ModelValidation` (and `OpenPonk` in `default`)
    * **Role:** Bridges model validators with OpenPonk diagrams (`OPValidationRegistry`, `OPValidationSession` using `OPDiagramHighlighter`, Spec2 `OPValidationResultsGUI`, and base Commander model commands).

#### 6. `openponk-git`
* **Purpose:** Git integration for uploading and downloading a diagram and its underlying model to/from remote Git repositories (GitHub, GitLab.com) directly from the OpenPonk workbench.
* **Architectural & Verification Note:** Loaded into the all-in-one build via `class-editor` (`complete-plugin`). While toolbar submenu population (`Model -> Git model repository`) is covered by a unit test (`OPGitModelMenuTest` in `OpenPonk-Git-Tests`), end-to-end upload/download functionality was historically tested only with UML diagrams and has not been actively used or verified in recent practice.
* **Baseline (`BaselineOfOpenPonkGit`):** External dependencies: None. Internal dependencies: `openponk` (`OpenPonk`).
* **Packages:**
  * **`OpenPonk-Git`** `(~20 classes, 0 ext)` & **`OpenPonk-Git-Tests`** `(10 classes)`
    * **Tags:** `[Commands, GUI, Manifest, Model]`
    * **Internal baseline requires:** `OpenPonk`
    * **Role:** Toolbar commands and dialogs for pushing/fetching a project (diagram + model) to/from GitHub and GitLab.

### CATEGORY 3: Notations (Editors & Models) (9 Repositories)

#### 7. `class-editor`
* **Purpose:** UML Class Diagram editor plugin (with generic UML profile support).
* **Architectural Note:** `OpenPonk-ClassEditor` itself is a pure UML class diagram editor and contains no OntoUML-specific logic. OntoUML is implemented in separate repositories that extend `class-editor`. While `OpenPonk-ClassEditor` can theoretically be loaded standalone without OntoUML (via the `plugin` or `default` baseline groups), standard builds always load the `complete` group, which uses `OpenPonk-ClassEditor-Plugins` to pull in the OntoUML ecosystem and `openponk-git`.
* **Baseline (`BaselineOfOpenPonkClassEditor`):**
  * **External dependencies:** None directly (transitively pulls `XMLWriter`, `XMLParser` via dependencies).
  * **Internal dependencies:**
    * Group `plugin` (loads `OpenPonk-ClassEditor`, `OpenPonk-ClassEditor-Tests`): requires `uml-xmi` (`OpenPonkUMLXMI`), `uml-bootstrap-generator` (`OpenPonkUMLBootstrapGenerator`), `uml-profiles` (`OpenPonkUMLProfiles`).
    * Group `complete-plugin` (loads `OpenPonk-ClassEditor-Plugins`): additionally requires `ontouml-verifications` (`OpenPonkOntoUMLVerifications`), `ontouml-patterns` (`OpenPonkOntoUMLPatterns`), `openponk-ontouml-to-uml-transformation` (`OpenPonkOntoUMLTransformations`), `openponk-git` (`OpenPonkGit`). To break the cycle with OntoUML packages depending back on `class-editor`, `baseline:` skips `OpenPonk-ClassEditor-Plugins` once `BaselineOfOpenPonkOntoUMLVerifications` is already loaded in `Smalltalk globals`.
    * Group `default`: `openponk` + `plugin`. Group `complete`: `default` + `complete-plugin`.
* **Active Packages:**
  * **`OpenPonk-ClassEditor`** `(~130 classes, ~25 ext)` & **`OpenPonk-ClassEditor-Tests`** `(~55 classes)`
    * **Tags:** `[Announcements, Commands, Commands-ProjectTree, Commands-RoassalShape, Controllers, DiagramElements, Examples, Forms, Manifest, Persistence, Plugin, Profiles, UI, ViewModels]`
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

#### 8. `borm-model`
* **Purpose:** Domain metamodel for the BORM (Business Object Relation Modeling) notation.
* **Baseline (`BaselineOfOpenPonkBormModel`):** External dependencies: None. Internal dependencies: `openponk-model` (`OpenPonkModel`), `synchronized-links` (`OpenPonkSynchronizedLinks`).
* **Packages:**
  * **`OpenPonk-BormModel`** `(~30 classes, 0 ext)` & **`OpenPonk-BormModel-Tests`** `(~10 classes)`
    * **Tags:** `[Conditions, Factory, Manifest, Models]`
    * **Internal baseline requires:** `OpenPonkModel`, `OpenPonkSynchronizedLinks`
    * **Role:** BORM domain model entities (participants, states, activities, transitions, communications, constraints/conditions).

#### 9. `borm-editor`
* **Purpose:** Diagram editor and simulator for BORM Object Relation Diagrams (ORD).
* **Baseline (`BaselineOfOpenPonkBormEditor`):** External dependencies: `PetitParser`. Internal dependencies: `borm-model` (`OpenPonkBormModel`), `simulation` (`OpenPonkSimulation`), `openponk` (`OpenPonk`). Groups: `default`, `plugin`, `no-simulation`, `no-simulation-plugin`.
* **Packages:**
  * **`OpenPonk-BormEditor`** `(~50 classes, <10 ext)` & **`OpenPonk-BormEditor-Tests`** `(~30 classes)`
    * **Tags:** `[Commands, Controllers, DiagramElements, Dsl, Examples, Icons, Manifest, UI, ViewModels]`
    * **Internal baseline requires:** `PetitParser`, `OpenPonkBormModel`
    * **Role:** BORM diagram view models, controllers, shapes, textual BormLang DSL (`BormLangGrammar`, `BormLangParser`, `BormLangSerializer`, `BormLangHighlighter`, `OPBormDslEditorCommand`, legacy Spec1 `OPDslEditor`), and UI commands.
  * **`OpenPonk-BormSimulation`** `(~15 classes, 0 ext)` & **`OpenPonk-BormSimulation-Tests`** `(<10 classes)`
    * **Tags:** `[Manifest, Simulation]`
    * **Internal baseline requires:** `OpenPonk-BormEditor`, `OpenPonkSimulation` (`OpenPonk-BormSimulation-Tests` requires `OpenPonk-BormSimulation` and `OpenPonk-BormEditor-Tests`)
    * **Role:** Token-based process simulator and visualization for BORM diagrams.
  * **`Borm-OpenCabe-Importer`** `[deprecated]` – Unloaded importer for the defunct Eclipse-based OpenCABE tool. Do not use.

#### 10. `BPMN`
* **Purpose:** Editor, validator, simulator, and XML serializer for BPMN 2.0 diagrams.
* **Baseline (`BaselineOfOpenPonkBPMN`):** External dependencies: None. Internal dependencies: `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`), `simulation` (`OpenPonkSimulation`).
* **Packages:**
  * **`OpenPonk-BPMN`** `(~115 classes, <5 ext)` & **`OpenPonk-BPMN-Tests`** `(~35 classes)`
    * **Tags:** `[Announcements, Commands, Controllers, DiagramElements, Examples, Factory, Icons, Manifest, Models, Plugin, ViewModels]`
    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`
    * **Role:** Core BPMN 2.0 domain model, view models, diagram elements, controllers, and workbench plugin.
  * **`OpenPonk-BPMN-XML`** `(<10 classes, 0 ext)` & **`OpenPonk-BPMN-XML-Tests`** `(2 classes)`
    * **Tags:** `[Manifest, Writer]`
    * **Internal baseline requires:** `OpenPonk-BPMN`
    * **Role:** BPMN 2.0 XML export.
  * **`OpenPonk-BPMN-Validation`** `(~55 classes, 0 ext)` & **`OpenPonk-BPMN-Validation-Tests`** `(~25 classes)`
    * **Tags:** `[Controllers, Manifest, Messages, Results, Settings, UI, Validation-Classes]`
    * **Internal baseline requires:** `OpenPonk-BPMN`
    * **Role:** BPMN 2.0 structural/semantic validation rules and diagnostic UI.
  * **`OpenPonk-BPMN-Simulation`** `(~25 classes, 0 ext)` & **`OpenPonk-BPMN-Simulation-Tests`** `(~15 classes)`
    * **Tags:** `[Manifest, Simulation]`
    * **Internal baseline requires:** `OpenPonk-BPMN`, `OpenPonkSimulation`
    * **Role:** BPMN token-flow simulation engine and UI integration.

#### 11. `petrinets`
* **Purpose:** Petri Nets diagram editor and simulator (Place/Transition nets and optional Boolean Colored Petri Nets).
* **Baseline (`BaselineOfOpenPonkPetriNets`):** External dependencies: None. Internal dependencies: `openponk` (`OpenPonk`), `simulation` (`OpenPonkSimulation`, which transitively provides `openponk-model`). Defines a separate `bcpn` group for Boolean Colored Petri Nets.
* **Packages:**
  * **`OpenPonk-PetriNets`** `(~70 classes, 0 ext)` & **`OpenPonk-PetriNets-Tests`** `(~30 classes)`
    * **Tags:** `[Announcements, Controllers, DiagramElements, Examples, Manifest, Models, Simulation, ViewModels]`
    * **Internal baseline requires:** `OpenPonkSimulation` (and `OpenPonk`)
    * **Role:** Place/Transition Petri net domain model, view models, visual shapes, controllers, and token-firing simulation.
  * **`OpenPonk-BooleanColoredPetriNets`** `[outdated]` – Loaded only via explicit `bcpn` baseline group (requires `OpenPonk-PetriNets`); excluded from the default build, but retained in the baseline and potentially functional.

#### 12. `fsm-editor`
* **Purpose:** Finite-State Machine (FSM) diagram editor, simulator, and GraphML serializer.
* **Baseline (`BaselineOfOpenPonkFsmEditor`):** External dependencies: `Mustache`. Internal dependencies: `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`).
* **Packages:**
  * **`OpenPonk-FSM`** `(~55 classes, 0 ext)` & **`OpenPonk-FSM-Tests`** `(~15 classes)`
    * **Tags:** `[Controllers, DiagramElements, Examples, Icons, Manifest, Models, Plugin, Serialization, Simulations, ViewModels]`
    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`, `Mustache`
    * **Role:** FSM metamodel, view models, diagram editor, state simulation, and GraphML serialization (`OPFsmGraphML`).

#### 13. `markov-chains`
* **Purpose:** Markov Chains diagram editor, simulator, and solver integration.
* **Baseline (`BaselineOfOpenPonkMarkovChains`):** External dependencies: `Mustache`. Internal dependencies: `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`).
* **Packages:**
  * **`OpenPonk-MarkovChains`** `(~50 classes, 0 ext)` & **`OpenPonk-MarkovChains-Tests`** `(<10 classes)`
    * **Tags:** `[Commands, Controllers, DiagramElements, Examples, Icons, Manifest, Models, Plugin, Serialization, Simulations, ViewModels, Zinc]`
    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`, `Mustache`
    * **Role:** Markov chain model, view models, visual editor, serialization, simulation, and Zinc HTTP client integration.

#### 14. `ERD`
* **Purpose:** Entity-Relationship Diagram (ERD) editor and SQL/DDL generator.
* **Baseline (`BaselineOfOpenPonkERD`):** External dependencies: `Mustache`. Internal dependencies: `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`).
* **Packages:**
  * **`OpenPonk-ERD`** `(~55 classes, 0 ext)` & **`OpenPonk-ERD-Tests`** `(~25 classes)`
    * **Tags:** `[Commands, Controllers, DiagramElements, Examples, Icons, Manifest, Models, Plugin, Serialization, ViewModels]`
    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`, `Mustache`
    * **Role:** ER domain model (entities, attributes, keys, relationships, cardinalities), view models, diagram editor, and Mustache-based serialization/export.

#### 15. `DEMO`
* **Purpose:** DEMO (Design & Engineering Methodology for Organizations) notation plugin.
* **Baseline (`BaselineOfOpenPonkDEMO`):** External dependencies: None. Internal dependencies: `openponk` (`OpenPonk`), `openponk-model` (`OpenPonkModel`). *(Supersedes the deprecated `demo-editor` repository).*
* **Packages:**
  * **`OpenPonk-DEMO`** `(~75 classes, 0 ext)` & **`OpenPonk-DEMO-Tests`** `(~25 classes)`
    * **Tags:** `[Controllers, DiagramElements, Examples, Icons, Manifest, Models, Plugin, ViewModels]`
    * **Internal baseline requires:** `OpenPonk`, `OpenPonkModel`
    * **Role:** DEMO enterprise ontology diagrams, models, view models, controllers, and shapes.

### CATEGORY 4: UML & OntoUML Ecosystem (9 Repositories)

#### 16. `uml-metamodel`
* **Purpose:** OMG UML 2.5 metamodel implementation in Pharo (generated from OMG UML 2.5 XMI specifications).
* **Architectural Warning:** `OpenPonk-UML-Metamodel` consists of **generated code** (produced by `uml-bootstrap-generator`). Never modify classes or methods in `OpenPonk-UML-Metamodel` manually without updating the generator in `uml-bootstrap-generator`.
* **Baseline (`BaselineOfOpenPonkUMLMetamodel`):** External dependencies: None. Internal dependencies: None.
* **Packages:**
  * **`OpenPonk-UML-Meta`** `(<10 classes, 0 ext)` & **`OpenPonk-UML-Meta-Tests`** `(<10 classes)`
    * **Tags:** `[Manifest, Models]`
    * **Internal baseline requires:** None.
    * **Role:** Hand-written meta-layer supporting UML metamodel reflection, opposite properties, and slot descriptors.
  * **`OpenPonk-UML-Metamodel`** `(~260 classes, 0 ext)`
    * **Tags:** `[Actions, Activities, Classification, CommonBehavior, CommonStructure, Deployments, InformationFlows, Interactions, Manifest, Meta, Packages, SimpleClassifiers, StateMachines, StructuredClassifiers, UseCases, Values]`
    * **Internal baseline requires:** `OpenPonk-UML-Meta`
    * **Role:** Generated implementation of the OMG UML 2.5 metamodel from OMG XMI specifications.
  * **`OP-UML-GTExtensions`** `[deprecated]` – Unloaded extensions for legacy GTInspector and Roassal2 (replaced by OpenPonk's Spec2 Project Tree). Do not use.

#### 17. `uml-bootstrap-generator`
* **Purpose:** Lightweight UML profile code generator (`default`) and OMG UML 2.5 metamodel bootstrap generator (`dev-plugin`).
* **Architectural Note:** Only `OpenPonk-UML-ProfileGenerator` (and its tests) loads in `default` (consumed by `class-editor` for `OPUmlProfileGeneratorWizard` and by `ontouml-profile` for `OntoUMLProfileFactory`). The generated bootstrap metamodel (`OpenPonk-UML-Bootstrap`) and metamodel generator (`OpenPonk-UML-BootstrapGenerator`) load exclusively via `dev-plugin` / `dev` (loaded in `BaselineOfOpenPonkPlugins` group `dev`).
* **Baseline (`BaselineOfOpenPonkUMLBootstrapGenerator`):** External dependencies: None directly. Internal dependencies: `xmi` (`OpenPonkXMI`), `uml-xmi` (`OpenPonkUMLXMI`), `pharo-changes-builder` (`ChangesBuilder`). Groups: `default` (`OpenPonk-UML-ProfileGenerator`, `OpenPonk-UML-ProfileGenerator-Tests`), `dev-plugin` (`OpenPonk-UML-Bootstrap`, `OpenPonk-UML-BootstrapGenerator`, `OpenPonk-UML-BootstrapGenerator-Tests`), `dev` (`default`, `dev-plugin`).
* **Packages:**
  * **`OpenPonk-UML-ProfileGenerator`** `(<5 classes, 0 ext)` & **`OpenPonk-UML-ProfileGenerator-Tests`** `(<5 classes)`
    * **Tags:** `[Manifest, Profile-Generator, Utilities]`
    * **Internal baseline requires:** `ChangesBuilder`, `OpenPonkUMLXMI`
    * **Role:** Lightweight UML profile generator producing Pharo profile classes, extension methods, and profile factories without depending on `OpenPonk-UML-Bootstrap`.
  * **`OpenPonk-UML-Bootstrap`** `(~50 classes, 0 ext)` *(loaded via `dev-plugin`)*
    * **Tags:** `[Classification, CommonStructure, Deployments, Manifest, Packages, SimpleClassifiers, StructuredClassifiers, Super, Values]`
    * **Internal baseline requires:** None.
    * **Role:** Minimal bootstrap metamodel used to parse raw OMG UML XMI specification files.
  * **`OpenPonk-UML-BootstrapGenerator`** `(~15 classes, 0 ext)` & **`OpenPonk-UML-BootstrapGenerator-Tests`** `(~11 classes)` *(loaded via `dev-plugin`)*
    * **Tags:** `[Bootstrap-Generator, Manifest, Metamodel-Generator, Reader, Templates]`
    * **Internal baseline requires:** `OpenPonkXMI`, `ChangesBuilder`, `OpenPonk-UML-Bootstrap`, `OpenPonk-UML-ProfileGenerator` (`OpenPonk-UML-BootstrapGenerator-Tests` additionally requires `OpenPonk-UML-ProfileGenerator-Tests`)
    * **Role:** Code generators producing the bootstrap model and `OpenPonk-UML-Metamodel`.

#### 18. `uml-profiles`
* **Purpose:** Core UML profile application mechanism and small example/test profiles.
* **Baseline (`BaselineOfOpenPonkUMLProfiles`):** External dependencies: None. Internal dependencies: `uml-metamodel` (`OpenPonkUMLMetamodel`), `uml-xmi` (`OpenPonkUMLXMI`).
* **Packages:**
  * **`OpenPonk-UML-Profiles`** `(<10 classes, <10 ext)` & **`OpenPonk-UML-Profiles-Tests`** `(3 classes)` – **Tags:** `[Manifest, Profile]` (tests: `[ClockProfile, IssuesProfile, Profile]`); **Requires:** `OpenPonkUMLMetamodel`, `OpenPonkUMLXMI`; **Role:** Core profile registration and stereotype application mechanism extending UML metamodel elements.
  * **`OpenPonk-ClockProfile`** `(<10 classes, <5 ext)` – **Tags:** `[Manifest, Metamodel, Model]`; **Requires:** `OpenPonk-UML-Profiles`; **Role:** Minimal example/test UML profile demonstrating how a profile is implemented and generated.
  * **`OpenPonk-IssuesProfile`** `(<10 classes, <5 ext)` – **Tags:** `[Manifest, Metamodel, Model]`; **Requires:** `OpenPonk-UML-Profiles`; **Role:** Minimal example/test UML profile demonstrating how a profile is implemented and generated.

#### 19. `uml-xmi`
* **Purpose:** XMI serialization and deserialization for `OpenPonk-UML-Metamodel` and applied UML profiles.
* **Baseline (`BaselineOfOpenPonkUMLXMI`):** External dependencies: None directly. Internal dependencies: `xmi` (`OpenPonkXMI`), `uml-metamodel` (`OpenPonkUMLMetamodel`).
* **Packages:**
  * **`OpenPonk-UML-XMI`** `(<10 classes, 0 ext)` & **`OpenPonk-UML-XMI-Tests`** `(3 classes)`
    * **Tags:** `[Manifest, Reader, Utility, Writer]`
    * **Internal baseline requires:** `OpenPonkXMI`, `OpenPonkUMLMetamodel`
    * **Role:** Converts between generic `OpenPonk-XMI` graphs and `OpenPonk-UML-Metamodel` instances (including stereotypes).

#### 20. `ontouml-profile`
* **Purpose:** OntoUML stereotype profile and UI extension plugin for the UML Class Editor.
* **Baseline (`BaselineOfOpenPonkOntoUMLProfile`):** External dependencies: None. Internal dependencies: `class-editor` (`OpenPonkClassEditor`).
* **Packages:**
  * **`OpenPonk-OntoUML-Profile`** `(~45 classes, ~10 ext)` & **`OpenPonk-OntoUML-Profile-Tests`** `(3 classes)`
    * **Tags:** `[Examples, Manifest, Model]`
    * **Internal baseline requires:** `OpenPonkClassEditor`
    * **Role:** OntoUML profile implementation (stereotypes such as `Kind`, `SubKind`, `Role`, `Phase`, `Relator`, `Mode`, etc.; originally scaffolded via `OntoUMLProfileFactory`, but maintained as a regular package—not locked generated code like `OpenPonk-UML-Metamodel`).
  * **`OpenPonk-OntoUML-Plugin`** `(<10 classes, <10 ext)` & **`OpenPonk-OntoUML-Plugin-Tests`** `(10 classes)`
    * **Tags:** `[Icons, Manifest, Menu, Plugin]`
    * **Internal baseline requires:** `OpenPonk-OntoUML-Profile`
    * **Role:** Extends `OpenPonk-ClassEditor` with OntoUML stereotype menus, icons, and visual conventions.

#### 21. `ontouml-modelquery`
* **Purpose:** Query layer for inspecting and filtering OntoUML-stereotyped UML models.
* **Baseline (`BaselineOfOpenPonkOntoUMLModelQuery`):** External dependencies: None. Internal dependencies: `ontouml-profile` (`OpenPonkOntoUMLProfile`).
* **Packages:**
  * **`OpenPonk-OntoUML-ModelQuery`** `(<10 classes, 0 ext)` & **`OpenPonk-OntoUML-ModelQuery-Tests`** `(1 class)`
    * **Tags:** `[Manifest, Uncategorized]`
    * **Internal baseline requires:** `OpenPonkOntoUMLProfile`
    * **Role:** Query utilities over OntoUML classes, associations, generalizations, and stereotypes.

#### 22. `ontouml-patterns`
* **Purpose:** Ontology-driven design patterns catalog and instantiation UI for OntoUML.
* **Baseline (`BaselineOfOpenPonkOntoUMLPatterns`):** External dependencies: None. Internal dependencies: `ontouml-profile` (`OpenPonkOntoUMLProfile`).
* **Packages:**
  * **`OpenPonk-OntoUML-Patterns`** `(~40 classes, 0 ext)` & **`OpenPonk-OntoUML-Patterns-Tests`** `(2 classes)`
    * **Tags:** `[Commands, Manifest, Patterns, UI]`
    * **Internal baseline requires:** `OpenPonkOntoUMLProfile`
    * **Role:** Pattern templates, wizards, and commands for applying OntoUML design patterns in the Class Editor.

#### 23. `ontouml-verifications`
* **Purpose:** OntoUML model verification and ontological anti-pattern detection.
* **Baseline (`BaselineOfOpenPonkOntoUMLVerifications`):** External dependencies: None. Internal dependencies: `ontouml-modelquery` (`OpenPonkOntoUMLModelQuery`).
* **Packages:**
  * **`OpenPonk-OntoUML-Verifications`** `(~55 classes, 0 ext)` & **`OpenPonk-OntoUML-Verifications-Tests`** `(~40 classes)`
    * **Tags:** `[Antipattern-Classes, Commands, Controllers, DocumentationReferences, Manifest, Messages, Results, Settings, Traits, UI, Verification-Classes]`
    * **Internal baseline requires:** `OpenPonkOntoUMLModelQuery`
    * **Role:** OntoUML syntax/semantics verification rules, anti-pattern detectors, and diagnostic UI.

#### 24. `openponk-ontouml-to-uml-transformation`
* **Purpose:** Model-to-model transformation from conceptual OntoUML models to implementation-level UML models.
* **Baseline (`BaselineOfOpenPonkOntoUMLTransformations`):** External dependencies: None. Internal dependencies: `ontouml-modelquery` (`OpenPonkOntoUMLModelQuery`).
* **Packages:**
  * **`OpenPonk-OntoUML-Transformations`** `(~15 classes, 0 ext)` & **`OpenPonk-OntoUML-Transformations-Tests`** `(2 classes)`
    * **Tags:** `[Commands, Controllers, Manifest, Transformations, Transformers, UI]`
    * **Internal baseline requires:** `OpenPonkOntoUMLModelQuery`
    * **Role:** Transforms OntoUML constructs into standard implementation UML class models.

### CATEGORY 5: Shared Libraries & Utilities (5 Repositories)

#### 25. `synchronized-links`
* **Purpose:** Self-synchronizing bidirectional relations for Smalltalk objects.
* **Baseline (`BaselineOfOpenPonkSynchronizedLinks`):** Dependencies: None. *(Consumed by `openponk` and `borm-model`.)*
* **Packages:**
  * **`OpenPonk-SynchronizedLinks`** `(<10 classes, 0 ext)` & **`OpenPonk-SynchronizedLinks-Tests`** `(<10 classes)` – **Tags:** `[Links, Manifest]`; **Role:** Maintains bidirectional consistency across $1:1$, $1:N$, and $M:N$ object references.

#### 26. `zoomer`
* **Purpose:** Tool for setting sharp, device-appropriate UI zoom levels.
* **Baseline (`BaselineOfOpenPonkZoomer`):** Dependencies: None. *(Consumed by `openponk` in `OpenPonk-Spec`.)*
* **Packages:**
  * **`OpenPonk-Zoomer`** `(<10 classes, 0 ext)` & **`OpenPonk-Zoomer-Tests`** `(2 classes)` – **Tags:** `[Model, UI]`; **Role:** World/canvas scaling widget and zoom factor management.

#### 27. `xmi`
* **Purpose:** Generic XMI file to XMI object-graph materialization and serialization.
* **Architectural Note:** Written as a generic XMI library, but consumed exclusively by the UML ecosystem (`uml-xmi`, `uml-bootstrap-generator`).
* **Baseline (`BaselineOfOpenPonkXMI`):** External dependencies: `XMLWriter` (loads `Core`). Internal dependencies: `xml-dom-visitor` (`OpenPonkXMLDOMVisitor`).
* **Packages:**
  * **`OpenPonk-XMI`** `(~15 classes, 0 ext)` & **`OpenPonk-XMI-Tests`** `(<10 classes)` – **Tags:** `[DOM, Manifest, Reader, Writer]`; **Requires:** `OpenPonkXMLDOMVisitor`, `XMLWriter`; **Role:** Reads and writes XMI files into/from intermediate XMI element graphs.
  * **`OpenPonk-XMI-Inspection`** `(<5 classes, ~10 ext)` & **`OpenPonk-XMI-Inspection-Tests`** `(1 class)` – **Tags:** `[Manifest]` (tests: `[Inspection]`); **Requires:** `OpenPonk-XMI`; **Role:** Inspector extensions for inspecting XMI graph nodes.

#### 28. `xml-dom-visitor`
* **Purpose:** Visitor pattern implementation for `XMLParser` DOM trees.
* **Architectural Note:** Required by `BaselineOfOpenPonkXMI`. Adds `#accept:` extension methods to `XMLCData`, `XMLDocument`, `XMLElement`, and `XMLString`; `OpenPonk-XMI` (`OPXMIReaderVisitor`) subclasses `XMLDOMVisitor`, while `XMLDOMElementVisitor` and `XMLDOMTypedElement` remain standalone public API classes without other OpenPonk references.
* **Baseline (`BaselineOfOpenPonkXMLDOMVisitor`):** External dependencies: `XMLParser` (loads `Core`). Internal dependencies: None.
* **Packages:**
  * **`OpenPonk-XMLDOMVisitor`** `(<10 classes, <10 ext)` & **`OpenPonk-XMLDOMVisitor-Tests`** `(4 classes)` – **Tags:** `[DOM, Manifest, Visitor]`; **Requires:** `XMLParser`; **Role:** Visitor double-dispatch and typed element wrappers over XML DOM nodes.

#### 29. `pharo-changes-builder`
* **Purpose:** Lightweight wrapper simplifying common `RBRefactoring` operations during programmatic code generation.
* **Architectural Note:** Actively used for code generation in `uml-bootstrap-generator`, `ontouml-profile` (`OntoUMLProfileFactory`), and `class-editor` (`OPUmlProfileGeneratorWizard`).
* **Baseline (`BaselineOfChangesBuilder`):** Dependencies: None.
* **Packages:**
  * **`ChangesBuilder`** `(~10 classes, <10 ext)` & **`ChangesBuilder-Tests`** `(10 classes)` – **Tags:** `[Model, Utilities]`; **Role:** Programmatic creation of packages, classes, instance variables, and methods via refactoring changesets.

### CATEGORY 6: Infrastructure, Web & Specifications – Non-Pharo Repositories (5 Repositories)
* **30. `.github`** – Organization-level GitHub profile (`profile/README.md`) and shared architectural/developer documentation (`architecture_core.md`, `repository_catalog.md`).
* **31. `ci-scripts`** – Shared GitHub Actions workflows and CI scripts used across OpenPonk repositories. *(Note: Individual Pharo repositories often define `.github/dependent-repositories.txt` listing downstream repositories whose CI builds are automatically triggered on push).*
* **32. `releaser`** – Python scripts for building and publishing numbered OpenPonk releases.
* **33. `specs-repository`** – Storage for formal specification files (such as OMG UML/XMI specifications consumed by `uml-bootstrap-generator`) and published XMI profile definitions referenced by exported OpenPonk/OntoUML XMI files and saved projects (e.g., `OntoUML/1.0/OntoUML.profile.xmi`, referenced via `https://openponk.org/OntoUML/1.0/`).
* **34. `openponk.github.io`** – Static HTML website for OpenPonk (edited directly in this repository).

### CATEGORY 7: Deprecated & Archived Repositories – Outside Active Builds (7 Repositories)
* **35. `openponk.github.io-sources`** `[deprecated]` – Legacy JavaScript static-site generator for the website; superseded by direct editing in `openponk.github.io`.
* **36. `demo-editor`** `[deprecated -> DEMO]` – Original non-functional implementation of the DEMO editor; completely replaced by `DEMO`.
* **37. `uml-shapes`** `[deprecated -> OpenPonk-Core]` – Early partial implementation of OMG UML Diagram Definition / Diagram Interchange. Generalized for all notations and merged into the `DiagramElements` (Shapes) architecture in `OpenPonk-Core`.
* **38. `file-dialog`** `[deprecated -> NewTools]` – Custom replacement for Pharo's native file dialog; superseded once its concepts were integrated into standard Pharo `NewTools`.
* **39. `uml-playground`** `[deprecated]` – Historical experimental repository from early UML metamodel development.
* **40. `xsd`** `[deprecated]` – Unmaintained XML Schema experiments (never loaded into OpenPonk).
* **41. `xsd-generator`** `[deprecated]` – Unmaintained XSD generator experiments (never loaded into OpenPonk).

## 2. Bottom-Up Dependency Layers & Maintenance Notes

### 2.1 Architectural Layers (Bottom-Up Dependency Flow)
* **Layer 0 — Standalone Low-Level Utilities & Base Metamodels (Zero Internal Dependencies):** `synchronized-links`, `zoomer`, `pharo-changes-builder`, `xml-dom-visitor` (external `XMLParser`), `uml-metamodel` (`OpenPonk-UML-Meta` $\rightarrow$ `OpenPonk-UML-Metamodel`).
* **Layer 1 — Core Infrastructure & Generic XMI Utilities:** `openponk` (`OpenPonk-Roassal` $\rightarrow$ `OpenPonk-Core`; `OpenPonk-Morphic` $\rightarrow$ `OpenPonk-Spec`; `OpenPonk-DevTools`), `xmi` (`OpenPonk-XMI` $\rightarrow$ `OpenPonk-XMI-Inspection`).
* **Layer 2 — Optional Base Metamodel, Validation Framework, Git Integration & UML XMI:** `openponk-model` (statically references `OpenPonk-Core` via `ViewModels`), `validation` (`OpenPonk-ModelValidation` $\rightarrow$ `OpenPonk-Validation`), `openponk-git`, `uml-xmi`.
* **Layer 3 — Domain Models, Simulation Framework, UML Profiles & Profile/Bootstrap Generator:** `borm-model`, `simulation` (`BaselineOfOpenPonkModelSimulation` $\rightarrow$ `BaselineOfOpenPonkSimulation`), `uml-profiles` (`OpenPonk-UML-Profiles` $\rightarrow$ `OpenPonk-ClockProfile`, `OpenPonk-IssuesProfile`), `uml-bootstrap-generator` (`OpenPonk-UML-ProfileGenerator`; `dev-plugin`: `OpenPonk-UML-Bootstrap` $\rightarrow$ `OpenPonk-UML-BootstrapGenerator`).
* **Layer 4 — Notation Editors (Core Plugins):** `class-editor` (`plugin` group), general notation editors (`DEMO`, `fsm-editor`, `markov-chains`, `ERD`), and simulated/validated notation editors (`borm-editor`, `BPMN`, `petrinets`).
* **Layer 5 — OntoUML Ecosystem & Class Editor Extensions:** `ontouml-profile` (`OpenPonk-OntoUML-Profile` $\rightarrow$ `OpenPonk-OntoUML-Plugin`) $\rightarrow$ `ontouml-modelquery`, `ontouml-patterns` $\rightarrow$ `ontouml-verifications`, `openponk-ontouml-to-uml-transformation`, and `class-editor` (`complete-plugin` group: `OpenPonk-ClassEditor-Plugins`).
* **Layer 6 — All-in-One Distribution Assembly:** `plugins` (`BaselineOfOpenPonkPlugins`).

### 2.2 Architectural Observations & Maintenance Notes
1. **Transitive Loading of `openponk-git` via `class-editor`:** `OpenPonk-Git` depends only on `openponk` core and provides workbench-level Git commands (`Model -> Git model repository`), but in the all-in-one build (`BaselineOfOpenPonkPlugins`) it is loaded transitively via `BaselineOfOpenPonkClassEditor` (`complete-plugin` $\rightarrow$ `OpenPonk-ClassEditor-Plugins`) rather than as a direct top-level dependency in `BaselineOfOpenPonkPlugins`.
2. **Public API Visitor Classes in `xml-dom-visitor`:** `OpenPonk-XMI` (`OPXMIReaderVisitor`) subclasses `XMLDOMVisitor`, whereas `XMLDOMElementVisitor` and `XMLDOMTypedElement` are retained as documented standalone public API classes without direct references from other OpenPonk packages.
3. **Remaining Dead Packages in Active Repositories:** `Borm-OpenCabe-Importer` (in `borm-editor`), `OP-UML-GTExtensions` (in `uml-metamodel`), and the 7 `[legacy / unmaintained]` packages in `class-editor` still reside in their respective Git repositories as unloaded code.