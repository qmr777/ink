# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/input/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes_input LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------------------------------------------------------------------ 子目录（顺序敏感）
add_subdirectory(internal)   # 提供 stroke_input_validation_helpers

# ------------------------------------------------------------------ 辅助宏

# ------------------------------------------------------------------ 基础库
## stroke_input
add_library(ink_strokes_input_stroke_input STATIC
  stroke_input.cc stroke_input.h
)
target_link_libraries(ink_strokes_input_stroke_input
  PUBLIC
    ink::geometry::angle
    ink::geometry::point
    ink::types::duration
    ink::types::physical_distance
    ink_brush_version
    absl::strings
)
add_library(ink::strokes::input::stroke_input ALIAS ink_strokes_input_stroke_input)

## stroke_input_batch
add_library(ink_strokes_input_stroke_input_batch STATIC
  stroke_input_batch.cc stroke_input_batch.h
)
target_link_libraries(ink_strokes_input_stroke_input_batch
  PUBLIC
    ink::strokes::input::stroke_input
    ink::geometry::affine_transform
    ink::geometry::angle
    ink::geometry::point
    ink::strokes::input::internal::stroke_input_validation_helpers
    ink::types::duration
    ink::types::physical_distance
    ink::types::internal::copy_on_write
    absl::check
    absl::status
    absl::statusor
    absl::strings
    absl::span
)
add_library(ink::strokes::input::stroke_input_batch ALIAS ink_strokes_input_stroke_input_batch)
