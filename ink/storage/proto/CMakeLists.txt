# ------------------------------------------------------------------------------
#  ink/storage/proto/CMakeLists.txt
#  Proto 库 —— 构建时通过 protoc 生成 .pb.cc / .pb.h
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)

# protobuf 由顶层 CMakeLists.txt 通过 FetchContent_MakeAvailable 引入，
# 目标 protobuf::libprotobuf 已全局可用。

# ------------------------------------------------------------------ protoc 查找：优先使用 PATH 中的，再回退到项目自带
find_program(PROTOC protoc)
if (NOT PROTOC)
    set(PROTOC "${CMAKE_SOURCE_DIR}/tools/protoc-33.5-win64/bin/protoc.exe")
    set(PROTOC_INCLUDE "${CMAKE_SOURCE_DIR}/tools/protoc-33.5-win64/include")
    message(STATUS "protoc not found in PATH, using bundled: ${PROTOC}")
else ()
    # 使用系统 protoc，include 目录由 protoc --proto_path 自行管理
    get_filename_component(PROTOC_DIR "${PROTOC}" DIRECTORY)
    set(PROTOC_INCLUDE "${PROTOC_DIR}/../include")
    message(STATUS "Using system protoc: ${PROTOC}")
endif ()

# 所有 .proto 文件（顺序无关，protoc 能自动处理依赖）
set(PROTO_FILES
        "${CMAKE_CURRENT_SOURCE_DIR}/options.proto"
        "${CMAKE_CURRENT_SOURCE_DIR}/color.proto"
        "${CMAKE_CURRENT_SOURCE_DIR}/coded_numeric_run.proto"
        "${CMAKE_CURRENT_SOURCE_DIR}/brush_family.proto"
        "${CMAKE_CURRENT_SOURCE_DIR}/brush.proto"
        "${CMAKE_CURRENT_SOURCE_DIR}/stroke_input_batch.proto"
        "${CMAKE_CURRENT_SOURCE_DIR}/incremental_stroke_inputs.proto"
        "${CMAKE_CURRENT_SOURCE_DIR}/mesh.proto"
)

# 生成的 .pb.cc / .pb.h 输出到构建根目录，保持 ink/storage/proto/ 路径结构
set(PROTO_OUT_DIR "${CMAKE_BINARY_DIR}")

set(PROTO_GEN_SOURCES
        "${PROTO_OUT_DIR}/ink/storage/proto/options.pb.cc"
        "${PROTO_OUT_DIR}/ink/storage/proto/color.pb.cc"
        "${PROTO_OUT_DIR}/ink/storage/proto/coded_numeric_run.pb.cc"
        "${PROTO_OUT_DIR}/ink/storage/proto/brush_family.pb.cc"
        "${PROTO_OUT_DIR}/ink/storage/proto/brush.pb.cc"
        "${PROTO_OUT_DIR}/ink/storage/proto/stroke_input_batch.pb.cc"
        "${PROTO_OUT_DIR}/ink/storage/proto/incremental_stroke_inputs.pb.cc"
        "${PROTO_OUT_DIR}/ink/storage/proto/mesh.pb.cc"
)
set(PROTO_GEN_HEADERS
        "${PROTO_OUT_DIR}/ink/storage/proto/options.pb.h"
        "${PROTO_OUT_DIR}/ink/storage/proto/color.pb.h"
        "${PROTO_OUT_DIR}/ink/storage/proto/coded_numeric_run.pb.h"
        "${PROTO_OUT_DIR}/ink/storage/proto/brush_family.pb.h"
        "${PROTO_OUT_DIR}/ink/storage/proto/brush.pb.h"
        "${PROTO_OUT_DIR}/ink/storage/proto/stroke_input_batch.pb.h"
        "${PROTO_OUT_DIR}/ink/storage/proto/incremental_stroke_inputs.pb.h"
        "${PROTO_OUT_DIR}/ink/storage/proto/mesh.pb.h"
)

# 构建时执行 protoc，生成所有 .pb.cc + .pb.h
# 两个列表都加入 OUTPUT 以标记为 GENERATED，避免 CMake 配置时检查源文件是否存在
add_custom_command(
        OUTPUT ${PROTO_GEN_SOURCES} ${PROTO_GEN_HEADERS}
        COMMAND "${PROTOC}" -I "${CMAKE_SOURCE_DIR}" -I "${PROTOC_INCLUDE}"
                --cpp_out "${PROTO_OUT_DIR}"
                ${PROTO_FILES}
        DEPENDS ${PROTO_FILES}
        COMMENT "Generating C++ code from proto files (protoc 33.5)"
        VERBATIM
)

# ------------------------------------------------------------------ 宏：创建单个 proto 库
function(add_ink_storage_proto_library target_name proto_file)
    get_filename_component(proto_basename ${proto_file} NAME_WE)
    set(generated_src "${PROTO_OUT_DIR}/ink/storage/proto/${proto_basename}.pb.cc")
    set(generated_hdr "${PROTO_OUT_DIR}/ink/storage/proto/${proto_basename}.pb.h")

    add_library(${target_name} STATIC
            ${generated_src}
            ${generated_hdr}
    )
    target_link_libraries(${target_name} PUBLIC
            protobuf::libprotobuf
    )
    # 生成的 .pb.h 用 #include "ink/storage/proto/xxx.pb.h" 引用，
    # 需要构建根目录作为 include 路径。
    target_include_directories(${target_name} PUBLIC
            "${CMAKE_SOURCE_DIR}"
            "${CMAKE_BINARY_DIR}"
    )
    target_compile_features(${target_name} PUBLIC cxx_std_20)
endfunction()

# ------------------------------------------------------------------ 独立库（无项目内 proto 依赖）
add_ink_storage_proto_library(ink_storage_proto_options_cc_proto options)
add_ink_storage_proto_library(ink_storage_proto_color_cc_proto color)
add_ink_storage_proto_library(ink_storage_proto_coded_numeric_run_cc_proto coded_numeric_run)

# ------------------------------------------------------------------ 有依赖的库
add_ink_storage_proto_library(ink_storage_proto_brush_family_cc_proto brush_family)
target_link_libraries(ink_storage_proto_brush_family_cc_proto PUBLIC
        ink_storage_proto_color_cc_proto
        ink_storage_proto_options_cc_proto
)

add_ink_storage_proto_library(ink_storage_proto_brush_cc_proto brush)
target_link_libraries(ink_storage_proto_brush_cc_proto PUBLIC
        ink_storage_proto_brush_family_cc_proto
        ink_storage_proto_color_cc_proto
)

add_ink_storage_proto_library(ink_storage_proto_stroke_input_batch_cc_proto stroke_input_batch)
target_link_libraries(ink_storage_proto_stroke_input_batch_cc_proto PUBLIC
        ink_storage_proto_coded_numeric_run_cc_proto
)

add_ink_storage_proto_library(ink_storage_proto_incremental_stroke_inputs_cc_proto incremental_stroke_inputs)
target_link_libraries(ink_storage_proto_incremental_stroke_inputs_cc_proto PUBLIC
        ink_storage_proto_stroke_input_batch_cc_proto
)

add_ink_storage_proto_library(ink_storage_proto_mesh_cc_proto mesh)
target_link_libraries(ink_storage_proto_mesh_cc_proto PUBLIC
        ink_storage_proto_coded_numeric_run_cc_proto
)

# ------------------------------------------------------------------ ALIAS 目标（供父目录引用）
add_library(ink::storage::proto::options_cc_proto ALIAS ink_storage_proto_options_cc_proto)
add_library(ink::storage::proto::color_cc_proto ALIAS ink_storage_proto_color_cc_proto)
add_library(ink::storage::proto::coded_numeric_run_cc_proto ALIAS ink_storage_proto_coded_numeric_run_cc_proto)
add_library(ink::storage::proto::brush_family_cc_proto ALIAS ink_storage_proto_brush_family_cc_proto)
add_library(ink::storage::proto::brush_cc_proto ALIAS ink_storage_proto_brush_cc_proto)
add_library(ink::storage::proto::stroke_input_batch_cc_proto ALIAS ink_storage_proto_stroke_input_batch_cc_proto)
add_library(ink::storage::proto::incremental_stroke_inputs_cc_proto ALIAS ink_storage_proto_incremental_stroke_inputs_cc_proto)
add_library(ink::storage::proto::mesh_cc_proto ALIAS ink_storage_proto_mesh_cc_proto)