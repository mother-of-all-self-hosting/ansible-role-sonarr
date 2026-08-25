<!--
SPDX-FileCopyrightText: 2023 Slavi Pantaleev
SPDX-FileCopyrightText: 2025 spatterIight
SPDX-FileCopyrightText: 2025, 2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Sonarr Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [Sonarr](https://sonarr.tv/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role *implicitly* depends on:

- [`com.devture.ansible.role.playbook_help`](https://github.com/devture/com.devture.ansible.role.playbook_help)
- [`com.devture.ansible.role.systemd_docker_base`](https://github.com/devture/com.devture.ansible.role.systemd_docker_base)

Check [`defaults/main.yml`](defaults/main.yml) for the full list of supported options.

💡 For an Ansible playbook which integrates this role and makes it easier to use, see the [Mother-of-All-Self-Hosting Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

## Limitations

This role configures Sonarr with security in mind by doing the following:

1. Running the container as a non-root user
2. Making the filesystem read-only
3. Dropping most capabilities

Unfortunately, due to upstream requirements, some admissions had to be made:

1. Several capabilities related to permissions are added to the container
   - SETUID
   - SETGID
   - CHOWN
   - FOWNER
   - DAC_OVERRIDE
2. A `tmpfs` volume is mounted with `exec` permissions

You can read more about these upstream requirements in the documentation:

1. <https://docs.linuxserver.io/misc/non-root/>
2. <https://docs.linuxserver.io/misc/read-only/>

## Development

### pre-commit

You can optionally install a Git pre-commit hook (via [mise](https://mise.jdx.dev/) + [prek](https://prek.j178.dev/)) that runs formatting and linting checks before each commit. See [`.pre-commit-config.yaml`](./.pre-commit-config.yaml) for which hooks are to be executed.

To install the hook, run the [`just`](https://github.com/casey/just) command below:

```sh
just prek-install-git-pre-commit-hook
```

### Molecule

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

Refer to [this page](./molecule/README.md) for details about how to utilize it.

### Releases

Releases are tagged automatically. `.github/workflows/autotag.yml` runs [`bin/compute-next-tag.sh`](./bin/compute-next-tag.sh) on every push, and that script derives the tag from `sonarr_version` in [`defaults/main.yml`](defaults/main.yml) and from the tags that already exist — never from commit messages. A commit that touches only documentation or CI is not released.

[`bin/test-compute-next-tag.sh`](./bin/test-compute-next-tag.sh) exercises the computation against throwaway repositories. It runs as a pre-commit hook whenever the script or `defaults/main.yml` changes, and can be run by hand at any time.

Sonarr's own version bumps are *not* automerged, not even at the patch level: Sonarr migrates its SQLite database on every start, forward-only, and the linuxserver.io tag family this role tracks truncates Sonarr's four-component versions to three, so its entire 4.0.x release stream looks patch-level to Renovate. See the `description` in [`.github/renovate.json`](.github/renovate.json).
