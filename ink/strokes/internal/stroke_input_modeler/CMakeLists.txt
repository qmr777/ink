cmake_minimum_required(VERSION 3.20)
project(ink_strokes_internal LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(
        ink_stroke_internal_input_model_impl INTERFACE
#        input_model_impl.h
)
target_link_libraries(ink_stroke_internal_input_model_impl INTERFACE
        ink::strokes::input::stroke_input_batch
        ink::strokes::internal::modeled_stroke_input
)
add_library(ink::strokes::internal::input_model_impl ALIAS ink_stroke_internal_input_model_impl)

add_library(
        ink_stroke_internal_passthrough_input_modeler STATIC
        passthrough_input_modeler.h
        passthrough_input_modeler.cc
)
target_link_libraries(ink_stroke_internal_passthrough_input_modeler PUBLIC
        ink::strokes::internal::input_model_impl
        ink::geometry::distance
        ink::geometry::vec
        ink::strokes::input::stroke_input
        ink::strokes::input::stroke_input_batch
        ink::strokes::internal::modeled_stroke_input
        ink::types::duration
        absl::span
)
add_library(ink::strokes::internal::passthrough_input_modeler ALIAS ink_stroke_internal_passthrough_input_modeler)

add_library(
        ink_stroke_internal_sliding_window_input_modeler STATIC
        sliding_window_input_modeler.h
        sliding_window_input_modeler.cc
)
target_link_libraries(ink_stroke_internal_sliding_window_input_modeler PUBLIC
        ink::strokes::internal::input_model_impl
        ink::geometry::angle
        ink::geometry::distance
        ink::geometry::point
        ink::geometry::vec
        ink::geometry::internal::lerp
        ink::strokes::input::stroke_input
        ink::strokes::input::stroke_input_batch
        ink::strokes::internal::modeled_stroke_input
        ink::types::duration
        absl::span
        absl::status
        absl::check
)
add_library(ink::strokes::internal::sliding_window_input_modeler ALIAS ink_stroke_internal_sliding_window_input_modeler)
