# ------------------------------------------------------------------------------
#  ink_native/ink/geometry/internal/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_geometry_internal LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS ON)
endif()
if(NOT DEFINED INK_BUILD_BENCHMARKS)
  set(INK_BUILD_BENCHMARKS ON)
endif()

# ------------------------------------------------------------------ 辅助宏
function(ink_testonly_lib name)
  add_library(${name} STATIC ${ARGN})
  set_target_properties(${name} PROPERTIES EXCLUDE_FROM_ALL ON)
endfunction()

# ------------------------------------------------------------------ 基础工具库
## modulo
add_library(ink_geometry_internal_modulo STATIC
  modulo.cc modulo.h
)
target_link_libraries(ink_geometry_internal_modulo
  PUBLIC
    absl::check
)
add_library(ink::geometry_internal::modulo ALIAS ink_geometry_internal_modulo)
target_include_directories(ink_geometry_internal_modulo PUBLIC "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>")

## mesh_constants
add_library(ink_geometry_internal_mesh_constants INTERFACE)
target_include_directories(ink_geometry_internal_mesh_constants INTERFACE
  ${CMAKE_CURRENT_SOURCE_DIR}
)
add_library(ink::geometry_internal::mesh_constants ALIAS ink_geometry_internal_mesh_constants)
target_include_directories(ink_geometry_internal_mesh_constants INTERFACE "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>")

## legacy_vector_utils
add_library(ink_geometry_internal_legacy_vector_utils STATIC
  legacy_vector_utils.cc legacy_vector_utils.h
)
target_link_libraries(ink_geometry_internal_legacy_vector_utils
  PUBLIC
    ink::geometry_point
    ink::geometry_vec
)
add_library(ink::geometry_internal::legacy_vector_utils ALIAS ink_geometry_internal_legacy_vector_utils)

## legacy_segment_intersection
add_library(ink_geometry_internal_legacy_segment_intersection STATIC
  legacy_segment_intersection.cc legacy_segment_intersection.h
)
target_link_libraries(ink_geometry_internal_legacy_segment_intersection
  PUBLIC
    ink::geometry_internal::legacy_vector_utils
    ink::geometry_point
    ink::geometry_segment
    absl::strings
)
add_library(ink::geometry_internal::legacy_segment_intersection ALIAS ink_geometry_internal_legacy_segment_intersection)

## legacy_triangle_contains
add_library(ink_geometry_internal_legacy_triangle_contains STATIC
  legacy_triangle_contains.cc legacy_triangle_contains.h
)
target_link_libraries(ink_geometry_internal_legacy_triangle_contains
  PUBLIC
    ink::geometry_internal::legacy_vector_utils
    ink::geometry_point
    ink::geometry_triangle
)
add_library(ink::geometry_internal::legacy_triangle_contains ALIAS ink_geometry_internal_legacy_triangle_contains)

## intersects_internal
add_library(ink_geometry_internal_intersects_internal STATIC
  intersects_internal.cc intersects_internal.h
)
target_link_libraries(ink_geometry_internal_intersects_internal
  PUBLIC
    ink::geometry_point
    ink::geometry_quad
    ink::geometry_rect
    ink::geometry_segment
    ink::geometry_triangle
    ink::geometry_vec
)
add_library(ink::geometry_internal::intersects_internal ALIAS ink_geometry_internal_intersects_internal)

## point_tessellation_helper
add_library(ink_geometry_internal_point_tessellation_helper INTERFACE)
target_link_libraries(ink_geometry_internal_point_tessellation_helper INTERFACE
  ink::geometry_point
  absl::span
)
add_library(ink::geometry_internal::point_tessellation_helper ALIAS ink_geometry_internal_point_tessellation_helper)

## convex_hull_helper
add_library(ink_geometry_internal_convex_hull_helper INTERFACE)
target_link_libraries(ink_geometry_internal_convex_hull_helper INTERFACE
  ink::geometry_point
  ink::geometry_triangle
  ink::geometry_vec
  absl::span
)
add_library(ink::geometry_internal::convex_hull_helper ALIAS ink_geometry_internal_convex_hull_helper)

## static_rtree
add_library(ink_geometry_internal_static_rtree STATIC
  static_rtree.cc static_rtree.h
)
target_link_libraries(ink_geometry_internal_static_rtree
  PUBLIC
    ink::geometry_internal::intersects_internal
    ink::geometry_envelope
    ink::geometry_rect
    ink::types_small_array
    absl::algorithm_container
    absl::inlined_vector
    absl::function_ref
    absl::check
    absl::strings
    absl::span
)
add_library(ink::geometry_internal::static_rtree ALIAS ink_geometry_internal_static_rtree)

## algorithms
add_library(ink_geometry_internal_algorithms STATIC
  algorithms.cc algorithms.h
)
target_link_libraries(ink_geometry_internal_algorithms
  PUBLIC
    ink::geometry_internal::intersects_internal
    ink::color
    ink::geometry_affine_transform
    ink::geometry_angle
    ink::geometry_envelope
    ink::geometry_mesh
    ink::geometry_mutable_mesh
    ink::geometry_point
    ink::geometry_rect
    ink::geometry_segment
    ink::geometry_triangle
    ink::geometry_vec
    absl::check
    absl::span
)
add_library(ink::geometry_internal::algorithms ALIAS ink_geometry_internal_algorithms)

## polyline_processing
add_library(ink_geometry_internal_polyline_processing STATIC
  polyline_processing.cc polyline_processing.h
)
target_link_libraries(ink_geometry_internal_polyline_processing
  PUBLIC
    ink::geometry_internal::algorithms
    ink::geometry_internal::static_rtree
    ink::geometry_distance
    ink::geometry_envelope
    ink::geometry_point
    ink::geometry_rect
    ink::geometry_segment
    absl::check
    absl::span
)
add_library(ink::geometry_internal::polyline_processing ALIAS ink_geometry_internal_polyline_processing)

## circle
add_library(ink_geometry_internal_circle STATIC
  circle.cc circle.h
)
target_link_libraries(ink_geometry_internal_circle
  PUBLIC
    ink::geometry_angle
    ink::geometry_point
    ink::geometry_vec
    absl::check
)
add_library(ink::geometry_internal::circle ALIAS ink_geometry_internal_circle)

## mesh_packing
add_library(ink_geometry_internal_mesh_packing STATIC
  mesh_packing.cc mesh_packing.h
)
target_link_libraries(ink_geometry_internal_mesh_packing
  PUBLIC
    ink::geometry_internal::mesh_constants
    ink::geometry_mesh_format
    ink::geometry_mesh_packing_types
    ink::geometry_point
    ink::types_small_array
    ink::types_internal::float
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
add_library(ink::geometry_internal::mesh_packing ALIAS ink_geometry_internal_mesh_packing)

## generic_tessellator
add_library(ink_geometry_internal_generic_tessellator INTERFACE)
target_link_libraries(ink_geometry_internal_generic_tessellator INTERFACE
  absl::core_headers
  absl::check
  absl::span
  tess2
)
add_library(ink::geometry_internal::generic_tessellator ALIAS ink_geometry_internal_generic_tessellator)

# ------------------------------------------------------------------ 测试
#if(INK_BUILD_TESTS)
#  enable_testing()
#  include(GoogleTest)
#
#  # testonly 库
#  ink_testonly_lib(test_matchers test_matchers.cc test_matchers.h)
#  target_link_libraries(test_matchers
#    PUBLIC
#      ink::geometry_internal::circle
#      GTest::gtest
#      ink::geometry_type_matchers
#      absl::strings
#  )
#
#  # ---- 单元测试 ----
#  add_executable(ink_geometry_internal_modulo_test modulo_test.cc)
#  target_link_libraries(ink_geometry_internal_modulo_test
#    PRIVATE
#      ink::geometry_internal::modulo
#      fuzztest::fuzztest
#  )
#  gtest_discover_tests(ink_geometry_internal_modulo_test)
#
#  add_executable(ink_geometry_internal_legacy_vector_utils_test legacy_vector_utils_test.cc)
#  target_link_libraries(ink_geometry_internal_legacy_vector_utils_test
#    PRIVATE
#      ink::geometry_internal::legacy_vector_utils
#      ink::geometry_point
#  )
#  gtest_discover_tests(ink_geometry_internal_legacy_vector_utils_test)
#
#  add_executable(ink_geometry_internal_legacy_segment_intersection_test legacy_segment_intersection_test.cc)
#  target_link_libraries(ink_geometry_internal_legacy_segment_intersection_test
#    PRIVATE
#      ink::geometry_internal::legacy_segment_intersection
#      ink::geometry_point
#      ink::geometry_segment
#      ink::geometry_type_matchers
#      absl::strings
#  )
#  gtest_discover_tests(ink_geometry_internal_legacy_segment_intersection_test)
#
#  add_executable(ink_geometry_internal_legacy_triangle_contains_test legacy_triangle_contains_test.cc)
#  target_link_libraries(ink_geometry_internal_legacy_triangle_contains_test
#    PRIVATE
#      ink::geometry_internal::legacy_triangle_contains
#      ink::geometry_point
#      ink::geometry_triangle
#  )
#  gtest_discover_tests(ink_geometry_internal_legacy_triangle_contains_test)
#
#  add_executable(ink_geometry_internal_algorithms_test algorithms_test.cc)
#  target_link_libraries(ink_geometry_internal_algorithms_test
#    PRIVATE
#      ink::geometry_internal::algorithms
#      ink::color
#      ink::geometry_affine_transform
#      ink::geometry_angle
#      ink::geometry_envelope
#      ink::geometry_mesh
#      ink::geometry_mesh_test_helpers
#      ink::geometry_mutable_mesh
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_triangle
#      ink::geometry_vec
#      ink::geometry_type_matchers
#      ink::color_type_matchers
#      absl::inlined_vector
#      absl::status
#      absl::statusor
#      absl::span
#  )
#  gtest_discover_tests(ink_geometry_internal_algorithms_test)
#
#  add_executable(ink_geometry_internal_polyline_processing_test polyline_processing_test.cc)
#  target_link_libraries(ink_geometry_internal_polyline_processing_test
#    PRIVATE
#      ink::geometry_internal::polyline_processing
#      ink::geometry_internal::static_rtree
#      ink::geometry_fuzz_domains
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_segment
#      ink::geometry_type_matchers
#      fuzztest::fuzztest
#  )
#  gtest_discover_tests(ink_geometry_internal_polyline_processing_test)
#
#  add_executable(ink_geometry_internal_circle_test circle_test.cc)
#  target_link_libraries(ink_geometry_internal_circle_test
#    PRIVATE
#      ink::geometry_internal::circle
#      ink::geometry_angle
#      ink::geometry_point
#      ink::geometry_segment
#      ink::geometry_type_matchers
#      ink::geometry_vec
#      absl::str_format
#  )
#  gtest_discover_tests(ink_geometry_internal_circle_test)
#
#  add_executable(ink_geometry_internal_mesh_packing_test mesh_packing_test.cc)
#  target_link_libraries(ink_geometry_internal_mesh_packing_test
#    PRIVATE
#      ink::geometry_internal::mesh_packing
#      ink::geometry_mesh_format
#      ink::geometry_mesh_packing_types
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::types_small_array
#      absl::flat_hash_map
#      absl::inlined_vector
#      absl::check
#      absl::span
#  )
#  gtest_discover_tests(ink_geometry_internal_mesh_packing_test)
#
#  add_executable(ink_geometry_internal_static_rtree_test static_rtree_test.cc)
#  target_link_libraries(ink_geometry_internal_static_rtree_test
#    PRIVATE
#      ink::geometry_internal::static_rtree
#      ink::geometry_distance
#      ink::geometry_point
#      ink::geometry_rect
#      ink::geometry_type_matchers
#      ink::types_small_array
#      absl::check
#      absl::span
#  )
#  gtest_discover_tests(ink_geometry_internal_static_rtree_test)
#
#  add_executable(ink_geometry_internal_generic_tessellator_test generic_tessellator_test.cc)
#  target_link_libraries(ink_geometry_internal_generic_tessellator_test
#    PRIVATE
#      ink::geometry_internal::generic_tessellator
#      ink::geometry_point
#      ink::geometry_internal::point_tessellation_helper
#      libtess2::libtess2
#  )
#  gtest_discover_tests(ink_geometry_internal_generic_tessellator_test)
#
#  # ---- benchmark ----
#  if(INK_BUILD_BENCHMARKS)
#    add_executable(ink_geometry_internal_static_rtree_benchmark static_rtree_benchmark.cc)
#    target_link_libraries(ink_geometry_internal_static_rtree_benchmark
#      PRIVATE
#        ink::geometry_internal::static_rtree
#        ink::geometry_rect
#        absl::span
#        benchmark::benchmark_main
#    )
#  endif()
#endif()