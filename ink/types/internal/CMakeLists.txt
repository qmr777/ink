# ------------------------------------------------------------------------------
#  ink_native/ink/types/internal/CMakeLists.txt
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)
project(ink_types_internal LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS ON)
endif()

# ------------------------------------------------------------------ 库定义
## copy_on_write
add_library(ink_types_internal_copy_on_write INTERFACE)
target_link_libraries(ink_types_internal_copy_on_write INTERFACE

  absl::check
)
add_library(ink::types_internal::copy_on_write ALIAS ink_types_internal_copy_on_write)

## float
add_library(ink_types_internal_float INTERFACE)
# 纯头文件，无依赖
add_library(ink::types_internal::float ALIAS ink_types_internal_float)

# ------------------------------------------------------------------ 测试
#if(INK_BUILD_TESTS)
#  enable_testing()
#  include(GoogleTest)
#
#  add_executable(ink_types_internal_copy_on_write_test copy_on_write_test.cc)
#  target_link_libraries(ink_types_internal_copy_on_write_test
#    PRIVATE
#      ink::types_internal::copy_on_write
#      GTest::gtest_main
#  )
#  gtest_discover_tests(ink_types_internal_copy_on_write_test)
#endif()