load("@build_bazel_rules_swift//swift:swift.bzl", "swift_library")
load("@build_bazel_rules_apple//apple:ios.bzl", "ios_unit_test")
load("@build_bazel_rules_ios//rules:test.bzl", ios_unit_test_rules_ios = "ios_unit_test")

swift_library(
    name = "Module",
    srcs = ["Module.swift"],
    module_name = "Module",
    visibility = ["//App:__subpackages__"],
    deps = [
        "//App/Data:Data",
        "//App/Utils:Utils"
    ]
)

swift_library(
    testonly = True,
    name = "ModuleTestsLib",
    srcs = ["Test.swift"],
    module_name = "ModuleTestsLib",
    deps = [":Module"],
)

ios_unit_test(
    name = "ModuleTests",
    testonly = True,
    minimum_os_version = "14.2",
    deps = [
        ":ModuleTestsLib",
    ],
)

ios_unit_test_rules_ios(
    name = "ModuleTestsRulesiOS",
    srcs = ["Test.swift"],
    deps = [":Module"],
    minimum_os_version = "14.2",
)