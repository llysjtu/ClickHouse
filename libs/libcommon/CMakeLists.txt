if (APPLE)
    if ("${CMAKE_SYSTEM_NAME}" STREQUAL "Darwin" AND NOT "${CMAKE_SYSTEM_VERSION}" VERSION_LESS "16.1.0")
        set (APPLE_SIERRA_OR_NEWER 1)
    else ()
        set (APPLE_SIERRA_OR_NEWER 0)
    endif ()
endif ()

set (CONFIG_COMMON ${CMAKE_CURRENT_BINARY_DIR}/include/common/config_common.h)
configure_file (${CMAKE_CURRENT_SOURCE_DIR}/include/common/config_common.h.in ${CONFIG_COMMON})

if (APPLE)
    add_library (apple_rt
        src/apple_rt.cpp
        include/common/apple_rt.h
    )
    target_include_directories (apple_rt PUBLIC ${COMMON_INCLUDE_DIR})
endif ()

add_library (common
    src/DateLUT.cpp
    src/DateLUTImpl.cpp
    src/exp10.cpp
    src/mremap.cpp
    src/JSON.cpp
    src/getMemoryAmount.cpp
    src/ThreadPool.cpp
    src/iostream_debug_helpers.cpp

    include/common/Types.h
    include/common/DateLUT.h
    include/common/DateLUTImpl.h
    include/common/LocalDate.h
    include/common/LocalDateTime.h
    include/common/ErrorHandlers.h
    include/common/exp10.h
    include/common/mremap.h
    include/common/likely.h
    include/common/logger_useful.h
    include/common/MultiVersion.h
    include/common/strong_typedef.h
    include/common/JSON.h
    include/common/getMemoryAmount.h
    include/common/ThreadPool.h
    include/common/iostream_debug_helpers.h

    include/ext/bit_cast.h
    include/ext/collection_cast.h
    include/ext/enumerate.h
    include/ext/function_traits.h
    include/ext/identity.h
    include/ext/map.h
    include/ext/range.h
    include/ext/scope_guard.h
    include/ext/size.h
    include/ext/unlock_guard.h
    include/ext/singleton.h

    ${CONFIG_COMMON}
)

# When testing for memory leaks, dont link tcmalloc or jemalloc.

if (USE_JEMALLOC)
        message (STATUS "Link jemalloc : ${JEMALLOC_LIBRARIES}")
    set (MALLOC_LIBRARIES ${JEMALLOC_LIBRARIES})
elseif (USE_TCMALLOC)
    if (DEBUG_TCMALLOC)
        message (STATUS "Link libtcmalloc_minimal_debug for testing: ${GPERFTOOLS_TCMALLOC_MINIMAL_DEBUG}")
        set (MALLOC_LIBRARIES ${GPERFTOOLS_TCMALLOC_MINIMAL_DEBUG})
    else ()
        message (STATUS "Link libtcmalloc_minimal : ${GPERFTOOLS_TCMALLOC_MINIMAL}")
        set (MALLOC_LIBRARIES ${GPERFTOOLS_TCMALLOC_MINIMAL})
    endif ()
else ()
    message (STATUS "Disabling libtcmalloc for valgrind better analysis")
endif ()

if (GLIBC_COMPATIBILITY)
    set (GLIBC_COMPATIBILITY_LIBRARIES glibc-compatibility)
endif ()

if (USE_INTERNAL_MEMCPY)
    set (MEMCPY_LIBRARIES memcpy)
endif ()

find_package (Threads)

target_include_directories (common BEFORE PRIVATE ${CCTZ_INCLUDE_DIR})
target_include_directories (common BEFORE PUBLIC ${CITYHASH_INCLUDE_DIR})
target_include_directories (common PUBLIC ${COMMON_INCLUDE_DIR})

target_link_libraries (
    common
    pocoext
    ${CITYHASH_LIBRARIES}
    ${CCTZ_LIBRARY}
    ${Boost_SYSTEM_LIBRARY}
    ${Boost_FILESYSTEM_LIBRARY}
    ${MALLOC_LIBRARIES}
    ${CMAKE_THREAD_LIBS_INIT}
    ${RT_LIBRARIES}
    ${GLIBC_COMPATIBILITY_LIBRARIES}
    ${MEMCPY_LIBRARIES})

if (ENABLE_TESTS)
    add_subdirectory (src/tests)
endif ()
