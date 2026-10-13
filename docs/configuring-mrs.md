<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Matrix Rooms Search

This is an [Ansible](https://www.ansible.com/) role which installs [Matrix Rooms Search](https://github.com/etkecc/mrs) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Matrix Rooms Search is a fully-featured, standalone, [Matrix](https://matrix.org/) rooms search service.

See the project's [documentation](https://github.com/etkecc/mrs/blob/main/README.md) to learn what Matrix Rooms Search does and why it might be useful to you.

## Adjusting the playbook configuration

To enable Matrix Rooms Search with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# mrs                                                                  #
#                                                                      #
########################################################################

mrs_enabled: true

########################################################################
#                                                                      #
# /mrs                                                                 #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Matrix Rooms Search you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
mrs_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Matrix Rooms Search under a subpath (by configuring the `mrs_path_prefix` variable) does not seem to be possible due to Matrix Rooms Search's technical limitations.

### Set the default homeserver URLs

It is possible to specify the default homeserver's URLs by adding the following configuration to your `vars.yml` file:

```yaml
mrs_servers:
  - matrix.org
```

### Protecting the API groups

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

### Matrix signing keys

`mrs_matrix_keys` is empty by default, and MRS does not generate a key of its own on first run: `/_matrix/key/v2/server` will report an empty `verify_keys` until one is configured. To generate one, run the image's `-genkey` flag once and put the resulting `ed25519 <id> <seed>` line into `mrs_matrix_keys`.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `mrs_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Matrix Rooms Search becomes available at the specified hostname like `https://example.com`.

You can call the `https://mrs.example.com/-/full` endpoint using admin credentials (see the `mrs_auth_admin_*` variables) to discover and parse content.

To see the list of supported public and private APIs, refer to the [API documentation](https://github.com/etkecc/mrs/blob/main/docs/swagger.yaml).

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu mrs` (or how you/your playbook named the service, e.g. `mash-mrs`).
