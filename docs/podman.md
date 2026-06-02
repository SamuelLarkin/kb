# Podman

## Restart Containes on Reboots

1. install the podman-restart.service in your user profile

```sh
systemctl --user enable --now podman-restart.service
```

2. enable linger for your own account

```sh
loginctl enable-linger
```
