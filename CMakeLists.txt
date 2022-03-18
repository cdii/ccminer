cmake_minimum_required(VERSION 3.21)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

include(HunterGate)
HunterGate(
  URL "https://github.com/cpp-pm/hunter/archive/v0.24.0.tar.gz"
  SHA1 "a3d7f4372b1dcd52faa6ff4a3bd5358e1d0e5efd")

project(ccminer LANGUAGES C CUDA CXX)

#===============================================================================
# cmake configuration
#===============================================================================
set(CMAKE_CONFIGURATION_TYPES Debug Release)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

foreach(config ${CMAKE_CONFIGURATION_TYPES})
  string(TOUPPER ${config} config)
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${config} ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY})
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${config} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
endforeach()

set_property(DIRECTORY PROPERTY VS_STARTUP_PROJECT ${PROJECT_NAME})
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

#===============================================================================
# compile options
#===============================================================================
add_library(ccminer_compile_options INTERFACE)
target_compile_features(ccminer_compile_options INTERFACE c_std_11 cxx_std_17 cuda_std_17)
target_compile_options(ccminer_compile_options
  INTERFACE
    $<$<COMPILE_LANG_AND_ID:C,GNU>:-Wall -Wextra -pedantic>
    $<$<COMPILE_LANG_AND_ID:C,MSVC>:/W4>
    $<$<COMPILE_LANG_AND_ID:CUDA,GNU>:-Xcompiler=-Wall -Xcompiler=-Wextra -Xcompiler=-pedantic>
    $<$<COMPILE_LANG_AND_ID:CUDA,MSVC>:-Xcompiler=/W4>
    $<$<COMPILE_LANG_AND_ID:CUDA,NVIDIA>:-default-stream=per-thread>
    $<$<COMPILE_LANG_AND_ID:CXX,GNU>:-Wall -Wextra -pedantic>
    $<$<COMPILE_LANG_AND_ID:CXX,MSVC>:/W4>)

target_compile_definitions(ccminer_compile_options INTERFACE USE_WRAPNVML)
if(MSVC)
  target_compile_definitions(ccminer_compile_options INTERFACE _CRT_SECURE_NO_WARNINGS)
endif()

#===============================================================================
# dependencies
#===============================================================================
hunter_add_package(CURL)
find_package(CUDAToolkit REQUIRED)
find_package(CURL REQUIRED)
add_library(ccminer_dependencies INTERFACE)
target_link_libraries(ccminer_dependencies INTERFACE CURL::libcurl CUDA::cuda_driver)

if(WIN32)
  hunter_add_package(pthreads-win32)
  find_package(pthreads-win32 CONFIG REQUIRED)
  target_link_libraries(ccminer_dependencies
    INTERFACE
      ${CMAKE_SOURCE_DIR}/deps/nvapi/amd64/nvapi64.lib
      pthreads-win32::pthreads)
else()
  find_package(Threads REQUIRED)
  target_link_libraries(ccminer_dependencies INTERFACE Threads::Threads)
endif()

#===============================================================================
add_library(ccminer_include_directories INTERFACE)
target_include_directories(ccminer_include_directories
  INTERFACE
    ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}
    ${CMAKE_SOURCE_DIR}
    deps
    deps/jansson)

if(WIN32)
  target_include_directories(ccminer_include_directories INTERFACE compat)
endif()

#===============================================================================
source_group("CUDA Header Files" REGULAR_EXPRESSION ".*(cuh)$")
source_group("CUDA Source Files" REGULAR_EXPRESSION ".*(cu)$")
if(WIN32)
  source_group("Resource Files" REGULAR_EXPRESSION ".*(rc)$")
  set(ccminer_win32_sources
    compat/getopt.h
    compat/getopt_long.c
    compat/gettimeofday.c
    compat/inttypes.h
    compat/stdbool.h
    compat/sys/time.h
    compat/unistd.h
    compat/winansi.c
    compat/winansi.h
    deps/nvapi/nvHLSLExtns.h
    deps/nvapi/nvHLSLExtnsInternal.h
    deps/nvapi/nvShaderExtnEnums.h
    deps/nvapi/nvapi.h
    deps/nvapi/nvapi_ccminer.h
    deps/nvapi/nvapi_lite_common.h
    deps/nvapi/nvapi_lite_d3dext.h
    deps/nvapi/nvapi_lite_salend.h
    deps/nvapi/nvapi_lite_salstart.h
    deps/nvapi/nvapi_lite_sli.h
    deps/nvapi/nvapi_lite_stereo.h
    deps/nvapi/nvapi_lite_surround.h
    res/ccminer.rc
    res/resource.h)
endif()

add_executable(${PROJECT_NAME}
  ${ccminer_win32_sources}
  Algo256/cuda_blake256.cu
  Algo256/cuda_cubehash256.cu
  Algo256/cuda_groestl256.cu
  Algo256/cuda_keccak256.cu
  Algo256/cuda_skein256.cu
  algos.h
  api.cpp
  bench.cpp
  bignum.cpp
  bignum.hpp
  bignum_ssl10.hpp
  ccminer-config.h
  ccminer.cpp
  compat.h
  crc32.c
  cuda.cpp
  cuda_checkhash.cu
  cuda_debug.cuh
  cuda_helper.h
  cuda_vector_uint2x4.h
  cuda_vectors.h
  deps/jansson/dump.c
  deps/jansson/error.c
  deps/jansson/hashtable.c
  deps/jansson/hashtable.h
  deps/jansson/jansson.h
  deps/jansson/jansson_config.h
  deps/jansson/jansson_private.h
  deps/jansson/jansson_private_config.h
  deps/jansson/load.c
  deps/jansson/memory.c
  deps/jansson/pack_unpack.c
  deps/jansson/strbuffer.c
  deps/jansson/strbuffer.h
  deps/jansson/strconv.c
  deps/jansson/utf.c
  deps/jansson/utf.h
  deps/jansson/util.h
  deps/jansson/value.c
  elist.h
  hashlog.cpp
  heavy/cuda_groestl512.cu
  heavy/cuda_hefty1.cu
  heavy/heavy.cu
  heavy/heavy.h
  hefty1.c
  hefty1.h
  lyra2/Lyra2.c
  lyra2/Lyra2.h
  lyra2/Sponge.c
  lyra2/Sponge.h
  lyra2/allium.cu
  lyra2/cuda_lyra2.cu
  lyra2/cuda_lyra2_sm2.cuh
  lyra2/cuda_lyra2_sm5.cuh
  lyra2/cuda_lyra2_vectors.h
  miner.h
  nvapi.cpp
  nvml.cpp
  nvml.h
  nvsettings.cpp
  pools.cpp
  serialize.hpp
  sph/aes_helper.c
  sph/blake.c
  sph/blake2b.c
  sph/blake2b.h
  sph/blake2s.c
  sph/blake2s.h
  sph/bmw.c
  sph/cubehash.c
  sph/echo.c
  sph/fugue.c
  sph/groestl.c
  sph/hamsi.c
  sph/hamsi_helper.c
  sph/haval.c
  sph/jh.c
  sph/keccak.c
  sph/luffa.c
  sph/ripemd.c
  sph/sha2.c
  sph/sha2big.c
  sph/shabal.c
  sph/shavite.c
  sph/simd.c
  sph/skein.c
  sph/sph_blake.h
  sph/sph_bmw.h
  sph/sph_cubehash.h
  sph/sph_echo.h
  sph/sph_fugue.h
  sph/sph_groestl.h
  sph/sph_hamsi.h
  sph/sph_haval.h
  sph/sph_jh.h
  sph/sph_keccak.h
  sph/sph_luffa.h
  sph/sph_ripemd.h
  sph/sph_sha2.c
  sph/sph_sha2.h
  sph/sph_shabal.h
  sph/sph_shavite.h
  sph/sph_simd.h
  sph/sph_skein.h
  sph/sph_streebog.h
  sph/sph_types.h
  sph/sph_whirlpool.h
  sph/streebog.c
  sph/whirlpool.c
  stats.cpp
  sysinfos.cpp
  uint256.h
  util.cpp)
set_property(TARGET ${PROJECT_NAME} PROPERTY CUDA_ARCHITECTURES 52 53 60 61 62 70 72 75 80 86)
target_link_libraries(${PROJECT_NAME}
  PRIVATE
    ccminer_compile_options
    ccminer_dependencies
    ccminer_include_directories)
