cmake_minimum_required( VERSION 3.1 )

include( CMakePackageConfigHelpers )

project ( cereal VERSION 1.2.2 )

option(SKIP_PORTABILITY_TEST "Skip portability (32 bit) tests" OFF)
option(JUST_INSTALL_CEREAL "Don't do anything besides installing the library" OFF)
option(THREAD_SAFE "Use mutexes to ensure thread safety" OFF)

add_library( cereal INTERFACE)
add_library( cereal::cereal ALIAS cereal )

if(THREAD_SAFE)
  find_package( Threads )
  target_link_libraries( cereal ${CMAKE_THREAD_LIBS_INIT} )
  target_compile_definitions( cereal -DCEREAL_THREAD_SAFE=1 )
endif()

if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /bigobj /W3 /WX")
else()
    set(CMAKE_CXX_FLAGS "-Wall -g -Wextra -Wshadow -pedantic -Wold-style-cast ${CMAKE_CXX_FLAGS}")
    option(WITH_WERROR "Compile with '-Werror' C++ compiler flag" ON)
    if(WITH_WERROR)
        set(CMAKE_CXX_FLAGS "-Werror ${CMAKE_CXX_FLAGS}")
    endif(WITH_WERROR)

    if(NOT DEFINED CMAKE_CXX_STANDARD OR CMAKE_CXX_STANDARD STREQUAL "98")
      set(CMAKE_CXX_STANDARD 11)
    endif()

    set(CMAKE_CXX_STANDARD_REQUIRED ON)

endif()

target_include_directories(cereal INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/include>
)

# Only exports cereal target to cereal-targets
install( TARGETS cereal EXPORT cereal-targets )

install( DIRECTORY include/cereal DESTINATION include COMPONENT dev )

# Create package build configuration in build directory

export( EXPORT cereal-targets
  FILE
    "${CMAKE_CURRENT_BINARY_DIR}/cereal-targets.cmake"
  NAMESPACE
    cereal::
)

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/cereal-config-version.cmake"
  VERSION
    ${${PROJECT_NAME}_VERSION}
  COMPATIBILITY
    AnyNewerVersion
)

configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cereal-config.cmake.in"
  "cereal-config.cmake"
  INSTALL_DESTINATION
    "${CMAKE_CURRENT_BINARY_DIR}"
)

# Export build configuration into local cmake repository

export( PACKAGE cereal )

# Install package configuration for installed targets

if(WIN32 AND NOT CYGWIN)
  set(CEREAL_PACKAGE_CONFIG_DIR CMake)
else()
  set(CEREAL_PACKAGE_CONFIG_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/cereal)
endif()

install( EXPORT cereal-targets
  DESTINATION
    ${CEREAL_PACKAGE_CONFIG_DIR}
  NAMESPACE
    cereal::
  COMPONENT
    dev
)

install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/cereal-config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/cereal-config-version.cmake"
  DESTINATION
    ${CEREAL_PACKAGE_CONFIG_DIR}
  COMPONENT
    dev
)

if(JUST_INSTALL_CEREAL)
    return()
endif()

# Boost serialization for performance sandbox
find_package(Boost COMPONENTS serialization)

if(Boost_FOUND)
  include_directories(SYSTEM ${Boost_INCLUDE_DIRS})
endif(Boost_FOUND)

enable_testing()
add_subdirectory(unittests)

add_subdirectory(sandbox)

add_subdirectory(doc)
