# Nox (cross-arch) Action

This action runs a [nox](https://github.com/wntrblm/nox/) session using
a particular architecture.

Here is an example demonstrating how to use it in a workflow with a matrix job:

```yaml
jobs:
  nox-cross-arch:
    name: Cross-arch tests with nox
    if: github.event_name != 'pull_request'

    strategy:
      fail-fast: false
      matrix:
        architecture:
          - "arm64"
        ubuntu_version:
          - "20.04"
        python_version:
          - "3.11"
        nox_session:
          - "pytest_min"
          - "pytest_max"

    runs-on: ubuntu-${{ matrix.ubuntu_version }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Run nox
        uses: frequenz-io/gh-action-nox-cross-arch@v0.x.x
        with:
          architecture: ${{ matrix.architecture }}
          ubuntu_version: ${{ matrix.ubuntu_version }}
          python_version: ${{ matrix.python_version }}
          nox_session: ${{ matrix.nox_session }}
```

> [!TIP]
> If you need to do some regular `nox` testing without cross-arch you can use the
> [`gh-action-nox`](https://github.com/frequenz-floss/gh-action-nox) action.

## Inputs

* `nox_session`: The nox session to run. Required.

  It must be defined in your project's `noxfile.py`.

* `nox_dependencies`: The dependencies to install to run `nox`. Default:
  `.[dev-noxfile]`.

  If the default is used it is expected that you project defines an optional
  dependencies section `dev-noxfile` in the `pyproject.toml` file that at least
  includes the `nox` package.

  If you don't have any particular extra dependencies you can set this input to
  `nox` to only install the `nox` package.

* `architecture`: The architecture to use, for example `arm64`. Required.

  Passed to the [`frequenz-floss/gh-action-run-python-in-qemu`][qemu-action]
  action, please consult the action for more details.

* `python_version`: The Python version to use, for example `3.11`. Required.

  Passed to the [`frequenz-floss/gh-action-run-python-in-qemu`][qemu-action]
  action, please consult the action for more details.

* `ubuntu_version`: The Ubuntu version to use, for example `20.04`. Required
  unless `dockerfile` is set.

  Passed to the [`frequenz-floss/gh-action-run-python-in-qemu`][qemu-action]
  action, please consult the action for more details.

* `dockerfile`: The Dockerfile to use to build the Docker image, for example
  `docker/Dockerfile.arm64`. Optional.

  Passed to the [`frequenz-floss/gh-action-run-python-in-qemu`][qemu-action]
  action, please consult the action for more details.

## Recommended use with matrix jobs

When using a matrix, it is recommended to create a dummy job to *merge* all the
matrix jobs, specially if you want to require all matrix jobs to pass to allow
merging a pull request. If you do this, you only need to add the dummy job as
a requirement and you don't need to update your requirements each time you
update your matrix.

```yaml
  nox-all:
    # The job name should match the name of the `nox` job.
    name: Test with nox
    needs: ["nox"]
    runs-on: ubuntu-20.04
    steps:
      - name: Return true
        run: "true"
```

## Example using a custom options

This example shows how to use the action with a custom `Dockerfile` and custom
a `nox` session and dependencies.

```yaml
jobs:
  nox:
    name: Test with nox (arm64)
    runs-on: ubuntu-20.04

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Run nox (arm64)
        uses: frequenz-io/gh-action-nox-cross-arch@v0.x.x
        with:
          architecture: arm64
          dockerfile: docker/Dockerfile.arm64
          python_version: 3.11
          nox_dependencies: nox
          nox_session: all
```

[qemu-action]: https://github.com/frequenz-floss/gh-action-run-python-in-qemu
