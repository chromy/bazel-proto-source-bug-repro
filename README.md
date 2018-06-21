# Bazel proto\_source\_root

## Issue 1: strict deps

```
$ bazel build //protos:a_and_b_proto
INFO: Build options have changed, discarding analysis cache.
INFO: Analysed target //protos:a_and_b_proto (0 packages loaded).
INFO: Found 1 target...
ERROR: /........................./proj/bazel-example/protos/BUILD:1:1: Generating Descriptor Set proto_library //protos:a_and_b_proto failed (Exit 1)
example/a.proto: example/b.proto is imported, but //protos:a_and_b_proto doesn't directly depend on a proto_library that 'srcs' it.
Target //protos:a_and_b_proto failed to build
Use --verbose_failures to see the command lines of failed build steps.
INFO: Elapsed time: 0.609s, Critical Path: 0.08s
INFO: 0 processes.
FAILED: Build did NOT complete successfully
```

With `-s`:

```
$ bazel build //protos:a_and_b_proto -s
INFO: Analysed target //protos:a_and_b_proto (0 packages loaded).
INFO: Found 1 target...
SUBCOMMAND: # //protos:a_and_b_proto [action 'Generating Descriptor Set proto_library //protos:a_and_b_proto']
(cd /........................./.cache/bazel/_bazel_hjd/d5202dfa29ea627a4a1c6fbe17e0bc9f/execroot/__main__ && \
  exec env - \
    PATH=/........................./google-cloud-sdk/bin:/........................./proj/depot_tools:/........................./.cabal/bin:/........................./flutter/bin:/........................./bin:/usr/lib/.../bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
  bazel-out/host/bin/external/com_google_protobuf/protoc '--proto_path=protos' '--descriptor_set_out=bazel-out/k8-fastbuild/genfiles/protos/a_and_b_proto-descriptor-set.proto.bin' '-Iprotos/example/a.proto=protos/example/a.proto' '-Iprotos/example/b.proto=protos/example/b.proto' --direct_dependencies protos/example/a.proto:protos/example/b.proto '--direct_dependencies_violation_msg=%s is imported, but //protos:a_and_b_proto doesn'\''t directly depend on a proto_library that '\''srcs'\'' it.' protos/example/a.proto protos/example/b.proto)
ERROR: /........................./proj/bazel-example/protos/BUILD:1:1: Generating Descriptor Set proto_library //protos:a_and_b_proto failed (Exit 1)
example/a.proto: example/b.proto is imported, but //protos:a_and_b_proto doesn't directly depend on a proto_library that 'srcs' it.
Target //protos:a_and_b_proto failed to build
Use --verbose_failures to see the command lines of failed build steps.
INFO: Elapsed time: 0.419s, Critical Path: 0.09s
INFO: 0 processes.
FAILED: Build did NOT complete successfully
```

This seems to be due to an error generating the `--direct_dependencies` command
line, if I substitute `--direct_dependencies
protos/example/a.proto:protos/example/b.proto` for
`--direct_dependencies example/a.proto:example/b.proto` it seems to work.

I can also work around this issue with `--strict_proto_deps=off`.

## Issue 2: proto\_cc\_library

```
$ bazel build --strict_proto_deps=off //protos:a_and_b_cc_proto
INFO: Build options have changed, discarding analysis cache.
INFO: Analysed target //protos:a_and_b_cc_proto (0 packages loaded).
INFO: Found 1 target...
ERROR: /.../proj/bazel-example/protos/BUILD:1:1: output 'protos/example/a.pb.h' was not created
ERROR: /.../proj/bazel-example/protos/BUILD:1:1: output 'protos/example/b.pb.h' was not created
ERROR: /.../proj/bazel-example/protos/BUILD:1:1: output 'protos/example/a.pb.cc' was not created
ERROR: /.../proj/bazel-example/protos/BUILD:1:1: output 'protos/example/b.pb.cc' was not created
ERROR: /.../proj/bazel-example/protos/BUILD:1:1: not all outputs were created or valid
Target //protos:a_and_b_cc_proto failed to build
Use --verbose_failures to see the command lines of failed build steps.
INFO: Elapsed time: 0.648s, Critical Path: 0.09s
INFO: 1 process, linux-sandbox.
FAILED: Build did NOT complete successfully
```

With `-s`:

```
$ bazel build --strict_proto_deps=off //protos:a_and_b_cc_proto -s
INFO: Analysed target //protos:a_and_b_cc_proto (0 packages loaded).
INFO: Found 1 target...
SUBCOMMAND: # //protos:a_and_b_proto [action 'Generating C++ proto_library //protos:a_and_b_proto']
(cd /........................./.cache/bazel/_bazel_hjd/d5202dfa29ea627a4a1c6fbe17e0bc9f/execroot/__main__ && \
  exec env - \
    PATH=/........................./google-cloud-sdk/bin:/........................./proj/depot_tools:/........................./.cabal/bin:/........................./flutter/bin:/........................./bin:/usr/lib/.../bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
  bazel-out/host/bin/external/com_google_protobuf/protoc '--proto_path=protos' '--cpp_out=bazel-out/k8-fastbuild/genfiles' '-Iprotos/example/a.proto=protos/example/a.proto' '-Iprotos/example/b.proto=protos/example/b.proto' protos/example/a.proto protos/example/b.proto)
ERROR: /../proj/bazel-example/protos/BUILD:1:1: output 'protos/example/a.pb.h' was not created
ERROR: /../proj/bazel-example/protos/BUILD:1:1: output 'protos/example/b.pb.h' was not created
ERROR: /../proj/bazel-example/protos/BUILD:1:1: output 'protos/example/a.pb.cc' was not created
ERROR: /../proj/bazel-example/protos/BUILD:1:1: output 'protos/example/b.pb.cc' was not created
ERROR: /../proj/bazel-example/protos/BUILD:1:1: not all outputs were created or valid
Target //protos:a_and_b_cc_proto failed to build
Use --verbose_failures to see the command lines of failed build steps.
INFO: Elapsed time: 0.399s, Critical Path: 0.09s
INFO: 1 process, linux-sandbox.
FAILED: Build did NOT complete successfully
```

The `protoc` command generates files
to: `bazel-out/k8-fastbuild/genfiles/example` rather than 
`bazel-out/k8-fastbuild/genfiles/protos/example` which `bazel` seems to expect.
