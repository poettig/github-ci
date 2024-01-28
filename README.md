# github-ci
Shared CI pipelines

### Example workflow for NPM build + deploy with environments

If environment is empty (neither main nor development branch), the project variable value is used.
If that does not exist, the workflow probably fails, have not tested though.

```yaml
name: Build and deploy

on:
  push:
  repository_dispatch:
    types:
      - ansible-production
      - ansible-staging

concurrency:
  group: ci-${{ github.event.client_payload.branch || github.ref_name }}
  cancel-in-progress: true

jobs:
  build_and_deploy:
    runs-on: ubuntu-latest
    environment: ${{ (github.event.client_payload.branch || github.ref_name) == 'main' && 'production' || ((github.event.client_payload.branch || github.ref_name) == 'development' && 'staging' || '') }}
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.client_payload.branch || github.ref_name }}

      - name: Build ${{ github.repository }}
        uses: poettig/github-ci/build_npm@main
        with:
          node_version: ${{ vars.NODE_VERSION }}

      # Only run if all necessary variables exist in the deploy environment
      - name: Deploy ${{ github.repository }}
        uses: poettig/github-ci/deploy_npm@main
        if: "${{ vars.TARGET_HOST && vars.TARGET_USER && vars.TARGET_DIRECTORY }}"
        with:
          target_host: ${{ vars.TARGET_HOST }}
          target_user: ${{ vars.TARGET_USER }}
          target_directory: ${{ vars.TARGET_DIRECTORY }}
          ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}

      # Only run if all necessary variables exist in the deploy environment
      - name: Restart requested services '${{ vars.RESTART_SERVICES }}'
        uses: poettig/github-ci/restart_services@main
        if: "${{ vars.RESTART_SERVICES && vars.TARGET_HOST && vars.TARGET_USER }}"
        with:
          services: ${{ vars.RESTART_SERVICES }}
          target_host: ${{ vars.TARGET_HOST }}
          target_user: ${{ vars.TARGET_USER }}
          ssh_key: ${{ secrets.DEPLOY_SSH_KEY }}
```

