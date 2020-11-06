cmake_minimum_required(VERSION 3.12)
project(rclrs_common)

# Make Corrosion available to assist with CMake build of Rust
include(FetchContent)
include(CMakePrintHelpers)

FetchContent_Declare(
  Corrosion
  GIT_REPOSITORY https://github.com/AndrewGaspar/corrosion.git
  GIT_TAG origin/master
)

FetchContent_MakeAvailable(Corrosion)

find_package(ament_cmake REQUIRED)
find_package(ament_cmake_export_crates REQUIRED)

# install(
#   FILES Cargo.toml
#   DESTINATION share/${PROJECT_NAME}/rust/
# )

install(
  DIRECTORY src/
  DESTINATION  share/${PROJECT_NAME}/rust/src
)

# ament_export_crates("share/${PROJECT_NAME}/rust")

# This defines a crate with the same name as in the Cargo.toml
# I.e: This defines "rclrs_common"
corrosion_import_crate(MANIFEST_PATH Cargo.toml)

# Debugging
get_directory_property(CURR_TARGETS BUILDSYSTEM_TARGETS)
message("Current Targets: ${CURR_TARGETS}")

ament_export_targets(rclrs_common HAS_LIBRARY_TARGET)

ament_package()