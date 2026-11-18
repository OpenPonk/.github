# OpenPonk Modeling Platform

**OpenPonk** is an open extensible metamodeling platform and diagram workbench built in Smalltalk-based [Pharo](https://pharo.org/), using Spec2 for its workbench GUI and Roassal3 for interactive vector canvas rendering. It supports heterogeneous domain metamodels (full OMG UML 2.5, OntoUML, BORM, BPMN 2.0, Petri Nets, FSM, Markov Chains, ERD, and DEMO), model verification, token-based simulation, and XMI/XML/GraphML/DDL serialization.

- **Website & Downloads:** [https://openponk.org](https://openponk.org)
- **All-in-One Build Umbrella:** [`OpenPonk/plugins`](https://github.com/OpenPonk/plugins) (`BaselineOfOpenPonkPlugins`)

## Architecture & Developer Documentation

For contributors and researchers, the platform architecture is documented in two reference guides in this repository:

1. **[Core Runtime Architecture & Component Layers (`architecture_core.md`)](https://github.com/OpenPonk/.github/blob/main/profile/architecture_core.md)**
   Global package and testing rules, the five runtime architectural layers (`Domain Model`, `ViewModel`, `DiagramElement`/`Shape`, `Controller`, `Workbench & Editor GUI`), instance multiplicities, element lifecycles (`Close`, `Hide`, `Remove from Model`), and the compact 28-repository overview.
2. **[Complete Repository, Package & Dependency Catalog (`repository_catalog.md`)](https://github.com/OpenPonk/.github/blob/main/profile/repository_catalog.md)**
   Detailed three-level inventory of all 39 repositories in the `OpenPonk` organization, `BaselineOf*` groups, package tags, external dependencies, and bottom-up dependency layers.

## Active Pharo Repositories Overview (28 Repositories)

| Category | Repositories |
| :--- | :--- |
| **Core & Platform (5)** | [`openponk`](https://github.com/OpenPonk/openponk) · [`openponk-model`](https://github.com/OpenPonk/openponk-model) · [`plugins`](https://github.com/OpenPonk/plugins) · [`simulation`](https://github.com/OpenPonk/simulation) · [`openponk-git`](https://github.com/OpenPonk/openponk-git) |
| **Notation Plugins (9)** | [`class-editor`](https://github.com/OpenPonk/class-editor) · [`borm-model`](https://github.com/OpenPonk/borm-model) · [`borm-editor`](https://github.com/OpenPonk/borm-editor) · [`BPMN`](https://github.com/OpenPonk/BPMN) · [`petrinets`](https://github.com/OpenPonk/petrinets) · [`fsm-editor`](https://github.com/OpenPonk/fsm-editor) · [`markov-chains`](https://github.com/OpenPonk/markov-chains) · [`ERD`](https://github.com/OpenPonk/ERD) · [`DEMO`](https://github.com/OpenPonk/DEMO) |
| **UML & OntoUML Ecosystem (9)** | [`uml-metamodel`](https://github.com/OpenPonk/uml-metamodel) · [`uml-bootstrap-generator`](https://github.com/OpenPonk/uml-bootstrap-generator) · [`uml-profiles`](https://github.com/OpenPonk/uml-profiles) · [`uml-xmi`](https://github.com/OpenPonk/uml-xmi) · [`ontouml-profile`](https://github.com/OpenPonk/ontouml-profile) · [`ontouml-modelquery`](https://github.com/OpenPonk/ontouml-modelquery) · [`ontouml-patterns`](https://github.com/OpenPonk/ontouml-patterns) · [`ontouml-verifications`](https://github.com/OpenPonk/ontouml-verifications) · [`openponk-ontouml-to-uml-transformation`](https://github.com/OpenPonk/openponk-ontouml-to-uml-transformation) |
| **Shared Libraries (5)** | [`synchronized-links`](https://github.com/OpenPonk/synchronized-links) · [`zoomer`](https://github.com/OpenPonk/zoomer) · [`xmi`](https://github.com/OpenPonk/xmi) · [`xml-dom-visitor`](https://github.com/OpenPonk/xml-dom-visitor) · [`pharo-changes-builder`](https://github.com/OpenPonk/pharo-changes-builder) |