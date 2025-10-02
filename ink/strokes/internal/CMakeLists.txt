# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/internal/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes_internal LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS ON)
endif()
if(NOT DEFINED INK_BUILD_BENCHMARKS)
  set(INK_BUILD_BENCHMARKS ON)
endif()

#------------------------------------------------------------------- 子目录
add_subdirectory(brush_tip_extruder)

# ------------------------------------------------------------------ 辅助宏
function(ink_testonly_lib name)
  add_library(${name} STATIC ${ARGN})
  set_target_properties(${name} PROPERTIES EXCLUDE_FROM_ALL ON)
endfunction()

# ------------------------------------------------------------------ 基础工具库
## mutable_multi_mesh
add_library(ink_strokes_internal_mutable_multi_mesh STATIC
  mutable_multi_mesh.cc mutable_multi_mesh.h
)
target_link_libraries(ink_strokes_internal_mutable_multi_mesh
  PUBLIC
    ink::geometry_mesh_format
    ink::geometry_mesh_index_types
    ink::geometry_mutable_mesh
    ink::geometry_point
    ink::geometry_triangle
    ink::types_small_array
    absl::inlined_vector
    absl::check
    absl::span
)
add_library(ink::strokes::internal::mutable_multi_mesh ALIAS ink_strokes_internal_mutable_multi_mesh)

## stroke_shape_update
add_library(ink_strokes_internal_stroke_shape_update STATIC
  stroke_shape_update.cc stroke_shape_update.h
)
target_link_libraries(ink_strokes_internal_stroke_shape_update
  PUBLIC
    ink::geometry_envelope
)
add_library(ink::strokes::internal::stroke_shape_update ALIAS ink_strokes_internal_stroke_shape_update)

## stroke_outline
add_library(ink_strokes_internal_stroke_outline STATIC
  stroke_outline.cc stroke_outline.h
)
target_link_libraries(ink_strokes_internal_stroke_outline
  PUBLIC
    absl::algorithm_container
    absl::check
    absl::numeric
    absl::span
)
add_library(ink::strokes::internal::stroke_outline ALIAS ink_strokes_internal_stroke_outline)
target_include_directories(ink_strokes_internal_stroke_outline PUBLIC "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>")

## extrusion_points
add_library(ink_strokes_internal_extrusion_points INTERFACE)
target_link_libraries(ink_strokes_internal_extrusion_points INTERFACE
  ink::geometry_point
)
add_library(ink::strokes::internal::extrusion_points ALIAS ink_strokes_internal_extrusion_points)

## legacy_vertex
add_library(ink_strokes_internal_legacy_vertex INTERFACE)
target_link_libraries(ink_strokes_internal_legacy_vertex INTERFACE
  ink::color
  ink::geometry_point
)
add_library(ink::strokes::internal::legacy_vertex ALIAS ink_strokes_internal_legacy_vertex)

## rounded_polygon
add_library(ink_strokes_internal_rounded_polygon STATIC
  rounded_polygon.cc rounded_polygon.h
)
target_link_libraries(ink_strokes_internal_rounded_polygon
  PUBLIC
    ink::geometry_angle
    ink::geometry_point
    ink::geometry_segment
    ink::geometry_vec
    ink::geometry_internal::circle
    absl::inlined_vector
    absl::check
    absl::span
)
add_library(ink::strokes::internal::rounded_polygon ALIAS ink_strokes_internal_rounded_polygon)

## constrain_brush_tip_extrusion
add_library(ink_strokes_internal_constrain_brush_tip_extrusion STATIC
  constrain_brush_tip_extrusion.cc constrain_brush_tip_extrusion.h
)
target_link_libraries(ink_strokes_internal_constrain_brush_tip_extrusion
  PUBLIC
    ink_strokes_internal_brush_tip_extrusion
    ink_strokes_internal_brush_tip_state
    ink::geometry_distance
    ink::geometry_internal::algorithms
    absl::check
)
add_library(ink::strokes::internal::constrain_brush_tip_extrusion ALIAS ink_strokes_internal_constrain_brush_tip_extrusion)

# ------------------------------------------------------------------ 模型与状态库
## stroke_input_modeler
add_library(ink_strokes_internal_stroke_input_modeler STATIC
  stroke_input_modeler.cc
  stroke_input_modeler/naive_input_modeler.cc
  stroke_input_modeler/sliding_window_input_modeler.cc
  stroke_input_modeler/spring_based_input_modeler.cc
  stroke_input_modeler.h
  stroke_input_modeler/naive_input_modeler.h
  stroke_input_modeler/sliding_window_input_modeler.h
  stroke_input_modeler/spring_based_input_modeler.h
)
target_link_libraries(ink_strokes_internal_stroke_input_modeler
  PUBLIC
    ink::brush_family
    ink::geometry_angle
    ink::geometry_distance
    ink::geometry_point
    ink::geometry_vec
    ink::geometry_internal::algorithms
    ink::strokes::input::stroke_input
    ink::strokes::input::stroke_input_batch
    ink::types_duration
    ink::types_numbers
    ink::types_physical_distance
    absl::cleanup
    absl::check
    absl::status
    absl::span
    InkStrokeModeler::params
    InkStrokeModeler::stroke_modeler
    InkStrokeModeler::types
)
add_library(ink::strokes::internal::stroke_input_modeler ALIAS ink_strokes_internal_stroke_input_modeler)

## brush_tip_state
add_library(ink_strokes_internal_brush_tip_state STATIC
  brush_tip_state.cc brush_tip_state.h
)
target_link_libraries(ink_strokes_internal_brush_tip_state
  PUBLIC
    ink::geometry_angle
    ink::geometry_point
    ink::geometry_internal::algorithms
)
add_library(ink::strokes::internal::brush_tip_state ALIAS ink_strokes_internal_brush_tip_state)

## easing_implementation
add_library(ink_strokes_internal_easing_implementation STATIC
  easing_implementation.cc easing_implementation.h
)
target_link_libraries(ink_strokes_internal_easing_implementation
  PUBLIC
    ink::easing_function
    ink::geometry_point
    ink::geometry_vec
    ink::geometry_internal::algorithms
    absl::inlined_vector
    absl::overload
    absl::check
    absl::log
    absl::strings
)
add_library(ink::strokes::internal::easing_implementation ALIAS ink_strokes_internal_easing_implementation)

## noise_generator
add_library(ink_strokes_internal_noise_generator STATIC
  noise_generator.cc noise_generator.h
)
target_link_libraries(ink_strokes_internal_noise_generator
  PUBLIC
    ink::geometry_internal::algorithms
    absl::check
)
add_library(ink::strokes::internal::noise_generator ALIAS ink_strokes_internal_noise_generator)

## brush_tip_modeler_helpers
add_library(ink_strokes_internal_brush_tip_modeler_helpers STATIC
  brush_tip_modeler_helpers.cc brush_tip_modeler_helpers.h
)
target_link_libraries(ink_strokes_internal_brush_tip_modeler_helpers
  PUBLIC
    ink_strokes_internal_brush_tip_state
    ink_strokes_internal_easing_implementation
    ink_strokes_internal_noise_generator
    ink_strokes_internal_stroke_input_modeler
    ink::brush_behavior
    ink::brush_tip
    ink::geometry_angle
    ink::geometry_point
    ink::geometry_vec
    ink::geometry_internal::algorithms
    ink::geometry_internal::modulo
    ink::strokes::input::stroke_input
    ink::types_duration
    ink::types_physical_distance
    absl::check
    absl::span
)
add_library(ink::strokes::internal::brush_tip_modeler_helpers ALIAS ink_strokes_internal_brush_tip_modeler_helpers)

## brush_tip_modeler
add_library(ink_strokes_internal_brush_tip_modeler STATIC
  brush_tip_modeler.cc brush_tip_modeler.h
)
target_link_libraries(ink_strokes_internal_brush_tip_modeler
  PUBLIC
    ink_strokes_internal_brush_tip_modeler_helpers
    ink_strokes_internal_brush_tip_state
    ink_strokes_internal_easing_implementation
    ink_strokes_internal_stroke_input_modeler
    ink::brush_behavior
    ink::brush_tip
    ink::geometry_angle
    ink::geometry_point
    ink::types_duration
    absl::algorithm_container
    absl::check
    absl::log
    absl::span
)
add_library(ink::strokes::internal::brush_tip_modeler ALIAS ink_strokes_internal_brush_tip_modeler)

## brush_tip_shape
add_library(ink_strokes_internal_brush_tip_shape STATIC
  brush_tip_shape.cc brush_tip_shape.h
)
target_link_libraries(ink_strokes_internal_brush_tip_shape
  PUBLIC
    ink_strokes_internal_brush_tip_state
    ink_strokes_internal_circular_extrusion_helpers
    ink_strokes_internal_extrusion_points
    ink::geometry_affine_transform
    ink::geometry_angle
    ink::geometry_point
    ink::geometry_rect
    ink::geometry_segment
    ink::geometry_vec
    ink::geometry_internal::algorithms
    ink::geometry_internal::circle
    ink::types_small_array
    absl::inlined_vector
    absl::check
    absl::span
)
add_library(ink::strokes::internal::brush_tip_shape ALIAS ink_strokes_internal_brush_tip_shape)

## brush_tip_extrusion
add_library(ink_strokes_internal_brush_tip_extrusion STATIC
  brush_tip_extrusion.cc brush_tip_extrusion.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extrusion
  PUBLIC
    ink_strokes_internal_brush_tip_shape
    ink_strokes_internal_brush_tip_state
    ink::geometry_distance
    ink::geometry_rect
    ink::geometry_internal::circle
    absl::inlined_vector
    absl::check
    absl::span
)
add_library(ink::strokes::internal::brush_tip_extrusion ALIAS ink_strokes_internal_brush_tip_extrusion)

## brush_tip_extruder
add_library(ink_strokes_internal_brush_tip_extruder STATIC
  brush_tip_extruder.cc brush_tip_extruder.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder
  PUBLIC
    ink_strokes_internal_brush_tip_extrusion
    ink_strokes_internal_brush_tip_shape
    ink_strokes_internal_brush_tip_state
    ink_strokes_internal_constrain_brush_tip_extrusion
    ink_strokes_internal_stroke_outline
    ink_strokes_internal_stroke_shape_update
    ink_strokes_internal_stroke_vertex
    ink::geometry_affine_transform
    ink::geometry_envelope
    ink::geometry_mutable_mesh
    ink::geometry_point
    ink_strokes_internal_brush_tip_extruder_geometry
    ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
    ink_strokes_internal_brush_tip_extruder_side
    absl::algorithm_container
    absl::inlined_vector
    absl::check
    absl::span
)
add_library(ink::strokes::internal::brush_tip_extruder ALIAS ink_strokes_internal_brush_tip_extruder)

## stroke_vertex
add_library(ink_strokes_internal_stroke_vertex STATIC
  stroke_vertex.cc stroke_vertex.h
)
target_link_libraries(ink_strokes_internal_stroke_vertex
  PUBLIC
    ink::geometry_mesh_format
    ink::geometry_mesh_packing_types
    ink::geometry_mutable_mesh
    ink::geometry_point
    ink::geometry_vec
    ink::geometry_internal::algorithms
    ink::types_small_array
    absl::algorithm_container
    absl::check
    absl::span
)
add_library(ink::strokes::internal::stroke_vertex ALIAS ink_strokes_internal_stroke_vertex)

## stroke_shape_builder
add_library(ink_strokes_internal_stroke_shape_builder STATIC
  stroke_shape_builder.cc stroke_shape_builder.h
)
target_link_libraries(ink_strokes_internal_stroke_shape_builder
  PUBLIC
    ink_strokes_internal_brush_tip_extruder
    ink_strokes_internal_brush_tip_modeler
    ink_strokes_internal_stroke_input_modeler
    ink_strokes_internal_stroke_outline
    ink_strokes_internal_stroke_shape_update
    ink_strokes_internal_stroke_vertex
    ink::brush_coat
    ink::brush_family
    ink::geometry_envelope
    ink::geometry_mutable_mesh
    ink::types_duration
    absl::inlined_vector
    absl::check
    absl::span
)
add_library(ink::strokes::internal::stroke_shape_builder ALIAS ink_strokes_internal_stroke_shape_builder)

## circular_extrusion_helpers
add_library(ink_strokes_internal_circular_extrusion_helpers STATIC
  circular_extrusion_helpers.cc circular_extrusion_helpers.h
)
target_link_libraries(ink_strokes_internal_circular_extrusion_helpers
  PUBLIC
    ink::geometry_angle
    ink::geometry_point
    ink::geometry_segment
    ink::geometry_vec
    ink::geometry_internal::algorithms
    ink::geometry_internal::circle
    absl::check
)
add_library(ink::strokes::internal::circular_extrusion_helpers ALIAS ink_strokes_internal_circular_extrusion_helpers)

# ------------------------------------------------------------------ 测试专用库
#if(INK_BUILD_TESTS)
#  ink_testonly_lib(type_matchers
#    type_matchers.cc type_matchers.h
#  )
#  target_link_libraries(type_matchers
#    PUBLIC
#      ink_strokes_internal_brush_tip_shape
#      ink_strokes_internal_brush_tip_state
#      ink_strokes_internal_stroke_input_modeler
#      GTest::gtest
#      ink::geometry_type_matchers
#      ink::geometry_internal::circle
#      ink::geometry_internal::test_matchers
#      ink::types_type_matchers
#      absl::inlined_vector
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
#  add_executable(ink_strokes_internal_mutable_multi_mesh_test mutable_multi_mesh_test.cc)
#  target_link_libraries(ink_strokes_internal_mutable_multi_mesh_test
#    PRIVATE
#      ink::strokes::internal::mutable_multi_mesh
#      ink::geometry_mesh_format
#      ink::geometry_triangle
#      ink::geometry_type_matchers
#      ink::types_small_array
#      absl::status_matchers
#      absl::statusor
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_mutable_multi_mesh_test)
#
#  add_executable(ink_strokes_internal_stroke_shape_update_test stroke_shape_update_test.cc)
#  target_link_libraries(ink_strokes_internal_stroke_shape_update_test
#    PRIVATE
#      ink::strokes::internal::stroke_shape_update
#      ink::geometry_envelope
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_stroke_shape_update_test)
#
#  add_executable(ink_strokes_internal_stroke_input_modeler_test stroke_input_modeler_test.cc)
#  target_link_libraries(ink_strokes_internal_stroke_input_modeler_test
#    PRIVATE
#      ink::strokes::internal::stroke_input_modeler
#      ink::brush_brush_family
#      ink::geometry_angle
#      ink::geometry_type_matchers
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::types_duration
#      ink::types_physical_distance
#      ink::types_type_matchers
#      absl::check
#      absl::status
#      absl::status_matchers
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_stroke_input_modeler_test)
#
#  add_executable(ink_strokes_internal_naive_input_modeler_test stroke_input_modeler/naive_input_modeler_test.cc)
#  target_link_libraries(ink_strokes_internal_naive_input_modeler_test
#    PRIVATE
#      ink::strokes::internal::stroke_input_modeler
#      ink::brush_brush_family
#      ink::geometry_angle
#      ink::geometry_type_matchers
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::types_duration
#      ink::types_physical_distance
#      ink::types_type_matchers
#      absl::check
#      absl::status
#      absl::status_matchers
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_naive_input_modeler_test)
#
#  add_executable(ink_strokes_internal_sliding_window_input_modeler_test stroke_input_modeler/sliding_window_input_modeler_test.cc)
#  target_link_libraries(ink_strokes_internal_sliding_window_input_modeler_test
#    PRIVATE
#      ink::strokes::internal::stroke_input_modeler
#      ink::brush_brush_family
#      ink::geometry_angle
#      ink::geometry_type_matchers
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::types_duration
#      ink::types_physical_distance
#      ink::types_type_matchers
#      absl::check
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_sliding_window_input_modeler_test)
#
#  add_executable(ink_strokes_internal_spring_based_input_modeler_test stroke_input_modeler/spring_based_input_modeler_test.cc)
#  target_link_libraries(ink_strokes_internal_spring_based_input_modeler_test
#    PRIVATE
#      ink::strokes::internal::stroke_input_modeler
#      ink_strokes_internal_type_matchers
#      ink::brush_brush_family
#      ink::geometry_angle
#      ink::geometry_type_matchers
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::types_duration
#      ink::types_physical_distance
#      ink::types_type_matchers
#      absl::check
#      absl::status
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_spring_based_input_modeler_test)
#
#  add_executable(ink_strokes_internal_brush_tip_state_test brush_tip_state_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_state_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_state
#      ink_strokes_internal_type_matchers
#      ink::geometry_angle
#      ink::geometry_type_matchers
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_state_test)
#
#  add_executable(ink_strokes_internal_easing_implementation_test easing_implementation_test.cc)
#  target_link_libraries(ink_strokes_internal_easing_implementation_test
#    PRIVATE
#      ink::strokes::internal::easing_implementation
#      ink::brush_easing_function
#      ink::brush_fuzz_domains
#      absl::inlined_vector
#      fuzztest::fuzztest
#  )
#  gtest_discover_tests(ink_strokes_internal_easing_implementation_test)
#
#  add_executable(ink_strokes_internal_noise_generator_test noise_generator_test.cc)
#  target_link_libraries(ink_strokes_internal_noise_generator_test
#    PRIVATE
#      ink::strokes::internal::noise_generator
#      fuzztest::fuzztest
#  )
#  gtest_discover_tests(ink_strokes_internal_noise_generator_test)
#
#  add_executable(ink_strokes_internal_brush_tip_modeler_helpers_test brush_tip_modeler_helpers_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_modeler_helpers_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_modeler_helpers
#      ink::strokes::internal::brush_tip_state
#      ink::strokes::internal::easing_implementation
#      ink::strokes::internal::noise_generator
#      ink::strokes::internal::stroke_input_modeler
#      ink_strokes_internal_type_matchers
#      ink::brush_brush_behavior
#      ink::brush_brush_tip
#      ink::brush_easing_function
#      ink::geometry_angle
#      ink::geometry_type_matchers
#      ink::geometry_vec
#      ink::strokes::input::stroke_input
#      ink::types_duration
#      ink::types_physical_distance
#      ink::types_type_matchers
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_modeler_helpers_test)
#
#  add_executable(ink_strokes_internal_brush_tip_modeler_test brush_tip_modeler_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_modeler_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_modeler
#      ink::strokes::internal::brush_tip_state
#      ink::strokes::internal::stroke_input_modeler
#      ink::brush_brush_behavior
#      ink::brush_brush_tip
#      ink::brush_easing_function
#      ink::geometry_angle
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::strokes::input::stroke_input
#      ink::types_duration
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_modeler_test)
#
#  add_executable(ink_strokes_internal_circular_extrusion_helpers_test circular_extrusion_helpers_test.cc)
#  target_link_libraries(ink_strokes_internal_circular_extrusion_helpers_test
#    PRIVATE
#      ink::strokes::internal::circular_extrusion_helpers
#      ink_strokes_internal_extrusion_points
#      ink::geometry_angle
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::geometry_vec
#      ink::geometry_internal::circle
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_circular_extrusion_helpers_test)
#
#  add_executable(ink_strokes_internal_constrain_brush_tip_extrusion_test constrain_brush_tip_extrusion_test.cc)
#  target_link_libraries(ink_strokes_internal_constrain_brush_tip_extrusion_test
#    PRIVATE
#      ink_strokes_internal_brush_tip_extrusion
#      ink_strokes_internal_brush_tip_state
#      ink_strokes_internal_constrain_brush_tip_extrusion
#      ink_strokes_internal_type_matchers
#      ink::geometry_angle
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_constrain_brush_tip_extrusion_test)
#
#  add_executable(ink_strokes_internal_brush_tip_shape_test brush_tip_shape_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_shape_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_shape
#      ink_strokes_internal_brush_tip_state
#      ink_strokes_internal_extrusion_points
#      ink::geometry_angle
#      ink::geometry_type_matchers
#      ink::geometry_internal::circle
#      ink::geometry_internal::test_matchers
#      absl::strings
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_shape_test)
#
#  add_executable(ink_strokes_internal_brush_tip_extrusion_test brush_tip_extrusion_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extrusion_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extrusion
#      ink::geometry_angle
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extrusion_test)
#
#  add_executable(ink_strokes_internal_brush_tip_extruder_test brush_tip_extruder_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder
#      ink::strokes::internal::brush_tip_state
#      ink::strokes::internal::stroke_shape_update
#      ink::strokes::internal::stroke_vertex
#      ink::geometry_angle
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::geometry_internal::algorithms
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_test)
#
#  add_executable(ink_strokes_internal_stroke_vertex_test stroke_vertex_test.cc)
#  target_link_libraries(ink_strokes_internal_stroke_vertex_test
#    PRIVATE
#      ink::strokes::internal::stroke_vertex
#      ink::geometry_mesh_format
#      ink::geometry_mesh_packing_types
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::geometry_vec
#      ink::geometry_internal::mesh_packing
#      ink::types_small_array
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_stroke_vertex_test)
#
#  add_executable(ink_strokes_internal_stroke_outline_test stroke_outline_test.cc)
#  target_link_libraries(ink_strokes_internal_stroke_outline_test
#    PRIVATE
#      ink::strokes::internal::stroke_outline
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_stroke_outline_test)
#
#  add_executable(ink_strokes_internal_rounded_polygon_test rounded_polygon_test.cc)
#  target_link_libraries(ink_strokes_internal_rounded_polygon_test
#    PRIVATE
#      ink::strokes::internal::rounded_polygon
#      ink::geometry_type_matchers
#      ink::geometry_internal::circle
#      absl::strings
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_rounded_polygon_test)
#
#  add_executable(ink_strokes_internal_stroke_shape_builder_test stroke_shape_builder_test.cc)
#  target_link_libraries(ink_strokes_internal_stroke_shape_builder_test
#    PRIVATE
#      ink::strokes::internal::stroke_shape_builder
#      ink::strokes::internal::stroke_input_modeler
#      ink::strokes::internal::stroke_shape_update
#      ink::strokes::internal::stroke_vertex
#      ink::brush_brush_coat
#      ink::brush_brush_family
#      ink::brush_brush_paint
#      ink::brush_brush_tip
#      ink::geometry_envelope
#      ink::geometry_mutable_mesh
#      ink::geometry_type_matchers
#      ink::geometry_internal::algorithms
#      ink::strokes::input::stroke_input_batch
#      ink::types_duration
#      absl::status
#      absl::statusor
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_stroke_shape_builder_test)
#
#  # benchmark
#  if(INK_BUILD_BENCHMARKS)
#    add_executable(ink_strokes_internal_stroke_shape_builder_benchmark stroke_shape_builder_benchmark.cc)
#    target_link_libraries(ink_strokes_internal_stroke_shape_builder_benchmark
#      PRIVATE
#        ink::strokes::internal::stroke_shape_builder
#        ink::strokes::internal::stroke_input_modeler
#        ink::strokes::internal::stroke_shape_update
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
#        ink::strokes::input::stroke_input
#        ink::strokes::input::stroke_input_batch
#        ink::types_duration
#        absl::check
#        absl::statusor
#        absl::strings
#        benchmark::benchmark_main
#    )
#  endif()
#endif()