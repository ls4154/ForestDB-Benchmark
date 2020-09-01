cmake_minimum_required(VERSION 2.8)

include_directories(BEFORE ${CMAKE_SOURCE_DIR}/utils)
include_directories(BEFORE ${CMAKE_SOURCE_DIR}/bench)
include_directories(BEFORE ${CMAKE_SOURCE_DIR}/include)
include_directories(BEFORE ${CMAKE_SOURCE_DIR}/wrappers)

if (NOT "${CMAKE_INCLUDE_PATH}" STREQUAL "")
    include_directories(BEFORE ${CMAKE_INCLUDE_PATH})
endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_GNU_SOURCE -D__DEBUG")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

if (NOT "${CMAKE_LIBRARY_PATH}" STREQUAL "")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${CMAKE_LIBRARY_PATH}")
endif()

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE)

if (NOT WIN32)
    set(PTHREAD_LIB pthread)
    set(LIBM m)
    set(LIBSNAPPY snappy)
    set(LIBLDB leveldb)
endif(NOT WIN32)

if ("${CMAKE_C_COMPILER_ID}" STREQUAL "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Qunused-arguments -g -fomit-frame-pointer -pthread")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -fomit-frame-pointer -pthread")
    set(LIBRT rt)
endif()

add_executable(leveldb_bench
               bench/couch_bench.cc
               wrappers/couch_leveldb.cc
               utils/avltree.cc
               utils/stopwatch.cc
               utils/iniparser.cc
               utils/crc32.cc
               utils/memleak.cc
               utils/zipfian_random.cc
               utils/keyloader.cc
               utils/keygen.cc)
target_link_libraries(leveldb_bench ${PTHREAD_LIB} ${LIBM} ${LIBLDB}
      -Wl,-rpath,$ENV{HOME}/spdk/build/lib:$ENV{HOME}/spdk/dpdk/build/lib -L$ENV{HOME}/spdk/build/lib  -L$ENV{HOME}/spdk/dpdk/build/lib -pthread
      -Wl,--whole-archive -Wl,--no-as-needed -lspdk_sock_posix -lspdk_nvme -lspdk_thread -lspdk_util -lspdk_log -lspdk_sock -lspdk_vmd -Wl,--no-whole-archive
      $ENV{HOME}/spdk/build/lib/libspdk_env_dpdk.a
      -Wl,--whole-archive,--no-as-needed $ENV{HOME}/spdk/dpdk/build/lib/librte_eal.a $ENV{HOME}/spdk/dpdk/build/lib/librte_mempool.a $ENV{HOME}/spdk/dpdk/build/lib/librte_ring.a
      $ENV{HOME}/spdk/dpdk/build/lib/librte_mbuf.a $ENV{HOME}/spdk/dpdk/build/lib/librte_mempool_ring.a $ENV{HOME}/spdk/dpdk/build/lib/librte_pci.a
      $ENV{HOME}/spdk/dpdk/build/lib/librte_bus_pci.a $ENV{HOME}/spdk/dpdk/build/lib/librte_kvargs.a $ENV{HOME}/spdk/dpdk/build/lib/librte_vhost.a
      $ENV{HOME}/spdk/dpdk/build/lib/librte_net.a $ENV{HOME}/spdk/dpdk/build/lib/librte_cryptodev.a $ENV{HOME}/spdk/dpdk/build/lib/librte_hash.a -Wl,--no-whole-archive
      -lnuma -ldl -L$ENV{HOME}/spdk/isa-l/.libs -lrt -luuid -lcrypto -laio)
set_target_properties(leveldb_bench PROPERTIES COMPILE_FLAGS "-D__LEVEL_BENCH")
file(COPY ${CMAKE_SOURCE_DIR}/bench_config.ini DESTINATION ./)
