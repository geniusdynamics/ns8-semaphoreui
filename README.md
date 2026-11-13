# ns8-semaphoreui

Semaphore is a responsive web UI for running Ansible playbooks, Terraform/OpenTofu and Pulumi code.

Module id: `semaphoreui`. Images are pinned in `build-images.sh` and updated through reviewed changes.

## Install

```bash
add-module ghcr.io/geniusdynamics/semaphoreui:latest 1
```

The command returns the instance id, for example `semaphoreui1`. Use the id NS8 returns.

## Configure

```bash
api-cli run configure-module --agent module/semaphoreui1 --data - <<'EOF'
{
  "host": "semaphoreui.example.org",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

| Parameter | Required | Type | Description |
|---|---|---|---|
| `host` | yes | string | Host name for the application, like 'semaphoreui.domain.org' |
| `lets_encrypt` | yes | boolean | Request a valid Let's Encrypt certificate. |
| `http2https` | yes | boolean | Redirect all the HTTP requests to HTTPS |

## Configuration output

`get-configuration` returns: `host`, `lets_encrypt`, `http2https`.

## Images

| Image | Version |
|---|---|
| `docker.io/mariadb` | `10.11.19` |
| `docker.io/semaphoreui/semaphore` | `v2.15.0` |

## Network and access

- Reserves 1 TCP port(s) on the node. The web application is published through Traefik on the configured host name.
- Runs rootless.
- Declares these NS8 authorizations: `traefik@node:routeadm`.

## Backup

The following paths are included in the NS8 backup (`imageroot/etc/state-include.conf`):

- `state/semaphoreui.sql`
- `state/database.env`
- `state/app.env`
- `volumes/semaphoreui-app`
- `volumes/semaphore-data`
- `volumes/semaphore-config`
- `volumes/semaphore-tmp`
- `volumes/mysql-data`

## Services

- `mariadb-app.service`
- `semaphoreui-app.service`
- `semaphoreui.service`

## Get the configuration

```bash
api-cli run get-configuration --agent module/semaphoreui1
```

## Update and uninstall

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/semaphoreui:latest","instances":["semaphoreui1"],"force":true}'
remove-module --no-preserve semaphoreui1
```

## Testing

Continuous integration:

- `Publish images` builds the module image on every push.
- `Test module` installs the image on a QEMU NS8 node and runs the Robot Framework suite in `tests/`. It runs after `Publish images` succeeds on `dev` and on `main`.
- `Build API JSON Schema docs` and the security scan run alongside.

The suite installs the module, configures a host name, checks that the web application answers through Traefik, takes UI screenshots and removes the module.

Run it locally with the NS8 standard infrastructure described in the [ns8-github-actions README](https://github.com/NethServer/ns8-github-actions/blob/v1/README.md#running-tests-locally).

## Upstream

Application documentation: https://docs.semaphoreui.com/

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
