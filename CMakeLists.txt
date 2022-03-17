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
    crypto/mman.c
    crypto/mman.h
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
  Algo256/blake256.cu
  Algo256/blake2s.cu
  Algo256/bmw.cu
  Algo256/cuda_blake256.cu
  Algo256/cuda_bmw.cu
  Algo256/cuda_bmw256.cu
  Algo256/cuda_cubehash256.cu
  Algo256/cuda_fugue256.cu
  Algo256/cuda_groestl256.cu
  Algo256/cuda_keccak256.cu
  Algo256/cuda_keccak256_sm3.cu
  Algo256/cuda_skein256.cu
  Algo256/decred.cu
  Algo256/keccak256.cu
  Algo256/vanilla.cu
  JHA/cuda_jha_compactionTest.cu
  JHA/cuda_jha_keccak512.cu
  JHA/jackpotcoin.cu
  JHA/jha.cu
  algos.h
  api.cpp
  bench.cpp
  bignum.cpp
  bignum.hpp
  bignum_ssl10.hpp
  blake2b.cu
  ccminer-config.h
  ccminer.cpp
  compat.h
  crc32.c
  crypto/aesb.cpp
  crypto/cn_aes.cuh
  crypto/cn_blake.cuh
  crypto/cn_groestl.cuh
  crypto/cn_jh.cuh
  crypto/cn_keccak.cuh
  crypto/cn_skein.cuh
  crypto/cpu/c_keccak.c
  crypto/cpu/c_keccak.h
  crypto/cryptolight-core.cu
  crypto/cryptolight-cpu.cpp
  crypto/cryptolight.cu
  crypto/cryptolight.h
  crypto/cryptonight-core.cu
  crypto/cryptonight-cpu.cpp
  crypto/cryptonight-extra.cu
  crypto/cryptonight.cu
  crypto/cryptonight.h
  crypto/int128_c.h
  crypto/oaes_config.h
  crypto/oaes_lib.cpp
  crypto/oaes_lib.h
  crypto/wildkeccak-cpu.cpp
  crypto/wildkeccak.cu
  crypto/wildkeccak.h
  crypto/xmr-rpc.cpp
  crypto/xmr-rpc.h
  cuda.cpp
  cuda_checkhash.cu
  cuda_debug.cuh
  cuda_fugue256.h
  cuda_groestlcoin.cu
  cuda_groestlcoin.h
  cuda_helper.h
  cuda_myriadgroestl.cu
  cuda_skeincoin.cu
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
  equi/blake2/blake2-config.h
  equi/blake2/blake2-impl.h
  equi/blake2/blake2-round.h
  equi/blake2/blake2.h
  equi/blake2/blake2b-load-sse2.h
  equi/blake2/blake2b-load-sse41.h
  equi/blake2/blake2b-round.h
  equi/blake2/blake2bx.cpp
  equi/cuda_equi.cu
  equi/eqcuda.hpp
  equi/equi-stratum.cpp
  equi/equi.cpp
  equi/equihash.cpp
  equi/equihash.h
  fuguecoin.cpp
  groestlcoin.cpp
  hashlog.cpp
  heavy/bastion.cu
  heavy/cuda_bastion.cu
  heavy/cuda_blake512.cu
  heavy/cuda_combine.cu
  heavy/cuda_groestl512.cu
  heavy/cuda_hefty1.cu
  heavy/cuda_keccak512.cu
  heavy/cuda_sha256.cu
  heavy/heavy.cu
  heavy/heavy.h
  hefty1.c
  hefty1.h
  lbry/cuda_lbry_merged.cu
  lbry/cuda_sha256_lbry.cu
  lbry/cuda_sha512_lbry.cu
  lbry/lbry.cu
  lyra2/Lyra2.c
  lyra2/Lyra2.h
  lyra2/Lyra2Z.c
  lyra2/Lyra2Z.h
  lyra2/Sponge.c
  lyra2/Sponge.h
  lyra2/allium.cu
  lyra2/cuda_lyra2.cu
  lyra2/cuda_lyra2Z.cu
  lyra2/cuda_lyra2Z_sm5.cuh
  lyra2/cuda_lyra2_sm2.cuh
  lyra2/cuda_lyra2_sm5.cuh
  lyra2/cuda_lyra2_vectors.h
  lyra2/cuda_lyra2v2.cu
  lyra2/cuda_lyra2v2_sm3.cuh
  lyra2/cuda_lyra2v3.cu
  lyra2/cuda_lyra2v3_sm3.cuh
  lyra2/lyra2RE.cu
  lyra2/lyra2REv2.cu
  lyra2/lyra2REv3.cu
  lyra2/lyra2Z.cu
  miner.h
  myriadgroestl.cpp
  neoscrypt/cuda_neoscrypt.cu
  neoscrypt/cuda_vectors.h
  neoscrypt/neoscrypt-cpu.c
  neoscrypt/neoscrypt.cpp
  neoscrypt/neoscrypt.h
  nvapi.cpp
  nvml.cpp
  nvml.h
  nvsettings.cpp
  pentablake.cu
  phi/cuda_phi2.cu
  phi/cuda_phi2_cubehash512.cu
  phi/phi.cu
  phi/phi2.cu
  polytimos.cu
  pools.cpp
  quark/cuda_bmw512.cu
  quark/cuda_bmw512_sm3.cuh
  quark/cuda_jh512.cu
  quark/cuda_quark.h
  quark/cuda_quark_blake512.cu
  quark/cuda_quark_blake512_sp.cuh
  quark/cuda_quark_compactionTest.cu
  quark/cuda_quark_groestl512.cu
  quark/cuda_quark_groestl512_sm2.cuh
  quark/cuda_quark_keccak512.cu
  quark/cuda_skein512.cu
  quark/groestl_functions_quad.h
  quark/groestl_simple.cuh
  quark/groestl_transf_quad.h
  quark/nist5.cu
  quark/quarkcoin.cu
  qubit/deep.cu
  qubit/luffa.cu
  qubit/qubit.cu
  qubit/qubit_luffa512.cu
  scrypt-jane.cpp
  scrypt.cpp
  scrypt/blake.cu
  scrypt/code/scrypt-conf.h
  scrypt/code/scrypt-jane-chacha.h
  scrypt/code/scrypt-jane-mix_chacha.h
  scrypt/code/scrypt-jane-portable-x86.h
  scrypt/code/scrypt-jane-portable.h
  scrypt/code/scrypt-jane-romix-basic.h
  scrypt/code/scrypt-jane-romix-template.h
  scrypt/fermi_kernel.cu
  scrypt/fermi_kernel.h
  scrypt/keccak.cu
  scrypt/keccak.h
  scrypt/kepler_kernel.cu
  scrypt/kepler_kernel.h
  scrypt/nv_kernel.cu
  scrypt/nv_kernel.h
  scrypt/nv_kernel2.cu
  scrypt/nv_kernel2.h
  scrypt/salsa_kernel.cu
  scrypt/salsa_kernel.h
  scrypt/scrypt-jane.h
  scrypt/sha256.cu
  scrypt/sha256.h
  scrypt/test_kernel.cu
  scrypt/test_kernel.h
  scrypt/titan_kernel.cu
  scrypt/titan_kernel.h
  serialize.hpp
  sha256/cuda_sha256d.cu
  sha256/cuda_sha256q.cu
  sha256/cuda_sha256t.cu
  sha256/sha256d.cu
  sha256/sha256q.cu
  sha256/sha256t.cu
  sia/sia-rpc.cpp
  sia/sia-rpc.h
  sia/sia.cu
  skein.cu
  skein2.cpp
  skunk/cuda_skunk.cu
  skunk/cuda_skunk_streebog.cu
  skunk/skein_header.h
  skunk/skunk.cu
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
  tribus/cuda_echo512_aes.cuh
  tribus/cuda_echo512_final.cu
  tribus/tribus.cu
  uint256.h
  util.cpp
  x11/bitcore.cu
  x11/c11.cu
  x11/cuda_streebog.cu
  x11/cuda_streebog_maxwell.cu
  x11/cuda_x11.h
  x11/cuda_x11_aes.cuh
  x11/cuda_x11_cubehash512.cu
  x11/cuda_x11_echo.cu
  x11/cuda_x11_luffa512.cu
  x11/cuda_x11_luffa512_Cubehash.cu
  x11/cuda_x11_shavite512.cu
  x11/cuda_x11_simd512.cu
  x11/cuda_x11_simd512_func.cuh
  x11/cuda_x11_simd512_sm2.cuh
  x11/exosis.cu
  x11/fresh.cu
  x11/s3.cu
  x11/sib.cu
  x11/streebog_arrays.cuh
  x11/timetravel.cu
  x11/veltor.cu
  x11/x11.cu
  x11/x11evo.cu
  x12/x12.cu
  x13/cuda_hsr_sm3.cu
  x13/cuda_x13_fugue512.cu
  x13/cuda_x13_hamsi512.cu
  x13/hsr.cu
  x13/sm3.c
  x13/sm3.h
  x13/x13.cu
  x15/cuda_whirlpool_tables.cuh
  x15/cuda_x14_shabal512.cu
  x15/cuda_x15_whirlpool.cu
  x15/cuda_x15_whirlpool_sm3.cu
  x15/whirlpool.cu
  x15/x14.cu
  x15/x15.cu
  x16/cuda_x16.h
  x16/cuda_x16_echo512.cu
  x16/cuda_x16_echo512_64.cu
  x16/cuda_x16_fugue512.cu
  x16/cuda_x16_shabal512.cu
  x16/cuda_x16_simd512_80.cu
  x16/x16r.cu
  x16/x16s.cu
  x17/cuda_x17_haval256.cu
  x17/cuda_x17_sha512.cu
  x17/hmq17.cu
  x17/sonoa.cu
  x17/x17.cu
  zr5.cu)
set_property(TARGET ${PROJECT_NAME} PROPERTY CUDA_ARCHITECTURES 61)
target_link_libraries(${PROJECT_NAME}
  PRIVATE
    ccminer_compile_options
    ccminer_dependencies
    ccminer_include_directories)
