cmake_minimum_required(VERSION 3.20)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ------------------------- 测试开关 -----------------------------------
# 沿用顶层选项
if(NOT DEFINED INK_BUILD_TESTS)
  set(INK_BUILD_TESTS ON)
endif()

# ------------------------- 通用宏 ------------------------------------
# 测试专用库（testonly = 1 等价）
function(ink_testonly_lib name)
  add_library(${name} STATIC ${ARGN})
  target_compile_definitions(${name} PRIVATE BRUSH_TESTONLY)
  set_target_properties(${name} PROPERTIES EXCLUDE_FROM_ALL ON)
endfunction()

# ------------------------- 1. 基础库 -------------------------------
# brush_tip
add_library(ink_brush_tip STATIC
  brush_tip.cc
  brush_tip.h
)
target_link_libraries(ink_brush_tip
  PUBLIC
    ink::brush_behavior
    ink::geometry_angle
    ink::geometry_mesh_format
    ink::geometry_vec
    ink::types_duration
    absl::core_headers
    absl::flat_hash_set
    absl::status
    absl::strings
    absl::str_format
)
add_library(ink::brush_tip ALIAS ink_brush_tip)
target_include_directories(ink_brush_tip PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/../..")

# easing_function
add_library(ink_easing_function STATIC
  easing_function.cc
  easing_function.h
)
target_link_libraries(ink_easing_function
  PUBLIC
    ink::geometry_point
    absl::status
    absl::strings
    absl::str_format
)
add_library(ink::easing_function ALIAS ink_easing_function)

# brush_behavior
add_library(ink_brush_behavior STATIC
  brush_behavior.cc
  brush_behavior.h
)
target_link_libraries(ink_brush_behavior
  PUBLIC
    ink::easing_function
    absl::status
    absl::strings
    absl::str_format
)
add_library(ink::brush_behavior ALIAS ink_brush_behavior)

# color_function
add_library(ink_color_function STATIC
  color_function.cc
  color_function.h
)
target_link_libraries(ink_color_function
  PUBLIC
    ink::color   # 来自 ink/color/
    absl::status
    absl::strings
    absl::span
)
add_library(ink::color_function ALIAS ink_color_function)

# brush_paint
add_library(ink_brush_paint STATIC
  brush_paint.cc
  brush_paint.h
)
target_link_libraries(ink_brush_paint
  PUBLIC
    ink::color_function
    ink::geometry_angle
    ink::geometry_mesh_format
    ink::geometry_vec
    absl::flat_hash_set
    absl::status
    absl::strings
    absl::str_format
    absl::time
)
add_library(ink::brush_paint ALIAS ink_brush_paint)

# brush_coat
add_library(ink_brush_coat STATIC
  brush_coat.cc
  brush_coat.h
)
target_link_libraries(ink_brush_coat
  PUBLIC
    ink::brush_behavior
    ink::brush_paint
    ink::brush_tip
    ink::geometry_mesh_format
    absl::flat_hash_set
    absl::inlined_vector
    absl::status
    absl::strings
    absl::str_format
)
add_library(ink::brush_coat ALIAS ink_brush_coat)

# brush_family
add_library(ink_brush_family STATIC
  brush_family.cc
  brush_family.h
)
target_link_libraries(ink_brush_family
  PUBLIC
    ink::brush_coat
    ink::brush_paint
    ink::brush_tip
    ink::types_duration
    absl::status
    absl::statusor
    absl::strings
    absl::span
)
add_library(ink::brush_family ALIAS ink_brush_family)

# brush（最外层聚合库）
add_library(ink_brush STATIC
  brush.cc
  brush.h
)
target_link_libraries(ink_brush
  PUBLIC
    ink::brush_coat
    ink::brush_family
    ink::color
    absl::status
    absl::statusor
    absl::strings
    absl::span
)
add_library(ink::brush ALIAS ink_brush)

# ------------------------- 2. 测试专用库 ------------------------------
#if(INK_BUILD_TESTS)
#  # type_matchers
#  ink_testonly_lib(type_matchers
#    type_matchers.cc
#    type_matchers.h
#  )
#  target_link_libraries(type_matchers
#    PUBLIC
#      ink::brush
#      ink::brush_behavior
#      ink::brush_coat
#      ink::brush_family
#      ink::brush_paint
#      ink::brush_tip
#      ink::color_function
#      ink::easing_function
#      ink::geometry_type_matchers
#      ink::types_type_matchers
#      GTest::gtest
#      absl::strings
#  )
#
#  # fuzz_domains
#  ink_testonly_lib(fuzz_domains
#    fuzz_domains.cc
#    fuzz_domains.h
#  )
#  target_link_libraries(fuzz_domains
#    PUBLIC
#      ink::brush
#      ink::brush_behavior
#      ink::brush_coat
#      ink::brush_family
#      ink::brush_paint
#      ink::brush_tip
#      ink::color_function
#      ink::easing_function
#      ink::color_fuzz_domains
#      ink::geometry_fuzz_domains
#      ink::types_fuzz_domains
#      fuzztest::fuzztest
#      absl::inlined_vector
#      absl::statusor
#      absl::time
#      absl::span
#  )
#
#  # ----------------------- 3. 单元测试 ------------------------------
#  enable_testing()
#  include(GoogleTest)
#
#  # 统一用 fuzztest 提供的 gtest_main
#  set(_test_main fuzztest::gtest_main)
#
#  macro(ink_cc_test name)
#    add_executable(${name} ${ARGN})
#    target_link_libraries(${name} PRIVATE ${_test_main})
#    gtest_discover_tests(${name})
#  endmacro()
#
#  ink_cc_test(brush_test brush_test.cc)
#  target_link_libraries(brush_test
#    PRIVATE
#      ink::brush
#      ink::brush_behavior
#      ink::brush_family
#      ink::brush_paint
#      ink::brush_tip
#      ink::easing_function
#      type_matchers
#      ink::color
#      ink::geometry_angle
#      absl::log absl::status absl::statusor absl::strings
#  )
#
#  ink_cc_test(brush_coat_test brush_coat_test.cc)
#  target_link_libraries(brush_coat_test
#    PRIVATE
#      ink::brush_coat
#      ink::brush_behavior
#      ink::brush_paint
#      ink::brush_tip
#      fuzz_domains
#      ink::geometry_mesh_format
#      absl::flat_hash_set absl::status absl::strings
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(brush_family_test brush_family_test.cc)
#  target_link_libraries(brush_family_test
#    PRIVATE
#      ink::brush_family
#      ink::brush_coat
#      ink::brush_paint
#      ink::brush_tip
#      ink::brush_behavior
#      ink::easing_function
#      fuzz_domains
#      type_matchers
#      ink::geometry_angle
#      ink::geometry_point
#      ink::geometry_vec
#      ink::types_duration
#      absl::status absl::statusor absl::strings absl::span
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(brush_tip_test brush_tip_test.cc)
#  target_link_libraries(brush_tip_test
#    PRIVATE
#      ink::brush_tip
#      ink::brush_behavior
#      ink::geometry_angle
#      ink::types_duration
#      absl::strings
#  )
#
#  ink_cc_test(color_function_test color_function_test.cc)
#  target_link_libraries(color_function_test
#    PRIVATE
#      ink::color_function
#      ink::color
#      fuzz_domains
#      absl::hash_testing
#      absl::status absl::status_matchers absl::strings
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(easing_function_test easing_function_test.cc)
#  target_link_libraries(easing_function_test
#    PRIVATE
#      ink::easing_function
#      fuzz_domains
#      absl::status absl::strings
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(brush_behavior_test brush_behavior_test.cc)
#  target_link_libraries(brush_behavior_test
#    PRIVATE
#      ink::brush_behavior
#      ink::easing_function
#      fuzz_domains
#      ink::types_duration
#      absl::status absl::status_matchers absl::strings
#  )
#
#  ink_cc_test(brush_paint_test brush_paint_test.cc)
#  target_link_libraries(brush_paint_test
#    PRIVATE
#      ink::brush_paint
#      ink::color_function
#      fuzz_domains
#      ink::color
#      ink::geometry_angle
#      ink::geometry_vec
#      absl::hash_testing
#      absl::status absl::status_matchers absl::strings absl::time
#      fuzztest::fuzztest
#  )
#
#  ink_cc_test(fuzz_domains_test fuzz_domains_test.cc)
#  target_link_libraries(fuzz_domains_test
#    PRIVATE
#      ink::brush
#      ink::brush_family
#      fuzz_domains
#      fuzztest::fuzztest
#  )
#endif()