# ------------------------------------------------------------------------------
#  ink_native/ink/types/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_types LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(internal)

# ------------------------------------------------------------------ 辅助宏

# ------------------------------------------------------------------ 基础库
## numbers（头文件 only）
add_library(ink_types_numbers INTERFACE numbers.h)
target_include_directories(ink_types_numbers INTERFACE
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>"
)
add_library(ink::types::numbers ALIAS ink_types_numbers)

## iterator_range（头文件 only）
add_library(ink_types_iterator_range INTERFACE iterator_range.h)
target_include_directories(ink_types_iterator_range INTERFACE
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>"
)
add_library(ink::types::iterator_range ALIAS ink_types_iterator_range)

## small_array（头文件 only）
add_library(ink_types_small_array INTERFACE small_array.h)
target_link_libraries(ink_types_small_array INTERFACE
  absl::algorithm_container
  absl::check
  absl::strings
  absl::span
)
add_library(ink::types::small_array ALIAS ink_types_small_array)
target_include_directories(ink_types_small_array INTERFACE "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>")

## physical_distance
add_library(ink_types_physical_distance STATIC
  physical_distance.cc physical_distance.h
)
target_link_libraries(ink_types_physical_distance
  PUBLIC
    absl::strings
)
add_library(ink::types::physical_distance ALIAS ink_types_physical_distance)
target_include_directories(ink_types_physical_distance PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>")

## duration
add_library(ink_types_duration STATIC
  duration.cc duration.h
)
target_link_libraries(ink_types_duration
  PUBLIC
    absl::strings
    absl::time
)
add_library(ink::types::duration ALIAS ink_types_duration)
target_include_directories(ink_types_duration PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>")
