# Python

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
