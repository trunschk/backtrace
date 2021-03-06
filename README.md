backtrace
=========

Work in progress.

This is a fork of [nir0s/backtrace](https://github.com/nir0s/backtrace).
Its aim is to slim down the original.

backtrace manipulates Python tracebacks to make them more readable.
It provides different configuration options for coloring and formatting.

NOTE: Didn't test this on Windows yet. Should work.. but don't know how well.


## Installation

backtrace officially supports Linux and OSX on Python 2.7 and 3.4+. Python 2.6 will also probably work, but with no guarantees.

```shell
pip install https://github.com/trunschk/backtrace/archive/master.tar.gz
```


## Usage

backtrace provides two methods for manipulating your tracebacks.

### Piping

You can pipe stderr into backtrace which will try to detect a traceback, parse it and display a beautified trace.

```text
$ backtrace -h
usage: backtrace [-h] [-r] [-a] [-s] [-c]

Beautify Tracebacks.

Just pipe stderr into backtrace like so:
  `python bad-program.py 2>&1 | backtrace`

optional arguments:
  -h, --help          show this help message and exit
  -r, --reverse       Reverse traceback entry order
  -a, --align         Right-align the backtrace
  -s, --strip-path    Strip the path to the module

$ python my-traceback-generating-program.py 2>&1 | backtrace
...

```

![Piping into backtrace](https://github.com/trunschk/backtrace/raw/master/img/piping.png)


### Inside your application

```python
import sys
import backtrace

sys.excepthook = backtrace.hook(
    reverse=False,
    align=False,
    strip_path=False,
    enable_on_envvar_only=False,
    on_tty=False,
    styles={})

...

# if you wanna restore the default hook...
sys.excepthook = sys.__excepthook__

...

```

![Using python API](https://github.com/trunschk/backtrace/raw/master/img/api.png)

You can pass the following flags to `hook` to change backtrace's behavior:

* If `reverse` is True, the traceback entries will be printed in reverse order.
* If `align` is True, all parts (line numbers, file names, etc..) will be
aligned to the left according to the longest entry. This allows for extended readability as your eyes don't have to move between columns to understand what's going on.
* If `strip_path` is True, only the file name will be shown, not its full
path. This is useful when you know you're running in the context of a single module or a single package containing only a root folder so you only need file names. This can help keep the traceback clean.
* If `enable_on_envvar_only` is True, only if the environment variable
`ENABLE_BACKTRACE` is set, backtrace will be activated.
* If `on_tty` is True, backtrace will be activated only if you're running
in a real terminal (i.e. not piped, redirected, etc..). This can help keep the original traceback when logging to files or piping to look for information.
* `styles` is a dictionary containing the styling for each part of the rebuilt traceback. See below.

#### Styles

Styles allow you to customize the coloring and structure of your new traceback. The defaults are:

```python
STYLES = {
    'backtrace': Fore.YELLOW + '{0}',
    'error': Fore.RED + Style.BRIGHT + '{0}',
    'line': Fore.RED + Style.BRIGHT + '{0}',
    'module': '{0}',
    'context': Style.BRIGHT + Fore.GREEN + '{0}',
    'call': Fore.YELLOW + '--> ' + Style.BRIGHT + '{0}',
}
```

Where:

* `backtrace` is the main traceback message.
* `error` is the error message presenting the exception message and its type.
* `line` is the line number of each entry.
* `module` is the calling module of each entry.
* `context` is the calling function/method of each entry.
* `call` is the called function/method/assignment of each entry.

and the `{0}` format place holder is the actual value of the field.

Sending a partial dictionary containing changes in only some parts of the traceback will have `backtrace` use the defaults for whatever wasn't specified.

You can do all sorts of stuff like removing a certain field by setting the formatting of that field to an empty string; add more verbose identifiers to each field by appending an ID in front of it or just adding paranthese around a field.


## Alternatives

* [colored_traceback](https://github.com/staticshock/colored-traceback.py) provides a way to color your tracebacks to make them more readable. It's a nice little tool but lacks actually re-formatting the traceback which is what the biggest problem is from my POV.
