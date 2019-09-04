[![Build status](https://badge.buildkite.com/c3449aba989713394a3237070971eb59b92ad19d6f69555a25.svg)](https://buildkite.com/bazel/rules-jsonnet-postsubmit)

# Jsonnet Rules

<div class="toc">
  <h2>Rules</h2>
  <ul>
    <li><a href="#jsonnet_library">jsonnet_library</a></li>
    <li><a href="#jsonnet_to_json">jsonnet_to_json</a></li>
    <li><a href="#jsonnet_to_json_test">jsonnet_to_json_test</a></li>
  </ul>
</div>

## Overview

These are build rules for working with [Jsonnet][jsonnet] files with Bazel.

[jsonnet]: http://google.github.io/jsonnet/doc/

## Setup

To use the Jsonnet rules, add the following to your `WORKSPACE` file to add the
external repositories for Jsonnet:

```python
http_archive(
    name = "io_bazel_rules_jsonnet",
    # TODO: Update this to reflect a later release.
    sha256 = "59bf1edb53bc6b5adb804fbfabd796a019200d4ef4dd5cc7bdee03acc7686806",
    strip_prefix = "rules_jsonnet-0.1.0",
    urls = ["https://github.com/bazelbuild/rules_jsonnet/archive/0.1.0.tar.gz"],
)
load("@io_bazel_rules_jsonnet//jsonnet:jsonnet.bzl", "jsonnet_repositories")

jsonnet_repositories()

load("@jsonnet_go//bazel:repositories.bzl", "jsonnet_go_repositories")

jsonnet_go_repositories()

load("@jsonnet_go//bazel:deps.bzl", "jsonnet_go_dependencies")

jsonnet_go_dependencies()
```

## Jsonnet Port Selection

By default, Bazel will use [the C++ port](https://github.com/google/jsonnet) of Jsonnet. To use [the Go port](https://github.com/google/go-jsonnet) of Jsonnet instead, invoke Bazel with the `--define jsonnet_port=go` command-line flag. To select the C++ port explicitly, invoke Bazel with the `--define jsonnet_port=cpp` command-line flag.

_bazel_ Flag | Jsonnet Port
------------ | ------------
(none)                     | C++
`--define jsonnet_port=cpp`| C++
`--define jsonnet_port=go` | Go

Note that the primary development focus of the Jsonnet project is now with the Go port. This repository's support for using the C++ port is deprecated, and may be removed in a future release. Before then, its default port will likely change from C++ to Go in the next release.


<a name="#jsonnet_library"></a>
## jsonnet_library

```python
jsonnet_library(name, srcs, deps, imports)
```

<table class="table table-condensed table-bordered table-params">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <thead>
    <tr>
      <th colspan="2">Attributes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <code>Name, required</code>
        <p>A unique name for this rule.</p>
      </td>
    </tr>
    <tr>
      <td><code>srcs</code></td>
      <td>
        <code>List of Labels, required</code>
        <p>
          List of <code>.jsonnet</code> files that comprises this Jsonnet
          library.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>deps</code></td>
      <td>
        <code>List of labels, optional</code>
        <p>
          List of targets that are required by the <code>srcs</code> Jsonnet
          files.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>imports</code></td>
      <td>
        <code>List of strings, optional</code>
        <p>
          List of import <code>-J</code> flags to be passed to the
          <code>jsonnet</code> compiler.
        </p>
      </td>
    </tr>
  </tbody>
</table>

### Example

Suppose you have the following directory structure:

```
[workspace]/
    WORKSPACE
    configs/
        BUILD
        backend.jsonnet
        frontend.jsonnet
```

You can use the `jsonnet_library` rule to build a collection of `.jsonnet`
files that can be imported by other `.jsonnet` files as dependencies:

`configs/BUILD`:

```python
load("@io_bazel_rules_jsonnet//jsonnet:jsonnet.bzl", "jsonnet_library")

jsonnet_library(
    name = "configs",
    srcs = [
        "backend.jsonnet",
        "frontend.jsonnet",
    ],
)
```

<a name="#jsonnet_to_json"></a>
## jsonnet\_to\_json

```python
jsonnet_to_json(name, src, deps, outs, multiple_outputs, imports, stamp_keys, ext_strs, ext_str_envs, ext_code, ext_code_envs ext_str_files, ext_str_file_vars, ext_code_files, ext_code_file_vars, yaml_stream)
```

<table class="table table-condensed table-bordered table-params">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <thead>
    <tr>
      <th colspan="2">Attributes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <code>Name, required</code>
        <p>A unique name for this rule.</p>
        <p>
          This name will be used as the name of the JSON file generated by this
          rule.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>src</code></td>
      <td>
        <code>Label, required</code>
        <p>
          The <code>.jsonnet</code> file to convert to JSON.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>deps</code></td>
      <td>
        <code>List of labels, optional</code>
        <p>
          List of targets that are required by the <code>src</code> Jsonnet
          file.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>outs</code></td>
      <td>
        <code>List of Filenames, required</code>
        <p>
          Names of the output .json files to be generated by this rule.
        </p>
        <p>
          If you are generating only a single JSON file and are not using
          jsonnet multiple output files, then this attribute should only
          contain the file name of the JSON file you are generating.
        </p>
        <p>
          If you are generating multiple JSON files using jsonnet multiple file
          output (<code>jsonnet -m</code>), then list the file names of all the
          JSON files to be generated. The file names specified here must match
          the file names specified in your <code>src</code> Jsonnet file.
        </p>
        <p>
          For the case where multiple file output is used but only for
          generating one output file, set the <code>multiple_outputs</code>
          attribute to 1 to explicitly enable the <code>-m</code> flag for
          multiple file output.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>multiple_outputs</code></td>
      <td>
        <code>bool, optional, default 0</code>
        <p>
          Set to 1 to explicitly enable multiple file output via the
          <code>jsonnet -m</code> flag.
        </p>
        <p>
          This is used for the case where multiple file output is used but only
          for generating a single output file. For example:
        </p>
        <p>
<pre>
local foo = import "foo.jsonnet";

{
  "foo.json": foo,
}
</pre>
        </p>
      </td>
    </tr>
    <tr>
      <td><code>imports</code></td>
      <td>
        <code>List of strings, optional</code>
        <p>
          List of import <code>-J</code> flags to be passed to the
          <code>jsonnet</code> compiler.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>stamp_keys</code></td>
      <td>
        <code>List of strings, optional</code>
        <p>
          Specify which variables in `ext_strs` and `ext_code` should get stamped by listing the matching dict keys.
        </p>
        <p>
          To get outside variables provided by a script invoked via `--workspace_status_command` into the build. For example:
        </p>
<pre>
jsonnet_to_json(
  name = "...",
  ext_strs = {
    cluster = "{CLUSTER}"
  },
  stamp_keys = ["cluster"]
)
</pre>
<pre>
$ cat .bazelrc
build --workspace_status_command=./print-workspace-status.sh

$ cat print-workspace-status.sh
cat &lt;&lt;EOF
VAR1 value1
# This can be overriden by users if they "export CLUSTER_OVERRIDE"
CLUSTER ${CLUSTER_OVERRIDE:-default-value2}
EOF
</pre>
</td>
    </tr>
    <tr>
      <td><code>ext_strs</code></td>
      <td>
        <code>String dict, optional</code>
        <p>
          Map of strings to pass to jsonnet as external variables via <code>--ext-str key=value</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_str_envs</code></td>
      <td>
        <code>String list, optional</code>
        <p>
          List of env var names containing strings to pass to jsonnet as external variables via <code>--ext-str key</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_code</code></td>
      <td>
        <code>String dict, optional</code>
        <p>
          Map of code to pass to jsonnet as external variables via
          <code>--ext-code key=value</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_code_envs</code></td>
      <td>
        <code>String list, optional</code>
        <p>
          List of env var names containing jsonnet code to pass to jsonnet as external variables via
          <code>--ext-code key</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_str_files</code></td>
      <td>
        <code>List of labels, optional but needed together with file_vars</code>
        <p>
          List of string files that map to the var name defined in file_vars at the same index and together are passed to jsonnet via
          <code>--ext-str-file var=file</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_str_file_vars</code></td>
      <td>
        <code>List of string, optional but needed together with files</code>
        <p>
          List of var names that maps to the file defined in files at the same index and together are passed to jsonnet via
          <code>--ext-str-file var=file</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_code_files</code></td>
      <td>
        <code>String dict, optional</code>
        <p>
          List of jsonnet code files that map to the var name defined in ext_code_file_vars at the same index and together are passed to jsonnet via
          <code>--ext-code-file var=file</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_code_file_vars</code></td>
      <td>
        <code>String dict, optional</code>
        <p>
          List of var names that maps to the code file defined in code_files at the same index and together are passed to jsonnet via
          <code>--ext-code-file var=file</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>tla_code_files</code></td>
      <td>
        <code>Label-keyed String dict, optional</code>
        <p>
          Dict of labels referencing code files and a var name, passed to jsonnet via <code>--tla-code-file var=file</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>yaml_stream</code></td>
      <td>
        <code>bool, optional, default is False</code>
        <p>
          Set to 1 to write output as a YAML stream of JSON documents.
        </p>
      </td>
    </tr>
  </tbody>
</table>

### Example

Suppose you have the following directory structure:

```
[workspace]/
    WORKSPACE
    workflows/
        BUILD
        workflow.libsonnet
        wordcount.jsonnet
        intersection.jsonnet
```

Say that `workflow.libsonnet` is a base configuration library for a workflow
scheduling system and `wordcount.jsonnet` and `intersection.jsonnet` both
import `workflow.libsonnet` to define workflows for performing a wordcount and
intersection of two files, respectively.

First, create a `jsonnet_library` target with `workflow.libsonnet`:

`workflows/BUILD`:

```python
load("@io_bazel_rules_jsonnet//jsonnet:jsonnet.bzl", "jsonnet_library")

jsonnet_library(
    name = "workflow",
    srcs = ["workflow.libsonnet"],
)
```

To compile `wordcount.jsonnet` and `intersection.jsonnet` to JSON, define two
`jsonnet_to_json` targets:

```python
jsonnet_to_json(
    name = "wordcount",
    src = "wordcount.jsonnet",
    outs = ["wordcount.json"],
    deps = [":workflow"],
)

jsonnet_to_json(
    name = "intersection",
    src = "intersection.jsonnet",
    outs = ["intersection.json"],
    deps = [":workflow"],
)
```

### Example: Multiple output files

To use Jsonnet's [multiple output files][multiple-output-files], suppose you
add a file `shell-workflows.jsonnet` that imports `wordcount.jsonnet` and
`intersection.jsonnet`:

`workflows/shell-workflows.jsonnet`:

```
local wordcount = import "workflows/wordcount.jsonnet";
local intersection = import "workflows/intersection.jsonnet";

{
  "wordcount-workflow.json": wordcount,
  "intersection-workflow.json": intersection,
}
```

To compile `shell-workflows.jsonnet` into the two JSON files,
`wordcount-workflow.json` and `intersection-workflow.json`, first create a
`jsonnet_library` target containing the two files that
`shell-workflows.jsonnet` depends on:

```python
jsonnet_library(
    name = "shell-workflows-lib",
    srcs = [
        "wordcount.jsonnet",
        "intersection.jsonnet",
    ],
    deps = [":workflow"],
)
```

Then, create a `jsonnet_to_json` target and set `outs` to the list of output
files to indicate that multiple output JSON files are generated:

```python
jsonnet_to_json(
    name = "shell-workflows",
    src = "shell-workflows.jsonnet",
    deps = [":shell-workflows-lib"],
    outs = [
        "wordcount-workflow.json",
        "intersection-workflow.json",
    ],
)
```

[multiple-output-files]: http://google.github.io/jsonnet/doc/commandline.html

<a name="#jsonnet_to_json_test"></a>
## jsonnet\_to\_json\_test

```python
jsonnet_to_json_test(name, src, deps, imports, golden, error=0, regex=False, yaml_stream=False, stamp_keys, ext_strs, ext_str_envs, ext_code, ext_code_envs ext_str_files, ext_str_file_vars, ext_code_files, ext_code_file_vars)
```

<table class="table table-condensed table-bordered table-params">
  <colgroup>
    <col class="col-param" />
    <col class="param-description" />
  </colgroup>
  <thead>
    <tr>
      <th colspan="2">Attributes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td>
        <code>Name, required</code>
        <p>A unique name for this rule.</p>
        <p>
          This name will be used as the name of the JSON file generated by this
          rule.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>src</code></td>
      <td>
        <code>Label, required</code>
        <p>
          The <code>.jsonnet</code> file to convert to JSON.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>deps</code></td>
      <td>
        <code>List of labels, optional</code>
        <p>
          List of targets that are required by the <code>src</code> Jsonnet
          file.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>imports</code></td>
      <td>
        <code>codefileList of strings, optional</code>
        <p>
          List of import <code>-J</code> flags to be passed to the
          <code>jsonnet</code> compiler.
        </p>
      </td>
    </tr>
        <tr>
      <td><code>stamp_keys</code></td>
      <td>
        <code>List of strings, optional</code>
        <p>
          Specify which variables in `ext_strs` and `ext_code` should get stamped by listing the matching dict keys.
        </p>
        <p>
          To get outside variables provided by a script invoked via `--workspace_status_command` into the build.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_strs</code></td>
      <td>
        <code>String dict, optional</code>
        <p>
          Map of strings to pass to jsonnet as external variables via <code>--ext-str key=value</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_str_envs</code></td>
      <td>
        <code>String list, optional</code>
        <p>
          List of env var names containing strings to pass to jsonnet as external variables via <code>--ext-str key</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_code</code></td>
      <td>
        <code>String dict, optional</code>
        <p>
          Map of code to pass to jsonnet as external variables via
          <code>--ext-code key=value</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_code_envs</code></td>
      <td>
        <code>String list, optional</code>
        <p>
          List of env var names containing jsonnet code to pass to jsonnet as external variables via
          <code>--ext-code key</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_str_files</code></td>
      <td>
        <code>List of labels, optional but needed together with file_vars</code>
        <p>
          List of string files that map to the var name defined in file_vars at the same index and together are passed to jsonnet via
          <code>--ext-str-file var=file</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_str_file_vars</code></td>
      <td>
        <code>List of string, optional but needed together with files</code>
        <p>
          List of var names that maps to the file defined in files at the same index and together are passed to jsonnet via
          <code>--ext-str-file var=file</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_code_files</code></td>
      <td>
        <code>String dict, optional</code>
        <p>
          List of jsonnet code files that map to the var name defined in ext_code_file_vars at the same index and together are passed to jsonnet via
          <code>--ext-code-file var=file</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>ext_code_file_vars</code></td>
      <td>
        <code>String dict, optional</code>
        <p>
          List of var names that maps to the code file defined in code_files at the same index and together are passed to jsonnet via
          <code>--ext-code-file var=file</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>golden</code></td>
      <td>
        <code>Label, optional</code>
        <p>
          The expected (combined stdout and stderr) output to compare to the
          output of running <code>jsonnet</code> on <code>src</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>error</code></td>
      <td>
        <code>Integer, optional, default is 0</code>
        <p>
          The expected error code from running <code>jsonnet</code> on
          <code>src</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>regex</code></td>
      <td>
        <code>bool, optional, default is False</code>
        <p>
          Set to 1 if <code>golden</code> contains a regex used to match
          the output of running <code>jsonnet</code> on <code>src</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>yaml_stream</code></td>
      <td>
        <code>bool, optional, default is False</code>
        <p>
          Set to 1 to write output as a YAML stream of JSON documents.
        </p>
      </td>
    </tr>
  </tbody>
</table>

### Example

Suppose you have the following directory structure:

```
[workspace]/
    WORKSPACE
    config/
        BUILD
        base_config.libsonnet
        test_config.jsonnet
        test_config.json
```

Suppose that `base_config.libsonnet` is a library Jsonnet file, containing the
base configuration for a service. Suppose that `test_config.jsonnet` is a test
configuration file that is used to test `base_config.jsonnet`, and
`test_config.json` is the expected JSON output from compiling
`test_config.jsonnet`.

The `jsonnet_to_json_test` rule can be used to verify that compiling a Jsonnet
file produces the expected JSON output. Simply define a `jsonnet_to_json_test`
target and provide the input test Jsonnet file and the `golden` file containing
the expected JSON output:

`config/BUILD`:

```python
load(
    "@io_bazel_rules_jsonnet//jsonnet:jsonnet.bzl",
    "jsonnet_library",
    "jsonnet_to_json_test",
)

jsonnet_library(
    name = "base_config",
    srcs = ["base_config.libsonnet"],
)

jsonnet_to_json_test(
    name = "test_config_test",
    src = "test_config",
    deps = [":base_config"],
    golden = "test_config.json",
)
```

To run the test: `bazel test //config:test_config_test`

### Example: Negative tests

Suppose you have the following directory structure:

```
[workspace]/
    WORKSPACE
    config/
        BUILD
        base_config.libsonnet
        invalid_config.jsonnet
        invalid_config.output
```

Suppose that `invalid_config.jsonnet` is a Jsonnet file used to verify that
an invalid config triggers an assertion in `base_config.jsonnet`, and
`invalid_config.output` is the expected error output.

The `jsonnet_to_json_test` rule can be used to verify that compiling a Jsonnet
file results in an expected error code and error output. Simply define a
`jsonnet_to_json_test` target and provide the input test Jsonnet file, the
expected error code in the `error` attribute, and the `golden` file containing
the expected error output:

`config/BUILD`:

```python
load(
    "@io_bazel_rules_jsonnet//jsonnet:jsonnet.bzl",
    "jsonnet_library",
    "jsonnet_to_json_test",
)

jsonnet_library(
    name = "base_config",
    srcs = ["base_config.libsonnet"],
)

jsonnet_to_json_test(
    name = "invalid_config_test",
    src = "invalid_config",
    deps = [":base_config"],
    golden = "invalid_config.output",
    error = 1,
)
```

To run the test: `bazel test //config:invalid_config_test`
