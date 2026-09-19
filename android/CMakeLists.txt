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

# NNUE networks are vendored in android/nnue (downloaded once from
# tests.stockfishchess.org / extracted from the official sf_18 release
# binary) so builds are deterministic and work offline. cmake's
# file(DOWNLOAD) hangs on the 302 redirect to data.stockfishchess.org
# anyway (cmake 3.22, HTTP/2).
# The big network (108919594 bytes) exceeds GitHub's 100 MB per-file limit,
# so it is stored split in two parts and reassembled here.
execute_process(
  COMMAND ${CMAKE_COMMAND} -E cat
          ${CMAKE_CURRENT_SOURCE_DIR}/nnue/nn-c288c895ea92.nnue.aa
          ${CMAKE_CURRENT_SOURCE_DIR}/nnue/nn-c288c895ea92.nnue.ab
  OUTPUT_FILE ${CMAKE_BINARY_DIR}/nn-c288c895ea92.nnue)
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/nnue/nn-37f18f62d772.nnue DESTINATION ${CMAKE_BINARY_DIR})
