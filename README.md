# conda-mirror-ng

[![Build Status](https://travis-ci.com/xhochy/conda-mirror-ng.svg?branch=master)](https://travis-ci.com/xhochy/conda-mirror-ng)
<!-- [![PyPI version](https://badge.fury.io/py/conda-mirror-ng.svg)](https://badge.fury.io/py/conda-mirror-ng) -->
<!-- [![codecov](https://codecov.io/gh/Valassis-Digital-Media/conda-mirror/branch/master/graph/badge.svg)](https://codecov.io/gh/Valassis-Digital-Media/conda-mirror) -->

Mirrors an upstream conda channel to a local directory.

## Install

`conda-mirror-ng` is available on PyPI and conda-forge.

Install with:

`pip install conda-mirror-ng`

or:

`conda install conda-mirror-ng -c conda-forge`

## Compatibility

`conda-mirror-ng` is intentionally a Py3.6+ only package

## CLI

CLI interface for `conda-mirror-ng`

```
usage: conda-mirror-ng [-h] [--upstream-channel UPSTREAM_CHANNEL]
                       [--target-directory TARGET_DIRECTORY]
                       [--temp-directory TEMP_DIRECTORY] [--platform PLATFORM]
                       [-v] [--config CONFIG] [--pdb] [--num-threads NUM_THREADS]
                       [--version] [--dry-run] [--no-validate-target]
                       [--minimum-free-space MINIMUM_FREE_SPACE]

CLI interface for conda-mirror-ng

optional arguments:
  -h, --help            show this help message and exit
  --upstream-channel UPSTREAM_CHANNEL
                        The target channel to mirror. Can be a channel on
                        anaconda.org like "conda-forge" or a full qualified
                        channel like "https://repo.continuum.io/pkgs/free/"
  --target-directory TARGET_DIRECTORY
                        The place where packages should be mirrored to
  --temp-directory TEMP_DIRECTORY
                        Temporary download location for the packages. Defaults
                        to a randomly selected temporary directory. Note that
                        you might need to specify a different location if your
                        default temp directory has less available space than
                        your mirroring target
  --platform PLATFORM   The OS platform(s) to mirror. one of: {'linux-64',
                        'linux-32','osx-64', 'win-32', 'win-64'}
  -v, --verbose         logging defaults to error/exception only. Takes up to
                        three '-v' flags. '-v': warning. '-vv': info. '-vvv':
                        debug.
  --config CONFIG       Path to the yaml config file
  --pdb                 Enable PDB debugging on exception
  --num-threads NUM_THREADS
                        Num of threads for validation. 1: Serial mode. 0: All
                        available.
  --version             Print version and quit
  --dry-run             Show what will be downloaded and what will be removed.
                        Will not validate existing packages
  --no-validate-target  Skip validation of files already present in target-
                        directory
  --minimum-free-space MINIMUM_FREE_SPACE
                        Threshold for free diskspace. Given in megabytes.
```

## Example Usage

WARNING: Invoking this command will pull ~10GB and take at least an hour

`conda-mirror-ng --upstream-channel conda-forge --target-directory local_mirror --platform linux-64`

## More Details

### blacklist/whitelist configuration

example-conf.yaml:

```yaml
blacklist:
    - license: "*agpl*"
    - license: None
    - license: ""

whitelist:
    - name: system
```

`blacklist` removes package(s) that match the condition(s) listed from the
upstream repodata.

`whitelist` re-includes any package(s) from blacklist that match the
whitelist conditions.

blacklist and whitelist both take lists of dictionaries. The keys in the
dictionary need to be values in the `repodata.json` metadata. The values are
(unix) globs to match on. Go here for the full repodata of the upstream
"defaults" channel:
http://conda.anaconda.org/anaconda/linux-64/repodata.json

Here are the contents of one of the entries in repodata['packages']

```python
{'botocore-1.4.10-py34_0.tar.bz2': {'arch': 'x86_64',
  'binstar': {'channel': 'main',
   'owner_id': '55fc8527d3234d09d4951c71',
   'package_id': '56b88ea1be1cc95a362b218e'},
  'build': 'py34_0',
  'build_number': 0,
  'date': '2016-04-11',
  'depends': ['docutils >=0.10',
   'jmespath >=0.7.1,<1.0.0',
   'python 3.4*',
   'python-dateutil >=2.1,<3.0.0'],
  'license': 'Apache',
  'md5': 'b35a5c1240ba672e0d9d1296141e383c',
  'name': 'botocore',
  'platform': 'linux',
  'requires': [],
  'size': 1831799,
  'version': '1.4.10'}}
```

See implementation details in the `conda_mirror:match` function for more
information.

#### Common usage patterns
##### Mirror **only** one specific package
If you wanted to match exactly the botocore package listed above with your
config, then you could use the following configuration to first blacklist
**all** packages and then include just the botocore packages:

```yaml
blacklist:
    - name: "*"
whitelist:
    - name: botocore
      version: 1.4.10
      build: py34_0
```
##### Mirror everything but agpl licenses
```yaml
blacklist:
    - license: "*agpl*"
```

##### Mirror only python 3 packages
```yaml
blacklist:
    - name: "*"
whitelist:
    - build: "*py3*"
```

## Testing

### Install test requirements

Note: Will install packages from pip

```
$ pip install -r test-requirements.txt
Requirement already satisfied: pytest in /home/edill/miniconda/lib/python3.5/site-packages (from -r test-requirements.txt (line 1))
Requirement already satisfied: coverage in /home/edill/miniconda/lib/python3.5/site-packages (from -r test-requirements.txt (line 2))
Requirement already satisfied: pytest-ordering in /home/edill/miniconda/lib/python3.5/site-packages (from -r test-requirements.txt (line 3))
Requirement already satisfied: py>=1.4.29 in /home/edill/miniconda/lib/python3.5/site-packages (from pytest->-r test-requirements.txt (line 1))
```

### Run the tests, invoking with the `coverage` tool.

```
$ coverage run run_tests.py
sys.argv=['run_tests.py']
========================================= test session starts ==========================================
platform linux -- Python 3.5.3, pytest-3.0.6, py-1.4.31, pluggy-0.4.0 -- /home/edill/miniconda/bin/python
cachedir: .cache
rootdir: /home/edill/dev/maxpoint/github/conda-mirror, inifile:
plugins: xonsh-0.5.2, ordering-0.4
collected 4 items

test/test_conda_mirror.py::test_match PASSED
test/test_conda_mirror.py::test_cli[https://repo.continuum.io/pkgs/free-linux-64] PASSED
test/test_conda_mirror.py::test_cli[conda-forge-linux-64] PASSED
test/test_conda_mirror.py::test_handling_bad_package PASSED

======================================= 4 passed in 4.41 seconds =======================================
```

## Releasing

To release you need three things

1. Commit rights to conda-mirror-ng
2. Access to `conda-mirror-ng` on PyPI
3. Access to `conda-mirror-ng-feedstock` on conda-forge

After you have all three of these things, add a git tag, upload an sdist using
`twine` and merge the automatically generated PR on conda-forge.
