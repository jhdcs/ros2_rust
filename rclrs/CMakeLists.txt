cmake_minimum_required(VERSION 3.12)
project(rclrs NONE)

# Get all propreties that cmake supports
execute_process(COMMAND cmake --help-property-list OUTPUT_VARIABLE CMAKE_PROPERTY_LIST)

# Convert command output into a CMake list
STRING(REGEX REPLACE ";" "\\\\;" CMAKE_PROPERTY_LIST "${CMAKE_PROPERTY_LIST}")
STRING(REGEX REPLACE "\n" ";" CMAKE_PROPERTY_LIST "${CMAKE_PROPERTY_LIST}")
# Fix https://stackoverflow.com/questions/32197663/how-can-i-remove-the-the-location-property-may-not-be-read-from-target-error-i
list(FILTER CMAKE_PROPERTY_LIST EXCLUDE REGEX "^LOCATION$|^LOCATION_|_LOCATION$")
# For some reason, "TYPE" shows up twice - others might too?
list(REMOVE_DUPLICATES CMAKE_PROPERTY_LIST)

# build whitelist by filtering down from CMAKE_PROPERTY_LIST in case cmake is
# a different version, and one of our hardcoded whitelisted properties
# doesn't exist!
unset(CMAKE_WHITELISTED_PROPERTY_LIST)
foreach(prop ${CMAKE_PROPERTY_LIST})
    if(prop MATCHES "^(INTERFACE|[_a-z]|IMPORTED_LIBNAME_|MAP_IMPORTED_CONFIG_)|^(COMPATIBLE_INTERFACE_(BOOL|NUMBER_MAX|NUMBER_MIN|STRING)|EXPORT_NAME|IMPORTED(_GLOBAL|_CONFIGURATIONS|_LIBNAME)?|NAME|TYPE|NO_SYSTEM_FROM_IMPORTED)$")
        list(APPEND CMAKE_WHITELISTED_PROPERTY_LIST ${prop})
    endif()
endforeach(prop)

function(print_properties)
    message ("CMAKE_PROPERTY_LIST = ${CMAKE_PROPERTY_LIST}")
endfunction(print_properties)

function(print_whitelisted_properties)
    message ("CMAKE_WHITELISTED_PROPERTY_LIST = ${CMAKE_WHITELISTED_PROPERTY_LIST}")
endfunction(print_whitelisted_properties)

function(print_target_properties tgt)
    if(NOT TARGET ${tgt})
      message("There is no target named '${tgt}'")
      return()
    endif()

    get_target_property(target_type ${tgt} TYPE)
    if(target_type STREQUAL "INTERFACE_LIBRARY")
        set(PROP_LIST ${CMAKE_WHITELISTED_PROPERTY_LIST})
    else()
        set(PROP_LIST ${CMAKE_PROPERTY_LIST})
    endif()

    foreach (prop ${PROP_LIST})
        string(REPLACE "<CONFIG>" "${CMAKE_BUILD_TYPE}" prop ${prop})
        # message ("Checking ${prop}")
        get_property(propval TARGET ${tgt} PROPERTY ${prop} SET)
        if (propval)
            get_target_property(propval ${tgt} ${prop})
            message ("${tgt} ${prop} = ${propval}")
        endif()
    endforeach(prop)
endfunction(print_target_properties)

include(FetchContent)

FetchContent_Declare(
  Corrosion
  GIT_REPOSITORY https://github.com/AndrewGaspar/corrosion.git
  GIT_TAG origin/master
)

FetchContent_MakeAvailable(Corrosion)

find_package(ament_cmake REQUIRED)

get_directory_property(_vars_before VARIABLES)
find_package(rclrs_common REQUIRED)

get_directory_property(_vars VARIABLES)

list(REMOVE_ITEM _vars _vars_before ${_vars_before})
message("Added variables:")
foreach(_var IN LISTS _vars)
    message(STATUS "${_var} = ${${_var}}")
endforeach()
message("")

# This defines a crate with the same name as in the Cargo.toml
# I.e: This defines "rclrs"
corrosion_import_crate(MANIFEST_PATH "${CMAKE_SOURCE_DIR}/Cargo.toml")

target_link_libraries(rclrs INTERFACE rclrs_common)

print_target_properties(rclrs)
print_target_properties(rclrs_common)

# foreach(_crate_dependency ${rclrs_common_CRATES})
#     set(_crates_dependencies "${_crates_dependencies}\n[dependencies.rclrs_common]\npath = '${_crate_dependency}'\n")
# endforeach()

# file(COPY "${CMAKE_SOURCE_DIR}/Cargo.toml" DESTINATION "${CMAKE_BINARY_DIR}/")
# file(APPEND "${CMAKE_BINARY_DIR}/Cargo.toml" "${_crates_dependencies}")

# install(
#   FILES ${CMAKE_BINARY_DIR}/Cargo.toml build.rs src/rcl_wrapper.h
#   DESTINATION share/${PROJECT_NAME}/rust/
# )

# install(
#   DIRECTORY src/
#   DESTINATION  share/${PROJECT_NAME}/rust/src
# )

# ament_export_crates("share/${PROJECT_NAME}/rust")

ament_package()