load("@build_bazel_rules_ios//rules:framework.bzl", "apple_framework")

apple_framework(
    name = "Utils",
    srcs = glob(["*.swift"]),
    bundle_id = "com.modules.Utils",
    link_dynamic = False,
    module_name = "Utils",
    platforms = {"ios": "14.2"},
    deps = ["//App/Data:Data"],
    visibility = ["//App:__subpackages__"],
)