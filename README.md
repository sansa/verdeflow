# VerdeFlow

VerdeFlow is an open source project for measuring and studying the energy use of software. It is intended for researchers, educators, and practitioners who need reproducible experiments and inspectable results.

## Status

This repository is the starting point for a redesigned VerdeFlow. The earlier prototype is not part of this public repository. Implementation, measurement protocols, and contribution guides will be added as the design is validated.

## Proposed architecture

VerdeFlow will use one public monorepo with a modular control plane and separately deployed measurement components:

- **Control plane:** experiment manifests, resource reservations, run orchestration, access control, and result APIs. Its modules may run as an API and a worker.
- **Target agent:** prepares and runs the software under test on a dedicated host.
- **Load generator:** applies a declared workload from a separate host.
- **Collector:** records physical meter readings and other observations independently of the target.
- **Web UI and CLI:** create experiments, inspect data, and export results through the same public API.

An experiment manifest will record the requested workload, exact code and build inputs, hardware and meter configuration, measurement method, and analysis plan. Each run will retain the manifest, raw observations, provenance, quality flags, and derived results so others can inspect and repeat the measurement.

The measurement protocol, calculations, raw data access, and reproducibility features will remain in the open source project. A future commercial offering may add managed hosting and organization features without requiring a separate scientific core.

## Documentation

See the [documentation index](doc/README.md) for research specifications, design material, and references from the earlier prototype.

Contributors and coding agents can use the [GitHub work-start convention](doc/contributing/GITHUB_WORKFLOW.md) when opening issues or creating branches.

## License

VerdeFlow is licensed under the [Apache License 2.0](LICENSE).
