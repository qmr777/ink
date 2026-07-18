# ------------------------------------------------------------------------------
#  ink/storage/proto/CMakeLists.txt
#  Proto 库 —— 构建时通过 protoc 生成 .pb.cc / .pb.h
# ------------------------------------------------------------------------------
cmake_minimum_required(VERSION 3.20)

# ------------------------------------------------------------------------------
# 1) Locate protoc.exe (Windows-only)
#    优先级: 上级 -Dprotobuf_dir=<dir>  >  本地 protoc-33.5-win64/  >  FATAL_ERROR
#    约定: <dir>/bin/protoc.exe  +  <dir>/include/google/...
# ------------------------------------------------------------------------------
set(_ink_proto_search_roots "")
if (DEFINED protobuf_dir AND NOT protobuf_dir STREQUAL "")
    list(APPEND _ink_proto_search_roots "${protobuf_dir}")
endif ()
list(APPEND _ink_proto_search_roots "${CMAKE_CURRENT_LIST_DIR}/protoc-33.5-win64")

set(_ink_protoc_found FALSE)
foreach (_root IN LISTS _ink_proto_search_roots)
    get_filename_component(_protoc_exe "${_root}/bin/protoc.exe" ABSOLUTE)
    if (EXISTS "${_protoc_exe}")
        set(PROTOC "${_protoc_exe}"
            CACHE FILEPATH "Path to protoc compiler")
        get_filename_component(PROTOC_INCLUDE "${_root}/include" ABSOLUTE)
        set(_ink_protoc_found TRUE)
        message(STATUS "[ink/storage/proto] protoc  = ${PROTOC}")
        message(STATUS "[ink/storage/proto] include = ${PROTOC_INCLUDE}")
        break()
    endif ()
endforeach ()
unset(_ink_proto_search_roots)


if (NOT _ink_protoc_found)
    message(FATAL_ERROR
        "[ink/storage/proto] protoc.exe not found.\n"
        "  This CMakeLists locates the protoc *compiler* via the -Dprotobuf_dir variable.\n"
        "  <protobuf_dir> must contain:\n"
        "      <protobuf_dir>/bin/protoc.exe\n"
        "      <protobuf_dir>/include/google/protobuf/...\n"
        "  Searched:\n"
        "    1) -Dprotobuf_dir=<dir>                       -> <dir>/bin/protoc.exe\n"
        "    2) <this-dir>/protoc-33.5-win64/bin/protoc.exe\n"
        "  Fix one of:\n"
        "    cmake -Dprotobuf_dir=<path-to-protoc-install> ...\n"
        "    (e.g. -Dprotobuf_dir=C:/protobuf/protoc-33.5-win64)\n"
        "    OR place protoc-33.5-win64/ alongside this CMakeLists.txt."
    )
endif ()


# ------------------------------------------------------------------------------
# 2) Proto 路径常量 (供下方 add_ink_storage_proto_library 使用)
#    .proto 的 import 形如:
#      import "ink/storage/proto/foo.proto"   -> PROTO_ROOT 必须是项目根
#      import "google/protobuf/descriptor.proto" -> 需要 PROTOC_INCLUDE
#    --cpp_out 镜像输入的相对路径, 所以 brush.proto ->
#      <OUT>/ink/storage/proto/brush.pb.{cc,h}
# ------------------------------------------------------------------------------
# PROTO_ROOT:
#   .proto 与本 CMakeLists.txt 同在 <proj>/ink/storage/proto/, 不管是自建还是
#   被 FetchContent, CMAKE_CURRENT_LIST_DIR 都是真实源路径, 上溯三级就是项目根.
#   这样不用区分 ink_SOURCE_DIR / CMAKE_SOURCE_DIR, 也不需要 CACHE 兜底.
get_filename_component(PROTO_ROOT "${CMAKE_CURRENT_LIST_DIR}/../../.." ABSOLUTE)
set(PROTO_OUT_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated"
    CACHE PATH "Output directory for generated .pb.cc / .pb.h")

# ---------------------------------------------- 关键修改：每个 proto 一条 custom_command
# 之前是 1 个 add_custom_command 输出 16 个文件, 跨函数 add_library 时 CMake
# 的依赖追踪会失效, 报 "Cannot find source file". 改为每个 proto 单独一条命令,
# 与它对应的 add_library 在同一函数作用域里, 依赖关系最干净.
function(add_ink_storage_proto_library target_name proto_file)
    get_filename_component(proto_basename ${proto_file} NAME_WE)
    set(_proto_abs "${CMAKE_CURRENT_SOURCE_DIR}/${proto_file}.proto")
    set(_proto_rel  "ink/storage/proto/${proto_file}.proto")
    set(generated_src "${PROTO_OUT_DIR}/ink/storage/proto/${proto_basename}.pb.cc")
    set(generated_hdr "${PROTO_OUT_DIR}/ink/storage/proto/${proto_basename}.pb.h")

    add_custom_command(
            OUTPUT  ${generated_src} ${generated_hdr}
            COMMAND "${PROTOC}"
                    -I "${PROTO_ROOT}"
                    -I "${PROTOC_INCLUDE}"
                    --cpp_out "${PROTO_OUT_DIR}"
                    "${_proto_rel}"
            DEPENDS ${_proto_abs}
            COMMENT "Generating ${proto_basename}.pb.{cc,h}"
            VERBATIM
    )

    add_library(${target_name} STATIC
            ${generated_src}
            ${generated_hdr}
    )
    target_link_libraries(${target_name} PUBLIC
            protobuf::libprotobuf
    )
    target_include_directories(${target_name} PUBLIC
            "${PROTO_OUT_DIR}"  # 生成的 .pb.h 在这里, 必须作为根, 才能解析 #include "ink/storage/proto/xxx.pb.h"
            "${CMAKE_SOURCE_DIR}"
            "${CMAKE_BINARY_DIR}"
            "${PROTO_ROOT}"
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

# ------------------------------------------------------------------ 伞型 INTERFACE 目标
# 一次性把所有 *_cc_proto 库 PUBLIC 链接起来。
# 下游只需要 link ink::storage::proto 即可拿到全部 .pb.h 头文件路径、
# protobuf::libprotobuf 运行时，以及所有生成消息类的构造函数/反射符号。
# 这样顶层 CMakeLists.txt 就不用列 8 个 ALIAS。
add_library(ink_storage_proto_umbrella INTERFACE)
target_link_libraries(ink_storage_proto_umbrella INTERFACE
        ink::storage::proto::options_cc_proto
        ink::storage::proto::color_cc_proto
        ink::storage::proto::coded_numeric_run_cc_proto
        ink::storage::proto::brush_family_cc_proto
        ink::storage::proto::brush_cc_proto
        ink::storage::proto::stroke_input_batch_cc_proto
        ink::storage::proto::incremental_stroke_inputs_cc_proto
        ink::storage::proto::mesh_cc_proto
)
add_library(ink::storage::proto ALIAS ink_storage_proto_umbrella)