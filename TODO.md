# pixi-riscv Packaging TODO

Goal: build a minimal ROS 2 Python control endpoint for `linux-riscv64`, starting from the smallest packages and using host-provided build tools where conda packages are not available yet.

Current policy:

- Prefer one recipe per package under `recipes/<name>/recipe.yaml`.
- Use host tools such as `gcc`, `cmake`, and `python3` until the `riscv` channel has enough bootstrap packages.
- Keep future conda dependencies commented in recipes when useful, so they can be enabled later.
- Do not target ROS 2 Desktop, Gazebo, Nav2, or SLAM on `riscv64` in this repository.
- Primary target is enough runtime to publish `geometry_msgs/Twist` to `/cmd_vel_teleop`.

## P0: Completed Bootstrap Packages

- [x] `mimick`
  - Provides the `riscv64`-capable Mimick fork needed to unblock ROS 2 Humble's `mimick_vendor` path.
- [x] `pybind11`
  - Provides `pybind11 2.13.6`, needed for Python 3.13 compatibility in the `rclpy` path.
- [x] `ament-package`
  - Provides the early Python metadata helper used by the ament build system.
- [x] `pyparsing`
  - Provides the pure-Python parser dependency used by `catkin-pkg`.
- [x] `catkin-pkg`
  - Provides the package manifest parser required by `ament-cmake-core`.
- [x] `patchelf`
  - Provides the ELF relocation tool needed by rattler-build for larger shared-library packages.
- [x] `gtest`
  - Provides GoogleTest C++ test libraries used by ament CMake integration packages.
- [x] `gmock`
  - Provides GoogleMock C++ test libraries used by ament CMake integration packages.

## P1: ament / colcon Foundation

- [x] `ament-index-python`
- [x] `ament-index-cpp`
- [x] `ament-cmake-gen-version-h`
  - Required by `ament-index-cpp`.
- [x] `ament-cmake-core`
- [x] `ament-cmake`
- [x] `ament-cmake-python`
- [x] `ament-cmake-ros`
- [x] `ament-cmake-target-dependencies`
- [x] `ament-cmake-include-directories`
- [x] `ament-cmake-libraries`
- [x] `ament-cmake-export-definitions`
- [x] `ament-cmake-export-dependencies`
- [x] `ament-cmake-export-include-directories`
- [x] `ament-cmake-export-interfaces`
- [x] `ament-cmake-export-libraries`
- [x] `ament-cmake-export-link-flags`
- [x] `ament-cmake-export-targets`
- [x] `ament-cmake-test`
- [x] `ament-cmake-pytest`
- [x] `ament-cmake-gtest`
- [x] `ament-cmake-gmock`

## P2: rosidl / Interface Generation

- [x] `rosidl-cli`
  - Required by `rosidl-adapter` and generator CLI entry points.
- [x] `rosidl-adapter`
- [x] `rosidl-parser`
- [x] `rosidl-cmake`
- [x] `rosidl-runtime-c`
- [x] `rosidl-runtime-cpp`
- [x] `rosidl-runtime-py`
- [x] `rosidl-generator-c`
- [x] `rosidl-generator-cpp`
- [x] `rosidl-generator-py`
- [x] `rosidl-typesupport-interface`

## P3: Low-Level Runtime

- [x] `rcutils`
  - Built early because `rosidl-runtime-c` requires it.
- [x] `rcpputils`
- [x] `tracetools`
  - Prefer disabled or minimized tracing while bootstrapping.
- [x] `libyaml-vendor` or system `yaml`
- [x] `rosidl-typesupport-c`
- [x] `rosidl-typesupport-cpp`

## P4: Message Packages

- [x] `builtin-interfaces`
- [x] `std-msgs`
- [x] `geometry-msgs`
  - Required for `geometry_msgs/Twist`.
- [x] `unique-identifier-msgs`
- [x] `action-msgs`

## P5: RMW / Communication Layer

- [x] `rosidl-typesupport-introspection-c`
  - Shared prerequisite for CycloneDDS and FastDDS RMW implementations.
- [x] `rosidl-typesupport-introspection-cpp`
  - Shared prerequisite for CycloneDDS and FastDDS RMW implementations.
- [x] `rmw`
- [x] `rmw-implementation-cmake`
  - Required CMake helper package for `rmw-implementation`.
- [x] `rmw-implementation`
  - Bootstrap proxy library only; real DDS communication still requires one concrete RMW implementation below.
- [x] `rmw-dds-common`
- [ ] `rmw-cyclonedds-cpp`
  - Preferred first RMW candidate if dependencies are manageable.
- [x] `rmw-fastrtps-cpp`
  - Alternative path if matching the existing source build path is more practical.
- [x] `rmw-fastrtps-shared-cpp`
- [ ] `rmw-fastrtps-dynamic-cpp`
- [x] `fastcdr`
  - FastDDS serialization dependency.
- [x] `foonathan-memory`
  - FastDDS allocator dependency providing CMake package `foonathan_memory`.
- [x] `asio`
  - Header-only standalone Asio dependency for FastDDS.
- [x] `tinyxml2`
  - XML parser dependency for FastDDS.
- [x] `fastrtps`
  - FastDDS transport library dependency.
- [x] `fastrtps-cmake-module`
  - CMake compatibility shim expected by ROS 2 Humble FastDDS packages.
- [x] `rosidl-typesupport-fastrtps-c`
- [x] `rosidl-typesupport-fastrtps-cpp`

## P6: rcl / Python Client Library

- [ ] `rcl`
- [ ] `rcl-action`
- [ ] `rcl-lifecycle`
- [ ] `rclpy`

## P7: Control Endpoint Applications

- [ ] `teleop-twist-keyboard`
- [ ] Minimal custom `rclpy` `/cmd_vel_teleop` publisher
  - Keep as a fallback if packaging `teleop-twist-keyboard` is not worth the dependency surface.

## Notes

- P0 and P1 have already been uploaded to the `riscv` channel.
- If a package is pure Python but needs host `python3`, avoid `noarch: python` for now; the current channel does not provide a conda Python package for the test environment.
- When a package already exists on the channel with the same version and build number, bump `build.number` before re-uploading.
- P4 message packages currently install interface files, generated `.idl`, ament index entries, and CMake metadata. Generated language runtime libraries and Python message modules are intentionally deferred until the client library/runtime packaging stages.
- P5 currently has the base RMW API and runtime-selection proxy. It does not yet provide a concrete DDS transport, so it cannot publish or subscribe until `rmw-cyclonedds-cpp` or the FastDDS packages are packaged.
