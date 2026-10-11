cmake_minimum_required(VERSION 3.4.1)

# Enable 16KB page size support for Android (required for Google Play)
add_link_options(-Wl,-z,max-page-size=16384)
add_link_options(-Wl,-z,common-page-size=16384)

file(GLOB_RECURSE cppPaths "../ios/Stockfish/src/*.cpp")
add_library(
  stockfish
  SHARED
  ../ios/FlutterStockfish/ffi.cpp
  ${cppPaths}
)

if(ANDROID_ABI STREQUAL arm64-v8a)
  if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(stockfish PRIVATE -DUSE_PTHREADS -DIS_64BIT -DUSE_POPCNT -DUSE_NEON=8)
  else()
    target_compile_options(stockfish PRIVATE -fno-exceptions -DUSE_PTHREADS -DNDEBUG -O3 -DIS_64BIT -DUSE_POPCNT -DUSE_NEON=8)
  endif()
else()
  if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(stockfish PRIVATE -DUSE_PTHREADS)
  else()
    target_compile_options(stockfish PRIVATE -fno-exceptions -DUSE_PTHREADS -DNDEBUG -O3)
  endif()
endif()

# Small-net-only release build (T0079): the big network (nn-c288c895ea92,
# 108919594 bytes, stored split as .aa/.ab in android/nnue) is NOT
# embedded. Three type-level lines (evaluate.h EvalFileDefaultNameBig →
# small net, network.h NetworkBig → small architecture,
# nnue_accumulator.h caches.big → small dims) route both NNUE slots onto
# the small net, so the engine starts, verifies and searches without the
# big net (evidence: shakhfit work/T0079-smallnet-verify-switch/).
# Only the small net is copied into the build dir for incbin.
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/nnue/nn-37f18f62d772.nnue DESTINATION ${CMAKE_BINARY_DIR})
