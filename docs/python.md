# Python

## Creating Virtual Environments

The preferred method is to use `uv` as it is much faster than other methods.
[uv documentation](https://docs.astral.sh/uv)

Install `uv`

```sh
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Create your environment.

```sh
uv venv --python-preference=only-managed --python=3.12 --relocatable --prompt=CAISI venv
```

Activate your environment and start populating it with your dependencies.

```sh
source venv/bin/activate
uv pip install XYZ
# or
uv pip install -r requirements.txt
```

## Profiling Imports

How to profile python's import statements.
Used in finding expensive imports that slow down `--help`.

```sh
python -X importtime myscript.py
```

or:

```sh
PYTHONPROFILEIMPORTTIME=1 myscript.python
```

## SimpleNamespace

[SimpleNamespace](https://docs.python.org/3/library/types.html#types.SimpleNamespace):
A simple object subclass that provides attribute access to its namespace, as
well as a meaningful repr.

## Simple Hack for None json.dumps Objects

Specify a `default` function to process objects that are not json serializable.
Note that `default=str` has to be after `indent` otherwise `indent` is not working.

```python
print(json.dumps(my_data, indent=2, default=str))
```

## Typer

Disable the long exception stack.

```python
app = typer.Typer(add_completion=False, pretty_exceptions_show_locals = False)
```
