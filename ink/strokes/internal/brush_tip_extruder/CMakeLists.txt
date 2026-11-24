# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/internal/brush_tip_extruder/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes_internal_brush_tip_extruder LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------------------------------------------------------------------ 基础工具库
## extruded_vertex
add_library(ink_strokes_internal_brush_tip_extruder_extruded_vertex STATIC
  extruded_vertex.cc extruded_vertex.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_extruded_vertex
  PUBLIC
    ink::color::color
    ink::geometry::point
    ink::geometry::triangle
    ink::geometry::internal::algorithms
    ink::geometry::internal::lerp
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
    ink::geometry::mesh_format
    ink::geometry::mutable_mesh
    ink::geometry::point
    ink::geometry::triangle
    ink::geometry::vec
    ink::strokes::internal::legacy_vertex
    ink::strokes::internal::stroke_vertex
    absl::check
)
add_library(ink::strokes::internal::brush_tip_extruder::mutable_mesh_view
  ALIAS ink_strokes_internal_brush_tip_extruder_mutable_mesh_view)

## side
add_library(ink_strokes_internal_brush_tip_extruder_side INTERFACE side.h)
target_link_libraries(ink_strokes_internal_brush_tip_extruder_side INTERFACE
  ink_strokes_internal_brush_tip_extruder_extruded_vertex
  ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
  ink::geometry::point
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
    ink::geometry::point
    ink::geometry::segment
    ink::geometry::triangle
    ink::geometry::vec
    ink::geometry::internal::algorithms
    ink::strokes::internal::stroke_vertex
    absl::algorithm_container
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
    ink::geometry::point
    ink::geometry::segment
    ink::geometry::triangle
    ink::geometry::vec
    ink::geometry::internal::algorithms
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
    ink::geometry::point
    ink::geometry::segment
    ink::geometry::triangle
    ink::geometry::vec
    ink::geometry::internal::legacy_segment_intersection
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
    ink::geometry::point
    ink::geometry::segment
    ink::geometry::triangle
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
    ink::geometry::distance
    ink::geometry::segment
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
    ink::geometry::distance
    ink::geometry::envelope
    ink::geometry::point
    ink::geometry::segment
    ink::geometry::triangle
    ink::geometry::vec
    ink::geometry::internal::algorithms
    ink::geometry::internal::legacy_segment_intersection
    ink::geometry::internal::legacy_triangle_contains
    ink::geometry::internal::lerp
    ink::strokes::internal::brush_tip_state
    ink::strokes::internal::legacy_vertex
    ink::strokes::internal::stroke_vertex
    absl::algorithm_container
    absl::core_headers
    absl::nullability
    absl::cleanup
    absl::btree
    absl::flat_hash_map
    absl::check
    absl::span
)
add_library(ink::strokes::internal::brush_tip_extruder::geometry
  ALIAS ink_strokes_internal_brush_tip_extruder_geometry)
