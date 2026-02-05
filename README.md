# dependency-patterns

Collection of data dependency patterns used in Eventum.

This repository provides a set of **dependency patterns** - from strict functional dependencies to temporal correlations. Each pattern is designed to be:

- **Executable**: each directory is an Eventum project with source files that can be run.
- **Descriptive**: each use case has a clear definition, rationale, and properties description in its `README.md` file.
- **Testable**: each use case include ClickHouse SQL query to validate properties of generated data.

The repository is intended for:

- **Testing synthetic data generators** to ensure generated datasets satisfy structural and statistical invariants
- **Learning and demonstration**, showing how different types of dependencies can be encoded and validated  
- **Research and experimentation** with synthetic datasets and correlation patterns  

## Usage

Run Eventum and ClickHouse services with `docker-compose`:

```sh
docker-compose up -d
```

Go to <http://localhost:9474> in you browser.
