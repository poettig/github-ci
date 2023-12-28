# github-ci
Shared CI pipelines

### Push workflow

You can additionally define `npm_build_script` in the `with` inputs if the build script differs from the name `build`.
`build-env` can be left out if there are no enviroment variables needed.

```yaml
name: Build and deploy on push

on:
  push:

jobs:
  on_push:
    uses: poettig/github-ci/workflows/npm_build_and_deploy.yml@main
    secrets: inherit
    with:
      branch: ${{ github.ref_name }}
      node_version: number
      build_env: '{"example": "hello world"}'
      target_host: example.com
      target_user: root
```

### Repository dispatch workflow

This builds on repository dispatch based on the type.
More variables can be defined via the matrix, but it will create a cross product of every possible combination.
You can additionally define `npm_build_script` in the `with` inputs if the build script differs from the name `build`.

```yaml
name: Build and deploy on webhook

on:
  repository_dispatch:
    types:
      - ansible

jobs:
  on_webhook:
    strategy:
      matrix:
        branch:
          - main
          - development
    uses: poettig/github-ci/workflows/npm_build_and_deploy.yml@main
    secrets: inherit
    with:
      branch: ${{ matrix.branch }}
      node_version: number
      build_env: '{"example": "hello world"}'
      target_host: example.com
      target_user: root
```

