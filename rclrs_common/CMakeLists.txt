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

# This defines a crate with the same name as in the Cargo.toml
# I.e: This defines "rclrs_common"
corrosion_import_crate(MANIFEST_PATH Cargo.toml)

ament_export_targets(export_rclrs_common HAS_LIBRARY_TARGET)

install(
  TARGETS rclrs_common
  EXPORT export_rclrs_common
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

ament_package()