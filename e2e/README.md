# TSF E2E Tests

End-to-end test suite for TSF (Trusted Software Factory) instances.

## Prerequisites

- **KUBECONFIG** pointing to a cluster with TSF installed
- **GitHub org** with a fork/clone of [konflux-ci/testrepo](https://github.com/konflux-ci/testrepo)
- **GitHub token** (PAT) with repo access to the org above
- **Quay org** accessible by the TSF instance's image-controller

## IDE Setup

The `e2e/` module is separate from the main CLI module. To get full IDE
navigation across both modules, create a `go.work` file in the repo root:

```
go work init . ./e2e
```

> **Note:** With `go.work` present, CLI builds will fail due to transitive
> dependency conflicts. Use `GOWORK=off make build` to build the CLI, or
> remove `go.work` when you don't need cross-module IDE support.

## Running tests

1. Build the test binary:
   ```
   make build
   ```

2. Set up your env file:
   ```
   cp my-test.env.template my-test.env
   # edit my-test.env and fill in the values
   ```

3. Source the env file and run:
   ```
   source my-test.env
   ./bin/tsf.test --ginkgo.v --ginkgo.label-filter="tsf-demo"
   ```
