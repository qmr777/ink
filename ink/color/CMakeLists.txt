# ------------------------------------------------------------------------------
#  ink/color/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.16)
project(ink_color LANGUAGES CXX)

# ------------------------------------------------------------------ compiler
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# ------------------------------------------------------------------ options
option(INK_COLOR_BUILD_TESTS "Build unit tests for ink/color" ON)

# ------------------------------------------------------------------ helpers
# 统一开启测试专用库的属性
function(ink_cc_testonly_library name)
  add_library(${name} STATIC ${ARGN})
  target_compile_definitions(${name} PRIVATE COLOR_TESTONLY)
  set_target_properties(${name} PROPERTIES EXCLUDE_FROM_ALL ON)
endfunction()

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
add_library(ink::color_space ALIAS ink_color_space)
target_include_directories(ink_color_space PUBLIC "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>")

# ------------------------------------------------------------------ color
add_library(ink_color STATIC
  color.cc
  color.h
  color_helpers.h
)
target_link_libraries(ink_color
  PUBLIC
    ink::color_space
    absl::log
    absl::str_format
)
add_library(ink::color ALIAS ink_color)
target_include_directories(ink_color PUBLIC "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}>")