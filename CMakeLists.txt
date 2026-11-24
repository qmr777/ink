# ------------------------------------------------------------------------------
#  ink_native/CMakeLists.txt  —— 等价于 MODULE.bazel 的顶层描述
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink
        VERSION 0.0.1
        LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ------------------------------------------------------------------ 选项

include(FetchContent)

# ------------------------------------------------------------------ 工具链
# 1. LLVM 16 工具链（示例，需本地预装）
#    调用时： cmake -DCMAKE_TOOLCHAIN_FILE=~/llvm_16_toolchain.cmake ...
#    文件内容见官方说明：https://clang.llvm.org/docs/CMake.html
#
# 2. Android NDK 工具链（示例）
#    调用时： cmake -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK_HOME/build/cmake/android.toolchain.cmake
#            -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-30 ...

# ------------------------------------------------------------------ 统一宏：FetchContent 快捷封装
function(ink_fetch_git name repo tag)
    FetchContent_Declare(
            ${name}
            GIT_REPOSITORY ${repo}
            GIT_TAG ${tag}
    )
    FetchContent_MakeAvailable(${name})
endfunction()

# ------------------------------------------------------------------ 生产依赖
ink_fetch_git(abseil-cpp
        https://github.com/abseil/abseil-cpp.git
        20260526.0
)

# 获取 abseil 构建目录，供 protobuf 的 find_package(absl CONFIG) 使用
FetchContent_GetProperties(abseil-cpp)
if (abseil-cpp_POPULATED)
    set(absl_DIR "${abseil-cpp_BINARY_DIR}" CACHE PATH "" FORCE)
endif ()

ink_fetch_git(ink_stroke_modeler
        https://github.com/google/ink-stroke-modeler.git
        e254fa1ea4644070ab760fc5f0f6cc9ac6a4f40c #20260520
)
FetchContent_GetProperties(ink_stroke_modeler)
if (NOT ink_stroke_modeler_POPULATED)
    FetchContent_Populate(ink_stroke_modeler)
endif ()
include_directories(${ink_stroke_modeler_SOURCE_DIR})

# libtess2不支持CMake，换一个别人fork过的
ink_fetch_git(libtess2
        https://github.com/mlogic1/libtess2.git
        3d2fdd08da550daca02f0638bd6fd93618d9d32b   # 2025-04-26
)

# protobuf（storage 模块依赖，需要 abseil 已就绪）
# 使用 protobuf_ABSL_PROVIDER=package 让 protobuf 通过 find_package(absl CONFIG)
# 查找已构建的 abseil，避免重复 add_subdirectory 导致目标冲突。
set(protobuf_ABSL_PROVIDER "package" CACHE STRING "" FORCE)
set(protobuf_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(protobuf_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
set(protobuf_WITH_ZLIB OFF CACHE BOOL "" FORCE)
set(protobuf_INSTALL OFF CACHE BOOL "" FORCE)
FetchContent_Declare(protobuf
        GIT_REPOSITORY https://github.com/protocolbuffers/protobuf.git
        GIT_TAG        v33.5
        GIT_SHALLOW    TRUE
)
FetchContent_MakeAvailable(protobuf)

# ------------------------------------------------------------------ 导出统一目标
# 现在所有依赖都已生成::目标，例如：
#   absl::strings
#   protobuf::protobuf
#   skia::skia
#
# 与之前 color/CMakeLists.txt 里假设的名字完全一致，无需改动。

# ------------------------------------------------------------------ 递归子目录

add_subdirectory(ink/types)
add_subdirectory(ink/color)
add_subdirectory(ink/geometry)
add_subdirectory(ink/brush)
add_subdirectory(ink/strokes)
add_subdirectory(ink/storage)
# 后续可以继续 add_subdirectory(stroke) / add_subdirectory(geometry) ...

# ---------------------------------------------------------------------------
#  统一导出目标 —— 对外只暴露 ink::ink
# ---------------------------------------------------------------------------
add_library(ink INTERFACE)
add_library(ink::ink ALIAS ink)

# 1. 把头文件根目录甩出去
target_include_directories(ink INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
)

# 2. 把内部所有子库一次性链接进来
#    下面这些名字全是子目录里已经存在的 ALIAS 或真实目标
target_link_libraries(ink INTERFACE
        ink::color::color
        ink::brush::color_function
        ink::color::color_space

        ink::types::duration
        ink::types::internal::copy_on_write
        ink::types::internal::float
        ink::types::iterator_range
        ink::types::numbers
        ink::types::physical_distance
        ink::types::small_array

        ink::brush::easing_function
        ink::brush::stock_brushes

        ink::geometry::angle
        ink::geometry::vec
        ink::geometry::point
        ink::geometry::segment
        ink::geometry::rect
        ink::geometry::triangle
        ink::geometry::quad
        ink::geometry::envelope
        ink::geometry::affine_transform
        ink::geometry::intersects
        ink::geometry::distance
        ink::geometry::mesh
        ink::geometry::mesh_format
        ink::geometry::mesh_packing_types
        ink::geometry::mutable_mesh
        ink::geometry::partitioned_mesh
        ink::geometry::convex_hull
        ink::geometry::tessellator

        ink::geometry::internal::modulo
        ink::geometry::internal::algorithms
        ink::geometry::internal::intersects_internal
        ink::geometry::internal::mesh_constants
        ink::geometry::internal::mesh_packing
        ink::geometry::internal::static_rtree
        ink::geometry::internal::convex_hull_helper
        ink::geometry::internal::point_tessellation_helper
        ink::geometry::internal::generic_tessellator
        ink::geometry::internal::polyline_processing
        ink::geometry::internal::outline_triangulation

        ink::strokes::in_progress_stroke
        ink::strokes::stroke
        ink::strokes::stroke_paint_animation_clock

        ink::strokes::internal::brush_tip_state
        ink::strokes::internal::rounded_polygon
        ink::strokes::internal::stroke_input_modeler
        ink::strokes::internal::stroke_shape_builder
        ink::strokes::internal::stroke_shape_update
        ink::strokes::internal::stroke_subtraction
        ink::strokes::internal::mesh_attribute_interpolation
        ink::strokes::internal::stroke_anti_aliasing
        ink::strokes::internal::stroke_vertex
        ink::strokes::internal::constrain_brush_tip_extrusion

        ink::strokes::input::stroke_input
        ink::strokes::input::stroke_input_batch
        ink::strokes::input::internal::stroke_input_validation_helpers

        ink::storage::numeric_run
        ink::storage::color
        ink::storage::mesh_format
        ink::storage::mesh
        ink::storage::mesh_vertices
        ink::storage::partitioned_mesh
        ink::storage::input_batch
        ink::storage::stroke_input_batch
        ink::storage::brush

        ink::storage::proto::options_cc_proto
        ink::storage::proto::color_cc_proto
        ink::storage::proto::coded_numeric_run_cc_proto
        ink::storage::proto::brush_family_cc_proto
        ink::storage::proto::brush_cc_proto
        ink::storage::proto::stroke_input_batch_cc_proto
        ink::storage::proto::incremental_stroke_inputs_cc_proto
        ink::storage::proto::mesh_cc_proto

        # absl
        absl::check
        absl::statusor

        # 显式声明 protobuf 运行时作为兜底，避免下游在自己项目中
        # 显式使用 google::protobuf::... 符号时找不到。
        # 已有 PUBLIC 链会传递，但显式写出更清晰、更不易被 visibility 裁剪。
#        protobuf::libprotobuf
)

# 禁止 re2 安装导出，避免 CMP0002 报错
set(RE2_BUILD_TESTING OFF CACHE BOOL "" FORCE)
set(INSTALL_RE2 OFF CACHE BOOL "" FORCE)