# Contributing to OpenPonk

Contributions to any repository in the `OpenPonk` organization are welcome.

## Before You Start

- **Supported Pharo version & installation:** See [openponk.org/#direct-installation](https://openponk.org/#direct-installation).
- **Platform architecture:** Read [Core Runtime Architecture & Component Layers](https://github.com/OpenPonk/.github/blob/main/profile/architecture_core.md).
- **Repository & package map:** Read [Complete Repository, Package & Dependency Catalog](https://github.com/OpenPonk/.github/blob/main/profile/repository_catalog.md).
- **Bug reports & feature requests:** Open an issue in the relevant notation repository, or in [`OpenPonk/openponk`](https://github.com/OpenPonk/openponk/issues) for core platform issues.

## Making Changes in Pharo

1. Fork the target repository on GitHub.
2. Load OpenPonk (or the target repository) into the supported Pharo version following [openponk.org/#direct-installation](https://openponk.org/#direct-installation), pointing Metacello or Iceberg at your fork.
3. Make your changes and add or update SUnit tests in the corresponding `*-Tests` package.
4. Run the test suite for the affected packages to ensure all tests pass.
5. Open **Browse → Iceberg** (Git Repositories Browser), right-click the modified repository, and commit and push your changes to your fork.
6. Open a pull request against the repository's default branch on GitHub.
