# Nox (cross-arch) Action

This action runs a [nox](https://github.com/wntrblm/nox/) session on the
current repository using a particular architecture.

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
      - name: Run nox
        uses: frequenz-io/gh-action-nox-cross-arch@v0.x.x
        with:
          architecture: ${{ matrix.architecture }}
          ubuntu_version: ${{ matrix.ubuntu_version }}
          python_version: ${{ matrix.python_version }}
          nox_session: ${{ matrix.nox_session }}
```

When using a matrix, it is recommended to create a dummy job to *merge* all the
matrix jobs, specially if you want to require all matrix jobs to pass to allow
merging a pull request. If you do this, you only need to add the dummy job as
a requirement and you don't need to update your requirements each time you
update your matrix.

```yaml
  nox-cross-arch-all:
    # The job name should match the name of the `nox-cross-arch` job.
    name: Cross-arch tests with nox
    needs: ["nox-cross-arch"]
    runs-on: ubuntu-20.04
    steps:
      - name: Return true
        run: "true"
```

> [!TIP]
> If you need to do some regular `nox` testing without cross-arch you can use the
> [`gh-action-nox`](https://github.com/frequenz-floss/gh-action-nox) action.

## Inputs

* `architecture`: The architecture to use. Required.

  It must be supported by [QEMU](https://www.qemu.org/) and in particular the
  [`docker/setup-qemu-action`](https://github.com/docker/setup-qemu-action)
  action. For example: `arm64`.

* `nox_session`: The nox session to run. Required.

  It must be defined in your project's `noxfile.py`.

* `ubuntu_version`: The Ubuntu version to use. Required unless `dockerfile` is
  set. Default: `""`.

  This will be used as the docker base image where `nox` will be run.

* `python_version`: The Python version to use. Required unless `dockerfile` is
  set. Default: `""`.

  This should be a python package version that is available in the Ubuntu, and
  in particular in the `ubuntu_version` you are using. The package
  `python${python_version}` (and other supporting packges for that version)
  will be installed. For example: `3.11`.

* `dockerfile`: The Dockerfile to use. Optional unless `ubuntu_version` and
  `python_version` are not set. When this is used, the docker context directory
  will be set to the directory containing the Dockerfile. Default: `""`.

* `nox_dependencies`: The dependencies to install to run `nox`. Default:
  `.[dev-noxfile]`.

  If the default is used it is expected that you project defines an optional
  dependencies section `dev-noxfile` in the `pyproject.toml` file that at least
  includes the `nox` package.

  If you don't have any particular extra dependencies you can set this input to
  `nox` to only install the `nox` package.

* `submodules`: How submodules should be fetched. This is passed to the
  `actions/checkout` action. Default: `true` (non-recursive fetching).

* `checkout_token`: The token to use to fetch the code and submodules. Default:
  `${{ secrets.github_token }}`.

> [!CAUTION]
> The `ubuntu_version`/`python_version` and `dockerfile` inputs are mutually
> exclusive. You must set one or the other, but not both.
>
> When using the `ubuntu_version`/`python_version` inputs, both must be
> defined and a `Dockerfile` is provided by the action. To test using other OSs
> you must provide your own Dockerfile. You can use the [Dockerfile in this
> action](resources/Dockerfile) (and the
> [entry_point](resources/run_nox_session)) as a starting point.

## Example

Fetching submodules recursively and using a custom `Dockerfile`, checkout token, and 
nox dependencies and session:

```yaml
jobs:
  nox:
    name: Test with nox (arm64)
    runs-on: ubuntu-20.04

    steps:
      - name: Run nox (arm64)
        uses: frequenz-io/gh-action-nox-cross-arch@v0.x.x
        with:
          architecture: arm64
          dockerfile: .github/workflows/Dockerfile.arm64
          nox_dependencies: nox
          nox_session: all
          submodules: recursive
          checkout_token: ${{ secrets.my_checkout_token }}
```
