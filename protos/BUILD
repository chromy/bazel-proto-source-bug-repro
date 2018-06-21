proto_library(
    name = "a_and_b_proto",
    srcs = [
        "example/a.proto",
        "example/b.proto",
    ],
    proto_source_root = "protos",
)

cc_proto_library(
    name = "a_and_b_cc_proto",
    deps = [":a_and_b_proto"],
)
