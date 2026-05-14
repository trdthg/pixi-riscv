# AI Contribution Guide

This repository bootstraps ROS 2 Humble packages for `linux-riscv64` on a native
RISC-V runner. Treat package version consistency as a correctness requirement,
not a cleanup task.

## Version Policy

- Check the ROS 2 Humble rosdistro before adding or updating ROS recipes:
  `https://raw.githubusercontent.com/ros/rosdistro/master/humble/distribution.yaml`
- Keep packages from the same upstream repository on the same rosdistro release
  version. Examples:
  - `rmw` and `rmw_implementation_cmake` come from `ros2/rmw`.
  - `rmw_fastrtps_cpp`, `rmw_fastrtps_dynamic_cpp`, and
    `rmw_fastrtps_shared_cpp` come from `ros2/rmw_fastrtps`.
- Do not mix newer non-Humble tags into the runtime chain unless the dependent
  packages are moved together and runtime tests prove compatibility.
- Prefer exact version pins in demo projects when the channel contains older
  experimental builds that should not be selected by `*`.

Current Humble RMW alignment:

- `rmw`: `6.1.2`
- `rmw_implementation_cmake`: `6.1.2`
- `rmw_implementation`: `2.8.5`
- `rmw_dds_common`: `1.6.0`
- `rmw_fastrtps_cpp`: `6.2.10`
- `rmw_fastrtps_dynamic_cpp`: `6.2.10`
- `rmw_fastrtps_shared_cpp`: `6.2.10`

## Recipe Style

- Use one recipe per package in `recipes/<name>/recipe.yaml`.
- Use host `gcc`, `g++`, `cmake`, `make`, and `python3` until equivalent
  `linux-riscv64` packages exist in this channel.
- Keep future conda toolchain dependencies commented in recipes for later
  conversion.
- Prefer upstream CMake installs once the prerequisite package graph exists.
  Hand-written headers, CMake config files, or shim libraries are acceptable
  only during bootstrap and must be documented in comments.
- If a recipe intentionally deviates from upstream build behavior, add a test
  that covers the behavior downstream packages need.

## Runtime Gates

- Shared-library packages in the RMW path should include an `ldd -r` test and
  fail on unresolved symbols.
- CMake package recipes should include a small `find_package(...)` test.
- Message/interface packages should verify generated `.idl`, generated C/C++
  headers, and at least one generated typesupport library.
- Client libraries should compile and run a minimal downstream program using
  their exported CMake targets.

## Workflow

- Let the push workflow build changed recipes when possible.
- Use manual workflow dispatch with `recipe=<name>` for targeted rebuilds.
- Verify uploads with the real package URL, for example:
  `https://prefix.dev/riscv/linux-riscv64/<package>-<version>-<build>.conda`
- Do not delete or yank packages as a first response to solver issues. Prefer
  project-side version pins, then document channel cleanup as a separate action.

## Documentation

- Update `TODO.md` when package status, version policy, or phase scope changes.
- Mention known bootstrap compromises and how they should be removed later.
- Keep user-facing demo instructions separate from packaging internals.
