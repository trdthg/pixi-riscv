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

- [ ] `ament-index-python`
  - Suggested next package. Pure Python and useful early.
- [ ] `ament-index-cpp`
- [ ] `ament-cmake-core`
- [ ] `ament-cmake`
- [ ] `ament-cmake-python`
- [ ] `ament-cmake-ros`
- [ ] `ament-cmake-target-dependencies`
- [ ] `ament-cmake-include-directories`
- [ ] `ament-cmake-libraries`
- [ ] `ament-cmake-export-definitions`
- [ ] `ament-cmake-export-dependencies`
- [ ] `ament-cmake-export-include-directories`
- [ ] `ament-cmake-export-interfaces`
- [ ] `ament-cmake-export-libraries`
- [ ] `ament-cmake-export-link-flags`
- [ ] `ament-cmake-export-targets`

## P2: rosidl / Interface Generation

- [ ] `rosidl-adapter`
- [ ] `rosidl-parser`
- [ ] `rosidl-cmake`
- [ ] `rosidl-runtime-c`
- [ ] `rosidl-runtime-cpp`
- [ ] `rosidl-runtime-py`
- [ ] `rosidl-generator-c`
- [ ] `rosidl-generator-cpp`
- [ ] `rosidl-generator-py`
- [ ] `rosidl-typesupport-interface`

## P3: Low-Level Runtime

- [ ] `rcutils`
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

- `mimick`, `pybind11`, and `ament-package` have already been uploaded to the `riscv` channel.
- If a package is pure Python but needs host `python3`, avoid `noarch: python` for now; the current channel does not provide a conda Python package for the test environment.
- When a package already exists on the channel with the same version and build number, bump `build.number` before re-uploading.
