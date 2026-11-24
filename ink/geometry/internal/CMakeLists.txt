# ------------------------------------------------------------------------------
#  ink_native/ink/geometry/internal/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_geometry_internal LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------------------------------------------------------------------ 辅助宏

# ------------------------------------------------------------------ 基础工具库
## modulo
add_library(ink_geometry_internal_modulo STATIC
  modulo.cc modulo.h
)
target_link_libraries(ink_geometry_internal_modulo
  PUBLIC
    absl::check
)
add_library(ink::geometry::internal::modulo ALIAS ink_geometry_internal_modulo)
target_include_directories(ink_geometry_internal_modulo
        PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../../..>"
)

## mesh_constants
add_library(ink_geometry_internal_mesh_constants INTERFACE)
target_include_directories(ink_geometry_internal_mesh_constants INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}
)
add_library(ink::geometry::internal::mesh_constants ALIAS ink_geometry_internal_mesh_constants)
target_include_directories(ink_geometry_internal_mesh_constants INTERFACE "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../../..>")

## legacy_vector_utils
add_library(ink_geometry_internal_legacy_vector_utils STATIC
  legacy_vector_utils.cc legacy_vector_utils.h
)
target_link_libraries(ink_geometry_internal_legacy_vector_utils
  PUBLIC
    ink::geometry::point
    ink::geometry::vec
)
add_library(ink::geometry::internal::legacy_vector_utils ALIAS ink_geometry_internal_legacy_vector_utils)

## legacy_segment_intersection
add_library(ink_geometry_internal_legacy_segment_intersection STATIC
  legacy_segment_intersection.cc legacy_segment_intersection.h
)
target_link_libraries(ink_geometry_internal_legacy_segment_intersection
  PUBLIC
    ink::geometry::internal::legacy_vector_utils
    ink::geometry::point
    ink::geometry::segment
    absl::strings
)
add_library(ink::geometry::internal::legacy_segment_intersection ALIAS ink_geometry_internal_legacy_segment_intersection)

## legacy_triangle_contains
add_library(ink_geometry_internal_legacy_triangle_contains STATIC
  legacy_triangle_contains.cc legacy_triangle_contains.h
)
target_link_libraries(ink_geometry_internal_legacy_triangle_contains
  PUBLIC
    ink::geometry::internal::legacy_vector_utils
    ink::geometry::point
    ink::geometry::triangle
)
add_library(ink::geometry::internal::legacy_triangle_contains ALIAS ink_geometry_internal_legacy_triangle_contains)

## intersects_internal
add_library(ink_geometry_internal_intersects_internal STATIC
  intersects_internal.cc intersects_internal.h
)
target_link_libraries(ink_geometry_internal_intersects_internal
  PUBLIC
    ink::geometry::point
    ink::geometry::quad
    ink::geometry::rect
    ink::geometry::segment
    ink::geometry::triangle
    ink::geometry::vec
)
add_library(ink::geometry::internal::intersects_internal ALIAS ink_geometry_internal_intersects_internal)

## point_tessellation_helper
add_library(ink_geometry_internal_point_tessellation_helper INTERFACE)
target_link_libraries(ink_geometry_internal_point_tessellation_helper INTERFACE
  ink::geometry::point
  absl::span
)
add_library(ink::geometry::internal::point_tessellation_helper ALIAS ink_geometry_internal_point_tessellation_helper)

## convex_hull_helper
add_library(ink_geometry_internal_convex_hull_helper INTERFACE)
target_link_libraries(ink_geometry_internal_convex_hull_helper INTERFACE
  ink::geometry::point
  ink::geometry::triangle
  ink::geometry::vec
  absl::span
)
add_library(ink::geometry::internal::convex_hull_helper ALIAS ink_geometry_internal_convex_hull_helper)

## static_rtree
add_library(ink_geometry_internal_static_rtree STATIC
  static_rtree.cc static_rtree.h
)
target_link_libraries(ink_geometry_internal_static_rtree
  PUBLIC
    ink::geometry::internal::intersects_internal
    ink::geometry::envelope
    ink::geometry::rect
    ink::types::small_array
    absl::algorithm_container
    absl::inlined_vector
    absl::function_ref
    absl::check
    absl::strings
    absl::span
)
add_library(ink::geometry::internal::static_rtree ALIAS ink_geometry_internal_static_rtree)

## algorithms
add_library(ink_geometry_internal_algorithms STATIC
  algorithms.cc algorithms.h
)
target_link_libraries(ink_geometry_internal_algorithms
  PUBLIC
    ink::geometry::internal::intersects_internal
    ink::geometry::affine_transform
    ink::geometry::envelope
    ink::geometry::mesh
    ink::geometry::mutable_mesh
    ink::geometry::point
    ink::geometry::rect
    ink::geometry::segment
    ink::geometry::triangle
    ink::geometry::vec
    absl::check
    absl::span
)
add_library(ink::geometry::internal::algorithms ALIAS ink_geometry_internal_algorithms)

## polyline_processing
add_library(ink_geometry_internal_polyline_processing STATIC
  polyline_processing.cc polyline_processing.h
)
target_link_libraries(ink_geometry_internal_polyline_processing
  PUBLIC
    ink::geometry::internal::algorithms
    ink::geometry::internal::static_rtree
    ink::geometry::distance
    ink::geometry::envelope
    ink::geometry::point
    ink::geometry::rect
    ink::geometry::segment
    absl::check
    absl::span
)
add_library(ink::geometry::internal::polyline_processing ALIAS ink_geometry_internal_polyline_processing)

## circle
add_library(ink_geometry_internal_circle STATIC
  circle.cc circle.h
)
target_link_libraries(ink_geometry_internal_circle
  PUBLIC
    ink::geometry::angle
    ink::geometry::point
    ink::geometry::vec
    absl::check
    absl::strings
)
add_library(ink::geometry::internal::circle ALIAS ink_geometry_internal_circle)

## mesh_packing
add_library(ink_geometry_internal_mesh_packing STATIC
  mesh_packing.cc mesh_packing.h
)
target_link_libraries(ink_geometry_internal_mesh_packing
  PUBLIC
    ink::geometry::internal::mesh_constants
    ink::geometry::mesh_format
    ink::geometry::mesh_packing_types
    ink::geometry::point
    ink::types::small_array
    ink::types::internal::float
    absl::algorithm_container

    absl::flat_hash_map
    absl::flat_hash_set
    absl::inlined_vector
    absl::check
    absl::log
    absl::status
    absl::statusor
    absl::strings
    absl::span
)
add_library(ink::geometry::internal::mesh_packing ALIAS ink_geometry_internal_mesh_packing)

## generic_tessellator
add_library(ink_geometry_internal_generic_tessellator INTERFACE)
target_link_libraries(ink_geometry_internal_generic_tessellator INTERFACE
  absl::core_headers
  absl::check
  absl::span
  tess2
)
add_library(ink::geometry::internal::generic_tessellator ALIAS ink_geometry_internal_generic_tessellator)

add_library(ink_geometry_internal_lerp STATIC
  lerp.cc lerp.h
)
target_link_libraries(
        ink_geometry_internal_lerp PUBLIC
        ink::color::color
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::vec
        ink::types::duration
)
add_library(ink::geometry::internal::lerp ALIAS ink_geometry_internal_lerp)

## outline_processing
add_library(ink_geometry_internal_outline_processing STATIC
  outline_processing.cc outline_processing.h
)
target_link_libraries(ink_geometry_internal_outline_processing
  PUBLIC
    ink::geometry::internal::intersects_internal
    ink::geometry::envelope
    ink::geometry::point
    ink::geometry::rect
    ink::geometry::segment
    ink::geometry::vec
    absl::algorithm_container
    absl::flat_hash_map
    absl::inlined_vector
    absl::span
)
add_library(ink::geometry::internal::outline_processing ALIAS ink_geometry_internal_outline_processing)

## outline_triangulation
add_library(ink_geometry_internal_outline_triangulation STATIC
  outline_triangulation.cc outline_triangulation.h
)
target_link_libraries(ink_geometry_internal_outline_triangulation
  PUBLIC
    ink::geometry::internal::outline_processing
    ink::geometry::point
    absl::algorithm_container
    absl::inlined_vector
    absl::check
    absl::span
)
add_library(ink::geometry::internal::outline_triangulation ALIAS ink_geometry_internal_outline_triangulation)
