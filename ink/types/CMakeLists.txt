# ------------------------------------------------------------------------------
#  ink_native/ink/types/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_types LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS ON)
endif()

add_subdirectory(internal)

# ------------------------------------------------------------------ 辅助宏
function(ink_testonly_lib name)
  add_library(${name} STATIC ${ARGN})
  set_target_properties(${name} PROPERTIES EXCLUDE_FROM_ALL ON)
endfunction()

# ------------------------------------------------------------------ 基础库
## numbers（头文件 only）
add_library(ink_types_numbers INTERFACE)
target_include_directories(ink_types_numbers INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}
)
add_library(ink::types_numbers ALIAS ink_types_numbers)

## iterator_range（头文件 only）
add_library(ink_types_iterator_range INTERFACE)
target_include_directories(ink_types_iterator_range INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}
)
add_library(ink::types_iterator_range ALIAS ink_types_iterator_range)

## small_array（头文件 only）
add_library(ink_types_small_array INTERFACE)
target_link_libraries(ink_types_small_array INTERFACE
  absl::algorithm_container
  absl::check
  absl::strings
  absl::span
)
add_library(ink::types_small_array ALIAS ink_types_small_array)

## physical_distance
add_library(ink_types_physical_distance STATIC
  physical_distance.cc physical_distance.h
)
target_link_libraries(ink_types_physical_distance
  PUBLIC
    absl::strings
)
add_library(ink::types_physical_distance ALIAS ink_types_physical_distance)
target_include_directories(ink_types_physical_distance PUBLIC "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>")

## duration
add_library(ink_types_duration STATIC
  duration.cc duration.h
)
target_link_libraries(ink_types_duration
  PUBLIC
    absl::strings
    absl::time
)
add_library(ink::types_duration ALIAS ink_types_duration)
target_include_directories(ink_types_duration PUBLIC "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>")

# ------------------------------------------------------------------ 测试专用库
#if(INK_BUILD_TESTS)
#  ink_testonly_lib(fuzz_domains
#    fuzz_domains.cc fuzz_domains.h
#  )
#  target_link_libraries(fuzz_domains
#    PUBLIC
#      ink::types_duration
#      ink::types_physical_distance
#      fuzztest::fuzztest
#  )
#
#  ink_testonly_lib(type_matchers
#    type_matchers.cc type_matchers.h
#  )
#  target_link_libraries(type_matchers
#    PUBLIC
#      ink::types_duration
#      ink::types_physical_distance
#      GTest::gtest
#      absl::strings
#  )
#endif()

# ------------------------------------------------------------------ 单元测试
#if(INK_BUILD_TESTS)
#  enable_testing()
#  include(GoogleTest)
#
#  # 统一用 fuzztest 提供的 gtest_main
#  set(_TEST_MAIN fuzztest::gtest_main)
#
#  add_executable(ink_types_duration_test duration_test.cc)
#  target_link_libraries(ink_types_duration_test
#    PRIVATE
#      ink::types_duration
#      ink::types_fuzz_domains
#      ink::types_type_matchers
#      absl::hash_testing
#      absl::strings
#      absl::time
#      ${_TEST_MAIN}
#  )
#  gtest_discover_tests(ink_types_duration_test)
#
#  add_executable(ink_types_physical_distance_test physical_distance_test.cc)
#  target_link_libraries(ink_types_physical_distance_test
#    PRIVATE
#      ink::types_physical_distance
#      ink::types_fuzz_domains
#      ink::types_type_matchers
#      absl::hash_testing
#      absl::strings
#      ${_TEST_MAIN}
#  )
#  gtest_discover_tests(ink_types_physical_distance_test)
#
#  add_executable(ink_types_small_array_test small_array_test.cc)
#  target_link_libraries(ink_types_small_array_test
#    PRIVATE
#      ink::types_small_array
#      absl::strings
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_types_small_array_test)
#
#  add_executable(ink_types_iterator_range_test iterator_range_test.cc)
#  target_link_libraries(ink_types_iterator_range_test
#    PRIVATE
#      ink::types_iterator_range
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_types_iterator_range_test)
#endif()