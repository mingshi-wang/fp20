# What it contains
* `helloworld` - simple hello world with protobuf
* `py_helloworld` - simple hello world with more testings
* `ray_demo` - a full example of ray distributive workers on local cluster
* `torch_demo` - Pytorch, Numpy and Pandas
* `pl_examples` - pytorch lightening examples (with some known testing errors)
# Run Ray Example and generate packages

```bash
./pants list ray_demo: # list the targets
./pants run ray_demo # run a binary
./pants package ray_demo:ray_demo # Package in PEX the pex is in dist/
./pants package ray_demo:ray_dist # Package in Wheel
```

# Run pyTorch and Numpy + Pandas

./pants run torch_demo:

# Run tests

```
./pants test py_helloworld/test/test_*
```

# example-python

An example repository to demonstrate Python support in Pants.

See [pantsbuild.org](https://www.pantsbuild.org/docs) for much more detailed documentation.

# Running Pants

You run Pants goals using the `./pants` wrapper script, which will bootstrap the
configured version of Pants if necessary.

Use `./pants --version` to see the version of Pants configured for the repo (which you can also find
in `pants.toml`).

# Goals

Pants commands are called _goals_. You can get a list of goals with

```
./pants help goals
```

# Targets

Targets are sets of source files with some attached metadata. Targets are provided in `BUILD` files.
Targets have types, such as `python_library`, `resources`, `python_binary`. Examples of metadata include
dependencies on other targets, Python version compatibility, entry points for binaries, and so on.

Pants goals can be invoked on targets or directly on source files/directories (which is often more intuitive and convenient).
In the latter case, Pants locates target metadata for the source files as needed.

## File specifications

Invoking goals on files is straightforward, e.g.,

```
./pants test helloworld/util/lang_test.py
```

You can use globs:

```
./pants lint helloworld/util/*.py
```

But note that these will be expanded by your shell, so this is equivalent to having used

```
./pants lint helloworld/util/lang.py helloworld/util/lang_test.py helloworld/util/resources.py helloworld/util/resources_test.py
```

If you want Pants itself to expand the globs (which is sometimes necessary), you must quote them in the shell:

```
./pants lint 'helloworld/util/*.py'
```

You can run on all changed files:

```
./pants --changed-since=HEAD lint
```

You can run on all changed files, and any of their "dependees":

```
./pants --changed-since=HEAD --changed-dependees=transitive test
```

## Target specifications

Targets are referenced on the command line using their address, of the form `path/to/dir:name`, e.g.,

```
./pants lint helloworld/util:util
```

You can omit the target name if it is the same as the immediately enclosing directory name, e.g.,

```
./pants lint helloworld/util
```

You can glob over all targets in a directory with a single trailing `:`, or over all targets in a directory
and all its subdirectories with a double trailing `::`, e.g.,

```
./pants lint helloworld::
```

## Globbing semantics

When you glob over files or targets, Pants knows to ignore ones that aren't relevant to the requested goal.
For example, if you run the `test` goal over a set of files that includes non-test files, Pants will just ignore
those, rather than error. So you can safely do things like

```
./pants test ::
```

To run all tests.

In some cases trying to run a goal on multiple files or targets will fail due to conflicts. For example, you cannot
`./pants repl helloworld::` because that globs over both Python 2 and Python 3 code, so there is
no way to select an interpreter compatible with both both to run the REPL on.


# Example Goals

Try these out in this repo!

## List targets

```
./pants list ::  # All targets.
./pants list 'helloworld/**/*.py'  # Just targets containing Python code.
```

## Run linters and formatters

```
./pants lint ::
./pants fmt 'helloworld/**/*.py'
```

## Run MyPy

```
./pants typecheck ::
```

## Run tests

```
./pants test ::  # Run all tests in the repo.
./pants test helloworld/util:test  # Run all the tests in this target.
./pants test helloworld/util/lang_test.py  # Run just the tests in this file.
./pants test helloworld/util/lang_test.py -- -k test_language_translator  # Run just this one test.
```

## Create a PEX binary

```
./pants package helloworld/main.py helloworld/main_py2.py
```

## Run a binary

```
./pants run helloworld/main.py
```

## Open a REPL

```
./pants repl helloworld/greet  # The REPL will have all relevant code and dependencies on its sys.path.
./pants repl --shell=ipython helloworld/greet
```

## Build a wheel / generate `setup.py`

This will build both a `.whl` bdist and a `.tar.gz` sdist.

```
./pants package helloworld/util:dist
```

We can also remove the `setup_py_commands` field from `helloworld/util/BUILD` to have Pants instead generate a 
`setup.py` file, with all the relevant code in a chroot.

## Build an AWS Lambda

(This example only works on Linux because it has an sdist. See https://www.pantsbuild.org/docs/awslambda-python.)

```
./pants package helloworld:helloworld-awslambda
```

## Count lines of code

```
./pants count-loc '**/*'
```
