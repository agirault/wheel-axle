# Wheel Axle - Python Wheel enhancement library

[![Gitter](https://img.shields.io/gitter/room/karellen/Lobby?logo=gitter)](https://app.gitter.im/#/room/#karellen_Lobby:gitter.im)
[![Build Status](https://img.shields.io/github/actions/workflow/status/karellen/wheel-axle/build.yml?branch=master)](https://github.com/karellen/wheel-axle/actions/workflows/build.yml)
[![Coverage Status](https://img.shields.io/coveralls/github/karellen/wheel-axle/master?logo=coveralls)](https://coveralls.io/r/karellen/wheel-axle?branch=master)

[![Wheel Axle Version](https://img.shields.io/pypi/v/wheel-axle?logo=pypi)](https://pypi.org/project/wheel-axle/)
[![Wheel Axle Python Versions](https://img.shields.io/pypi/pyversions/wheel-axle?logo=pypi)](https://pypi.org/project/wheel-axle/)

[![Wheel Axle Downloads Per Day](https://img.shields.io/pypi/dd/wheel-axle?logo=pypi)](https://pypistats.org/packages/wheel-axle)
[![Wheel Axle Downloads Per Week](https://img.shields.io/pypi/dw/wheel-axle?logo=pypi)](https://pypistats.org/packages/wheel-axle)
[![Wheel Axle Downloads Per Month](https://img.shields.io/pypi/dm/wheel-axle?logo=pypi)](https://pypistats.org/packages/wheel-axle)

## Problem

1. `bdist_wheel` does not support symlinks.
2. `bdist_wheel` does not support overwriting in a convenient way:
    * whether the distribution is pure-Python
    * distribution ABI tag
    * Extend Python tag override capability

## Solution

**WARNING: THIS IS EXPERIMENTAL BETA SOFTWARE. THERE ARE NO WARRANTIES OF ANY KIND. USE AT YOUR OWN RISK.
ADDITIONAL INCLUDED DISCLAIMERS ALSO APPLY.**

Wheel-Axle (`Axle`, `bdist_axle`) is a drop-in replacement/augmentation utility for `wheel` (`bdist_wheel`)
that extends and builds spec-compliant wheels.

During the build, `Axle` is able to capture and record symlinks in the following schema paths (locations):

* purelib
* platlib
* scripts
* headers
* data

Additionally, `Axle` is able to customize the tags via additional command line options.

While the generated wheel is fully spec-compliant, additional symlinks functionality is not possible without its
companion library [Wheel Axle Runtime](https://github.com/karellen/wheel-axle-runtime). Thus, every wheel generated by
the `bdist_axle` automatically becomes dependent on `wheel-axle-runtime` that provides post-install logic required.

## Implementation

The body of the library is as ugly and messy as `distutils`/`setuptools` are, and consists of, mainly, in
hacking/overwriting various `setuptools` commands to detect, handle and record symlinks and their targets. Once that
problem is solved, the list of symlinks is recorded in the `.dist-info/symlinks.txt`
in the following CSV format:

1. symlink name
2. symlink target
3. a boolean (0 or 1) flag indicating whether the target is a directory

**NOTE: Symlinks may be relative, absolute and/or broken. Symlink targets are recorded verbatim (even when broken) and
are NOT otherwise interpreted. THIS IS INTENTIONAL. Please
see [Wheel Axle Runtime Security Notice](https://github.com/karellen/wheel-axle-runtime#security)
for additional information.**

A special `<distribution name and version>.pth` file is also added to the distribution. When the wheel is installed
this `.pth` file triggers the post-install logic via
[wheel-axle-runtime](https://github.com/karellen/wheel-axle-runtime).

## Usage

`python setup.py bdist_wheel <arguments>` can be replaced with `python setup.py bdist_axle <arguments>`. The replacement
is drop-in.

Additional functionality is available via the following options:

```commandline
  --root-is-pure    set to manually override whether the wheel is
                    pure (default: None)
  --abi-tag         set to override ABI tag (default: None)
```

Using `--python-tag`, `--root-is-pure` and `--abi-tag` allows you to create wheels that carry platform-dependent data
while otherwise containing pure-Python libraries.
