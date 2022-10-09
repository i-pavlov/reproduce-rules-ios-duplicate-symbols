This repo reproduces linker issue while using a combination of swift_library and apple_framework rules.

## Dependency graph
1. `//App/Utils:Utils` depends on `//App/Data:Data`
2. `//App/Module:Module` depends on both `//App/Utils:Utils` and `//App/Data:Data`
3. `//App/Module:ModuleTests` depends on `//App/Module:Module`

<details>
  <summary>Dependency graph diagram</summary>
  
![dependency_graph drawio](https://user-images.githubusercontent.com/6316558/194778343-618ad4e6-4af5-4d69-ba3e-30741e7bbd97.png)
</details>

## How to reproduce

To reproduce the issue: run `ModuleTests` by 
```
bazel test //App/Module:ModuleTests
```

## Issue details
`libData` symbols are duplicated in tests binary

``` bash 
 /Users/ipavlov/Developer/bazel_platforms_test/App/Module/BUILD:24:14: Linking App/Module/ModuleTests.__internal__.__test_bundle_bin failed: (Aborted): wrapped_clang failed: error executing command 
  (cd /private/var/tmp/_bazel_ipavlov/23ebc518c1b7878a1b2c6f2ef4486ae1/sandbox/darwin-sandbox/332/execroot/__main__ && \
  exec env - \
    APPLE_SDK_PLATFORM=iPhoneSimulator \
    APPLE_SDK_VERSION_OVERRIDE=15.2 \
    PATH=/bin:/usr/bin:/usr/local/bin \
    XCODE_VERSION_OVERRIDE=13.2.1.13C100 \
    ZERO_AR_DATE=1 \
  external/local_config_cc/wrapped_clang @bazel-out/ios-sim_arm64-min14.2-applebin_ios-ios_sim_arm64-dbg-ST-19517cdb79de/bin/App/Module/ModuleTests.__internal__.__test_bundle_bin-2.params)
# Configuration: 59b07bb4057e96974d443e7f597806a5e3cd105462e50da71b86a983ab7c4496
# Execution platform: @local_config_platform//:host

Use --sandbox_debug to see verbose messages from the sandbox and retain the sandbox build root for debugging
duplicate symbol 'nominal type descriptor for Data.Item' in:
    bazel-out/ios-sim_arm64-min14.2-applebin_ios-ios_sim_arm64-dbg-ST-19517cdb79de/bin/App/Data/libData.a(Contract.swift.o)
    bazel-out/ios-sim_arm64-min14.2-applebin_ios-ios_sim_arm64-dbg-ST-37412340fa95/bin/App/Data/libData.a(Contract.swift.o)
```

If print subcommands, it's visible that `libData` was compiled twice.


``` bash
SUBCOMMAND: # //App/Data:Data [action 'Compiling Swift module //App/Data:Data', configuration: 92d4594cb8fd2799aa91d544c582450cb49f7bc02075d927c8db7c90cf3652a7, execution platform: @local_config_platform//:host]
(cd /private/var/tmp/_bazel_ipavlov/23ebc518c1b7878a1b2c6f2ef4486ae1/execroot/__main__ && \
  exec env - \
    APPLE_SDK_PLATFORM=iPhoneSimulator \
    APPLE_SDK_VERSION_OVERRIDE=15.2 \
    SWIFT_AVOID_WARNING_USING_OLD_DRIVER=1 \
    XCODE_VERSION_OVERRIDE=13.2.1.13C100 \
  bazel-out/darwin_arm64-opt-exec-2B5CBBC6-ST-f0056b8e5833/bin/external/build_bazel_rules_swift/tools/worker/worker swiftc @bazel-out/ios-sim_arm64-min14.2-applebin_ios-ios_sim_arm64-dbg-ST-37412340fa95/bin/App/Data/Data.swiftmodule-0.params)
# Configuration: 92d4594cb8fd2799aa91d544c582450cb49f7bc02075d927c8db7c90cf3652a7
# Execution platform: @local_config_platform//:host
```

And another run, with sime params, but different configuration.

``` bash
SUBCOMMAND: # //App/Data:Data [action 'Compiling Swift module //App/Data:Data', configuration: 03f1ac5615d5fcff438b70835b28294822a3e2d8e6fd637bb3830da04dd4addb, execution platform: @local_config_platform//:host]
(cd /private/var/tmp/_bazel_ipavlov/23ebc518c1b7878a1b2c6f2ef4486ae1/execroot/__main__ && \
  exec env - \
    APPLE_SDK_PLATFORM=iPhoneSimulator \
    APPLE_SDK_VERSION_OVERRIDE=15.2 \
    SWIFT_AVOID_WARNING_USING_OLD_DRIVER=1 \
    XCODE_VERSION_OVERRIDE=13.2.1.13C100 \
  bazel-out/darwin_arm64-opt-exec-2B5CBBC6-ST-dedf359489b7/bin/external/build_bazel_rules_swift/tools/worker/worker swiftc @bazel-out/ios-sim_arm64-min14.2-applebin_ios-ios_sim_arm64-dbg-ST-19517cdb79de/bin/App/Data/Data.swiftmodule-0.params)
# Configuration: 03f1ac5615d5fcff438b70835b28294822a3e2d8e6fd637bb3830da04dd4addb
# Execution platform: @local_config_platform//:host
```
