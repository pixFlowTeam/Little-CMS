cmake_minimum_required(VERSION 3.16)

# 避免源码树污染
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
  message(FATAL_ERROR "In-source builds are not permitted. Make a separate folder for building:\nmkdir build; cd build; cmake ..\nBefore that, remove the files already created:\nrm -rf CMakeCache.txt CMakeFiles")
endif()

project(lcms2 C)

# 设置 C 标准
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# 设置版本信息
set(LCMS2_VERSION_MAJOR 2)
set(LCMS2_VERSION_MINOR 17)
set(LCMS2_VERSION_MICRO 0)
set(LCMS2_VERSION "${LCMS2_VERSION_MAJOR}.${LCMS2_VERSION_MINOR}")

# 包含目录
include(GNUInstallDirs)
include(CheckIncludeFile)
include(CheckFunctionExists)
include(CheckLibraryExists)
include(CheckCSourceCompiles)

# 编译器检查
if(MSVC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /W3")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /Od /Zi")
    set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} /O2")
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g -O0")
    set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O2")
endif()

# 选项
option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(BUILD_STATIC_LIBS "Build static libraries" ON)
option(ENABLE_JPEG "Enable JPEG support" OFF)
option(ENABLE_TIFF "Enable TIFF support" OFF)
option(ENABLE_ZLIB "Enable ZLIB support" OFF)
option(ENABLE_THREADS "Enable thread support" ON)
option(ENABLE_SSE2 "Enable SSE2 optimizations" ON)
option(ENABLE_DEBUG "Enable debug build" OFF)
option(BUILD_UTILS "Build utility programs" ON)
option(BUILD_TESTS "Build test programs" ON)

# 检查编译器特性
check_include_file("time.h" HAVE_TIME_H)
check_function_exists("gmtime_r" HAVE_GMTIME_R)
check_function_exists("gmtime_s" HAVE_GMTIME_S)
check_function_exists("timespec_get" HAVE_TIMESPEC_GET)

# 检查数学库
check_library_exists("m" "sqrt" "" HAVE_LIBM)
if(HAVE_LIBM)
    set(MATH_LIBRARIES m)
endif()

# 检查线程支持
if(ENABLE_THREADS)
    find_package(Threads REQUIRED)
    set(THREAD_LIBRARIES ${CMAKE_THREAD_LIBS_INIT})
    add_definitions(-DHasTHREADS=1)
else()
    add_definitions(-DHasTHREADS=0)
endif()

# 检查 JPEG 支持
if(ENABLE_JPEG)
    find_package(JPEG REQUIRED)
    add_definitions(-DHasJPEG=1)
    set(JPEG_LIBRARIES ${JPEG_LIBRARIES})
else()
    add_definitions(-DHasJPEG=0)
endif()

# 检查 TIFF 支持
if(ENABLE_TIFF)
    find_package(TIFF REQUIRED)
    add_definitions(-DHasTIFF=1)
    set(TIFF_LIBRARIES ${TIFF_LIBRARIES})
else()
    add_definitions(-DHasTIFF=0)
endif()

# 检查 ZLIB 支持
if(ENABLE_ZLIB)
    find_package(ZLIB REQUIRED)
    add_definitions(-DHasZLIB=1)
    set(ZLIB_LIBRARIES ${ZLIB_LIBRARIES})
else()
    add_definitions(-DHasZLIB=0)
endif()

# 检查 SSE2 支持
if(ENABLE_SSE2)
    check_c_source_compiles("
        #include <emmintrin.h>
        int main() { __m128i n = _mm_set1_epi8(42); return 0; }
    " HAVE_SSE2)
    if(NOT HAVE_SSE2)
        add_definitions(-DCMS_DONT_USE_SSE2=1)
    endif()
else()
    add_definitions(-DCMS_DONT_USE_SSE2=1)
endif()

# 检查大端序
if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(arm|aarch64|mips|mips64|ppc|ppc64|sparc|sparc64)$")
    add_definitions(-DWORDS_BIGENDIAN=1)
endif()

# 检查可见性支持
if(NOT MSVC)
    check_c_compiler_flag("-fvisibility=hidden" HAVE_VISIBILITY)
    if(HAVE_VISIBILITY)
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fvisibility=hidden")
        add_definitions(-DHAVE_FUNC_ATTRIBUTE_VISIBILITY=1)
    endif()
endif()

# 设置编译定义
add_definitions(
    -DLCMS2_VERSION_MAJOR=${LCMS2_VERSION_MAJOR}
    -DLCMS2_VERSION_MINOR=${LCMS2_VERSION_MINOR}
    -DLCMS2_VERSION_MICRO=${LCMS2_VERSION_MICRO}
)

if(HAVE_GMTIME_R)
    add_definitions(-DHAVE_GMTIME_R=1)
endif()

if(HAVE_GMTIME_S)
    add_definitions(-DHAVE_GMTIME_S=1)
endif()

if(HAVE_TIMESPEC_GET)
    add_definitions(-DHAVE_TIMESPEC_GET=1)
endif()

# 源文件
set(LCMS2_SOURCES
    src/cmsalpha.c
    src/cmscam02.c
    src/cmscgats.c
    src/cmscnvrt.c
    src/cmserr.c
    src/cmsgamma.c
    src/cmsgmt.c
    src/cmshalf.c
    src/cmsintrp.c
    src/cmsio0.c
    src/cmsio1.c
    src/cmslut.c
    src/cmsmd5.c
    src/cmsmtrx.c
    src/cmsnamed.c
    src/cmsopt.c
    src/cmspack.c
    src/cmspcs.c
    src/cmsplugin.c
    src/cmsps2.c
    src/cmssamp.c
    src/cmssm.c
    src/cmstypes.c
    src/cmsvirt.c
    src/cmswtpnt.c
    src/cmsxform.c
)

# 头文件
set(LCMS2_HEADERS
    include/lcms2.h
    include/lcms2_plugin.h
)

# 创建库
if(BUILD_SHARED_LIBS)
    add_library(lcms2 SHARED ${LCMS2_SOURCES})
    set_target_properties(lcms2 PROPERTIES
        VERSION ${LCMS2_VERSION}
        SOVERSION ${LCMS2_VERSION_MAJOR}
        OUTPUT_NAME lcms2
    )
    
    # 链接库
    target_link_libraries(lcms2 ${MATH_LIBRARIES} ${THREAD_LIBRARIES})
    
    # 安装
    install(TARGETS lcms2
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
endif()

if(BUILD_STATIC_LIBS)
    add_library(lcms2_static STATIC ${LCMS2_SOURCES})
    set_target_properties(lcms2_static PROPERTIES
        OUTPUT_NAME lcms2
    )
    
    # 链接库
    target_link_libraries(lcms2_static ${MATH_LIBRARIES} ${THREAD_LIBRARIES})
    
    # 安装
    install(TARGETS lcms2_static
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
endif()

# 安装头文件
install(FILES ${LCMS2_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# 创建 pkg-config 文件
configure_file(
    ${CMAKE_SOURCE_DIR}/lcms2.pc.in
    ${CMAKE_BINARY_DIR}/lcms2.pc
    @ONLY
)

install(FILES ${CMAKE_BINARY_DIR}/lcms2.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)

# 构建工具程序
if(BUILD_UTILS)
    add_subdirectory(utils)
endif()

# 构建测试程序
if(BUILD_TESTS)
    add_subdirectory(testbed)
endif()

# 显示配置信息
message(STATUS "Little-CMS 2 Configuration:")
message(STATUS "  Version: ${LCMS2_VERSION}")
message(STATUS "  Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "  Shared libraries: ${BUILD_SHARED_LIBS}")
message(STATUS "  Static libraries: ${BUILD_STATIC_LIBS}")
message(STATUS "  JPEG support: ${ENABLE_JPEG}")
message(STATUS "  TIFF support: ${ENABLE_TIFF}")
message(STATUS "  ZLIB support: ${ENABLE_ZLIB}")
message(STATUS "  Thread support: ${ENABLE_THREADS}")
message(STATUS "  SSE2 support: ${ENABLE_SSE2}")
message(STATUS "  Debug build: ${ENABLE_DEBUG}")
