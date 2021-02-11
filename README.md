# litex-conda-compilers

Conda build recipes for C compilers needed for LiteX environments.

The C compilers consist of;

| Tool     | Version | Target     | C Library                                            | C Library Version  |
| -------- | ------- | ---------- | ---------------------------------------------------- | ------------------ |
| binutils | 2.32    | Bare metal |                                                      |                    |
| gcc      | 9.1.0   | Bare metal | no C library                                         |                    |
| gcc      | 9.1.0   | Bare metal | [picolibc](https://github.com/keith-packard/picolibc)| WIP                |
| gcc      | 9.1.0   | Bare metal | [newlib](https://sourceware.org/newlib/)             | 3.1.0              |
| gcc      | 9.1.0   | Linux      | [musl](https://musl-libc.org)                        | git ac304227       |
| gdb      | 8.2     | N/A        |                                                      |                    |

The architectures currently supported are;

| Architecture Name | Abbrev    | Soft Core |
| ----------------- | --------- | --------- |
| LatticeMico32     | `lm32`    | [lm32]() |
| RISC-V 32bit      | `rv32`    | [VexRISCV](), [picorv32](), [minvera](), [tiaga?]() |
| RISC-V 64bit      | `rv64`    | [Rocket](), [BlackParrot?]() |
| OpenRISC1000      | `or1k`    | [mor1k]() |
| PowerPC 64bit     | `ppc64le` | [Microwatt]() |

## Cypress FX2 support

 * sdcc (Current version: 3.5.0)

# Building

This repository is set up to be built by Travis CI, using the GitHub
integration to Travis CI.

See [`.travis.yml`](.travis.yml) for the build configuration given to
Travis CI, and the [`.travis`](.travis) directory for scripts referenced.

The Travis CI output can be found on the https://travis-ci.org/ for the
GitHub account and GitHub repository.  For instance, for the main:

https://github.com/timvideos/conda-hdmi2usb-packages

GitHub repository, the Travis CI results can be seen at:

https://travis-ci.org/timvideos/conda-hdmi2usb-packages

On a successful build in the `timvideos` Travis CI, the resulting packages
are uploaded to:

https://anaconda.org/TimVideos/repo

and can be installed with:

```
conda install --channel "TimVideos" package
```

These packages are mostly used by
[`litex-buildenv`](https://github.com/timvideos/litex-buildenv).

## Building via Travis CI in your own repository

If you [enable Travis CI on your GitHub
fork](https://travis-ci.com/getting_started) of `conda-hdmi2usb-packages`
then your Travis CI results will be at:

```
https://travis-ci.org/${GITHUB_USER}/conda-hdmi2usb-packages
```

Since the repository includes `.travis.yml` and all the other Travis CI
setup, you should just need to turn on the Travis CI integration at GitHub,
and push changes to your GitHub repo, to trigger a Travis CI build, then
watch the https://travis-ci.org/ site for the build progress.

A full build of everything will take the Travis CI infrastructure a few
hours if it all builds successfully.

## Common Travis CI build failures

If the build fails, see the [common Travis CI build
problems](https://docs.travis-ci.com/user/common-build-problems/)
for assistance investigating the issues.  Common issues with this
repository include package dependencies (eg, where Conda has changed),
output log file size (Travis CI has a 4MB maximum, and some package
builds like `gcc` generate a *lot* of output), and builds timing out
(either for maximum time allocation, or for ["no output" for more than
10 minutes](https://docs.travis-ci.com/user/common-build-problems/#build-times-out-because-no-output-was-received)).

## Testing conda builds locally

It is recommend to build these packages in a fresh disposable environment
such as a clean docker container.
While the goal is for the conda environment to be totally self contained,
there is a constant battle to make sure this happens.

The commands from the following subsections are enough to build a package
in a clean docker container based on the *ubuntu* Docker image.

### Prerequisites

The only required prerequisites are:
* [Git](https://git-scm.com/),
* Conda installed and initialized, e.g., using
[Miniconda](https://docs.conda.io/en/latest/miniconda.html)
(which includes self contained versions of the required *python3* with
*pip* and *setuptools* tooling),
* [conda-build-prepare](https://github.com/litex-hub/conda-build-prepare)
and of course this repository cloned to a local directory.

All of these requirements can be satisfied using the following commands:

```bash
# Install git and wget (might require using `sudo`)
apt-get update
apt-get install -y git wget

# Download Miniconda and install in CONDA_PATH
CONDA_PATH=~/conda
wget -c https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
chmod a+x Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh -p $CONDA_PATH -b -f

# Initialize Conda in the shell
eval "$("$CONDA_PATH/bin/conda" "shell.bash" 'hook' 2> /dev/null)"

# Install `conda-build-prepare`
python3 -m pip install git+https://github.com/litex-hub/conda-build-prepare@v0.1#egg=conda-build-prepare

# Clone the litex-conda-compilers repository
git clone https://github.com/litex-hub/litex-conda-compilers.git
cd litex-conda-compilers
```

### Required environment variables

The recipe might require exporting some additional environment variables.
Such variables must be set before preparing the recipe, which is done
with one of the commands from the next subsection.

Currently required additional environment variables are:
* `TOOLCHAIN_ARCH` (by: `binutils`, `gcc/*`, `gdb` and `toolchain/linux-musl`
recipes) – must contain a supported architecture name, ie. one of these:
  * `lm32` (excluding `gcc/linux-musl` and `toolchain/linux-musl` recipes),
  * `or1k`,
  * `ppc64le` (only `binutils` and `gcc/linux-musl` recipes),
  * `riscv32`,
  * `riscv64`,
  * `sh`.

The optional `DATE_NUM` and `DATE_STR` environment variables are used by the
recipes to set the `build/number` and `build/string` keys during preparation.
In case they're unset, recipe preparation provides them with default values
based on the latest commit's committer date from the repository holding the
recipe.

### Preparing and building the package

After getting all prerequisites and setting the required variables, it
is recommended to prepare the recipe before building, as it gives you
the advantages described on [the `conda-build-prepare`'s
GitHub page](https://github.com/litex-hub/conda-build-prepare).
The tool is also used within the CI, which makes the locally built
package much more similar to the one built by the CI workflow.

If the provided commands are to be used unmodified, it is important to first
set the `RECIPE_PATH` variable with the appropriate recipe path for the
package chosen to be built.

The `PREPARED_RECIPE_OUTPUTDIR` variable holds a path for the directory that will
be created with output files after running `conda-build-prepare`.
It can be any valid path ending with the name chosen for that new directory.
The output files are described in detail on [the `conda-build-prepare`'s
GitHub page](https://github.com/litex-hub/conda-build-prepare).

The arguments passed with `--channels` and `--packages` switches to the
`conda-build-prepare` are similar to those used within the CI workflow.

The following commands are meant to be run from the repository root.
All aforementioned variables can be modified to, e.g., build a package other
than the `binutils-riscv64` used as an example or use another path for the
`conda-build-prepare` output directory.

```bash
# Set example variables
PREPARED_RECIPE_OUTPUTDIR=cbp-outdir
RECIPE_PATH=binutils
export TOOLCHAIN_ARCH=riscv64

# Prepare the RECIPE with `conda-build-prepare`
ADDITIONAL_PACKAGES="conda-build=3.20.3 conda-verify jinja2 pexpect python=3.7"
python3 -m conda_build_prepare --channels litex-hub --packages $ADDITIONAL_PACKAGES \
            --dir $PREPARED_RECIPE_OUTPUTDIR $RECIPE_PATH

# Activate prepared environment where `conda build` will be run
conda activate $PREPARED_RECIPE_OUTPUTDIR/conda-env

# Build the package
conda build $PREPARED_RECIPE_OUTPUTDIR/recipe
```

### Additional information

Expect packages like `binutils` to take 3-5 minutes to build, packages
like `gcc/nostdc` to take 10-15 minutes to build, and packages like
`gcc/newlib` to take 25-40 minutes to build, on a relatively fast
build system (eg, SSD, i7, reasonable amount of RAM).  Beware that
`gcc/newlib` wants to see `gcc/nostdc` *of the same version* already
installed before it will build; this means that `gcc/newlib` is
non-trivial to build individually.

To build one architecture of tools, without any cleanup will need a
VM with maybe 12-15GiB of space available (a 10GiB disk image is not
quite big enough). Building more architectures at once will need more
disk space.

**NOTE**: By preference only packages built by Travis CI should be
uploaded to the Anaconda repository, so that the externally visible
packages have consistent package versions (and do not conflict).  But
it can be useful to build locally to debug `conda-build` config issues
without waiting for a full Travis CI cycle.
