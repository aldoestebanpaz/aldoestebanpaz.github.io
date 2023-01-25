- [Python usage notes](#python-usage-notes)
  - [Installing packages](#installing-packages)
  - [The \_\_init\_\_.py file](#the-__init__py-file)
  - [How to create a project](#how-to-create-a-project)
  - [What is a pyproject](#what-is-a-pyproject)
  - [Testing with pytest](#testing-with-pytest)
    - [Invoking 'pytest' versus 'python -m pytest'](#invoking-pytest-versus-python--m-pytest)
    - [Python test discovery convention](#python-test-discovery-convention)
  - [Language syntax](#language-syntax)
    - ['*\<list\>'](#list)
    - ['**\<dict\>'](#dict)
    - ['*args' and '**kwargs'](#args-and-kwargs)

# Python usage notes

## Installing packages

```sh
# install in system directory (admin/root required)
pip install <PACKAGE>
# Or, install in user's home directory
pip install --user <PACKAGE>
```

## The \_\_init\_\_.py file

TODO

The '\_\_init\_\_.py' file makes Python treat directories containing it as modules.

Furthermore, this is the first file to be loaded in a module, so you can use it to execute code that you want to run each time a module is loaded, or specify the submodules to be exported.

## How to create a project

The most easiest way is using 'pipenv':

**Install 'pipenv' if it is not installed yet**

```sh
# Install in system directory (admin/root required)
pip install pipenv
# Or, install in user's home directory
pip install --user pipenv
```

**Move into the project directory**

```sh
mkdir myproject
cd myproject
```

**Start installing the packages**

```sh
# Install dependencies from Pipfile, if there is one:
pipenv install
# Or, install dependencies, including the development packages, from Pipfile, if there is one:
pipenv install --dev
# Or, add a package to your new project:
pipenv install <PACKAGE>
```

Example:

```sh
pipenv install requests
```

Pipenv will install the Requests library and create a 'Pipfile' for you in your project’s directory. The Pipfile is used to track which dependencies your project needs in case you need to re-install them, such as when you share your project with others.


**Install development packages**

There are usually some Python packages that are only required in your development environment and not in your production environment, such as unit testing packages. Pipenv will let you keep the two environments separate using the '--dev' flag.

Example:

```sh
pipenv install --dev pytest
pipenv install --dev pytest-mock
pipenv install --dev pytest-cov
```

**Activate the virtual environment**

You can start a pipenv shell:

```sh
pipenv shell
python my_project.py
exit
```

Or, you can also invoke shell commands in your virtual environment, without explicitly activating it first, by using the run keyword.

Example:

```sh
pipenv run python my_project.py
# Or, for example run pytest:
pipenv run pytest
# Or, for example see the code coverage report in HTML format:
pipenv run pytest — cov=. — cov-report html
```

**Custom Script Shortcuts**

Pipenv supports creating custom shortcuts in the (optional) `[scripts]` section of your Pipfile.

You can then run `pipenv run <shortcut name>` in your terminal to run the command in the context of your pipenv virtual environment.

Example:

```ini
[scripts]
echospam = "echo I am really a very silly example"
```

Then:

```sh
pipenv run echospam "indeed"
I am really a very silly example indeed
```

You can also display the names and commands of your shortcuts by running `pipenv scripts`.

Example:

```sh
pipenv scripts
#   command   script
#   echospam  echo I am really a very silly example
```

**Custom environment variables with the .env file**

If a .env file is present in your project, `pipenv shell` and `pipenv run` will automatically load it, for you.

Example:

```sh
cat .env
#   HELLO=WORLD

pipenv run python
#   ...
#   >>> import os
#   >>> os.environ['HELLO']
#   'WORLD'
```

**Create the lock file**

The package names, together with its version and a list of its own dependencies, can be frozen by updating the Pipfile.lock. This is done using the lock keyword.

```sh
pipenv lock
```

'pipenv lock' is used to create a Pipfile.lock, which declares all dependencies (and sub-dependencies) of your project, their latest available versions, and the current hashes for the downloaded files. This ensures repeatable, and most importantly deterministic, builds.

Pipfile.lock takes advantage of some great new security improvements in pip. By default, the Pipfile.lock will be generated with the sha256 hashes of each downloaded package. This will allow pip to guarantee you’re installing what you intend to when on a compromised network, or downloading dependencies from an untrusted PyPI endpoint.

## What is a pyproject

TODO

setup.cfg (if the pip version is older than 21.3, you’ll also need a setup.py file) and pyproject.toml files purpose ?
https://ianhopkinson.org.uk/2022/02/understanding-setup-py-setup-cfg-and-pyproject-toml-in-python/

See [https://github.com/carlosperate/awesome-pyproject](https://github.com/carlosperate/awesome-pyproject) for details.

## Testing with pytest

TODO

### Invoking 'pytest' versus 'python -m pytest'

Running pytest with `pipenv run pytest` instead of `pipenv run python -m pytest` yields nearly equivalent behaviour, except that the latter will add the current directory to 'sys.path', which is standard python behavior.

### Python test discovery convention

pytest will run all files of the form 'test_*.py' or '*_test.py' in the current directory and its subdirectories. More generally, it follows standard test discovery rules.

From those files, pytest collect test items:
- functions or methods outside of class prefixed with 'test'.
- functions or methods, prefixed with 'test', inside 'Test' prefixed classes (without an __init__ method).



See [pytest import mechanisms](https://docs.pytest.org/en/7.1.x/explanation/pythonpath.html) and [Good Integration Practices](https://docs.pytest.org/en/7.1.x/explanation/goodpractices.html) for details.

## Language syntax

TODO

### '*\<list\>'

'*\<list\>'  extracts the contents of a list and passes them as positional parameters to a function.

Example:

```python
def rgb_color(r,g,b):
  print(r,g,b)

# normal call
rgb_color(1, 2, 3)

# using a list
black = [0, 0, 0]
rgb_color(*black)

# using list inline
rgb_color(*[0, 0, 0])
```

### '**\<dict\>'

'**\<dict\>' extracts the contents of a dictionary and passes them as keyword parameters to a function.

Example:

```python
def func(a=1, b=2, c=3):
   print a
   print b
   print b

# normal call
func(1, 2, 3)

# using a dict
params = {'a': 2, 'b': 3, 'c': 4}
func(**params)

# using dict inline
func(**{'a': 2, 'b': 3, 'c': 4})
```

### '*args' and '**kwargs'

The special syntax, '\*args' and '\*\*kwargs' in function definitions is used to pass a variable number of arguments to a function.
- the single asterisk form (*args) is used to pass a non-keyworded, variable-length argument list,
- and the double asterisk form (*kwargs) is used to pass a keyworded, variable-length argument list.

In other words, '*args' extracts positional parameters and '**kwargs' extract keyword parameters.

Example:

```python
def func(*args, **kwargs):
   for arg in args:
        print "another arg: ", arg
   for key in kwargs:
        print "another keyword arg: %s: %s" % (key, kwargs[key])
```



