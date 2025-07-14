# ns8-semaphoreui

This is a module for [NethServer 8](https://github.com/NethServer/ns8-core) that installs [Semaphore](https://github.com/ansible-semaphore/semaphore), a modern open-source Ansible UI.

---

## ✅ Install

Instantiate the module with:

```bash
add-module ghcr.io/geniusdynamics/semaphoreui:latest 1
```

Example output:

```json
{
  "module_id": "semaphoreui1",
  "image_name": "semaphoreui",
  "image_url": "ghcr.io/geniusdynamics/semaphoreui:latest"
}
```

---

## ⚙️ Configure

Assuming your instance is named `semaphoreui1`, configure it by setting the domain and HTTPS options:

```bash
api-cli run configure-module --agent module/semaphoreui1 --data - <<EOF
{
  "host": "semaphoreui.domain.com",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

This command:

* Starts the Semaphore UI
* Creates a virtual host via Traefik

---

## 📤 Email Configuration via SmartHost

Email/SMTP settings are automatically discovered from the centralized SmartHost Redis configuration using the `discover-smarthost` hook.

### Automatically injected values:

* `SEMAPHORE_EMAIL_SENDER`
* `SEMAPHORE_EMAIL_HOST`
* `SEMAPHORE_EMAIL_PORT`
* `SEMAPHORE_EMAIL_USERNAME`
* `SEMAPHORE_EMAIL_PASSWORD`

You do not need to pass these manually — they're refreshed at container start and on SmartHost changes.

---

## 🔔 Alert Integrations (Gotify & Telegram)

Optional alert integrations can be added by including the following variables in your JSON configuration:

### Gotify Alerts

* `SEMAPHORE_GOTIFY_ALERT`: "True" or "False"
* `SEMAPHORE_GOTIFY_URL`: Your Gotify server URL
* `SEMAPHORE_GOTIFY_TOKEN`: Application token

### Telegram Alerts

* `SEMAPHORE_TELEGRAM_ALERT`: "True" or "False"
* `SEMAPHORE_TELEGRAM_CHAT`: Telegram Chat ID
* `SEMAPHORE_TELEGRAM_TOKEN`: Bot token

All alert settings are written into `app.env` and used on container startup.

---

## 🔍 Get the configuration

```bash
api-cli run get-configuration --agent module/semaphoreui1
```

---

## ❌ Uninstall

```bash
remove-module --no-preserve semaphoreui1
```

---

## 📡 SmartHost setting discovery

At container startup, `bin/discover-smarthost` populates `state/smarthost.env` by querying Redis. When SmartHost settings change, the event handler `events/smarthost-changed/10reload_services` restarts the container automatically.

---

## 🔎 Debugging Tips

### Check environment variables:

```bash
runagent -m semaphoreui1 env
```

### Launch interactive shell as agent:

```bash
runagent -m semaphoreui1
```

### Inspect running containers:

```bash
podman ps
```

### Check inside the container:

```bash
podman exec semaphoreui-app env
podman exec -ti semaphoreui-app sh
```

---

## 🔮 Testing

Test the module using the following script:

```bash
./test-module.sh <NODE_ADDR> ghcr.io/geniusdynamics/semaphoreui:latest
```

Test logic uses [Robot Framework](https://robotframework.org).

---

## 🌎 UI Translation

This module supports [Weblate](https://hosted.weblate.org/projects/ns8/) for localization.

To contribute:

1. Add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repo.
2. Add the repository to [hosted.weblate.org](https://hosted.weblate.org) or ask a NethServer developer to include it.
