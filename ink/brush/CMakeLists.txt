cmake_minimum_required(VERSION 3.20)
project(ink_brush LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------------------------- 通用宏 ------------------------------------

# ------------------------- 1. 基础库 -------------------------------
# brush_tip
add_library(ink_brush_tip STATIC
        brush_tip.cc
        brush_tip.h
)
target_link_libraries(ink_brush_tip
        PUBLIC
        ink::brush::brush_behavior
        ink_brush_version
        ink::geometry::angle
        ink::geometry::mesh_format
        ink::geometry::vec
        ink::types::duration
        absl::flat_hash_set
        absl::status
        absl::status_macros
        absl::strings
        absl::str_format
)
add_library(ink::brush::brush_tip ALIAS ink_brush_tip)
target_include_directories(ink_brush_tip PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/../..")

add_library(ink_brush_version INTERFACE)
target_link_libraries(ink_brush_version INTERFACE
        absl::check
        absl::strings
)
target_include_directories(ink_brush_version INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/../..")

# easing_function
add_library(ink_easing_function STATIC
        easing_function.cc
        easing_function.h
)
target_link_libraries(ink_easing_function
        PUBLIC
        ink_brush_version
        ink::geometry::point
        absl::status
        absl::strings
        absl::str_format
)
add_library(ink::brush::easing_function ALIAS ink_easing_function)

# brush_behavior
add_library(ink_brush_behavior STATIC
        brush_behavior.cc
        brush_behavior.h
)
target_link_libraries(ink_brush_behavior
        PUBLIC
        ink::brush::easing_function
        ink_brush_version
        absl::status
        absl::status_macros
        absl::strings
        absl::str_format
)
add_library(ink::brush::brush_behavior ALIAS ink_brush_behavior)

# color_function
add_library(ink_color_function STATIC
        color_function.cc
        color_function.h
)
target_link_libraries(ink_color_function
        PUBLIC
        ink_brush_version
        ink::color::color
        ink::color::color_space
        ink::geometry::affine_transform
        ink::geometry::angle
        ink::geometry::point
        absl::status
        absl::strings
        absl::span
)
add_library(ink::brush::color_function ALIAS ink_color_function)

# brush_paint
add_library(ink_brush_paint STATIC
        brush_paint.cc
        brush_paint.h
)
target_link_libraries(ink_brush_paint
        PUBLIC
        ink::brush::color_function
        ink_brush_version
        ink::geometry::angle
        ink::geometry::mesh_format
        ink::geometry::vec
        absl::algorithm_container
        absl::flat_hash_set
        absl::overload
        absl::status
        absl::status_macros
        absl::strings
        absl::str_format
        absl::time
)
add_library(ink::brush::brush_paint ALIAS ink_brush_paint)

# brush_coat
add_library(ink_brush_coat STATIC
        brush_coat.cc
        brush_coat.h
)
target_link_libraries(ink_brush_coat
        PUBLIC
        ink::brush::brush_paint
        ink::brush::brush_tip
        ink_brush_version
        ink::geometry::mesh_format
        absl::flat_hash_set
        absl::inlined_vector
        absl::status
        absl::status_macros
        absl::strings
        absl::str_format
        absl::span
)
add_library(ink::brush::brush_coat ALIAS ink_brush_coat)

# brush_family
add_library(ink_brush_family STATIC
        brush_family.cc
        brush_family.h
)
target_link_libraries(ink_brush_family
        PUBLIC
        ink::brush::brush_coat
        ink::brush::brush_paint
        ink::brush::brush_tip
        ink_brush_version
        ink::types::duration
        absl::status
        absl::status_macros
        absl::statusor
        absl::strings
        absl::time
        absl::span
)
add_library(ink::brush::brush_family ALIAS ink_brush_family)

# stock_brushes
add_library(ink_stock_brushes STATIC
        stock_brushes.cc
        stock_brushes.h
)
target_link_libraries(ink_stock_brushes
        PUBLIC
        ink::brush::brush_behavior
        ink::brush::brush_coat
        ink::brush::brush_family
        ink::brush::brush_tip
        ink::brush::easing_function
        ink::geometry::angle
        absl::check
        absl::statusor
        absl::string_view
)
add_library(ink::brush::stock_brushes ALIAS ink_stock_brushes)

# brush（最外层聚合库）
add_library(ink_brush STATIC
        brush.cc
        brush.h
)
target_link_libraries(ink_brush
        PUBLIC
        ink::brush::brush_coat
        ink::brush::brush_family
        ink::color::color
        absl::status
        absl::status_macros
        absl::statusor
        absl::strings
        absl::span
)
add_library(ink::brush::brush ALIAS ink_brush)

add_library(ink_brush_family_internal_accessor INTERFACE
        internal/brush_family_internal_accessor.h)
add_library(ink::brush::brush_family_internal_accessor ALIAS ink_brush_family_internal_accessor)
