cmake_minimum_required(VERSION 3.19)

file(READ "${CMAKE_CURRENT_SOURCE_DIR}/wyvrn.json" WYVRN_JSON)
string(JSON WYVRN_PROJECT_VERSION GET "${WYVRN_JSON}" version)

project(asio VERSION ${WYVRN_PROJECT_VERSION} LANGUAGES CXX)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

option(ASIO_USE_BOOST     "Build Asio with Boost integration (otherwise standalone)" OFF)
option(ASIO_NO_DEPRECATED "Disable deprecated Asio API surface"                      OFF)

add_library(asio INTERFACE)
add_library(asio::asio ALIAS asio)

target_include_directories(asio INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

if(NOT ASIO_USE_BOOST)
    target_compile_definitions(asio INTERFACE ASIO_STANDALONE)
endif()

if(ASIO_NO_DEPRECATED)
    target_compile_definitions(asio INTERFACE ASIO_NO_DEPRECATED)
endif()

if(WIN32)
    target_link_libraries(asio INTERFACE ws2_32 wsock32 mswsock)
endif()

if(THREADS_HAVE_PTHREAD_ARG)
    target_compile_options(asio INTERFACE "-pthread")
endif()

install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/"
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING
        PATTERN "*.hpp"
        PATTERN "*.ipp"
        PATTERN "Makefile.am" EXCLUDE)

install(TARGETS asio EXPORT asioTargets)

install(EXPORT asioTargets
    FILE asioTargets.cmake
    NAMESPACE asio::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/asio)

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/asioConfig.cmake"
    "include(\"\${CMAKE_CURRENT_LIST_DIR}/asioTargets.cmake\")\n")

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/asioConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
    ARCH_INDEPENDENT)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/asioConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/asioConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/asio)
