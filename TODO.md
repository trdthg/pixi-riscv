# pixi-riscv Packaging TODO

Goal: build a minimal ROS 2 Python control endpoint for `linux-riscv64`, starting from the smallest packages and using host-provided build tools where conda packages are not available yet.

Current policy:

- Prefer one recipe per package under `recipes/<name>/recipe.yaml`.
- Use host tools such as `gcc`, `cmake`, and `python3` until the `riscv` channel has enough bootstrap packages.
- Keep future conda dependencies commented in recipes when useful, so they can be enabled later.
- Align ROS package versions with the ROS 2 Humble rosdistro before adding or updating runtime packages.
- Do not target ROS 2 Desktop, Gazebo, Nav2, or SLAM on `riscv64` in this repository.
- Primary target is enough runtime to publish `geometry_msgs/Twist` to `/cmd_vel_teleop`.
- `conda-forge/linux-riscv64` currently has repodata but no usable package records, so do not assume `python`, `cmake`, compiler, or system-library packages exist there.

## P-1: Conda Runtime Bootstrap

- [x] `cmake`
  - Bootstrap with host compiler and CMake's own `bootstrap` script so future recipes can depend on `$PREFIX/bin/cmake`.
- [ ] `python`
  - Preferred long-term fix for Python path instability. Package CPython into the channel, then make Python packages depend on it and install into `$PREFIX/lib/pythonX.Y/site-packages`.
  - First target is CPython 3.12.x to match the current RoboStack Humble `py312` ABI choice.
  - Bootstrap dependency chain likely includes at least `openssl`, `zlib`, `bzip2`, `xz`, `libffi`, `sqlite`, `readline`, and `ncurses` if we want a relocatable conda-style Python.
  - A host-linked CPython package is possible as an intermediate step, but it should be marked as bootstrap-only because it depends on the runner/system libraries.
- [ ] `make`
  - Bootstrap GNU Make so recipes can stop assuming a system `make`.
- [ ] `ninja`
  - Bootstrap Ninja for CMake projects that prefer the Ninja generator.

## P0: Completed Bootstrap Packages

- [x] `mimick`
  - Provides the `riscv64`-capable Mimick fork needed to unblock ROS 2 Humble's `mimick_vendor` path.
- [x] `pybind11`
  - Provides `pybind11 2.13.6`, needed for Python 3.13 compatibility in the `rclpy` path.
- [x] `ament-package`
  - Provides the early Python metadata helper used by the ament build system.
- [x] `pyparsing`
  - Provides the pure-Python parser dependency used by `catkin-pkg`.
- [x] `setuptools`
  - Provides `find_packages` and setup helpers required by generated ament Python install steps.
- [x] `lark`
  - Provides the parser backend required by `rosidl-parser`.
- [x] `rpyutils`
  - Provides Python runtime utilities used by `rosidl-generator-py`.
- [x] `empy`
  - Provides the `em` Python module used by `rosidl_adapter` templates. Use the ROS 2 Humble-compatible 3.x API.
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
- [x] `rosidl-default-generators`
- [x] `rosidl-default-runtime`
- [x] `python-cmake-module`
  - Required by `rosidl-generator-py` during real interface generation.

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
- [x] `rmw-fastrtps-dynamic-cpp`
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

- [x] `rcl`
- [x] `rcl-action`
- [x] `rcl-lifecycle`
- [x] `rclpy`
- [x] `libstatistics-collector`
  - Required by `rclcpp`.
- [x] `rclcpp`
  - C++ client library added as an optional extension beyond the Python control endpoint.

## P7: Control Endpoint Applications

- [ ] `teleop-twist-keyboard`
- [ ] Minimal custom `rclpy` `/cmd_vel_teleop` publisher
  - Keep as a fallback if packaging `teleop-twist-keyboard` is not worth the dependency surface.

## Notes

- P0 and P1 have already been uploaded to the `riscv` channel.
- If a package is pure Python but needs host `python3`, avoid `noarch: python` for now; the current channel does not provide a conda Python package for the test environment.
- When a package already exists on the channel with the same version and build number, bump `build.number` before re-uploading.
- Current Humble RMW alignment is `rmw` / `rmw_implementation_cmake` 6.1.2, `rmw_implementation` 2.8.5, `rmw_dds_common` 1.6.0, and `rmw_fastrtps*` 6.2.10.
- Do not use `rmw_implementation` 3.x with the Humble FastDDS RMW chain; it probes newer RMW symbols and produces startup errors with Humble `rmw_fastrtps`.
- `builtin-interfaces`, `unique-identifier-msgs`, `std-msgs`, and `geometry-msgs` now use upstream CMake / rosidl generation and install generated C, C++, Python, and typesupport artifacts.
- `action-msgs` still needs the same real rosidl generation conversion before moving deeper into `rcl-action`.
- P5 currently uses the FastDDS RMW packages as the concrete DDS transport. Keep `ldd -r` tests on RMW shared libraries to catch unresolved runtime symbols.
