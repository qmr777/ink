# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/internal/brush_tip_extruder/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes_internal_brush_tip_extruder LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS ON)
endif()

# ------------------------------------------------------------------ 基础工具库
## extruded_vertex
add_library(ink_strokes_internal_brush_tip_extruder_extruded_vertex STATIC
  extruded_vertex.cc extruded_vertex.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_extruded_vertex
  PUBLIC
    ink::color
    ink::geometry_point
    ink::geometry_triangle
    ink::geometry_internal::algorithms
    ink::strokes::internal::legacy_vertex
    ink::strokes::internal::stroke_vertex
    absl::check
)
add_library(ink::strokes::internal::brush_tip_extruder::extruded_vertex
  ALIAS ink_strokes_internal_brush_tip_extruder_extruded_vertex)

## mutable_mesh_view
add_library(ink_strokes_internal_brush_tip_extruder_mutable_mesh_view STATIC
  mutable_mesh_view.cc mutable_mesh_view.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
  PUBLIC
    ink_strokes_internal_brush_tip_extruder_extruded_vertex
    ink::geometry_mesh_format
    ink::geometry_mutable_mesh
    ink::geometry_point
    ink::geometry_triangle
    ink::geometry_vec
    ink::strokes::internal::legacy_vertex
    ink::strokes::internal::stroke_vertex
    absl::check
)
add_library(ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
  ALIAS ink_strokes_internal_brush_tip_extruder_mutable_mesh_view)

## side
add_library(ink_strokes_internal_brush_tip_extruder_side INTERFACE)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_side INTERFACE
  ink_strokes_internal_brush_tip_extruder_extruded_vertex
  ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
  ink::geometry_point
)
add_library(ink::strokes::internal::brush_tip_extruder::side
  ALIAS ink_strokes_internal_brush_tip_extruder_side)

## derivative_calculator_helpers
add_library(ink_strokes_internal_brush_tip_extruder_derivative_calculator_helpers STATIC
  derivative_calculator_helpers.cc derivative_calculator_helpers.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_derivative_calculator_helpers
  PUBLIC
    ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
    ink_strokes_internal_brush_tip_extruder_side
    ink::geometry_point
    ink::strokes::internal::stroke_vertex
    absl::check
    absl::span
)
add_library(ink::strokes::internal::brush_tip_extruder::derivative_calculator_helpers
  ALIAS ink_strokes_internal_brush_tip_extruder_derivative_calculator_helpers)

## derivative_calculator
add_library(ink_strokes_internal_brush_tip_extruder_derivative_calculator STATIC
  derivative_calculator.cc derivative_calculator.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_derivative_calculator
  PUBLIC
    ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
    ink::geometry_point
    ink::geometry_segment
    ink::geometry_triangle
    ink::geometry_vec
    ink::geometry_internal::algorithms
    ink::strokes::internal::stroke_vertex
    absl::algorithm_container
    absl::check
    absl::span
)
add_library(ink::strokes::internal::brush_tip_extruder::derivative_calculator
  ALIAS ink_strokes_internal_brush_tip_extruder_derivative_calculator)

## directed_partial_outline
add_library(ink_strokes_internal_brush_tip_extruder_directed_partial_outline STATIC
  directed_partial_outline.cc directed_partial_outline.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_directed_partial_outline
  PUBLIC
    ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
    ink_strokes_internal_brush_tip_extruder_side
    ink::geometry_point
    ink::geometry_segment
    ink::geometry_triangle
    ink::geometry_vec
    ink::geometry_internal::legacy_segment_intersection
    absl::check
)
add_library(ink::strokes::internal::brush_tip_extruder::directed_partial_outline
  ALIAS ink_strokes_internal_brush_tip_extruder_directed_partial_outline)

## find_clockwise_winding_segment
add_library(ink_strokes_internal_brush_tip_extruder_find_clockwise_winding_segment STATIC
  find_clockwise_winding_segment.cc find_clockwise_winding_segment.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_find_clockwise_winding_segment
  PUBLIC
    ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
    ink_strokes_internal_brush_tip_extruder_side
    ink::geometry_point
    ink::geometry_segment
    ink::geometry_triangle
    absl::span
)
add_library(ink::strokes::internal::brush_tip_extruder::find_clockwise_winding_segment
  ALIAS ink_strokes_internal_brush_tip_extruder_find_clockwise_winding_segment)

## simplify
add_library(ink_strokes_internal_brush_tip_extruder_simplify STATIC
  simplify.cc simplify.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_simplify
  PUBLIC
    ink_strokes_internal_brush_tip_extruder_extruded_vertex
    ink::geometry_distance
    ink::geometry_segment
)
add_library(ink::strokes::internal::brush_tip_extruder::simplify
  ALIAS ink_strokes_internal_brush_tip_extruder_simplify)

## geometry
add_library(ink_strokes_internal_brush_tip_extruder_geometry STATIC
  geometry.cc geometry.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_geometry
  PUBLIC
    ink_strokes_internal_brush_tip_extruder_derivative_calculator
    ink_strokes_internal_brush_tip_extruder_derivative_calculator_helpers
    ink_strokes_internal_brush_tip_extruder_directed_partial_outline
    ink_strokes_internal_brush_tip_extruder_extruded_vertex
    ink_strokes_internal_brush_tip_extruder_find_clockwise_winding_segment
    ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
    ink_strokes_internal_brush_tip_extruder_side
    ink_strokes_internal_brush_tip_extruder_simplify
    ink::geometry_distance
    ink::geometry_envelope
    ink::geometry_point
    ink::geometry_segment
    ink::geometry_triangle
    ink::geometry_vec
    ink::geometry_internal::algorithms
    ink::geometry_internal::legacy_segment_intersection
    ink::geometry_internal::legacy_triangle_contains
    ink::strokes::internal::brush_tip_state
    ink::strokes::internal::legacy_vertex
    ink::strokes::internal::stroke_vertex
    absl::algorithm_container
    absl::core_headers
    absl::cleanup
    absl::btree
    absl::flat_hash_map
    absl::check
    absl::span
)
add_library(ink::strokes::internal::brush_tip_extruder::geometry
  ALIAS ink_strokes_internal_brush_tip_extruder_geometry)

# ------------------------------------------------------------------ 测试
#if(INK_BUILD_TESTS)
#  enable_testing()
#  include(GoogleTest)
#
#  # mutable_mesh_view_test
#  add_executable(ink_strokes_internal_brush_tip_extruder_mutable_mesh_view_test mutable_mesh_view_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_mutable_mesh_view_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::extruded_vertex
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::geometry_mutable_mesh
#      ink::geometry_type_matchers
#      ink::strokes::internal::legacy_vertex
#      ink::strokes::internal::stroke_vertex
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_mutable_mesh_view_test)
#
#  # extruded_vertex_test
#  add_executable(ink_strokes_internal_brush_tip_extruder_extruded_vertex_test extruded_vertex_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_extruded_vertex_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::extruded_vertex
#      ink::color_type_matchers
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::strokes::internal::legacy_vertex
#      ink::strokes::internal::stroke_vertex
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_extruded_vertex_test)
#
#  # derivative_calculator_test
#  add_executable(ink_strokes_internal_brush_tip_extruder_derivative_calculator_test derivative_calculator_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_derivative_calculator_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::derivative_calculator
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::geometry_vec
#      ink::strokes::internal::stroke_vertex
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_derivative_calculator_test)
#
#  # derivative_calculator_helpers_test
#  add_executable(ink_strokes_internal_brush_tip_extruder_derivative_calculator_helpers_test derivative_calculator_helpers_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_derivative_calculator_helpers_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::derivative_calculator_helpers
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::strokes::internal::brush_tip_extruder::side
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::strokes::internal::stroke_vertex
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_derivative_calculator_helpers_test)
#
#  # directed_partial_outline_test
#  add_executable(ink_strokes_internal_brush_tip_extruder_directed_partial_outline_test directed_partial_outline_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_directed_partial_outline_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::directed_partial_outline
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::strokes::internal::brush_tip_extruder::side
#      ink::geometry_segment
#      ink::geometry_triangle
#      ink::strokes::internal::legacy_vertex
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_directed_partial_outline_test)
#
#  # find_clockwise_winding_segment_test
#  add_executable(ink_strokes_internal_brush_tip_extruder_find_clockwise_winding_segment_test find_clockwise_winding_segment_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_find_clockwise_winding_segment_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::find_clockwise_winding_segment
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::strokes::internal::brush_tip_extruder::side
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::strokes::internal::legacy_vertex
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_find_clockwise_winding_segment_test)
#
#  # simplify_test
#  add_executable(ink_strokes_internal_brush_tip_extruder_simplify_test simplify_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_simplify_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::extruded_vertex
#      ink::strokes::internal::brush_tip_extruder::simplify
#      ink::geometry_point
#      ink::geometry_type_matchers
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_simplify_test)
#
#  # geometry_test 及其子测试
#  add_executable(ink_strokes_internal_brush_tip_extruder_geometry_test geometry_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_geometry_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::geometry
#      ink::strokes::internal::brush_tip_extruder::extruded_vertex
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::strokes::internal::brush_tip_extruder::side
#      ink::strokes::internal::brush_tip_extruder::simplify
#      ink::geometry_distance
#      ink::geometry_envelope
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_type_matchers
#      ink::strokes::internal::brush_tip_state
#      ink::strokes::internal::legacy_vertex
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_geometry_test)
#
#  add_executable(ink_strokes_internal_brush_tip_extruder_geometry_non_position_attributes_test geometry_non_position_attributes_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_geometry_non_position_attributes_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::geometry
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::strokes::internal::brush_tip_state
#      ink::strokes::internal::stroke_vertex
#      ink::types_small_array
#      absl::check
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_geometry_non_position_attributes_test)
#
#  add_executable(ink_strokes_internal_brush_tip_extruder_geometry_extrusion_break_test geometry_extrusion_break_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_geometry_extrusion_break_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::geometry
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::strokes::internal::brush_tip_extruder::side
#      ink::geometry_envelope
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::strokes::internal::stroke_vertex
#      absl::strings
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_geometry_extrusion_break_test)
#
#  add_executable(ink_strokes_internal_brush_tip_extruder_geometry_save_test geometry_save_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_geometry_save_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::extruded_vertex
#      ink::strokes::internal::brush_tip_extruder::geometry
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::strokes::internal::brush_tip_extruder::side
#      ink::geometry_envelope
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::strokes::internal::brush_tip_state
#      ink::strokes::internal::legacy_vertex
#      absl::base_nullability
#      absl::span
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_geometry_save_test)
#
#  add_executable(ink_strokes_internal_brush_tip_extruder_geometry_vertex_label_test geometry_vertex_label_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_geometry_vertex_label_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::geometry
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::strokes::internal::brush_tip_extruder::side
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::strokes::internal::brush_tip_state
#      ink::strokes::internal::stroke_vertex
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_geometry_vertex_label_test)
#
#  add_executable(ink_strokes_internal_brush_tip_extruder_geometry_mesh_derivatives_test geometry_mesh_derivatives_test.cc)
#  target_link_libraries(ink_strokes_internal_brush_tip_extruder_geometry_mesh_derivatives_test
#    PRIVATE
#      ink::strokes::internal::brush_tip_extruder::geometry
#      ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::geometry_vec
#      ink::strokes::internal::brush_tip_state
#      ink::types_small_array
#      absl::check
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_strokes_internal_brush_tip_extruder_geometry_mesh_derivatives_test)
#endif()