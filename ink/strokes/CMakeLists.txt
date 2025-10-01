# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS ON)
endif()
if(NOT DEFINED INK_BUILD_BENCHMARKS)
  set(INK_BUILD_BENCHMARKS ON)
endif()

# ------------------------------------------------------------------ 子目录（顺序敏感）
add_subdirectory(input)   # 提供 stroke_input / stroke_input_batch / ...
add_subdirectory(internal) # 提供 stroke_input_modeler / stroke_shape_builder / ...

# ------------------------------------------------------------------ 主库
## stroke
add_library(ink_strokes_stroke STATIC
  stroke.cc stroke.h
)
target_link_libraries(ink_strokes_stroke
  PUBLIC
    ink::brush
    ink::brush_coat
    ink::brush_family
    ink::color
    ink::geometry_mutable_mesh
    ink::geometry_partitioned_mesh
    ink::strokes::input::stroke_input_batch
    ink::strokes::internal::stroke_input_modeler
    ink::strokes::internal::stroke_shape_builder
    ink::strokes::internal::stroke_vertex
    ink::types_duration
    absl::check
    absl::log
    absl::status
    absl::statusor
    absl::span
)
add_library(ink::strokes::stroke ALIAS ink_strokes_stroke)

## in_progress_stroke
add_library(ink_strokes_in_progress_stroke STATIC
  in_progress_stroke.cc in_progress_stroke.h
)
target_link_libraries(ink_strokes_in_progress_stroke
  PUBLIC
    ink::strokes::stroke
    ink::brush
    ink::brush_coat
    ink::geometry_envelope
    ink::geometry_mesh_format
    ink::geometry_mutable_mesh
    ink::geometry_partitioned_mesh
    ink::strokes::input::stroke_input
    ink::strokes::input::stroke_input_batch
    ink::strokes::input::internal::stroke_input_validation_helpers
    ink::strokes::internal::stroke_input_modeler
    ink::strokes::internal::stroke_shape_builder
    ink::strokes::internal::stroke_shape_update
    ink::strokes::internal::stroke_vertex
    ink::types_duration

    absl::flat_hash_set
    absl::inlined_vector
    absl::check
    absl::log
    absl::status
    absl::statusor
    absl::str_format
    absl::span
)
add_library(ink::strokes::in_progress_stroke ALIAS ink_strokes_in_progress_stroke)

# ------------------------------------------------------------------ 测试
#if(INK_BUILD_TESTS)
#  enable_testing()
#  include(GoogleTest)
#
#  # 统一用 fuzztest 提供的 gtest_main
#  set(_TEST_MAIN fuzztest::gtest_main)
#
#  # stroke_test
#  add_executable(ink_strokes_stroke_test stroke_test.cc)
#  target_link_libraries(ink_strokes_stroke_test
#    PRIVATE
#      ink::strokes::stroke
#      ink::brush
#      ink::brush_brush_coat
#      ink::brush_brush_family
#      ink::brush_brush_paint
#      ink::brush_brush_tip
#      ink::brush_fuzz_domains
#      ink::brush_type_matchers
#      ink::color
#      ink::geometry_angle
#      ink::geometry_envelope
#      ink::geometry_mesh_test_helpers
#      ink::geometry_partitioned_mesh
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::strokes::input::fuzz_domains
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::strokes::input::type_matchers
#      ink::types_duration
#      absl::check
#      absl::status
#      absl::statusor
#      absl::string_view
#      absl::span
#      ${_TEST_MAIN}
#  )
#  gtest_discover_tests(ink_strokes_stroke_test)
#
#  # in_progress_stroke_test
#  add_executable(ink_strokes_in_progress_stroke_test in_progress_stroke_test.cc)
#  target_link_libraries(ink_strokes_in_progress_stroke_test
#    PRIVATE
#      ink::strokes::in_progress_stroke
#      ink::strokes::stroke
#      ink::brush
#      ink::brush_brush_family
#      ink::brush_brush_paint
#      ink::brush_type_matchers
#      ink::color
#      ink::geometry_angle
#      ink::geometry_envelope
#      ink::geometry_mesh_format
#      ink::geometry_mutable_mesh
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::geometry_internal::algorithms
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::strokes::input::type_matchers
#      ink::types_duration
#      absl::check
#      absl::status
#      absl::statusor
#      absl::string_view
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_in_progress_stroke_test)
#
#  # benchmark
#  if(INK_BUILD_BENCHMARKS)
#    add_executable(ink_strokes_stroke_benchmark stroke_benchmark.cc)
#    target_link_libraries(ink_strokes_stroke_benchmark
#      PRIVATE
#        ink::strokes::stroke
#        ink::brush
#        ink::brush_brush_behavior
#        ink::brush_brush_family
#        ink::brush_brush_paint
#        ink::brush_brush_tip
#        ink::brush_easing_function
#        ink::color
#        ink::geometry_angle
#        ink::geometry_rect
#        ink::strokes::input::recorded_test_inputs
#        ink::strokes::input::stroke_input_batch
#        absl::check
#        absl::statusor
#        benchmark::benchmark_main
#    )
#  endif()
#endif()