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

# 必须先于下方：proto 库会生成并暴露 ink::storage::proto::*_cc_proto
add_subdirectory(proto)

# ------------------------------------------------------------------ 辅助宏

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
        ink::types::iterator_range
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
        ink::color::color
        ink::color::color_space
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
        ink::geometry::mesh_format
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
        ink::geometry::internal::mesh_packing
        ink::geometry::mesh
        ink::geometry::mesh_format
        ink::geometry::mesh_packing_types
        ink::geometry::point
        ink::storage::mesh_format
        ink::storage::numeric_run
        ink::storage::proto::coded_numeric_run_cc_proto
        ink::storage::proto::mesh_cc_proto
        ink::types::iterator_range
        ink::types::small_array
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
        ink::types::iterator_range
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
        ink::geometry::mesh
        ink::geometry::mesh_format
        ink::geometry::mesh_index_types
        ink::geometry::mutable_mesh
        ink::geometry::partitioned_mesh
        ink::storage::mesh
        ink::storage::mesh_format
        ink::storage::numeric_run
        ink::storage::proto::coded_numeric_run_cc_proto
        ink::storage::proto::mesh_cc_proto
        ink::strokes::internal::stroke_anti_aliasing
        ink::strokes::internal::stroke_vertex
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
        ink::geometry::angle
        ink::storage::numeric_run
        ink::storage::proto::stroke_input_batch_cc_proto
        ink::strokes::input::stroke_input
        ink::types::duration
        ink::types::iterator_range
        ink::types::physical_distance
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
        ink::geometry::angle
        ink::geometry::rect
        ink::storage::input_batch
        ink::storage::proto::coded_numeric_run_cc_proto
        ink::storage::proto::stroke_input_batch_cc_proto
        ink::strokes::input::stroke_input
        ink::strokes::input::stroke_input_batch
        ink::types::duration
        ink::types::iterator_range
        ink::types::numbers
        ink::types::physical_distance
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
        ink::brush::brush
        ink::brush::brush_behavior
        ink::brush::brush_coat
        ink::brush::brush_family
        ink::brush::brush_family_internal_accessor
        ink::brush::brush_paint
        ink::brush::brush_tip
        ink::brush::color_function
        ink::brush::easing_function
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::vec
        ink::storage::color
        ink::storage::proto::brush_cc_proto
        ink::storage::proto::brush_family_cc_proto
        ink::storage::proto::options_cc_proto
        ink::storage::proto::stroke_input_batch_cc_proto
        ink::types::duration
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
