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
- [ ] `rcpputils`
- [ ] `tracetools`
  - Prefer disabled or minimized tracing while bootstrapping.
- [ ] `libyaml-vendor` or system `yaml`
- [ ] `rosidl-typesupport-c`
- [ ] `rosidl-typesupport-cpp`

## P4: Message Packages

- [ ] `builtin-interfaces`
- [ ] `std-msgs`
- [ ] `geometry-msgs`
  - Required for `geometry_msgs/Twist`.
- [ ] `unique-identifier-msgs`
- [ ] `action-msgs`

## P5: RMW / Communication Layer

- [ ] `rmw`
- [ ] `rmw-implementation`
- [ ] `rmw-dds-common`
- [ ] `rmw-cyclonedds-cpp`
  - Preferred first RMW candidate if dependencies are manageable.
- [ ] `rmw-fastrtps-cpp`
  - Alternative path if matching the existing source build path is more practical.
- [ ] `rmw-fastrtps-shared-cpp`
- [ ] `rmw-fastrtps-dynamic-cpp`

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
