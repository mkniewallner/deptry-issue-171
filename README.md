# deptry-issue-171

This repository highlights the issue detailed in https://github.com/fpgmaas/deptry/issues/171.

## project_with_httpx_at_root

This project contains an import to [httpx](https://pypi.org/project/httpx/) (which is missing from the
dependencies) from module `foo` in a `sub_directory` directory, as well as one from module `foo` in the root directory.

As there is also a `httpx` module in the root directory, Python will rightfully consider `httpx` to be a local module,
as what's in `PYTHONPATH` has priority over other sources:

```console
$ cd project_with_httpx_at_root

$ poetry install

$ poetry run python -m foo
I've been imported

$ poetry run python -m sub_directory.foo
I've been imported
```

deptry, on the other hand, will report `httpx` as a missing dependency, despite Python considering it a local module
regardless of from where it's imported:

```console
$ poetry run deptry . --json-output output.json; cat output.json
[...]
{
    "obsolete": [],
    "missing": [
        "httpx"
    ],
    "transitive": [],
    "misplaced_dev": []
}
```

## project_with_httpx_in_sub_directory

This project contains an import to [httpx](https://pypi.org/project/httpx/) (which is missing from the
dependencies) from module `foo` in a `sub_directory` directory.

There is an `httpx` module in the code, but only in a `sub_directory` directory. Python will rightfully consider `httpx`
to be an external module, as `PYTHONPATH` only contains the modules that are in the root directory (`sub_directory`):

```console
$ cd project_with_httpx_in_sub_directory

$ poetry install

$ poetry run python -m sub_directory.foo
Traceback (most recent call last):
  File "<REDACTED>/lib/python3.10/runpy.py", line 196, in _run_module_as_main
    return _run_code(code, main_globals, None,
  File "<REDACTED>/lib/python3.10/runpy.py", line 86, in _run_code
    exec(code, run_globals)
  File "<REDACTED>/deptry-issue-171/project_with_httpx_in_sub_directory/sub_directory/foo.py", line 1, in <module>
    from httpx import Client
ModuleNotFoundError: No module named 'httpx'
```

deptry, on the other hand, will not report `httpx` as a missing dependency, despite Python considering it an external
module:

```console
$ poetry run deptry . --json-output output.json; cat output.json
[...]
{
    "obsolete": [],
    "missing": [],
    "transitive": [],
    "misplaced_dev": []
}
```
