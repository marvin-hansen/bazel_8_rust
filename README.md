# Reproduction of a rules_rust issue with Bazel 8

Bazel 8 ends the WORKSPACE format in  2024 after which Bzlmod becomes the new standard format for Bazel.

The WORKSPACE file will be disabled by default in Bazel 8 (November 2024) and will be removed in Bazel 9 (late 2025).

See [this migration guide](https://bazel.build/external/migration) 
and this [issue](https://github.com/bazelbuild/bazel/issues/23315) for more details.

This repo reproduces an issue in the rules_rust that causes multiple use cases to fail to build with Bazel 8.

## How to reproduce

1. Clone this repo
2. cd in any or the examples
3. run bazel build //...

It is assumed that Bazelisk is installed and it will download Bazel [8.0.0-pre.20240826.1 (2024-09-07)](https://bazel.build/release/rolling) to build each example. 

The all_crate_deps example fails with the following error:

```bash
bazel build //...
ERROR: /private/var/tmp/_bazel_marvin/2ca8fbba9913b18ada099dc013e74ad8/external/rules_rust+/crate_universe/private/module_extensions/cargo_bazel_bootstrap.bzl:66:17: Traceback (most recent call last):
        File "/private/var/tmp/_bazel_marvin/2ca8fbba9913b18ada099dc013e74ad8/external/rules_rust+/crate_universe/extension.bzl", line 369, column 37, in _crate_impl
                _generate_hub_and_spokes(module_ctx, cargo_bazel, cfg, annotations, cargo_lockfile = cargo_lockfile, manifests = manifests)
        File "/private/var/tmp/_bazel_marvin/2ca8fbba9913b18ada099dc013e74ad8/external/rules_rust+/crate_universe/extension.bzl", line 139, column 16, in _generate_hub_and_spokes
                cargo_bazel([
        File "/private/var/tmp/_bazel_marvin/2ca8fbba9913b18ada099dc013e74ad8/external/rules_rust+/crate_universe/private/module_extensions/cargo_bazel_bootstrap.bzl", line 66, column 17, in run
                fail("%s returned with exit code %d:\n%s" % (pretty_args, result.return_code, result.stderr
                
                /2ca8fbba9913b18ada099dc013e74ad8/external/rules_rust++rust_host_tools+rust_host_tools/bin/rustc returned with exit code 101:
thread 'main' panicked at src/utils.rs:49:22:
Could not rename paths: Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
ERROR: error evaluating module extension crate in @@rules_rust+//crate_universe:extension.bzl
INFO: Elapsed time: 1.890s
INFO: 0 processes.
ERROR: Build did NOT complete successfully
FAILED: 
    Fetching ... @@rules_rust+//crate_universe:extension.bzl; starting
```

The deps_direct example throws:

```bash
bazel build //...
bazel build //...
ERROR: /private/var/tmp/_bazel_marvin/1b5c31766ff67fea6f28348355649997/external/rules_rust+/crate_universe/private/module_extensions/cargo_bazel_bootstrap.bzl:66:17: Traceback (most recent call last):
        File "/private/var/tmp/_bazel_marvin/1b5c31766ff67fea6f28348355649997/external/rules_rust+/crate_universe/extension.bzl", line 383, column 37, in _crate_impl
                _generate_hub_and_spokes(module_ctx, cargo_bazel, cfg, annotations, packages = packages)
        File "/private/var/tmp/_bazel_marvin/1b5c31766ff67fea6f28348355649997/external/rules_rust+/crate_universe/extension.bzl", line 139, column 16, in _generate_hub_and_spokes
                cargo_bazel([
        File "/private/var/tmp/_bazel_marvin/1b5c31766ff67fea6f28348355649997/external/rules_rust+/crate_universe/private/module_extensions/cargo_bazel_bootstrap.bzl", line 66, column 17, in run
                fail("%s returned with exit code %d:\n%s" % (pretty_args, result.return_code, result.stderr))
                
                /1b5c31766ff67fea6f28348355649997/external/rules_rust++rust_host_tools+rust_host_tools/bin/rustc returned with exit code 101:
thread 'main' panicked at src/utils.rs:49:22:
Could not rename paths: Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
ERROR: Analysis of target '//rest_tokio:doc_test' failed; build aborted: error evaluating module extension crate in @@rules_rust+//crate_universe:extension.bzl
INFO: Elapsed time: 5.924s
INFO: 0 processes.
ERROR: Build did NOT complete successfully
```

The hello_world_no_cargo example fails with:

```bash
bazel build //...
bazel build //...
ERROR: /private/var/tmp/_bazel_marvin/dd1db39991f6a5dcc40cdb59e104b4e1/external/rules_rust+/crate_universe/private/module_extensions/cargo_bazel_bootstrap.bzl:66:17: Traceback (most recent call last):
        File "/private/var/tmp/_bazel_marvin/dd1db39991f6a5dcc40cdb59e104b4e1/external/rules_rust+/crate_universe/extension.bzl", line 383, column 37, in _crate_impl
                _generate_hub_and_spokes(module_ctx, cargo_bazel, cfg, annotations, packages = packages)
        File "/private/var/tmp/_bazel_marvin/dd1db39991f6a5dcc40cdb59e104b4e1/external/rules_rust+/crate_universe/extension.bzl", line 139, column 16, in _generate_hub_and_spokes
                cargo_bazel([
        File "/private/var/tmp/_bazel_marvin/dd1db39991f6a5dcc40cdb59e104b4e1/external/rules_rust+/crate_universe/private/module_extensions/cargo_bazel_bootstrap.bzl", line 66, column 17, in run
                fail("%s returned with exit code %d:\n%s" % (pretty_args, result.return_code, result.stderr))
                
        ++rust_host_tools+rust_host_tools/bin/rustc returned with exit code 101:
thread 'main' panicked at src/utils.rs:49:22:
Could not rename paths: Os { code: 2, kind: NotFound, message: "No such file or directory" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
ERROR: Analysis of target '//:hello_world' failed; build aborted: error evaluating module extension crate in @@rules_rust+//crate_universe:extension.bzl
INFO: Elapsed time: 1.979s, Critical Path: 0.00s
INFO: 1 process: 1 internal.
ERROR: Build did NOT complete successfully
FAILED: 
    Fetching ... @@rules_rust+//crate_universe:extension.bzl; starting
    Fetching ...e_extension+local_config_xcode; Building xcode-locator        
```