# FPF Tools APT packages LFS repo

This repository holds packages served on `packages.freedom.press`.
Currently, this includes [Dangerzone](https://dangerzone.rocks/).

## Prerequisites

- [git-lfs](https://git-lfs.github.com/) to store large files.
- Docker, to use the required version of [reprepro](https://salsa.debian.org/brlink/reprepro), across OSes.

## Installation

First, set up a machine with the GPG key used for signing Release files.

Then, build a container image using the `Dockerfile` in this repo:

```
docker build -t apt-tools-prod-builder .
```

## Usage

When you want to release some files on the apt repository, you will need to add
them in the `dangerzone/*` folders, and then rebuild the debian repository from
scratch.

- Commit new package files to each suite in `dangerzone`. You may want to
  prune older versions as new ones are released, to keep the repo
  manageable. You can use the scripts located in `./tools` for this:

  * `./tools/remove-version X.Y.Z` will remove all Dangerzone packages from the
    repository that match a specific version.
  * `./tools/add-version` will get the debian file from the main dangerzone
    repository, put it in the proper folder and create symlinks.
  * `./tools/reset-repo` will reset the repository, it can be useful before
    publishing it.

- Then, you can run `./tools/publish`, to populate the Debian database.
  * Here is how you can do this inside a container:

    ```
    docker run --rm -v .:/home/user/apt-tools-prod apt-tools-prod-builder ./tools/publish
    ```

- Run `./tools/publish --sign` to sign the release files. This part must run
  on an environment that has access to the private PGP key.
- Commit the results, and create a PR.

When PRs are merged, `packages.freedom.press` will pull new files and
serve the contents of `repo/public`.

## Adding and removing a distribution

You need to do the following things when adding and/or removing a distribution:

- Update the `dangerzone/*` folders accordingly. They are named after the distribution version
- Update the `repo/conf/distributions` file and add/remove your distribution version.
- Update the `.github/workflows/ci.yaml` file, with the updated distribution list.
