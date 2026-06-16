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

## Multiprocessing with Expensive Resources

You can avoid global variables by attaching the expensive resource as an attribute of the worker function itself.
Since the initializer runs once per process, you can modify the function object passed to it, and that modified function object is then used for all tasks in that specific process.

Here is the updated example:

```python title="Multiprocessing Example"
import multiprocessing
import time
import os

# Simulate an expensive-to-create resource
class ExpensiveResource:
    def __init__(self):
        print(f"Creating resource in process {os.getpid()}...")
        time.sleep(1)  # Simulate expensive initialization
        self.value = 100

    def process(self, data):
        return self.value * data

# Worker function that will hold the resource as an attribute
def worker_task(task_data):
    # Access the resource attached to this function object
    return worker_task.resource.process(task_data)

def worker_initializer(resource_instance):
    """Called once per worker process to attach the resource to the function."""
    # Attach the resource directly to the function object
    # This avoids using the global keyword or global scope
    worker_task.resource = resource_instance

if __name__ == "__main__":
    # 1. Create the expensive resource in the main process
    # (It will be pickled and sent to workers)
    resource = ExpensiveResource()

    # 2. Prepare data to process
    data = [1, 2, 3, 4, 5, 6, 7, 8]

    # 3. Use Pool with the initializer to attach the resource
    with multiprocessing.Pool(
        processes=multiprocessing.cpu_count(),
        initializer=worker_initializer,
        initargs=(resource,)
    ) as pool:
        # 4. Map tasks; each worker uses its own attached resource instance
        results = pool.map(worker_task, data)

    print(f"Results: {results}")
```

Key Points:

- Function Attributes: Instead of global resource, we use worker_task.resource = resource_instance. In Python, functions are objects and can have attributes.
- Scope: The resource is stored in the function's namespace within the worker process, keeping the global namespace clean.
- Access: Inside worker_task, we access the resource via worker_task.resource, ensuring each process uses the instance initialized specifically for it.

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
