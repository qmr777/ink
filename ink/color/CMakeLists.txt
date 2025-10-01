# ------------------------------------------------------------------------------
#  ink/color/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.16)
project(ink_color LANGUAGES CXX)

# ------------------------------------------------------------------ compiler
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ------------------------------------------------------------------ options
option(INK_COLOR_BUILD_TESTS "Build unit tests for ink/color" ON)

# ------------------------------------------------------------------ dependencies
# 下方假设已通过 FetchContent 或 find_package 提供下列目标：
#   absl::core_headers
#   absl::log
#   absl::strings
#   absl::str_format
#   absl::hash_testing
#   GTest::gtest
#   GTest::gtest_main
#   fuzztest::fuzztest
#   fuzztest::gtest_main
#
# 若尚未提供，请先在顶层 CMakeLists.txt 中通过 FetchContent 拉取：
#   https://github.com/abseil/abseil-cpp
#   https://github.com/google/fuzztest
#   https://github.com/google/googletest

# ------------------------------------------------------------------ helpers
# 统一开启测试专用库的属性
function(ink_cc_testonly_library name)
  add_library(${name} STATIC ${ARGN})
  target_compile_definitions(${name} PRIVATE COLOR_TESTONLY)
  set_target_properties(${name} PROPERTIES EXCLUDE_FROM_ALL ON)
endfunction()

# ------------------------------------------------------------------ color_space
add_library(ink_color_space STATIC
  color_space.cc
  color_space.h
)
target_link_libraries(ink_color_space
  PUBLIC
    absl::log
    absl::str_format
)
add_library(ink::color_space ALIAS ink_color_space)

# ------------------------------------------------------------------ color
add_library(ink_color STATIC
  color.cc
  color.h
  color_helpers.h
)
target_link_libraries(ink_color
  PUBLIC
    ink::color_space
    absl::log
    absl::str_format
)
add_library(ink::color ALIAS ink_color)

# ------------------------------------------------------------------ test-only libraries
#if(INK_COLOR_BUILD_TESTS)
#  # fuzz_domains
#  ink_cc_testonly_library(color_fuzz_domains
#    fuzz_domains.cc
#    fuzz_domains.h
#  )
#  target_link_libraries(color_fuzz_domains
#    PUBLIC
#      ink::color
#      ink::color_space
#      fuzztest::fuzztest
#  )
#
#  # type_matchers
#  ink_cc_testonly_library(color_type_matchers
#    type_matchers.cc
#    type_matchers.h
#  )
#  target_link_libraries(color_type_matchers
#    PUBLIC
#      ink::color
#      GTest::gtest
#      absl::strings
#      absl::str_format
#  )
#endif()

# ------------------------------------------------------------------ tests
#if(INK_COLOR_BUILD_TESTS)
#  # 统一使用 fuzztest 提供的 gtest_main
#  set(INK_TEST_MAIN fuzztest::gtest_main)
#
#  # color_test
#  add_executable(ink_color_test
#    color_test.cc
#  )
#  target_link_libraries(ink_color_test
#    PRIVATE
#      ink::color
#      ink::color_space
#      fuzz_domains
#      type_matchers
#      absl::hash_testing
#      absl::strings
#      absl::str_format
#      fuzztest::fuzztest
#      ${INK_TEST_MAIN}
#  )
#  gtest_discover_tests(ink_color_test
#    PROPERTIES
#      # 对应 bazel 的 shard_count=10
#      PROCESSORS 10
#  )
#
#  # color_helpers_test
#  add_executable(ink_color_helpers_test
#    color_helpers_test.cc
#  )
#  target_link_libraries(ink_color_helpers_test
#    PRIVATE
#      ink::color
#      type_matchers
#      fuzztest::fuzztest
#      ${INK_TEST_MAIN}
#  )
#  gtest_discover_tests(ink_color_helpers_test)
#
#  # color_space_test
#  add_executable(ink_color_space_test
#    color_space_test.cc
#  )
#  target_link_libraries(ink_color_space_test
#    PRIVATE
#      ink::color_space
#      fuzz_domains
#      type_matchers
#      absl::strings
#      fuzztest::fuzztest
#      ${INK_TEST_MAIN}
#  )
#  gtest_discover_tests(ink_color_space_test
#    PROPERTIES
#      PROCESSORS 10
#  )
#endif()