# Pixi

```sh
curl -fsSL https://pixi.sh/install.sh | sh
```

## AutoVenv

```sh
#!/usr/bin/env -S pixi run --script
# /// script
# requires-python = ">=3.11"
# dependencies = ["black"]
#
# [tool.pixi.dependencies]
# isort = "*"
# ///
print("bar")
```
