# The Foreman GitHub Actions

This repository contains [reusable workflows](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows)
to reduce duplication in actions in the Foreman project.

At this moment it's considered experimental.

## Rubocop

To call Rubocop within your CI, use the following workflow:

```yaml
name: CI

on: pull_request

concurrency:
  group: ${{ github.ref_name }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  rubocop:
    name: Rubocop
    uses: theforeman/actions/.github/workflows/rubocop.yml@v0
```

## Foreman plugin Ruby tests

To run the Foreman tests once Rubocop has passed, use the following workflow:

```yaml
name: CI

on:
  pull_request:
  push:
    branches:
      - 'develop'
      - '*-stable'

concurrency:
  group: ${{ github.ref_name }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  rubocop:
    name: Rubocop
    uses: theforeman/actions/.github/workflows/rubocop.yml@v0

  test:
    name: Ruby
    needs: rubocop
    uses: theforeman/actions/.github/workflows/foreman_plugin.yml@v0
    with:
      plugin: MY_PLUGIN
```

By default, this will run with the Ruby/NodeJS combination that is configured in Foreman's `.github/matrix.json`.
The Foreman version is specified using `foreman_version`, which defaults to `develop`.

To test out pull request number 1234, you can use:

```yaml
jobs:
  test:
    name: Ruby
    uses: theforeman/actions/.github/workflows/foreman_plugin.yml@v0
    with:
      plugin: MY_PLUGIN
      foreman_version: refs/pull/1234/head
```

You can adjust this matrix by setting the `matrix_include` and `matrix_exclude` inputs to the workflow:

```yaml
name: CI

on:
  pull_request:
  push:
    branches:
      - 'develop'
      - '*-stable'

jobs:
  test:
    name: Ruby
    uses: theforeman/actions/.github/workflows/foreman_plugin.yml@v0
    with:
      plugin: MY_PLUGIN
      matrix_include: '[{"ruby": "3.0", "node": "20"}]'
      matrix_exclude: '[{"ruby": "2.5", "node": "10"}, {"ruby": "2.5", "node": "12"}]'
```

You can run tests against multiple Foreman versions by using a matrix:

```yaml
name: CI

on:
  pull_request:
  push:
    branches:
      - 'develop'
      - '*-stable'

concurrency:
  group: ${{ github.ref_name }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  test:
    name: Ruby
    strategy:
      fail-fast: false
      matrix:
        foreman:
          - 3.9-stable
          - develop
    uses: theforeman/actions/.github/workflows/foreman_plugin.yml@v0
    with:
      plugin: MY_PLUGIN
      foreman_version: ${{ matrix.foreman }}
```

If you need to set additional environment variables (e.g. to handle specific dependencies in your Gemfile), you can provide them via `environment_variables`:

```yaml
name: CI

on:
  pull_request:
  push:
    branches:
      - 'develop'
      - '*-stable'

concurrency:
  group: ${{ github.ref_name }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  rubocop:
    name: Rubocop
    uses: theforeman/actions/.github/workflows/rubocop.yml@v0

  test:
    name: Ruby
    needs: rubocop
    uses: theforeman/actions/.github/workflows/foreman_plugin.yml@v0
    with:
      plugin: MY_PLUGIN
      environment_variables: |
        CUSTOM_ENV_VARIABLE_ONE=FOO
        CUSTOM_ENV_VARIABLE_TWO=BAR
```

## Foreman plugin JavaScript/React tests

To run the Foreman plugin JavaScript/React tests, use the following workflow:

```yaml
name: JavaScript

on:
  pull_request:
  push:
    branches:
      - 'develop'
      - '*-stable'

concurrency:
  group: ${{ github.ref_name }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  test:
    name: JavaScript
    uses: theforeman/actions/.github/workflows/foreman_plugin_js.yml@v0
    with:
      plugin: MY_PLUGIN
```

By default, this will run with the NodeJS versions that is configured in Foreman's `.github/matrix.json`.

You can alter the behavior the same way as with the Ruby tests.

## Gem test

To test a simple gem that only needs Ruby and bundler, use the following workflow:

```yaml
name: CI

on: pull_request

concurrency:
  group: ${{ github.ref_name }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  test:
    name: Tests
    uses: theforeman/actions/.github/workflows/test-gem.yml@v0
```

By default it uses `bundle exec rake spec` but it's possible to override the command:

```yaml
jobs:
  test:
    name: Tests
    uses: theforeman/actions/.github/workflows/test-gem.yml@v0
    with:
      command: bundle exec rake test
```

## Gem release

To release a gem, use the following workflow

```yaml
name: Release

on:
  push:
    # Pattern matched against refs/tags
    tags:
      - '**'

jobs:
  release:
    name: Release gem
    uses: theforeman/actions/.github/workflows/release-gem.yml@v0
    with:
      allowed_owner: MY_USERNAME
    secrets:
      api_key: ${{ secrets.RUBYGEM_API_KEY }}
```
