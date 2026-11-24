# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------------------------------------------------------------------ 子目录（顺序敏感）
add_subdirectory(input)   # 提供 stroke_input / stroke_input_batch / ...
add_subdirectory(internal) # 提供 stroke_input_modeler / stroke_shape_builder / ...

# ------------------------------------------------------------------ 主库
## stroke
add_library(ink_strokes_stroke STATIC
        stroke.cc stroke.h
)
target_link_libraries(ink_strokes_stroke
        PUBLIC
        ink::brush::brush
        ink::brush::brush_coat
        ink::brush::brush_family
        ink::color::color
        ink::geometry::affine_transform
        ink::geometry::partitioned_mesh
        ink::strokes::input::stroke_input_batch
        ink::strokes::internal::stroke_input_modeler
        ink::strokes::internal::stroke_segmentation
        ink::strokes::internal::stroke_shape_builder
        ink::strokes::internal::stroke_subtraction
        ink::strokes::internal::stroke_vertex
        ink::types::duration
        absl::algorithm_container
        absl::check
        absl::log
        absl::status
        absl::status_macros
        absl::statusor
        absl::span
)
add_library(ink::strokes::stroke ALIAS ink_strokes_stroke)

## in_progress_stroke
add_library(ink_strokes_in_progress_stroke STATIC
        in_progress_stroke.cc in_progress_stroke.h
)
target_link_libraries(ink_strokes_in_progress_stroke
        PUBLIC
        ink::strokes::stroke
        ink::brush::brush
        ink::brush::brush_coat
        ink::geometry::envelope
        ink::geometry::mesh_format
        ink::geometry::mutable_mesh
        ink::geometry::partitioned_mesh
        ink::strokes::input::stroke_input
        ink::strokes::input::stroke_input_batch
        ink::strokes::input::internal::stroke_input_validation_helpers
        ink::strokes::internal::modeled_stroke_input
        ink::strokes::internal::stroke_input_modeler
        ink::strokes::internal::stroke_shape_builder
        ink::strokes::internal::stroke_shape_update
        ink::strokes::internal::stroke_vertex
        ink::types::duration
        absl::nullability
        absl::flat_hash_set
        absl::inlined_vector
        absl::check
        absl::log
        absl::status
        absl::status_macros
        absl::statusor
        absl::str_format
        absl::span
)
add_library(ink::strokes::in_progress_stroke ALIAS ink_strokes_in_progress_stroke)

## stroke_paint_animation_clock
add_library(ink_strokes_stroke_paint_animation_clock STATIC
        stroke_paint_animation_clock.cc stroke_paint_animation_clock.h
)
target_link_libraries(ink_strokes_stroke_paint_animation_clock
        PUBLIC
        ink::brush::brush_family
        ink::brush::brush_paint
        absl::check
        absl::time
)
add_library(ink::strokes::stroke_paint_animation_clock ALIAS ink_strokes_stroke_paint_animation_clock)
