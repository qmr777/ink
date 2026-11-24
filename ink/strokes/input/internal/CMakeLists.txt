# ------------------------------------------------------------------------------
#  ink_native/ink/strokes/input/internal/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_strokes_input_internal LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------------------------------------------------------------------ 库
add_library(ink_strokes_input_internal_stroke_input_validation_helpers STATIC
  stroke_input_validation_helpers.cc
  stroke_input_validation_helpers.h
)
target_link_libraries(ink_strokes_input_internal_stroke_input_validation_helpers
  PUBLIC
    ink::strokes::input::stroke_input   # 来自上层 input/
    ink::types::duration
    absl::status
    absl::status_macros
    absl::strings
)
add_library(ink::strokes::input::internal::stroke_input_validation_helpers
  ALIAS ink_strokes_input_internal_stroke_input_validation_helpers)
