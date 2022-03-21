if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
  set(LLVM_OUT_OF_TREE_BUILD TRUE)
endif()

if(${LLVM_OUT_OF_TREE_BUILD})
  set(TORCH_MLIR_DIALECTS_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
  set(TORCH_MLIR_DIALECTS_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}")
  message(STATUS "Building torch-mlir-dialects project at ${TORCH_MLIR_DIALECTS_SOURCE_DIR} (into ${TORCH_MLIR_DIALECTS_BINARY_DIR})")
  set(MLIR_GENERATED_INCLUDE_DIR ${MLIR_INCLUDE_DIR})
else()
option(MLIR_ENABLE_BINDINGS_PYTHON "Enables MLIR Python Bindings" OFF)

set(TORCH_MLIR_DIALECTS_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(TORCH_MLIR_DIALECTS_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}")
message(STATUS "Building torch-mlir-dialects project at ${TORCH_MLIR_DIALECTS_SOURCE_DIR} (into ${TORCH_MLIR_DIALECTS_BINARY_DIR})")

# TODO: Fix this upstream so that global include directories are not needed.
set(MLIR_MAIN_SRC_DIR ${LLVM_MAIN_SRC_DIR}/../mlir)
set(MLIR_INCLUDE_DIR ${LLVM_MAIN_SRC_DIR}/../mlir/include)
set(MLIR_GENERATED_INCLUDE_DIR ${LLVM_BINARY_DIR}/tools/mlir/include)

# TODO: Needed for tablegen. Remove.
include_directories(SYSTEM ${MLIR_INCLUDE_DIR})
include_directories(SYSTEM ${MLIR_GENERATED_INCLUDE_DIR})
include_directories(SYSTEM ${TORCH_MLIR_DIALECTS_SOURCE_DIR}/include)

# Configure CMake and tablegen.
list(APPEND CMAKE_MODULE_PATH ${MLIR_MAIN_SRC_DIR}/cmake/modules)
list(APPEND CMAKE_MODULE_PATH ${LLVM_MAIN_SRC_DIR}/cmake)
set(MLIR_TABLEGEN_EXE mlir-tblgen)

include(TableGen)
include(AddLLVM)
include(AddMLIR)
endif()

function(torch_mlir_dialects_target_includes target)
  set(_dirs 
    $<BUILD_INTERFACE:${LLVM_BUILD_MAIN_SRC_DIR}/include>
    $<BUILD_INTERFACE:${LLVM_INCLUDE_DIR}>
    $<BUILD_INTERFACE:${MLIR_INCLUDE_DIRS}>
    $<BUILD_INTERFACE:${MLIR_GENERATED_INCLUDE_DIR}>
    $<BUILD_INTERFACE:${TORCH_MLIR_DIALECTS_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${TORCH_MLIR_DIALECTS_BINARY_DIR}/include>
  )
  # In LLVM parlance, the actual target may just be an interface and may not
  # be responsible for actually compiling anything. The corresponding obj.
  # target, when present, is just used for compilation and does not
  # contribute to the interface properties.
  # TODO: Normalize this upstream.
  target_include_directories(${target} PUBLIC ${_dirs})
  if(TARGET obj.${target})
    target_include_directories(obj.${target} PRIVATE ${_dirs})
  endif()
endfunction()

include_directories(SYSTEM ${MLIR_INCLUDE_DIRS})
include_directories(SYSTEM ${MLIR_GENERATED_INCLUDE_DIR})
include_directories(SYSTEM ${TORCH_MLIR_DIALECTS_SOURCE_DIR}/include)

add_subdirectory(include)
add_subdirectory(lib)
add_subdirectory(tools)
add_subdirectory(test)

# TODO: Needed for tablegen. Remove.
