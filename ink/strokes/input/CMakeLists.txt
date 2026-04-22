# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/input/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes_input LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS ON)
endif()

# ------------------------------------------------------------------ 子目录（顺序敏感）
add_subdirectory(internal)   # 提供 stroke_input_validation_helpers

# ------------------------------------------------------------------ 辅助宏
function(ink_testonly_lib name)
  add_library(${name} STATIC ${ARGN})
  set_target_properties(${name} PROPERTIES EXCLUDE_FROM_ALL ON)
endfunction()

# ------------------------------------------------------------------ 基础库
## stroke_input
add_library(ink_strokes_input_stroke_input STATIC
  stroke_input.cc stroke_input.h
)
target_link_libraries(ink_strokes_input_stroke_input
  PUBLIC
    ink::geometry_angle
    ink::geometry_point
    ink::types_duration
    ink::types_physical_distance
    absl::strings
)
add_library(ink::strokes::input::stroke_input ALIAS ink_strokes_input_stroke_input)

## stroke_input_batch
add_library(ink_strokes_input_stroke_input_batch STATIC
  stroke_input_batch.cc stroke_input_batch.h
)
target_link_libraries(ink_strokes_input_stroke_input_batch
  PUBLIC
    ink::strokes::input::stroke_input
    ink::geometry_affine_transform
    ink::geometry_angle
    ink::geometry_point
    ink::strokes::input::internal::stroke_input_validation_helpers
    ink::types_duration
    ink::types_physical_distance
    ink::types_internal::copy_on_write
    absl::check
    absl::status
    absl::statusor
    absl::strings
    absl::span
)
add_library(ink::strokes::input::stroke_input_batch ALIAS ink_strokes_input_stroke_input_batch)

# ------------------------------------------------------------------ 测试专用库
#if(INK_BUILD_TESTS)
#  ink_testonly_lib(fuzz_domains
#    fuzz_domains.cc fuzz_domains.h
#  )
#  target_link_libraries(fuzz_domains
#    PUBLIC
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::geometry_angle
#      ink::geometry_fuzz_domains
#      ink::geometry_point
#      ink::geometry_rect
#      ink::types_duration
#      ink::types_fuzz_domains
#      ink::types_physical_distance
#      absl::statusor
#      fuzztest::fuzztest
#  )
#
#  ink_testonly_lib(type_matchers
#    type_matchers.cc type_matchers.h
#  )
#  target_link_libraries(type_matchers
#    PUBLIC
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      GTest::gtest
#      ink::geometry_type_matchers
#      ink::types_type_matchers
#      absl::span
#  )
#
#  ink_testonly_lib(synthetic_test_inputs
#    synthetic_test_inputs.cc synthetic_test_inputs.h
#  )
#  target_link_libraries(synthetic_test_inputs
#    PUBLIC
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::geometry_angle
#      ink::geometry_rect
#      ink::types_duration
#      ink::types_physical_distance
#      absl::check
#  )
#
#  ink_testonly_lib(recorded_test_inputs
#    recorded_test_inputs.cc
#    recorded_test_inputs_data.cc
#    recorded_test_inputs_data.h
#    recorded_test_inputs.h
#  )
#  target_link_libraries(recorded_test_inputs
#    PUBLIC
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::geometry_affine_transform
#      ink::geometry_angle
#      ink::geometry_envelope
#      ink::geometry_point
#      ink::geometry_rect
#      ink::types_duration
#      absl::check
#      absl::statusor
#      absl::span
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
#  add_executable(ink_strokes_input_stroke_input_test stroke_input_test.cc)
#  target_link_libraries(ink_strokes_input_stroke_input_test
#    PRIVATE
#      ink::strokes::input::stroke_input
#      ink::geometry_angle
#      ink::types_duration
#      ink::types_physical_distance
#      absl::strings
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_input_stroke_input_test)
#
#  add_executable(ink_strokes_input_stroke_input_batch_test stroke_input_batch_test.cc)
#  target_link_libraries(ink_strokes_input_stroke_input_batch_test
#    PRIVATE
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::strokes::input::fuzz_domains
#      ink::strokes::input::type_matchers
#      ink::geometry_angle
#      ink::types_duration
#      ink::types_physical_distance
#      absl::status
#      absl::statusor
#      absl::strings
#      absl::span
#      ${_TEST_MAIN}
#  )
#  gtest_discover_tests(ink_strokes_input_stroke_input_batch_test)
#
#  add_executable(ink_strokes_input_stroke_input_batch_iterator_test stroke_input_batch_iterator_test.cc)
#  target_link_libraries(ink_strokes_input_stroke_input_batch_iterator_test
#    PRIVATE
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::strokes::input::fuzz_domains
#      ink::strokes::input::type_matchers
#      ink::geometry_angle
#      ink::geometry_type_matchers
#      ink::types_duration
#      absl::status
#      ${_TEST_MAIN}
#  )
#  gtest_discover_tests(ink_strokes_input_stroke_input_batch_iterator_test)
#
#  add_executable(ink_strokes_input_stroke_input_batch_transform_test stroke_input_batch_transform_test.cc)
#  target_link_libraries(ink_strokes_input_stroke_input_batch_transform_test
#    PRIVATE
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::strokes::input::type_matchers
#      ink::geometry_affine_transform
#      ink::geometry_angle
#      ink::types_duration
#      absl::status
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_input_stroke_input_batch_transform_test)
#
#  add_executable(ink_strokes_input_synthetic_test_inputs_test synthetic_test_inputs_test.cc)
#  target_link_libraries(ink_strokes_input_synthetic_test_inputs_test
#    PRIVATE
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::strokes::input::synthetic_test_inputs
#      ink::geometry_envelope
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::types_duration
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_input_synthetic_test_inputs_test)
#
#  add_executable(ink_strokes_input_recorded_test_inputs_test recorded_test_inputs_test.cc)
#  target_link_libraries(ink_strokes_input_recorded_test_inputs_test
#    PRIVATE
#      ink::strokes::input::recorded_test_inputs
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::geometry_envelope
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_input_recorded_test_inputs_test)
#endif()