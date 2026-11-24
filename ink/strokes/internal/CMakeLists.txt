# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/internal/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes_internal LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#------------------------------------------------------------------- 子目录
add_subdirectory(brush_tip_extruder)
add_subdirectory(stroke_input_modeler)

# ------------------------------------------------------------------ 辅助宏

# ------------------------------------------------------------------ 基础工具库
## mutable_multi_mesh
add_library(ink_strokes_internal_mutable_multi_mesh STATIC
        mutable_multi_mesh.cc mutable_multi_mesh.h
)
target_link_libraries(ink_strokes_internal_mutable_multi_mesh
        PUBLIC
        ink::geometry::mesh_format
        ink::geometry::mesh_index_types
        ink::geometry::mutable_mesh
        ink::geometry::point
        ink::geometry::triangle
        ink::types::small_array
        absl::inlined_vector
        absl::check
        absl::span
)
add_library(ink::strokes::internal::mutable_multi_mesh ALIAS ink_strokes_internal_mutable_multi_mesh)

## stroke_shape_update
add_library(ink_strokes_internal_stroke_shape_update STATIC
        stroke_shape_update.cc stroke_shape_update.h
)
target_link_libraries(ink_strokes_internal_stroke_shape_update
        PUBLIC
        ink::geometry::envelope
)
add_library(ink::strokes::internal::stroke_shape_update ALIAS ink_strokes_internal_stroke_shape_update)

add_library(ink_strokes_internal_modeled_stroke_input STATIC
        modeled_stroke_input.cc modeled_stroke_input.h
)
target_link_libraries(
        ink_strokes_internal_modeled_stroke_input PUBLIC
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::vec
        ink::geometry::internal::lerp
        ink::strokes::input::stroke_input
        ink::types::duration
        ink::types::physical_distance
)
add_library(ink::strokes::internal::modeled_stroke_input ALIAS ink_strokes_internal_modeled_stroke_input)

## stroke_input_modeler
add_library(ink_strokes_internal_stroke_input_modeler STATIC
        stroke_input_modeler.cc
        stroke_input_modeler.h
)
target_link_libraries(ink_strokes_internal_stroke_input_modeler
        PUBLIC
        ink::strokes::internal::modeled_stroke_input
        ink::brush::brush_family
        ink::strokes::input::stroke_input_batch
        ink::strokes::internal::input_model_impl
        ink::strokes::internal::passthrough_input_modeler
        ink::strokes::internal::sliding_window_input_modeler
        ink::types::duration
        absl::nullability
        absl::check
        absl::span
)
target_include_directories(ink_strokes_internal_stroke_input_modeler
        PUBLIC
        ${ink_stroke_modeler_SOURCE_DIR}
)
add_library(ink::strokes::internal::stroke_input_modeler ALIAS ink_strokes_internal_stroke_input_modeler)

## brush_tip_state
add_library(ink_strokes_internal_brush_tip_state STATIC
        brush_tip_state.cc brush_tip_state.h
)
target_link_libraries(ink_strokes_internal_brush_tip_state
        PUBLIC
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::internal::lerp
)
add_library(ink::strokes::internal::brush_tip_state ALIAS ink_strokes_internal_brush_tip_state)

## brush_tip_modeler_helpers
add_library(ink_strokes_internal_brush_tip_modeler_helpers STATIC
        brush_tip_modeler_helpers.cc brush_tip_modeler_helpers.h
)
target_link_libraries(ink_strokes_internal_brush_tip_modeler_helpers
        PUBLIC
        ink_strokes_internal_brush_tip_state
        ink_strokes_internal_easing_implementation
        ink_strokes_internal_noise_generator
        ink_strokes_internal_modeled_stroke_input
        ink::brush::brush_behavior
        ink::brush::brush_tip
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::vec
        ink::geometry::internal::lerp
        ink::geometry::internal::modulo
        ink::strokes::input::stroke_input
        ink::types::duration
        ink::types::physical_distance
        absl::check
        absl::span
)
add_library(ink::strokes::internal::brush_tip_modeler_helpers ALIAS ink_strokes_internal_brush_tip_modeler_helpers)

## brush_tip_modeler
add_library(ink_strokes_internal_brush_tip_modeler STATIC
        brush_tip_modeler.cc brush_tip_modeler.h
)
target_link_libraries(ink_strokes_internal_brush_tip_modeler
        PUBLIC
        ink_strokes_internal_brush_tip_modeler_helpers
        ink_strokes_internal_brush_tip_state
        ink_strokes_internal_easing_implementation
        ink_strokes_internal_modeled_stroke_input
        ink_strokes_internal_noise_generator
        ink::brush::brush_behavior
        ink::brush::brush_tip
        ink::geometry::angle
        ink::geometry::point
        ink::types::duration
        absl::algorithm_container
        absl::nullability
        absl::check
        absl::log
        absl::span
)
add_library(ink::strokes::internal::brush_tip_modeler ALIAS ink_strokes_internal_brush_tip_modeler)

## stroke_outline
add_library(ink_strokes_internal_stroke_outline STATIC
        stroke_outline.cc stroke_outline.h
)
target_link_libraries(ink_strokes_internal_stroke_outline
        PUBLIC
        absl::algorithm_container
        absl::nullability
        absl::check
        absl::numeric
        absl::span
)
add_library(ink::strokes::internal::stroke_outline ALIAS ink_strokes_internal_stroke_outline)
target_include_directories(ink_strokes_internal_stroke_outline PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/../../..")

## extrusion_points
add_library(ink_strokes_internal_extrusion_points INTERFACE extrusion_points.h)
target_link_libraries(ink_strokes_internal_extrusion_points INTERFACE
        ink::geometry::point
)
add_library(ink::strokes::internal::extrusion_points ALIAS ink_strokes_internal_extrusion_points)

## legacy_vertex
add_library(ink_strokes_internal_legacy_vertex INTERFACE legacy_vertex.h)
target_link_libraries(ink_strokes_internal_legacy_vertex INTERFACE
        ink::color::color
        ink::geometry::point
)
add_library(ink::strokes::internal::legacy_vertex ALIAS ink_strokes_internal_legacy_vertex)

## rounded_polygon
add_library(ink_strokes_internal_rounded_polygon STATIC
        rounded_polygon.cc rounded_polygon.h
)
target_link_libraries(ink_strokes_internal_rounded_polygon
        PUBLIC
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::segment
        ink::geometry::vec
        ink::geometry::internal::circle
        absl::inlined_vector
        absl::check
        absl::span
)
add_library(ink::strokes::internal::rounded_polygon ALIAS ink_strokes_internal_rounded_polygon)

## constrain_brush_tip_extrusion
add_library(ink_strokes_internal_constrain_brush_tip_extrusion STATIC
        constrain_brush_tip_extrusion.cc constrain_brush_tip_extrusion.h
)
target_link_libraries(ink_strokes_internal_constrain_brush_tip_extrusion
        PUBLIC
        ink_strokes_internal_brush_tip_extrusion
        ink_strokes_internal_brush_tip_state
        ink::geometry::distance
        ink::geometry::internal::lerp
        absl::check
)
add_library(ink::strokes::internal::constrain_brush_tip_extrusion ALIAS ink_strokes_internal_constrain_brush_tip_extrusion)

# ------------------------------------------------------------------ 模型与状态库

## easing_implementation
add_library(ink_strokes_internal_easing_implementation STATIC
        easing_implementation.cc easing_implementation.h
)
target_link_libraries(ink_strokes_internal_easing_implementation
        PUBLIC
        ink::brush::easing_function
        ink::geometry::point
        ink::geometry::vec
        ink::geometry::internal::lerp
        absl::inlined_vector
        absl::overload
        absl::check
        absl::log
        absl::strings
)
add_library(ink::strokes::internal::easing_implementation ALIAS ink_strokes_internal_easing_implementation)

## noise_generator
add_library(ink_strokes_internal_noise_generator STATIC
        noise_generator.cc noise_generator.h
)
target_link_libraries(ink_strokes_internal_noise_generator
        PUBLIC
        ink::geometry::internal::lerp
        absl::check
)
add_library(ink::strokes::internal::noise_generator ALIAS ink_strokes_internal_noise_generator)

## brush_tip_shape
add_library(ink_strokes_internal_brush_tip_shape STATIC
        brush_tip_shape.cc brush_tip_shape.h
)
target_link_libraries(ink_strokes_internal_brush_tip_shape
        PUBLIC
        ink_strokes_internal_brush_tip_state
        ink_strokes_internal_circular_extrusion_helpers
        ink_strokes_internal_extrusion_points
        ink::geometry::affine_transform
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::rect
        ink::geometry::segment
        ink::geometry::vec
        ink::geometry::internal::algorithms
        ink::geometry::internal::circle
        ink::types::small_array
        absl::inlined_vector
        absl::check
        absl::span
)
add_library(ink::strokes::internal::brush_tip_shape ALIAS ink_strokes_internal_brush_tip_shape)

## brush_tip_extrusion
add_library(ink_strokes_internal_brush_tip_extrusion STATIC
        brush_tip_extrusion.cc brush_tip_extrusion.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extrusion
        PUBLIC
        ink_strokes_internal_brush_tip_shape
        ink_strokes_internal_brush_tip_state
        ink_strokes_internal_rounded_polygon
        ink::geometry::distance
        ink::geometry::rect
        ink::geometry::internal::circle
        absl::inlined_vector
        absl::check
        absl::span
)
add_library(ink::strokes::internal::brush_tip_extrusion ALIAS ink_strokes_internal_brush_tip_extrusion)

## brush_tip_extruder
add_library(ink_strokes_internal_brush_tip_extruder STATIC
        brush_tip_extruder.cc brush_tip_extruder.h
)
target_link_libraries(ink_strokes_internal_brush_tip_extruder
        PUBLIC
        ink_strokes_internal_brush_tip_extrusion
        ink_strokes_internal_brush_tip_shape
        ink_strokes_internal_brush_tip_state
        ink_strokes_internal_constrain_brush_tip_extrusion
        ink_strokes_internal_stroke_outline
        ink_strokes_internal_stroke_shape_update
        ink_strokes_internal_stroke_vertex
        ink::geometry::affine_transform
        ink::geometry::envelope
        ink::geometry::mutable_mesh
        ink::geometry::point
        ink_strokes_internal_brush_tip_extruder_geometry
        ink_strokes_internal_brush_tip_extruder_mutable_mesh_view
        ink_strokes_internal_brush_tip_extruder_side
        absl::algorithm_container
        absl::inlined_vector
        absl::check
        absl::span
)
add_library(ink::strokes::internal::brush_tip_extruder ALIAS ink_strokes_internal_brush_tip_extruder)

## stroke_vertex
add_library(ink_strokes_internal_stroke_vertex STATIC
        stroke_vertex.cc stroke_vertex.h
)
target_link_libraries(ink_strokes_internal_stroke_vertex
        PUBLIC
        ink::geometry::mesh_format
        ink::geometry::mesh_packing_types
        ink::geometry::mutable_mesh
        ink::geometry::point
        ink::geometry::vec
        ink::geometry::internal::lerp
        ink::types::small_array
        absl::algorithm_container
        absl::check
        absl::span
)
add_library(ink::strokes::internal::stroke_vertex ALIAS ink_strokes_internal_stroke_vertex)

## stroke_shape_builder
add_library(ink_strokes_internal_stroke_shape_builder STATIC
        stroke_shape_builder.cc stroke_shape_builder.h
)
target_link_libraries(ink_strokes_internal_stroke_shape_builder
        PUBLIC
        ink_strokes_internal_brush_tip_extruder
        ink_strokes_internal_brush_tip_modeler
        ink_strokes_internal_modeled_stroke_input
        ink_strokes_internal_stroke_input_modeler
        ink_strokes_internal_stroke_outline
        ink_strokes_internal_stroke_shape_update
        ink_strokes_internal_stroke_vertex
        ink::brush::brush_coat
        ink::brush::brush_family
        ink::geometry::envelope
        ink::geometry::mutable_mesh
        ink::types::duration
        absl::inlined_vector
        absl::span
)
add_library(ink::strokes::internal::stroke_shape_builder ALIAS ink_strokes_internal_stroke_shape_builder)

## circular_extrusion_helpers
add_library(ink_strokes_internal_circular_extrusion_helpers STATIC
        circular_extrusion_helpers.cc circular_extrusion_helpers.h
)
target_link_libraries(ink_strokes_internal_circular_extrusion_helpers
        PUBLIC
        ink::geometry::angle
        ink::geometry::point
        ink::geometry::segment
        ink::geometry::vec
        ink::geometry::internal::algorithms
        ink::geometry::internal::circle
        absl::check
)
add_library(ink::strokes::internal::circular_extrusion_helpers ALIAS ink_strokes_internal_circular_extrusion_helpers)

## mesh_attribute_interpolation
add_library(ink_strokes_internal_mesh_attribute_interpolation STATIC
        mesh_attribute_interpolation.cc mesh_attribute_interpolation.h
)
target_link_libraries(ink_strokes_internal_mesh_attribute_interpolation
        PUBLIC
        ink::geometry::mesh
        ink::geometry::mesh_format
        ink::geometry::mutable_mesh
        ink::types::numbers
        ink::types::small_array
        absl::inlined_vector
        absl::check
        absl::span
)
add_library(ink::strokes::internal::mesh_attribute_interpolation ALIAS ink_strokes_internal_mesh_attribute_interpolation)

## stroke_anti_aliasing
add_library(ink_strokes_internal_stroke_anti_aliasing STATIC
        stroke_anti_aliasing.cc stroke_anti_aliasing.h
)
target_link_libraries(ink_strokes_internal_stroke_anti_aliasing
        PUBLIC
        ink_strokes_internal_mesh_attribute_interpolation
        ink_strokes_internal_stroke_vertex
        ink::geometry::mesh_format
        ink::geometry::mutable_mesh
        ink::geometry::point
        ink::geometry::segment
        ink::geometry::triangle
        ink::geometry::vec
        ink::strokes::internal::brush_tip_extruder::derivative_calculator
        absl::algorithm_container
        absl::flat_hash_map
        absl::inlined_vector
        absl::check
        absl::span
)
add_library(ink::strokes::internal::stroke_anti_aliasing ALIAS ink_strokes_internal_stroke_anti_aliasing)

add_library(ink_stroke_internal_stroke_subtraction STATIC
        stroke_subtraction.cc stroke_subtraction.h
)
target_link_libraries(ink_stroke_internal_stroke_subtraction
        PUBLIC
        ink_strokes_internal_mesh_attribute_interpolation
        ink_strokes_internal_stroke_anti_aliasing
        ink_strokes_internal_stroke_vertex
        ink::geometry::affine_transform
        ink::geometry::envelope
        ink::geometry::mesh
        ink::geometry::mesh_format
        ink::geometry::mutable_mesh
        ink::geometry::partitioned_mesh
        ink::geometry::point
        ink::geometry::triangle
        ink::geometry::vec
        ink::geometry::internal::outline_processing
        ink::geometry::internal::outline_triangulation
        absl::flat_hash_map
        absl::flat_hash_set
        absl::inlined_vector
        absl::check
        absl::status
        absl::statusor
        absl::span
)
add_library(ink::strokes::internal::stroke_subtraction ALIAS ink_stroke_internal_stroke_subtraction)

add_library(ink_stroke_internal_stroke_segmentation STATIC
        stroke_segmentation.cc stroke_segmentation.h
)
target_link_libraries(ink_stroke_internal_stroke_segmentation
        PUBLIC
        ink::strokes::internal::stroke_vertex
        ink::geometry::affine_transform
        ink::geometry::mesh
        ink::geometry::mesh_format
        ink::geometry::mutable_mesh
        ink::geometry::partitioned_mesh
        ink::geometry::point
        absl::algorithm_container
        absl::status
        absl::statusor
        absl::span
)
add_library(ink::strokes::internal::stroke_segmentation ALIAS ink_stroke_internal_stroke_segmentation)
