# Contributing

Thanks for your interest in improving this role! This guide covers how to set up a development
environment, run the linters and formatters, and execute the test suite locally so your changes
match what CI expects.

## Ways to contribute

- **Report bugs** or request features by opening an [issue](https://github.com/sebdanielsson/ansible-role-cloudflared/issues).
- **Submit changes** via a pull request from a topic branch (see [Pull requests](#pull-requests)).

## Prerequisites

- **Python** 3.14 (the pinned version lives in [`.python-version`](.python-version))
- **[uv](https://docs.astral.sh/uv/)** for managing the Python environment and dependencies
- **Docker** — required by Molecule to spin up test containers

## Development environment

### 1. Clone the repository

```sh
git clone https://github.com/sebdanielsson/ansible-role-cloudflared.git
cd ansible-role-cloudflared
```

### 2. Create and activate a virtual environment

`uv` reads `.python-version` and provisions the correct interpreter automatically.

```sh
uv venv
source .venv/bin/activate
```

### 3. Install Python dependencies

```sh
uv pip install -r requirements.txt
```

This installs `ansible-core`, `molecule`, the Molecule Docker plugin, and the `docker` SDK — all
pinned in [`requirements.txt`](requirements.txt).

### 4. Install the required Ansible collections

```sh
ansible-galaxy collection install -r requirements.yml
```

## Linting

CI runs two linters; run both before pushing.

### ansible-lint

```sh
ansible-lint
```

Configuration lives in [`.ansible-lint`](.ansible-lint).

### YAML

`yamllint` rules are defined in [`.yamllint.yaml`](.yamllint.yaml):

```sh
yamllint .
```

CI additionally enforces YAML _formatting_ with [`yamlfmt`](https://github.com/google/yamlfmt),
configured in [`.yamlfmt.yaml`](.yamlfmt.yaml). Check it the same way CI does:

```sh
go run github.com/google/yamlfmt/cmd/yamlfmt@main -conf .yamlfmt.yaml -lint
```

Drop the `-lint` flag to apply the formatting in place.

## Testing

Tests use [Molecule](https://ansible.readthedocs.io/projects/molecule/) with the Docker driver.
There are two scenarios:

- **`molecule/default`** — applies the role and relies on Molecule's own sequence (converge →
  idempotence) to catch breakage. There is no `verify.yml` here, so `molecule verify` is a no-op in
  this scenario; if you add assertions, add the playbook and wire it up in `molecule.yml`.
- **`molecule/migration`** — prepares a host with the old, distro-specific Cloudflare apt suite and
  asserts in `verify.yml` that the role migrates it to `Suites: any` (see the
  [scenario README](molecule/migration/README.md)).

Run the default scenario:

```sh
molecule test
```

Run the migration scenario:

```sh
molecule test -s migration
```

While iterating, converge and verify against a persistent container instead of recreating it each
time:

```sh
molecule converge   # apply the role
molecule verify     # run the assertions (migration scenario only)
molecule login      # shell into the test container for debugging
molecule destroy    # tear the container down when done
```

### Testing against different distributions

CI runs the default scenario across Debian 13, Ubuntu 24.04, Fedora 42, and RHEL UBI 9, and the
migration scenario across Debian 12 and Ubuntu 24.04. Locally you can target a specific image with
the same environment variables CI uses:

```sh
MOLECULE_DOCKER_IMAGE=debian MOLECULE_DOCKER_TAG=13 molecule test
MOLECULE_DOCKER_IMAGE=fedora MOLECULE_DOCKER_TAG=42 molecule test
MOLECULE_DOCKER_IMAGE=redhat/ubi9 MOLECULE_DOCKER_TAG=9.6 molecule test
```

The Archlinux path has no CI coverage — if you touch `tasks/pacman/`, please test it by hand and
say so in the PR.

## Commit messages

Commits on `main` must follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/)
— the version number and changelog are derived from them (see [Releases](#releases)). PRs are
squash-merged, so it is the **PR title** that ends up on `main` and has to conform. CI enforces this
([`Lint PR` workflow](.github/workflows/lint-pr.yml)); edit the title and the check re-runs.

| Type                                           | Effect on the next release           |
| ---------------------------------------------- | ------------------------------------ |
| `feat: ...`                                    | minor bump, listed under _Features_  |
| `fix: ...`                                     | patch bump, listed under _Bug Fixes_ |
| `feat!: ...` or a `BREAKING CHANGE:` footer    | major bump                           |
| `docs:`, `chore:`, `ci:`, `test:`, `refactor:` | no release                           |

Renovate's dependency PRs are forced to `chore` in [`renovate.json`](renovate.json): it only bumps
CI and test tooling here, which shouldn't bump the role version.

## Pull requests

1. Create a topic branch off `main` (e.g. `feat/...`, `fix/...`, `docs/...`).
2. Make your change, keeping it focused and idempotent.
3. Run the linters and `molecule test` locally.
4. Document new variables in [`README.md`](README.md) and `defaults/main.yml`.
5. Open a PR against `main`, titled as a conventional commit. CI must pass before merge.

## Releases

Releases are automated with [Release Please](https://github.com/googleapis/release-please). Every
push to `main` updates a standing **release PR** that bumps the version and
[`CHANGELOG.md`](CHANGELOG.md) based on the conventional commits since the last release.

Merging that PR is the release: it tags the commit, publishes the GitHub release, and imports the
role to [Ansible Galaxy](https://galaxy.ansible.com/) — all in the
[`Release Please` workflow](.github/workflows/release-please.yml). Maintainers merge the release PR;
nobody creates tags by hand.

Tags here are bare (`2.5.1`, no `v` prefix), matching the existing Galaxy version history.

To force a specific version, add a `Release-As: 1.2.3` footer to a commit on `main`.
