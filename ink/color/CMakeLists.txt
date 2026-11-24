# ------------------------------------------------------------------------------
#  ink/color/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_color LANGUAGES CXX)

# ------------------------------------------------------------------ compiler
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ------------------------------------------------------------------ color_space
add_library(ink_color_space STATIC
        color_space.cc
        color_space.h
)
target_link_libraries(ink_color_space
        PUBLIC
        absl::log
        absl::str_format
)
add_library(ink::color::color_space ALIAS ink_color_space)
target_include_directories(ink_color_space PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>")

# ------------------------------------------------------------------ color
add_library(ink_color STATIC
        color.cc
        color.h
)
target_link_libraries(ink_color
        PUBLIC
        ink::color::color_space
        absl::check
        absl::log
        absl::str_format
)
add_library(ink::color::color ALIAS ink_color)
target_include_directories(ink_color PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/../..>")
