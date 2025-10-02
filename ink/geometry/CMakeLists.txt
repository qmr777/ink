# ------------------------------------------------------------------------------
#  ink_native/ink/geometry/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS OFF)
endif()
if(NOT DEFINED INK_BUILD_BENCHMARKS)
  set(INK_BUILD_BENCHMARKS OFF)
endif()

add_subdirectory(internal)

# ------------------------------------------------------------------ 辅助宏
function(ink_testonly_lib name)
  add_library(${name} STATIC ${ARGN})
  set_target_properties(${name} PROPERTIES EXCLUDE_FROM_ALL ON)
endfunction()

function(ink_cc_test name)
  add_executable(${name} ${ARGN})
  target_link_libraries(${name} PRIVATE ${ARGN}_deps)
  gtest_discover_tests(${name})
endfunction()

# ------------------------------------------------------------------ 基础几何库
## angle
add_library(ink_geometry_angle STATIC
  angle.cc angle.h
)
target_link_libraries(ink_geometry_angle
  PUBLIC
    ink::geometry_internal::modulo
    ink::types_numbers
    absl::str_format
)
add_library(ink::geometry_angle ALIAS ink_geometry_angle)
target_include_directories(ink_geometry_angle PUBLIC "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>")

## vec
add_library(ink_geometry_vec STATIC
  vec.cc vec.h
)
target_link_libraries(ink_geometry_vec
  PUBLIC
    ink::geometry_angle
    ink::types_numbers
    absl::check
    absl::str_format
)
add_library(ink::geometry_vec ALIAS ink_geometry_vec)

## point
add_library(ink_geometry_point STATIC
  point.cc point.h
)
target_link_libraries(ink_geometry_point
  PUBLIC
    ink::geometry_vec
    absl::str_format
)
add_library(ink::geometry_point ALIAS ink_geometry_point)

## segment
add_library(ink_geometry_segment STATIC
  segment.cc segment.h
)
target_link_libraries(ink_geometry_segment
  PUBLIC
    ink::geometry_point
    ink::geometry_vec
    absl::strings
)
add_library(ink::geometry_segment ALIAS ink_geometry_segment)

## rect
add_library(ink_geometry_rect STATIC
  rect.cc rect.h
)
target_link_libraries(ink_geometry_rect
  PUBLIC
    ink::geometry_point
    ink::geometry_segment
    ink::geometry_vec
    absl::check
    absl::str_format
)
add_library(ink::geometry_rect ALIAS ink_geometry_rect)

## triangle
add_library(ink_geometry_triangle STATIC
  triangle.cc triangle.h
)
target_link_libraries(ink_geometry_triangle
  PUBLIC
    ink::geometry_point
    ink::geometry_segment
    ink::geometry_vec
    absl::check
    absl::str_format
)
add_library(ink::geometry_triangle ALIAS ink_geometry_triangle)

## quad
add_library(ink_geometry_quad STATIC
  quad.cc quad.h
)
target_link_libraries(ink_geometry_quad
  PUBLIC
    ink::geometry_angle
    ink::geometry_point
    ink::geometry_rect
    ink::geometry_segment
    ink::geometry_vec
    absl::check
    absl::log
)
add_library(ink::geometry_quad ALIAS ink_geometry_quad)

## envelope
add_library(ink_geometry_envelope STATIC
  envelope.cc envelope.h
)
target_link_libraries(ink_geometry_envelope
  PUBLIC
    ink::geometry_point
    ink::geometry_quad
    ink::geometry_rect
    ink::geometry_segment
    ink::geometry_triangle
    absl::str_format
)
add_library(ink::geometry_envelope ALIAS ink_geometry_envelope)

## affine_transform
add_library(ink_geometry_affine_transform STATIC
  affine_transform.cc affine_transform.h
)
target_link_libraries(ink_geometry_affine_transform
  PUBLIC
    ink::geometry_angle
    ink::geometry_point
    ink::geometry_quad
    ink::geometry_rect
    ink::geometry_segment
    ink::geometry_triangle
    ink::geometry_vec
    absl::strings
)
add_library(ink::geometry_affine_transform ALIAS ink_geometry_affine_transform)

## intersects
add_library(ink_geometry_intersects STATIC
  intersects.cc intersects.h
)
target_link_libraries(ink_geometry_intersects
  PUBLIC
    ink::geometry_affine_transform
    ink::geometry_mesh_index_types
    ink::geometry_partitioned_mesh
    ink::geometry_point
    ink::geometry_quad
    ink::geometry_rect
    ink::geometry_segment
    ink::geometry_triangle
    ink::geometry_internal::algorithms
    ink::geometry_internal::intersects_internal
)
add_library(ink::geometry_intersects ALIAS ink_geometry_intersects)

## distance
add_library(ink_geometry_distance STATIC
  distance.cc distance.h
)
target_link_libraries(ink_geometry_distance
  PUBLIC
    ink::geometry_intersects
    ink::geometry_point
    ink::geometry_quad
    ink::geometry_rect
    ink::geometry_segment
    ink::geometry_triangle
)
add_library(ink::geometry_distance ALIAS ink_geometry_distance)

## mesh_format
add_library(ink_geometry_mesh_format STATIC
  mesh_format.cc mesh_format.h
)
target_link_libraries(ink_geometry_mesh_format
  PUBLIC
    ink::geometry_internal::mesh_constants
    ink::types_small_array
    absl::algorithm_container
    absl::flat_hash_set
    absl::log
    absl::status
    absl::statusor
    absl::strings
    absl::span
)
add_library(ink::geometry_mesh_format ALIAS ink_geometry_mesh_format)

## mesh_index_types
add_library(ink_geometry_mesh_index_types INTERFACE)
target_include_directories(ink_geometry_mesh_index_types INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}
)
add_library(ink::geometry_mesh_index_types ALIAS ink_geometry_mesh_index_types)

## mesh_packing_types
add_library(ink_geometry_mesh_packing_types STATIC
  mesh_packing_types.cc mesh_packing_types.h
)
target_link_libraries(ink_geometry_mesh_packing_types
  PUBLIC
    ink::types_small_array
    absl::str_format
)
add_library(ink::geometry_mesh_packing_types ALIAS ink_geometry_mesh_packing_types)
target_include_directories(ink_geometry_mesh_packing_types PUBLIC "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>")

## mutable_mesh
add_library(ink_geometry_mutable_mesh STATIC
  mutable_mesh.cc mutable_mesh.h
)
target_link_libraries(ink_geometry_mutable_mesh
  PUBLIC
    ink::geometry_envelope
    ink::geometry_mesh
    ink::geometry_mesh_format
    ink::geometry_mesh_packing_types
    ink::geometry_point
    ink::geometry_rect
    ink::geometry_triangle
    ink::geometry_vec
    ink::geometry_internal::mesh_packing
    ink::types_small_array
    ink::types_internal::float
    absl::algorithm_container

    absl::flat_hash_map
    absl::flat_hash_set
    absl::inlined_vector
    absl::check
    absl::status
    absl::statusor
    absl::strings
    absl::span
)
add_library(ink::geometry_mutable_mesh ALIAS ink_geometry_mutable_mesh)

## mesh
add_library(ink_geometry_mesh STATIC
  mesh.cc mesh.h
)
target_link_libraries(ink_geometry_mesh
  PUBLIC
    ink::geometry_envelope
    ink::geometry_mesh_format
    ink::geometry_mesh_packing_types
    ink::geometry_point
    ink::geometry_triangle
    ink::geometry_internal::mesh_packing
    ink::types_small_array
    ink::types_internal::float
    absl::algorithm_container

    absl::check
    absl::status
    absl::statusor
    absl::strings
    absl::span
)
add_library(ink::geometry_mesh ALIAS ink_geometry_mesh)

## partitioned_mesh
add_library(ink_geometry_partitioned_mesh STATIC
  partitioned_mesh.cc partitioned_mesh.h
)
target_link_libraries(ink_geometry_partitioned_mesh
  PUBLIC
    ink::geometry_affine_transform
    ink::geometry_envelope
    ink::geometry_mesh
    ink::geometry_mesh_format
    ink::geometry_mesh_index_types
    ink::geometry_mesh_packing_types
    ink::geometry_mutable_mesh
    ink::geometry_point
    ink::geometry_quad
    ink::geometry_rect
    ink::geometry_segment
    ink::geometry_triangle
    ink::geometry_internal::algorithms
    ink::geometry_internal::intersects_internal
    ink::geometry_internal::mesh_packing
    ink::geometry_internal::static_rtree
    absl::core_headers

    absl::flat_hash_map
    absl::inlined_vector
    absl::function_ref
    absl::check
    absl::status
    absl::statusor
    absl::strings
    absl::synchronization
    absl::span
)
add_library(ink::geometry_partitioned_mesh ALIAS ink_geometry_partitioned_mesh)

## convex_hull
add_library(ink_geometry_convex_hull INTERFACE)
target_link_libraries(ink_geometry_convex_hull INTERFACE
  ink::geometry_point
  ink::geometry_internal::convex_hull_helper
  ink::geometry_internal::point_tessellation_helper
  absl::span
)
add_library(ink::geometry_convex_hull ALIAS ink_geometry_convex_hull)

## tessellator
add_library(ink_geometry_tessellator STATIC
  tessellator.cc tessellator.h
)
target_link_libraries(ink_geometry_tessellator
  PUBLIC
    ink::geometry_mesh
    ink::geometry_mesh_format
    ink::geometry_point
    ink::geometry_internal::generic_tessellator
    ink::geometry_internal::point_tessellation_helper
    absl::status
    absl::statusor
    absl::strings
    absl::span
)
add_library(ink::geometry_tessellator ALIAS ink_geometry_tessellator)

#
## ------------------------------------------------------------------ 测试
#if(INK_BUILD_TESTS)
#  enable_testing()
#  include(GoogleTest)
#
#  # testonly 库
#  ink_testonly_lib(mesh_test_helpers mesh_test_helpers.cc mesh_test_helpers.h)
#  target_link_libraries(mesh_test_helpers
#    PUBLIC
#      ink::geometry_affine_transform
#      ink::geometry_angle
#      ink::geometry_mesh_format
#      ink::geometry_mutable_mesh
#      ink::geometry_partitioned_mesh
#      ink::geometry_point
#      ink::geometry_vec
#      absl::check
#      absl::statusor
#  )
#
#  ink_testonly_lib(fuzz_domains fuzz_domains.cc fuzz_domains.h)
#  target_link_libraries(fuzz_domains
#    PUBLIC
#      ink::geometry_angle
#      ink::geometry_mesh_format
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_triangle
#      ink::geometry_vec
#      absl::check
#      absl::span
#      fuzztest::fuzztest
#  )
#
#  ink_testonly_lib(type_matchers type_matchers.cc type_matchers.h)
#  target_link_libraries(type_matchers
#    PUBLIC
#      ink::geometry_affine_transform
#      ink::geometry_angle
#      ink::geometry_envelope
#      ink::geometry_mesh
#      ink::geometry_mesh_format
#      ink::geometry_mesh_index_types
#      ink::geometry_mesh_packing_types
#      ink::geometry_partitioned_mesh
#      ink::geometry_point
#      ink::geometry_quad
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_triangle
#      ink::geometry_vec
#      ink::types_small_array
#      GTest::gtest
#      absl::strings
#  )
#
#  # ---- 单元测试 ----
#  ink_cc_test(affine_transform_test affine_transform_test.cc)
#  target_link_libraries(affine_transform_test
#    PRIVATE
#      ink::geometry_affine_transform
#      ink::geometry_angle
#      ink::geometry_fuzz_domains
#      ink::geometry_point
#      ink::geometry_quad
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_triangle
#      ink::geometry_vec
#      ink::geometry_type_matchers
#      absl::strings
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(angle_test angle_test.cc)
#  target_link_libraries(angle_test
#    PRIVATE
#      ink::geometry_angle
#      ink::geometry_fuzz_domains
#      ink::types_numbers
#      absl::hash_testing
#      absl::strings
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(distance_test distance_test.cc)
#  target_link_libraries(distance_test
#    PRIVATE
#      ink::geometry_distance
#      ink::geometry_angle
#      ink::geometry_point
#      ink::geometry_quad
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_triangle
#  )
#
#  ink_cc_test(envelope_test envelope_test.cc)
#  target_link_libraries(envelope_test
#    PRIVATE
#      ink::geometry_envelope
#      ink::geometry_angle
#      ink::geometry_point
#      ink::geometry_quad
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_triangle
#      ink::geometry_type_matchers
#      absl::strings
#  )
#
#  ink_cc_test(intersects_test intersects_test.cc)
#  target_link_libraries(intersects_test
#    PRIVATE
#      ink::geometry_intersects
#      ink::geometry_affine_transform
#      ink::geometry_angle
#      ink::geometry_partitioned_mesh
#      ink::geometry_point
#      ink::geometry_quad
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_triangle
#      ink::geometry_mesh_test_helpers
#  )
#
#  ink_cc_test(quad_test quad_test.cc)
#  target_link_libraries(quad_test
#    PRIVATE
#      ink::geometry_quad
#      ink::geometry_angle
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_vec
#      ink::geometry_type_matchers
#  )
#
#  ink_cc_test(rect_test rect_test.cc)
#  target_link_libraries(rect_test
#    PRIVATE
#      ink::geometry_rect
#      ink::geometry_fuzz_domains
#      ink::geometry_point
#      ink::geometry_segment
#      ink::geometry_vec
#      ink::geometry_type_matchers
#      absl::strings
#      fuzztest::fuzztest
#  )
#  # shard_count=10 等价
#  set_tests_properties(rect_test PROPERTIES PROCESSORS 10)
#
#  ink_cc_test(segment_test segment_test.cc)
#  target_link_libraries(segment_test
#    PRIVATE
#      ink::geometry_segment
#      ink::geometry_fuzz_domains
#      ink::geometry_type_matchers
#      absl::strings
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(triangle_test triangle_test.cc)
#  target_link_libraries(triangle_test
#    PRIVATE
#      ink::geometry_triangle
#      ink::geometry_fuzz_domains
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_segment
#      absl::strings
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(point_test point_test.cc)
#  target_link_libraries(point_test
#    PRIVATE
#      ink::geometry_point
#      ink::geometry_vec
#      ink::geometry_type_matchers
#      absl::hash_testing
#      absl::strings
#  )
#
#  ink_cc_test(mesh_format_test mesh_format_test.cc)
#  target_link_libraries(mesh_format_test
#    PRIVATE
#      ink::geometry_mesh_format
#      ink::geometry_fuzz_domains
#      absl::hash_testing
#      absl::status absl::status_matchers absl::statusor absl::strings absl::span
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(mesh_packing_types_test mesh_packing_types_test.cc)
#  target_link_libraries(mesh_packing_types_test
#    PRIVATE
#      ink::geometry_mesh_packing_types
#      absl::strings
#  )
#
#  ink_cc_test(mutable_mesh_test mutable_mesh_test.cc)
#  target_link_libraries(mutable_mesh_test
#    PRIVATE
#      ink::geometry_mutable_mesh
#      ink::geometry_mesh
#      ink::geometry_mesh_format
#      ink::geometry_mesh_packing_types
#      ink::geometry_fuzz_domains
#      ink::geometry_mesh_test_helpers
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_triangle
#      ink::geometry_type_matchers
#      ink::geometry_internal::mesh_packing
#      ink::types_small_array
#      absl::inlined_vector absl::status absl::strings absl::span
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(mesh_test mesh_test.cc)
#  target_link_libraries(mesh_test
#    PRIVATE
#      ink::geometry_mesh
#      ink::geometry_mesh_format
#      ink::geometry_mesh_packing_types
#      ink::geometry_mesh_test_helpers
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::geometry_internal::mesh_packing
#      absl::status absl::statusor absl::span
#  )
#
#  ink_cc_test(mesh_test_helpers_test mesh_test_helpers_test.cc)
#  target_link_libraries(mesh_test_helpers_test
#    PRIVATE
#      ink::geometry_mesh_test_helpers
#      ink::geometry_affine_transform
#      ink::geometry_angle
#      ink::geometry_mesh_format
#      ink::geometry_mutable_mesh
#      ink::geometry_partitioned_mesh
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::geometry_vec
#      absl::status absl::statusor absl::strings
#  )
#
#  ink_cc_test(partitioned_mesh_test partitioned_mesh_test.cc)
#  target_link_libraries(partitioned_mesh_test
#    PRIVATE
#      ink::geometry_partitioned_mesh
#      ink::geometry_affine_transform
#      ink::geometry_angle
#      ink::geometry_mesh
#      ink::geometry_mesh_format
#      ink::geometry_mesh_index_types
#      ink::geometry_mesh_packing_types
#      ink::geometry_mesh_test_helpers
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_quad
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_triangle
#      ink::geometry_type_matchers
#      absl::inlined_vector absl::check absl::status absl::status_matchers absl::statusor absl::strings absl::span
#  )
#
#  ink_cc_test(convex_hull_test convex_hull_test.cc)
#  target_link_libraries(convex_hull_test
#    PRIVATE
#      ink::geometry_convex_hull
#      ink::geometry_point
#  )
#
#  ink_cc_test(tessellator_test tessellator_test.cc)
#  target_link_libraries(tessellator_test
#    PRIVATE
#      ink::geometry_tessellator
#      ink::geometry_mesh
#      ink::geometry_mesh_format
#      ink::geometry_point
#      ink::geometry_type_matchers
#      absl::status absl::status_matchers absl::statusor
#  )
#
#  # 手动标记的 fuzz 测试
#  ink_cc_test(tessellator_fuzz_test tessellator_fuzz_test.cc)
#  target_link_libraries(tessellator_fuzz_test
#    PRIVATE
#      ink::geometry_tessellator
#      ink::geometry_fuzz_domains
#      ink::geometry_point
#      fuzztest::fuzztest
#  )
#  set_tests_properties(tessellator_fuzz_test PROPERTIES LABELS "manual")
#
#  # ---- benchmark ----
#  if(INK_BUILD_BENCHMARKS)
#    ink_cc_test(vec_benchmark vec_benchmark.cc)
#    target_link_libraries(vec_benchmark
#      PRIVATE
#        ink::geometry_vec
#        benchmark::benchmark_main
#    )
#  endif()
#endif()
#)
#
