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
option(INK_BUILD_TESTS    "Build all C++ unit tests" OFF)
option(INK_BUILD_BENCHMARKS "Build all benchmarks"   OFF)

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
    GIT_TAG        ${tag}
  )
  FetchContent_MakeAvailable(${name})
endfunction()

# ------------------------------------------------------------------ 生产依赖
ink_fetch_git(abseil-cpp
  https://github.com/abseil/abseil-cpp.git
  20250512.0
)

ink_fetch_git(ink_stroke_modeler
  https://github.com/google/ink-stroke-modeler.git
  2cd45e8683025c28fa2efcf672ad46607e8af869   # 2025-08-28
)
FetchContent_GetProperties(ink_stroke_modeler)
if(NOT ink_stroke_modeler_POPULATED)
  FetchContent_Populate(ink_stroke_modeler)
endif()
include_directories(${ink_stroke_modeler_SOURCE_DIR})

#ink_fetch_git(libtess2
#  https://github.com/memononen/libtess2.git
#  9a450cc9e5b4b79c36b89648f8b92fe65b6dd407   # 2025-04-26
#)

# libtess2不支持CMake，换一个别人fork过的
ink_fetch_git(libtess2
  https://github.com/mlogic1/libtess2.git
  3d2fdd08da550daca02f0638bd6fd93618d9d32b   # 2025-04-26
)

# Protobuf 官方仓库
#ink_fetch_git(protobuf
#  https://github.com/protocolbuffers/protobuf.git
#  v31.1
#)

# Skia 巨大，建议本地镜像；这里仍用 FetchContent 示例
#ink_fetch_git(skia
#  https://github.com/google/skia.git
#  fb92a016ef1dd54b027d022e72689497eb5b6c0e   # 2025-09-08
#)

# Skia 用户配置 overlay（等价于 local_repository）
#add_subdirectory(skia_user_config)   # 目录里需包含 CMakeLists.txt


# ------------------------------------------------------------------ 测试/基准依赖（仅当需要）
if(INK_BUILD_TESTS OR INK_BUILD_BENCHMARKS)
  ink_fetch_git(googletest
    https://github.com/google/googletest.git
    v1.17.0
  )
  # 让 GoogleTest 提供 GTest::gtest, GTest::gtest_main, GMock::gmock ...
endif()

if(INK_BUILD_BENCHMARKS)
  ink_fetch_git(google_benchmark
    https://github.com/google/benchmark.git
    v1.8.3
  )
endif()

if(INK_BUILD_TESTS)
  ink_fetch_git(fuzztest
    https://github.com/google/fuzztest.git
    2025-02-14
  )
  # FuzzTest 依赖 riegeli，提前拉取高版本以规避 Abseil 冲突
#  ink_fetch_git(riegeli
#    https://github.com/google/riegeli.git
#    0.0.0-20250706-c4d1f27
#  )
endif()

# ------------------------------------------------------------------ 导出统一目标
# 现在所有依赖都已生成::目标，例如：
#   absl::strings
#   protobuf::protobuf
#   skia::skia
#   GTest::gtest
#   benchmark::benchmark
#   fuzztest::fuzztest
#   fuzztest::gtest_main
#
# 与之前 color/CMakeLists.txt 里假设的名字完全一致，无需改动。

# ------------------------------------------------------------------ 递归子目录

add_subdirectory(ink/types)
add_subdirectory(ink/color)
add_subdirectory(ink/geometry)
add_subdirectory(ink/brush)
add_subdirectory(ink/strokes)
# 后续可以继续 add_subdirectory(stroke) / add_subdirectory(geometry) ...

# ---------------------------------------------------------------------------
#  统一导出目标 —— 对外只暴露 ink::ink
# ---------------------------------------------------------------------------
add_library(ink INTERFACE)

# 1. 把头文件根目录甩出去
target_include_directories(ink INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
)

# 2. 把内部所有子库一次性链接进来
#    下面这些名字全是子目录里已经存在的 ALIAS 或真实目标
target_link_libraries(ink INTERFACE
        ink::color
        ink::color_function
        ink::color_space

        ink::types_duration
        ink::types_internal::copy_on_write
        ink::types_internal::float
        ink::types_iterator_range
        ink::types_numbers
        ink::types_physical_distance
        ink::types_small_array

        ink::easing_function

        ink::geometry_angle
        ink::geometry_vec
        ink::geometry_point
        ink::geometry_segment
        ink::geometry_rect
        ink::geometry_triangle
        ink::geometry_quad
        ink::geometry_envelope
        ink::geometry_affine_transform
        ink::geometry_intersects
        ink::geometry_distance
        ink::geometry_mesh
        ink::geometry_mesh_format
        ink::geometry_mesh_packing_types
        ink::geometry_mutable_mesh
        ink::geometry_partitioned_mesh
        ink::geometry_convex_hull
        ink::geometry_tessellator

        ink::geometry_internal::modulo
        ink::geometry_internal::algorithms
        ink::geometry_internal::intersects_internal
        ink::geometry_internal::mesh_constants
        ink::geometry_internal::mesh_packing
        ink::geometry_internal::static_rtree
        ink::geometry_internal::convex_hull_helper
        ink::geometry_internal::point_tessellation_helper
        ink::geometry_internal::generic_tessellator
        ink::geometry_internal::polyline_processing


        ink::strokes::in_progress_stroke
        ink::strokes::stroke

        ink::strokes::internal::brush_tip_state
        ink::strokes::internal::rounded_polygon
        ink::strokes::internal::stroke_input_modeler
        ink::strokes::internal::stroke_shape_builder
        ink::strokes::internal::stroke_shape_update
        ink::strokes::internal::stroke_vertex
        ink::strokes::internal::constrain_brush_tip_extrusion

        ink::strokes::input::stroke_input
        ink::strokes::input::stroke_input_batch
        ink::strokes::input::internal::stroke_input_validation_helpers
)

# 禁止 re2 安装导出，避免 CMP0002 报错
set(RE2_BUILD_TESTING OFF CACHE BOOL "" FORCE)
set(INSTALL_RE2 OFF CACHE BOOL "" FORCE)