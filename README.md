<!--
SPDX-FileCopyrightText: 2023 Nikita Chernyi
SPDX-FileCopyrightText: 2026 Slavi Pantaleev
SPDX-FileCopyrightText: 2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Matrix Rooms Search API Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [Matrix Rooms Search](https://github.com/etkecc/mrs) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role *implicitly* depends on:

- [`com.devture.ansible.role.playbook_help`](https://github.com/devture/com.devture.ansible.role.playbook_help)
- [`com.devture.ansible.role.systemd_docker_base`](https://github.com/devture/com.devture.ansible.role.systemd_docker_base)

Check [`defaults/main.yml`](defaults/main.yml) for the full list of supported options.

💡 For an Ansible playbook which integrates this role and makes it easier to use, see the [Mother-of-All-Self-Hosting Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

## Protecting the API groups

MRS splits its HTTP API into five groups, each with its own credentials and its own optional IP allowlist:

| Group | Variables | Endpoints |
| --- | --- | --- |
| `admin` | `mrs_auth_admin_login`, `mrs_auth_admin_password`, `mrs_auth_admin_ips` | `/-/status`, `/-/discover`, `/-/parse`, `/-/reindex`, `/-/full` |
| `metrics` | `mrs_auth_metrics_*` | `/metrics` |
| `catalog` | `mrs_auth_catalog_*` | `/catalog/rooms`, `/catalog/servers` |
| `discovery` | `mrs_auth_discovery_*` | `/discover/bulk`, and the rate limit exemption on `/discover/<server>` |
| `moderation` | `mrs_auth_moderation_*` | `/mod/list`, `/mod/list-reported`, `/mod/ban/<room>`, `/mod/unban/<room>`, `/mod/unreport` |

⚠️ **Configure credentials (or an IP allowlist) for every group you expose.** All of these variables default to an empty value, and MRS compares presented credentials against the configured ones with a plain equality check: an empty login together with an empty password compares equal to a group that was left empty. A caller sending `Authorization: Basic Og==` — an empty login and an empty password — is let straight through any group with no credentials and no IP allowlist. That includes `/-/full` and `/-/reindex`, which start a federation-wide crawl, and `/mod/ban`, which removes rooms from the index.

The role points out any group left in that state at the end of an installation run, but it does not refuse to install.

## Matrix signing keys

`mrs_matrix_keys` is empty by default, and MRS does not generate a key of its own on first run: `/_matrix/key/v2/server` will report an empty `verify_keys` until one is configured. To generate one, run the image's `-genkey` flag once and put the resulting `ed25519 <id> <seed>` line into `mrs_matrix_keys`.

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
