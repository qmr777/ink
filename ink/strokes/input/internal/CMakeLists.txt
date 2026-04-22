# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/input/internal/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes_input_internal LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS ON)
endif()

# ------------------------------------------------------------------ 库
add_library(ink_strokes_input_internal_stroke_input_validation_helpers STATIC
  stroke_input_validation_helpers.cc
  stroke_input_validation_helpers.h
)
target_link_libraries(ink_strokes_input_internal_stroke_input_validation_helpers
  PUBLIC
    ink::strokes::input::stroke_input   # 来自上层 input/
    ink::types_duration
    absl::status
    absl::strings
)
add_library(ink::strokes::input::internal::stroke_input_validation_helpers
  ALIAS ink_strokes_input_internal_stroke_input_validation_helpers)

# ------------------------------------------------------------------ 测试
#if(INK_BUILD_TESTS)
#  enable_testing()
#  include(GoogleTest)
#
#  add_executable(ink_strokes_input_internal_stroke_input_validation_helpers_test
#    stroke_input_validation_helpers_test.cc
#  )
#  target_link_libraries(ink_strokes_input_internal_stroke_input_validation_helpers_test
#    PRIVATE
#      ink::strokes::input::internal::stroke_input_validation_helpers
#      ink::geometry_angle
#      ink::strokes::input::stroke_input
#      ink::types_duration
#      ink::types_physical_distance
#      absl::status
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_input_internal_stroke_input_validation_helpers_test)
#endif()