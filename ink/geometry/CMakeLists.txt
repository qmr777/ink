# ------------------------------------------------------------------------------
#  ink_native/ink/geometry/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(internal)

# ------------------------------------------------------------------ 辅助宏

# ------------------------------------------------------------------ 基础几何库
## angle
add_library(ink_geometry_angle STATIC
        angle.cc angle.h
)
target_link_libraries(ink_geometry_angle
        PUBLIC
        ink::geometry::internal::modulo
        ink::types::numbers
        absl::str_format
)
add_library(ink::geometry::angle ALIAS ink_geometry_angle)
target_include_directories(ink_geometry_angle PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>")

## vec
add_library(ink_geometry_vec STATIC
        vec.cc vec.h
)
target_link_libraries(ink_geometry_vec
        PUBLIC
        ink::geometry::angle
        ink::types::numbers
        absl::check
        absl::str_format
)
add_library(ink::geometry::vec ALIAS ink_geometry_vec)

## point
add_library(ink_geometry_point STATIC
        point.cc point.h
)
target_link_libraries(ink_geometry_point
        PUBLIC
        ink::geometry::vec
        absl::str_format
)
add_library(ink::geometry::point ALIAS ink_geometry_point)

## segment
add_library(ink_geometry_segment STATIC
        segment.cc segment.h
)
target_link_libraries(ink_geometry_segment
        PUBLIC
        ink::geometry::point
        ink::geometry::vec
        ink::geometry::internal::lerp
        absl::strings
)
add_library(ink::geometry::segment ALIAS ink_geometry_segment)

## rect
add_library(ink_geometry_rect STATIC
        rect.cc rect.h
)
target_link_libraries(ink_geometry_rect
        PUBLIC
        ink::geometry::point
        ink::geometry::segment
        ink::geometry::vec
        absl::check
        absl::str_format
)
add_library(ink::geometry::rect ALIAS ink_geometry_rect)

## triangle
add_library(ink_geometry_triangle STATIC
        triangle.cc triangle.h
)
target_link_libraries(ink_geometry_triangle
        PUBLIC
        ink::geometry::point
        ink::geometry::segment
        ink::geometry::vec
        absl::check
        absl::str_format
)
add_library(ink::geometry::triangle ALIAS ink_geometry_triangle)

## quad
add_library(ink_geometry_quad STATIC
        quad.cc quad.h
)
target_link_libraries(ink_geometry_quad
        PUBLIC
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::rect
        ink::geometry::segment
        ink::geometry::vec
        absl::check
        absl::log
)
add_library(ink::geometry::quad ALIAS ink_geometry_quad)

## envelope
add_library(ink_geometry_envelope STATIC
        envelope.cc envelope.h
)
target_link_libraries(ink_geometry_envelope
        PUBLIC
        ink::geometry::point
        ink::geometry::quad
        ink::geometry::rect
        ink::geometry::segment
        ink::geometry::triangle
        absl::str_format
)
add_library(ink::geometry::envelope ALIAS ink_geometry_envelope)

## affine_transform
add_library(ink_geometry_affine_transform STATIC
        affine_transform.cc affine_transform.h
)
target_link_libraries(ink_geometry_affine_transform
        PUBLIC
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::quad
        ink::geometry::rect
        ink::geometry::segment
        ink::geometry::triangle
        ink::geometry::vec
        absl::strings
)
add_library(ink::geometry::affine_transform ALIAS ink_geometry_affine_transform)

## intersects
add_library(ink_geometry_intersects STATIC
        intersects.cc intersects.h
)
target_link_libraries(ink_geometry_intersects
        PUBLIC
        ink::geometry::affine_transform
        ink::geometry::mesh_index_types
        ink::geometry::partitioned_mesh
        ink::geometry::point
        ink::geometry::quad
        ink::geometry::rect
        ink::geometry::segment
        ink::geometry::triangle
        ink::geometry::internal::algorithms
        ink::geometry::internal::intersects_internal
)
add_library(ink::geometry::intersects ALIAS ink_geometry_intersects)

## distance
add_library(ink_geometry_distance STATIC
        distance.cc distance.h
)
target_link_libraries(ink_geometry_distance
        PUBLIC
        ink::geometry::intersects
        ink::geometry::point
        ink::geometry::quad
        ink::geometry::rect
        ink::geometry::segment
        ink::geometry::triangle
)
add_library(ink::geometry::distance ALIAS ink_geometry_distance)

## mesh_format
add_library(ink_geometry_mesh_format STATIC
        mesh_format.cc mesh_format.h
)
target_link_libraries(ink_geometry_mesh_format
        PUBLIC
        ink::geometry::internal::mesh_constants
        ink::types::small_array
        absl::algorithm_container
        absl::flat_hash_set
        absl::log
        absl::status
        absl::statusor
        absl::strings
        absl::span
)
add_library(ink::geometry::mesh_format ALIAS ink_geometry_mesh_format)
target_include_directories(ink_geometry_mesh_format PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>")

## mesh_index_types
add_library(ink_geometry_mesh_index_types INTERFACE)
target_include_directories(ink_geometry_mesh_index_types INTERFACE
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>"
)
add_library(ink::geometry::mesh_index_types ALIAS ink_geometry_mesh_index_types)

## mesh_packing_types
add_library(ink_geometry_mesh_packing_types STATIC
        mesh_packing_types.cc mesh_packing_types.h
)
target_link_libraries(ink_geometry_mesh_packing_types
        PUBLIC
        ink::types::small_array
        absl::str_format
)
add_library(ink::geometry::mesh_packing_types ALIAS ink_geometry_mesh_packing_types)
target_include_directories(ink_geometry_mesh_packing_types PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>")

## mutable_mesh
add_library(ink_geometry_mutable_mesh STATIC
        mutable_mesh.cc mutable_mesh.h
)
target_link_libraries(ink_geometry_mutable_mesh
        PUBLIC
        ink::geometry::envelope
        ink::geometry::mesh
        ink::geometry::mesh_format
        ink::geometry::mesh_packing_types
        ink::geometry::point
        ink::geometry::rect
        ink::geometry::triangle
        ink::geometry::vec
        ink::geometry::internal::mesh_packing
        ink::types::small_array
        ink::types::internal::float
        absl::algorithm_container
        absl::nullability
        absl::flat_hash_map
        absl::flat_hash_set
        absl::inlined_vector
        absl::check
        absl::status
        absl::status_macros
        absl::statusor
        absl::strings
        absl::span
)
add_library(ink::geometry::mutable_mesh ALIAS ink_geometry_mutable_mesh)

## mesh
add_library(ink_geometry_mesh STATIC
        mesh.cc mesh.h
)
target_link_libraries(ink_geometry_mesh
        PUBLIC
        ink::geometry::envelope
        ink::geometry::mesh_format
        ink::geometry::mesh_packing_types
        ink::geometry::point
        ink::geometry::triangle
        ink::geometry::internal::mesh_packing
        ink::types::small_array
        ink::types::internal::float
        absl::algorithm_container
        absl::nullability
        absl::check
        absl::status
        absl::status_macros
        absl::statusor
        absl::strings
        absl::span
)
add_library(ink::geometry::mesh ALIAS ink_geometry_mesh)

## partitioned_mesh
add_library(ink_geometry_partitioned_mesh STATIC
        partitioned_mesh.cc partitioned_mesh.h
)
target_link_libraries(ink_geometry_partitioned_mesh
        PUBLIC
        ink::geometry::affine_transform
        ink::geometry::envelope
        ink::geometry::mesh
        ink::geometry::mesh_format
        ink::geometry::mesh_index_types
        ink::geometry::mesh_packing_types
        ink::geometry::mutable_mesh
        ink::geometry::point
        ink::geometry::quad
        absl::span
        ink::geometry::rect
        ink::geometry::segment
        ink::geometry::triangle
        ink::geometry::internal::algorithms
        ink::geometry::internal::intersects_internal
        ink::geometry::internal::mesh_packing
        ink::geometry::internal::static_rtree
        absl::core_headers
        absl::nullability
        absl::flat_hash_map
        absl::inlined_vector
        absl::function_ref
        absl::check
        absl::status
        absl::status_macros
        absl::statusor
        absl::strings
        absl::synchronization
        absl::span
)
add_library(ink::geometry::partitioned_mesh ALIAS ink_geometry_partitioned_mesh)

## convex_hull
add_library(ink_geometry_convex_hull INTERFACE)
target_link_libraries(ink_geometry_convex_hull INTERFACE
        ink::geometry::point
        ink::geometry::internal::convex_hull_helper
        ink::geometry::internal::point_tessellation_helper
        absl::span
)
add_library(ink::geometry::convex_hull ALIAS ink_geometry_convex_hull)

## tessellator
add_library(ink_geometry_tessellator STATIC
        tessellator.cc tessellator.h
)
target_link_libraries(ink_geometry_tessellator
        PUBLIC
        ink::geometry::mesh
        ink::geometry::mesh_format
        ink::geometry::point
        ink::geometry::internal::generic_tessellator
        ink::geometry::internal::point_tessellation_helper
        absl::status
        absl::statusor
        absl::strings
        absl::span
)
add_library(ink::geometry::tessellator ALIAS ink_geometry_tessellator)
