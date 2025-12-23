# Python (Current)

Latest stable version of Python: 3.14.2
Some notes about how Python is working now.

1). First regarding Python itself:

- Always use Python Install Manager to manage Python installations <https://peps.python.org/pep-0773/>.

2). `uv` has quickly been adopted in favor of it's speed and manageability. <https://docs.astral.sh/uv/>

## Python installation manager - pymanager

```pwsh
PS C:\Users\user> pymanager
Python installation manager 25.2
Copyright (c) Python Software Foundation. All Rights Reserved.

Usage:
    pymanager exec <regular Python options>
                                 Launch the default runtime with specified
                                 options, installing it if needed. This is the
                                 equivalent of the python command, but with
                                 auto-install.
    pymanager exec -V:<TAG>      Launch runtime identified by <TAG>, which
                                 should include the company name if not
                                 PythonCore. Regular Python options may follow
                                 this option. The runtime will be installed if
                                 needed.
    pymanager exec -3<VERSION>   Equivalent to -V:PythonCore\3<VERSION>. The
                                 version must begin with a '3', platform
                                 overrides are permitted, and regular Python
                                 options may follow. The runtime will be
                                 installed if needed.
    pymanager help [<CMD>]       Show help for Python installation manager
                                 commands
    pymanager install <TAG>      Download new Python runtimes, or pass --update
                                 to update existing installs.
    pymanager list [<FILTER>]    Show installed Python runtimes, optionally
                                 filtering by <FILTER>.
    pymanager uninstall <TAG>    Remove one or more runtimes from your machine.
                                 Pass --purge to clean up all runtimes and
                                 cached files.

Find additional information at <https://docs.python.org/dev/using/windows>.

Global options: (options must follow the command)
    -v, --verbose    Increased output (log_level=20)
    -vv              Further increased output (log_level=10)
    -q, --quiet      Less output (log_level=30)
    -qq              Even less output (log_level=40)
    -y, --yes        Always accept confirmation prompts (confirm=false)
    -h, -?, --help   Show help for a specific command
    --config=<PATH>  Override configuration with JSON file
```

## uv package manager

```pwsh
PS C:\Users\user> uv
An extremely fast Python package manager.

Usage: uv.exe [OPTIONS] <COMMAND>

Commands:
  auth     Manage authentication
  run      Run a command or script
  init     Create a new project
  add      Add dependencies to the project
  remove   Remove dependencies from the project
  version  Read or update the project's version
  sync     Update the project's environment
  lock     Update the project's lockfile
  export   Export the project's lockfile to an alternate format
  tree     Display the project's dependency tree
  format   Format Python code in the project
  tool     Run and install commands provided by Python packages
  python   Manage Python versions and installations
  pip      Manage Python packages with a pip-compatible interface
  venv     Create a virtual environment
  build    Build Python packages into source distributions and wheels
  publish  Upload distributions to an index
  cache    Manage uv's cache
  self     Manage the uv executable
  help     Display documentation for a command

Cache options:
  -n, --no-cache               Avoid reading from or writing to the cache, instead using a temporary directory for the duration of the
                               operation [env: UV_NO_CACHE=]
      --cache-dir <CACHE_DIR>  Path to the cache directory [env: UV_CACHE_DIR=]

Python options:
      --managed-python       Require use of uv-managed Python versions [env: UV_MANAGED_PYTHON=]
      --no-managed-python    Disable use of uv-managed Python versions [env: UV_NO_MANAGED_PYTHON=]
      --no-python-downloads  Disable automatic downloads of Python. [env: "UV_PYTHON_DOWNLOADS=never"]

Global options:
  -q, --quiet...                                   Use quiet output
  -v, --verbose...                                 Use verbose output
      --color <COLOR_CHOICE>                       Control the use of color in output [possible values: auto, always, never]
      --native-tls                                 Whether to load TLS certificates from the platform which has native certificate store [env:
                                                   UV_NATIVE_TLS=]
      --offline                                    Disable network access [env: UV_OFFLINE=]
      --allow-insecure-host <ALLOW_INSECURE_HOST>  Allow insecure connections to a host [env: UV_INSECURE_HOST=]
      --no-progress                                Hide all progress outputs [env: UV_NO_PROGRESS=]
      --directory <DIRECTORY>                      Change to the given directory prior to running the command [env: UV_WORKING_DIR=]
      --project <PROJECT>                          Discover a project in the given directory [env: UV_PROJECT=]
      --config-file <CONFIG_FILE>                  The path to a `uv.toml` file to use for configuration [env: UV_CONFIG_FILE=]
      --no-config                                  Avoid discovering configuration files (`pyproject.toml`, `uv.toml`) [env:
                                                   UV_NO_CONFIG=]
  -h, --help                                       Display the concise help for this command
  -V, --version                                    Display the uv version

Use `uv help` for more details.
```

## Python usage

```pwsh
# Python Notes

Usage:
    py <regular Python options>
                         Launch the default runtime with specified options. This
                         is the equivalent of the python command.
    py -V:<TAG>          Launch runtime identified by <TAG>, which should
                         include the company name if not PythonCore. Regular
                         Python options may follow this option.
    py -3<VERSION>       Equivalent to -V:PythonCore\3<VERSION>. The version
                         must begin with the digit 3, platform overrides are
                         permitted, and regular Python options may follow. py -3
                         is the equivalent of the python3 command.
    py exec <any of the above>
                         Equivalent to any of the above launch options, and the
                         requested runtime will be installed if needed.
    py help [<CMD>]      Show help for Python installation manager commands
    py install <TAG>     Download new Python runtimes, or pass --update to
                         update existing installs.
    py list [<FILTER>]   Show installed Python runtimes, optionally filtering by
                         <FILTER>.
    py uninstall <TAG>   Remove one or more runtimes from your machine. Pass
                         --purge to clean up all runtimes and cached files.

Find additional information at <https://docs.python.org/dev/using/windows>.
Tool then asks `View online help? [y/N]` and if yes -> <https://docs.python.org/dev/using/windows.html>
```

Python help output (from v3.14.2 on Windows 11 x64)

```python
py --help   
usage: C:\Users\user\AppData\Local\Python\pythoncore-3.14-64\python.exe [option] ... [-c cmd | -m mod | file | -] [arg] ...
Options (and corresponding environment variables):
-b     : issue warnings about converting bytes/bytearray to str and comparing
         bytes/bytearray with str or bytes with int. (-bb: issue errors)
-B     : do not write .pyc files on import; also PYTHONDONTWRITEBYTECODE=x
-c cmd : program passed in as string (terminates option list)
-d     : turn on parser debugging output (for experts only, only works on
         debug builds); also PYTHONDEBUG=x
-E     : ignore PYTHON* environment variables (such as PYTHONPATH)
-h     : print this help message and exit (also -? or --help)
-i     : inspect interactively after running script; forces a prompt even
         if stdin does not appear to be a terminal; also PYTHONINSPECT=x
-I     : isolate Python from the user's environment (implies -E, -P and -s)
-m mod : run library module as a script (terminates option list)
-O     : remove assert and __debug__-dependent statements; add .opt-1 before
         .pyc extension; also PYTHONOPTIMIZE=x
-OO    : do -O changes and also discard docstrings; add .opt-2 before
         .pyc extension
-P     : do not prepend a potentially unsafe path to sys.path; also
         PYTHONSAFEPATH
-q     : do not print version and copyright messages on interactive startup
-s     : do not add user site directory to sys.path; also PYTHONNOUSERSITE=x
-S     : do not imply 'import site' on initialization
-u     : force the stdout and stderr streams to be unbuffered;
         this option has no effect on stdin; also PYTHONUNBUFFERED=x
-v     : verbose (trace import statements); also PYTHONVERBOSE=x
         can be supplied multiple times to increase verbosity
-V     : print the Python version number and exit (also --version)
         when given twice, print more information about the build
-W arg : warning control; arg is action:message:category:module:lineno
         also PYTHONWARNINGS=arg
-x     : skip first line of source, allowing use of non-Unix forms of #!cmd
-X opt : set implementation-specific option
--check-hash-based-pycs always|default|never:
         control how Python invalidates hash-based .pyc files
--help-env: print help about Python environment variables and exit
--help-xoptions: print help about implementation-specific -X options and exit
--help-all: print complete help information and exit

Arguments:
file   : program read from script file
-      : program read from stdin (default; interactive mode if a tty)
arg ...: arguments passed to program in sys.argv[1:]

```

See also @AGENTS.md, AGENTS-MEM.MD
