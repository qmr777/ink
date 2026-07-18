# ------------------------------------------------------------------------------
#  ink_native/ink/storage/CMakeLists.txt
#  等价于 ink/storage/BUILD.bazel
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_storage LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 关键:关闭 protobuf / utf8_range / abseil 的 install 步骤,
# 否则 utf8_validity 的 install(EXPORT) 会尝试拉 absl::strings 触发
#   "install(EXPORT ... utf8_validity) ... absl_strings is not in any export set"
# (protobuf 33.x 在 Windows MSVC 下的已知问题)
set(protobuf_BUILD_TESTS    OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(protobuf_INSTALL        OFF CACHE BOOL "" FORCE)
set(utf8_range_ENABLE_TESTS OFF CACHE BOOL "" FORCE)
set(utf8_range_ENABLE_INSTALL OFF CACHE BOOL "" FORCE)
set(absl_ENABLE_INSTALL     OFF CACHE BOOL "" FORCE)

include(FetchContent)

FetchContent_Declare(
        protobuf
        https://github.com/protocolbuffers/protobuf.git
        v33.5
)
FetchContent_MakeAvailable(protobuf)

# 沿用顶层选项
if (NOT DEFINED INK_BUILD_TESTS)
    set(INK_BUILD_TESTS ON)
endif ()
if (NOT DEFINED INK_BUILD_BENCHMARKS)
    set(INK_BUILD_BENCHMARKS OFF)
endif ()

# 必须先于下方：proto 库会生成并暴露 ink::storage::proto::*_cc_proto
add_subdirectory(proto)

# ------------------------------------------------------------------ 辅助宏
# 测试专用库（testonly = 1 等价）
function(ink_testonly_lib name)
    add_library(${name} STATIC ${ARGN})
    target_compile_definitions(${name} PRIVATE INK_STORAGE_TESTONLY)
    set_target_properties(${name} PROPERTIES EXCLUDE_FROM_ALL ON)
endfunction()

# 单元测试可执行文件
function(ink_cc_test name)
    add_executable(${name} ${ARGN})
    gtest_discover_tests(${name})
endfunction()

# 注意：
#  ink/storage 下大量库依赖 //ink/storage/proto:xxx_cc_proto 生成的 .pb.h
#  proto 头文件由同级的 ink/storage/proto/CMakeLists.txt 调用 protoc 生成，
#  并以 ink::storage::proto::<name>_cc_proto 形式对外暴露。

# ------------------------------------------------------------------ numeric_run
add_library(ink_storage_numeric_run STATIC
        numeric_run.cc
        numeric_run.h
)
target_link_libraries(ink_storage_numeric_run
        PUBLIC
        ink::types_iterator_range
        ink::storage::proto::coded_numeric_run_cc_proto
        absl::status
        absl::statusor
)
add_library(ink::storage::numeric_run ALIAS ink_storage_numeric_run)
target_include_directories(ink_storage_numeric_run
        PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>"
)

# ------------------------------------------------------------------ color
add_library(ink_storage_color STATIC
        color.cc
        color.h
)
target_link_libraries(ink_storage_color
        PUBLIC
        ink::color
        ink::color_space
        ink::storage::proto::color_cc_proto
        absl::check
        absl::log
)
add_library(ink::storage::color ALIAS ink_storage_color)
target_include_directories(ink_storage_color
        PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>"
)

# ------------------------------------------------------------------ mesh_format
add_library(ink_storage_mesh_format STATIC
        mesh_format.cc
        mesh_format.h
)
target_link_libraries(ink_storage_mesh_format
        PUBLIC
        ink::geometry_internal::mesh_constants
        ink::geometry_mesh_format
        ink::types_small_array
        ink::storage::proto::mesh_cc_proto
        absl::log
        absl::status
        absl::status_macros
        absl::statusor
        absl::strings
)
add_library(ink::storage::mesh_format ALIAS ink_storage_mesh_format)
target_include_directories(ink_storage_mesh_format
        PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>"
)

# ------------------------------------------------------------------ mesh
add_library(ink_storage_mesh STATIC
        mesh.cc
        mesh.h
)
target_link_libraries(ink_storage_mesh
        PUBLIC
        ink::geometry_internal::mesh_packing
        ink::geometry_mesh
        ink::geometry_mesh_format
        ink::geometry_mesh_packing_types
        ink::geometry_point
        ink::storage::mesh_format
        ink::storage::numeric_run
        ink::storage::proto::coded_numeric_run_cc_proto
        ink::storage::proto::mesh_cc_proto
        ink::types_iterator_range
        ink::types_small_array
        absl::check
        absl::status
        absl::status_macros
        absl::statusor
        absl::strings
        absl::span
)
add_library(ink::storage::mesh ALIAS ink_storage_mesh)

# ------------------------------------------------------------------ mesh_vertices
add_library(ink_storage_mesh_vertices STATIC
        mesh_vertices.cc
        mesh_vertices.h
)
target_link_libraries(ink_storage_mesh_vertices
        PUBLIC
        ink::storage::numeric_run
        ink::storage::proto::mesh_cc_proto
        ink::strokes::internal::legacy_vertex
        ink::types_iterator_range
        absl::status
        absl::status_macros
        absl::statusor
)
add_library(ink::storage::mesh_vertices ALIAS ink_storage_mesh_vertices)

# ------------------------------------------------------------------ partitioned_mesh
add_library(ink_storage_partitioned_mesh STATIC
        partitioned_mesh.cc
        partitioned_mesh.h
)
target_link_libraries(ink_storage_partitioned_mesh
        PUBLIC
        ink::geometry_mesh
        ink::geometry_mesh_format
        ink::geometry_mesh_index_types
        ink::geometry_partitioned_mesh
        ink::storage::mesh
        ink::storage::mesh_format
        ink::storage::numeric_run
        ink::storage::proto::coded_numeric_run_cc_proto
        ink::storage::proto::mesh_cc_proto
        absl::status
        absl::status_macros
        absl::statusor
        absl::strings
        absl::span
)
add_library(ink::storage::partitioned_mesh ALIAS ink_storage_partitioned_mesh)

# ------------------------------------------------------------------ input_batch
add_library(ink_storage_input_batch STATIC
        input_batch.cc
        input_batch.h
)
target_link_libraries(ink_storage_input_batch
        PUBLIC
        ink::geometry_angle
        ink::storage::numeric_run
        ink::storage::proto::stroke_input_batch_cc_proto
        ink::strokes::input::stroke_input
        ink::types_duration
        ink::types_iterator_range
        ink::types_physical_distance
        absl::status
        absl::status_macros
        absl::statusor
)
add_library(ink::storage::input_batch ALIAS ink_storage_input_batch)

# ------------------------------------------------------------------ stroke_input_batch
add_library(ink_storage_stroke_input_batch STATIC
        stroke_input_batch.cc
        stroke_input_batch.h
)
target_link_libraries(ink_storage_stroke_input_batch
        PUBLIC
        ink::geometry_angle
        ink::geometry_rect
        ink::storage::input_batch
        ink::storage::proto::coded_numeric_run_cc_proto
        ink::storage::proto::stroke_input_batch_cc_proto
        ink::strokes::input::stroke_input
        ink::strokes::input::stroke_input_batch
        ink::types_duration
        ink::types_iterator_range
        ink::types_numbers
        ink::types_physical_distance
        absl::status
        absl::status_macros
        absl::statusor
)
add_library(ink::storage::stroke_input_batch ALIAS ink_storage_stroke_input_batch)

# ------------------------------------------------------------------ brush
add_library(ink_storage_brush STATIC
        brush.cc
        brush.h
)
target_link_libraries(ink_storage_brush
        PUBLIC
        ink::brush
        ink::brush_behavior
        ink::brush_coat
        ink::brush_family
        ink::brush_family_internal_accessor
        ink::brush_paint
        ink::brush_tip
        ink::color_function
        ink::easing_function
        ink::geometry_angle
        ink::geometry_point
        ink::geometry_vec
        ink::storage::color
        ink::storage::proto::brush_cc_proto
        ink::storage::proto::brush_family_cc_proto
        ink::storage::proto::options_cc_proto
        ink::storage::proto::stroke_input_batch_cc_proto
        ink::types_duration
        absl::algorithm_container
        absl::flat_hash_map
        absl::flat_hash_set
        absl::inlined_vector
        absl::status
        absl::status_macros
        absl::statusor
        absl::strings
        absl::time
        absl::span
)
add_library(ink::storage::brush ALIAS ink_storage_brush)

# ------------------------------------------------------------------ 测试专用库
#if(INK_BUILD_TESTS)
#  # proto_matchers（testonly = 1）
#  ink_testonly_lib(proto_matchers
#    proto_matchers.cc
#    proto_matchers.h
#  )
#  target_link_libraries(proto_matchers
#    PUBLIC
#      GTest::gtest
#      protobuf::libprotobuf
#  )
#endif()

# ------------------------------------------------------------------ 单元测试
#if(INK_BUILD_TESTS)
#  enable_testing()
#  include(GoogleTest)
#
#  # 统一用 fuzztest 提供的 gtest_main（fuzztest 测试）
#  set(_FUZZTEST_MAIN fuzztest::gtest_main)
#
#  # color_test
#  ink_cc_test(color_test color_test.cc)
#  target_link_libraries(color_test
#    PRIVATE
#      ink::storage::color
#      ink::storage::proto_matchers
#      ink::color
#      ink::color_space
#      ink::storage::proto::color_cc_proto
#      fuzztest::fuzztest
#      ${_FUZZTEST_MAIN}
#  )
#
#  # stroke_input_batch_test
#  ink_cc_test(stroke_input_batch_test stroke_input_batch_test.cc)
#  target_link_libraries(stroke_input_batch_test
#    PRIVATE
#      ink::storage::numeric_run
#      ink::storage::stroke_input_batch
#      ink::geometry_angle
#      ink::storage::proto::stroke_input_batch_cc_proto
#      ink::strokes::input::fuzz_domains
#      ink::strokes::input::stroke_input
#      ink::strokes::input::stroke_input_batch
#      ink::strokes::input::type_matchers
#      ink::types_duration
#      ink::types_iterator_range
#      ink::types_physical_distance
#      absl::check
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      absl::span
#      fuzztest::fuzztest
#      ${_FUZZTEST_MAIN}
#  )
#
#  # input_batch_test
#  ink_cc_test(input_batch_test input_batch_test.cc)
#  target_link_libraries(input_batch_test
#    PRIVATE
#      ink::storage::input_batch
#      ink::geometry_angle
#      ink::geometry_type_matchers
#      ink::storage::proto::stroke_input_batch_cc_proto
#      ink::strokes::input::stroke_input
#      ink::strokes::input::type_matchers
#      ink::types_duration
#      ink::types_iterator_range
#      ink::types_numbers
#      ink::types_physical_distance
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      GTest::gtest_main
#      protobuf::libprotobuf
#  )
#
#  # mesh_test
#  ink_cc_test(mesh_test mesh_test.cc)
#  target_link_libraries(mesh_test
#    PRIVATE
#      ink::storage::mesh
#      ink::storage::numeric_run
#      ink::geometry_fuzz_domains
#      ink::geometry_mesh
#      ink::geometry_mesh_format
#      ink::geometry_point
#      ink::geometry_type_matchers
#      ink::storage::proto::mesh_cc_proto
#      ink::types_iterator_range
#      absl::inlined_vector
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      absl::span
#      fuzztest::fuzztest
#      ${_FUZZTEST_MAIN}
#      protobuf::libprotobuf
#  )
#
#  # mesh_format_test
#  ink_cc_test(mesh_format_test mesh_format_test.cc)
#  target_link_libraries(mesh_format_test
#    PRIVATE
#      ink::storage::mesh_format
#      ink::storage::proto_matchers
#      ink::geometry_fuzz_domains
#      ink::geometry_mesh_format
#      ink::geometry_type_matchers
#      ink::storage::proto::mesh_cc_proto
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      fuzztest::fuzztest
#      ${_FUZZTEST_MAIN}
#  )
#
#  # mesh_vertices_test
#  ink_cc_test(mesh_vertices_test mesh_vertices_test.cc)
#  target_link_libraries(mesh_vertices_test
#    PRIVATE
#      ink::storage::mesh_vertices
#      ink::geometry_type_matchers
#      ink::storage::proto::mesh_cc_proto
#      ink::strokes::internal::legacy_vertex
#      ink::types_iterator_range
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      GTest::gtest_main
#      protobuf::libprotobuf
#  )
#
#  # partitioned_mesh_test
#  ink_cc_test(partitioned_mesh_test partitioned_mesh_test.cc)
#  target_link_libraries(partitioned_mesh_test
#    PRIVATE
#      ink::storage::mesh_format
#      ink::storage::numeric_run
#      ink::storage::partitioned_mesh
#      ink::geometry_mesh
#      ink::geometry_mesh_format
#      ink::geometry_mesh_index_types
#      ink::geometry_partitioned_mesh
#      ink::geometry_type_matchers
#      ink::storage::proto::mesh_cc_proto
#      ink::types_iterator_range
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      absl::span
#      fuzztest::fuzztest
#      ${_FUZZTEST_MAIN}
#      protobuf::libprotobuf
#  )
#
#  # brush_test（shard_count = 10）
#  ink_cc_test(brush_test brush_test.cc)
#  target_link_libraries(brush_test
#    PRIVATE
#      ink::storage::brush
#      ink::storage::color
#      ink::storage::proto_matchers
#      ink::brush
#      ink::brush_behavior
#      ink::brush_coat
#      ink::brush_family
#      ink::brush_family_internal_accessor
#      ink::brush_paint
#      ink::brush_tip
#      ink::easing_function
#      ink::brush_fuzz_domains
#      ink::brush_type_matchers
#      ink_brush_version
#      ink::color
#      ink::geometry_angle
#      ink::storage::proto::brush_cc_proto
#      ink::storage::proto::brush_family_cc_proto
#      ink::storage::proto::color_cc_proto
#      ink::storage::proto::options_cc_proto
#      ink::storage::proto::stroke_input_batch_cc_proto
#      ink::types_duration
#      absl::flat_hash_set
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      absl::strings
#      absl::string_view
#      absl::time
#      fuzztest::fuzztest
#      ${_FUZZTEST_MAIN}
#      protobuf::libprotobuf
#  )
#  set_tests_properties(brush_test PROPERTIES PROCESSORS 10)
#
#  # numeric_run_test
#  ink_cc_test(numeric_run_test numeric_run_test.cc)
#  target_link_libraries(numeric_run_test
#    PRIVATE
#      ink::storage::numeric_run
#      ink::storage::proto::coded_numeric_run_cc_proto
#      ink::types_iterator_range
#      absl::status
#      absl::status_matchers
#      absl::statusor
#      fuzztest::fuzztest
#      ${_FUZZTEST_MAIN}
#  )
#endif()
