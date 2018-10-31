---
title: "Configuration Scripting"
date: 2018-10-31
categories: [programming,python,script]
tags: [programming,python,config,script]
---
A particular go-to patterns of mine is an imperative, configuration scripting: a set of steps that can be used to to set properties and execute methods. 

## Configuration Scripts

Here a sample configuration script that is actually in-use at the [Broad Institute][broad] where I work. It enables a computational biologist to validate and load their genomic data from a CSV, harmonize it, validate it, transform it (to JSON), and upload it to HDFS:

```
dataset     ExChip_CAMP "Type 2 Diabetes"
tech        ExChip
cases       563
controls    3065

# import common settings
include     ../common/common.txt

# CSV field names
marker      VAR_ID
var_id      chr{chrom}_{pos}_{ref}_{alt}
maf         MAF_PH
pvalue      P_FIRTH
odds_ratio  ODDS_RATIO
stderr      SE

# process input files, each with a different ancestry
ancestry    European
process     /.../data.1.csv

ancestry    Hispanic
process     /.../data.2.csv

ancestry    African
process     /.../data.3.csv
```

The above example might seem quite trivial (and it is, there are some scripts that are more complex), but it's _extremely_ flexible, dead-simple to use and read for a non-programmer, and doesn't require any heavyweight, hierarchal libraries like XML or JSON that may require extra code-side traversal.

## Example Usage

It's really quite simple...

A `ConfigScript` consists of just 2 things:

* a `SimpleNamespace` to hold properties; and
* a dictionary of command to method;

If the method pointed to by a command is one of `str`, `int`, `float`, or `bool`, then it is assumed to be a property that should be parsed and stored in the `SimpleNamespace`. Otherwise, it is executed, being passed the namespace and any other arguments provided on the line.

To make a `ConfigScript` that can set two properties: `a` and `b`, and execute a command to sum them with an optional list of numbers and print the sum of them should only require:

```python
def show_sum(ns, *extra):
    print(ns.a + ns.b + sum(float(x) for x in extra))

ConfigScript(a=float, b=float, sum=show_sum).run()
```

_By default, with no arguments, `run` will read from STDIN, otherwise it expects a filename._

## Show Me the Code!

The code is quite simple (using [Python][python] 3.x, porting to Python 2.x should be trivial):

```python
import os
import os.path
import shlex
import sys
import types


class ConfigScript:
    "A Configuration Script."

    class QuitException(Exception):
        "A custom exception that can be caught by ConfigScript.run."
        pass
    
    def __init__(self, **cmds):
        "Initialize with a set of properties and commands."
        props = [str, int, float, bool]

        # initialize the namespace
        self.ns = types.SimpleNamespace(
            **{k: None for k, v in cmds.items() if v in props}
        )

        # case-insensitive commands and override built-ins
        self.cmds = {
            'quit': self._quit,
            'include': self._include,
            **{k.lower(): v for k, v in cmds.items()},
        }
    
    def _quit(self, ns):
        "Raise a QuitException, which is caught by run."
        raise ConfigScript.QuitException()
    
    def _include(self, ns, source):
        "Load and execute the lines of a script."
        cwd = os.getcwd()

        # reading from standard in doesn't change the diretory
        if source != '-':
            path, source = os.path.split(source)

            # relative to the current directory
            os.chdir(os.path.join(cwd, path))
        
        # open the source
        fp = sys.stdin if source == '-' else open(source, 'r')

        try:
            for line in fp:
                cmdline = shlex.split(line, comments=True)
                if len(cmdline) == 0:
                    continue

                cmd, *rest = cmdline

                # lookup the method
                method = self.cmds.get(cmd)
                if method is None:
                    raise RuntimeError(cmd + ' ?')

                # expand environment variables
                args = [os.path.expandvars(arg) for arg in rest]
                
                # execute the method
                if method in [int, float, str]:
                    setattr(ns, cmd, method(*args))
                elif method == bool:
                    setattr(ns, cmd, str2bool(*args))
                else:
                    method(ns, *args)
        finally:
            if fp != sys.stdin:
                fp.close()

            # restore the original working directory
            os.chdir(cwd)
    
    def run(self, script=None):
        "Load a configuration script and execute it."
        try:
            self._include(self.ns, script or '-')
        except ConfigScript.QuitException:
            pass

def str2bool(flag):
    "Convert a string to a boolean value or raise a ValueError."
    if flag.lower() in ['y', 'yes', '1', 't', 'true', 'on']:
        return True
    if flag.lower() in ['n', 'no', '0', 'f', 'false', 'off']:
        return False

    raise ValueError('Cannot convert "%s" to a bool' % flag)
```

That's it! 

At some point I'll see if I can make this code available to all on [PyPI][pypi]. Until then, please attribute the code if you copy/paste it, but otherwise feel free to use it in your own projects and I hope you find "configuration scripting" as easy and as helpful as I do!

# fin.

[broad]: https://broadinstitute.org/
[python]: https://python.org/
[pypi]: https://pypi.org/