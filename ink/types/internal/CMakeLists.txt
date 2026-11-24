# ------------------------------------------------------------------------------
#  ink_native/ink/types/internal/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_types_internal LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------------------------------------------------------------------ 库定义
## copy_on_write
add_library(ink_types_internal_copy_on_write INTERFACE copy_on_write.h)
target_link_libraries(ink_types_internal_copy_on_write INTERFACE
  absl::nullability
  absl::check
)
add_library(ink::types::internal::copy_on_write ALIAS ink_types_internal_copy_on_write)

## float
add_library(ink_types_internal_float INTERFACE float.h)
# 纯头文件，无依赖
add_library(ink::types::internal::float ALIAS ink_types_internal_float)
