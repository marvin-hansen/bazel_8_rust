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

The error seems to root in crate_universe, as indicated by the error message shown below.

```bash
ERROR: /private/var/tmp/_bazel_marvin/2ca8fbba9913b18ada099dc013e74ad8/external/rules_rust+/crate_universe/private/module_extensions/cargo_bazel_bootstrap.bzl:66:17: Traceback (most recent call last):
        File "/private/var/tmp/_bazel_marvin/2ca8fbba9913b18ada099dc013e74ad8/external/rules_rust+/crate_universe/extension.bzl", 
        line 369, column 37, in _crate_impl
                _generate_hub_and_spokes(module_ctx, cargo_bazel, cfg, annotations, cargo_lockfile = cargo_lockfile, manifests = manifests)
        File "/private/var/tmp/_bazel_marvin/2ca8fbba9913b18ada099dc013e74ad8/external/rules_rust+/crate_universe/extension.bzl", 
        line 139, column 16, in _generate_hub_and_spokes
                cargo_bazel([
        File "/private/var/tmp/_bazel_marvin/2ca8fbba9913b18ada099dc013e74ad8/external/rules_rust+/crate_universe/private/module_extensions/cargo_bazel_bootstrap.bzl", 
        line 66, column 17, in run
                fail("%s returned with exit code %d:\n%s" % (pretty_args, result.return_code, result.stderr))
```