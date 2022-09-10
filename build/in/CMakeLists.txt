# [general:link]
cmake_minimum_required(
  VERSION 3.16)

# [cmp:135]
cmake_policy(SET CMP0135 NEW)

# [artifact:compdb] [compdp]
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# [fetch]
include(FetchContent)

# [fetch:po] [project_options v0.24.0]
FetchContent_Declare(
  water_river_project_options 
  URL "https://github.com/aminya/project_options/archive/refs/tags/v0.24.0.zip"
  DOWNLOAD_EXTRACT_TIMESTAMP
)
FetchContent_MakeAvailable(water_river_project_options)
include("${water_river_project_options_SOURCE_DIR}/Index.cmake")
include("${water_river_project_options_SOURCE_DIR}/src/DynamicProjectOptions.cmake")

# [deps:boost]
set(BOOST_ENABLE_CMAKE         ON)
set(Boost_USE_MULTITHREADED    ON)
set(Boost_USE_RELEASE_LIBS     ON)
set(Boost_USE_STATIC_LIBS      ON)
if(APPLE)
  set(Boost_USE_STATIC_RUNTIME OFF)
else()
  set(Boost_USE_STATIC_RUNTIME ON)
endif()
FetchContent_Declare(
  Boost
  GIT_REPOSITORY "https://github.com/boostorg/boost.git"
  GIT_TAG boost-1.80.0
  DOWNLOAD_EXTRACT_TIMESTAMP
)
FetchContent_MakeAvailable(Boost)

# [deps:threads]
if(APPLE)
  set(CMAKE_THREAD_LIBS_INIT       -lpthread)
  set(CMAKE_HAVE_THREADS_LIBRARY   1)
  set(CMAKE_USE_WIN32_THREADS_INIT 0)
  set(CMAKE_USE_PTHREADS_INIT      1)
  set(THREADS_PREFER_PTHREAD_FLAG  ON)
  find_package(Threads REQUIRED)
else()
  set(DEPENDENCIES_CONFIGURED Threads)
  foreach(DEPENDENCY ${DEPENDENCIES_CONFIGURED})
    find_package(${DEPENDENCY} CONFIG REQUIRED)
endforeach()
endif()
include("${Boost_SOURCE_DIR}/tools/boost_install/BoostConfig.cmake")

# [dist:pkg:vcpkg:todo] [todo vcpkg]

# [project]
project(
  water.river
  VERSION 0.0.1
  DESCRIPTION "river: a server"
  HOMEPAGE_URL "https://edant.io"
  LANGUAGES CXX
)

# [dist:sha] [sha]
set(GIT_SHA "Unknown" CACHE STRING "SHA this build was generated from")
string(SUBSTRING "${GIT_SHA}" 0 8 GIT_SHORT_SHA)

# [config:po] [project options]
dynamic_project_options(
  ENABLE_CACHE
  ENABLE_INTERPROCEDURAL_OPTIMIZATION
  ENABLE_NATIVE_OPTIMIZATION
  ENABLE_CONAN
  ENABLE_PCH
  PCH_HEADERS
  <vector>
  <string>
)

# [config:files] [configure files]
configure_file(
  ../../include/configured_files/metadata.hpp.in
  ../../include/configured_files/metadata.hpp)

# [target:src]
add_executable("${PROJECT_NAME}"
  ../../src/main.cpp)

# [target:include]
target_include_directories("${PROJECT_NAME}" PRIVATE
  ../../include)

# [build:link]
target_link_libraries("${PROJECT_NAME}" PRIVATE
  project_options
  project_warnings
  Threads::Threads
  Boost::headers
  Boost::program_options
)

set(CPACK_PACKAGE_FILE_NAME_DIST
  "${CMAKE_PROJECT_NAME}-${CMAKE_PROJECT_VERSION}-${GIT_SHORT_SHA}-${CMAKE_SYSTEM_NAME}")
set(CPACK_PACKAGE_FILE_NAME_BUILD
  "${CMAKE_BUILD_TYPE}-${CMAKE_CXX_COMPILER_ID}-${CMAKE_CXX_COMPILER_VERSION}")
set(CPACK_PACKAGE_FILE_NAME
  "${CPACK_PACKAGE_FILE_NAME_DIST}-${CPACK_PACKAGE_FILE_NAME_BUILD}")

include(CPack)

#
# [reference]
#

# [cmp:135] [DOWNLOAD_EXTRACT_TIMESTAMP]
#   - notes:
#     for local settings maybe, use
#     cmake_policy(SET CMP0135 NEW)
#     for global settings maybe, use
#     set(DOWNLOAD_EXTRACT_TIMESTAMP true)
# [fetch] [fetch]
#     - for:
#       FetchContent_Declare
#       FetchContent_MakeAvailable
#       FetchContent_GetProperties
#       FetchContent_Populate
# [fetch:po] [project_options v0.24.0]
#   - for:
#     dynamic_project_options
#     project_options
#     conan
#     cpack
# [deps:boost]
# [dist:pkg:vcpkg] [vcpkg]
#   - notes:
#     - run_vcpkg() should be called before
#       defining project()
#     - to enable, call run_vcpkg() before
#       project()
# [dist:sha] [sha]
# [artifact:compdb] [compdp]
# [config:po] [project options]
#   [ENABLE_PCH]
#     PCH is disabled by default in developer
#     mode because these headers become globally
#     included and they can mask other errors
#   [PCH_HEADERS]
#     This is a list of headers to pre-compile.
#     These are the common ones.
#   [some other options]
#     ${ENABLE_CLANG_TIDY}
#     ${ENABLE_CPPCHECK}
#     ${ENABLE_DOXYGEN}
#     ${ENABLE_COVERAGE}
#     ENABLE_USER_LINKER
#     ENABLE_BUILD_WITH_TIME_TRACE
#     ENABLE_UNITY
#     ENABLE_VS_ANALYSIS
#     WARNINGS_AS_ERRORS
#     passes a profile to conan
#     see https://docs.conan.io/en/latest/reference/profiles.html
#     CONAN_PROFILE ${profile_path}
#     extra options to pass to conan
#     CONAN_OPTIONS
#     override the defaults for the MSVC warnings
#     MSVC_WARNINGS
#     override the defaults for the CLANG warnings
#     CLANG_WARNINGS
#     override the defaults for the GCC warnings
#     GCC_WARNINGS
#     ENABLE_SANITIZER_ADDRESS
#     ENABLE_SANITIZER_UNDEFINED_BEHAVIOR
#     ENABLE_SANITIZER_THREAD
#     ENABLE_SANITIZER_MEMORY
#     ENABLE_INCLUDE_WHAT_YOU_USE
# [config:files] [configure files]
#   - notes:
#     use ESCAPE_QUOTES in the call to
#     configure_file if embedding strings

# [general:link] [useful links]
#   - https://github.com/aminya/project_options
#   - https://github.com/cpp-best-practices/project_options

# [deps:threads] [threads]
#   - notes:
#     assume built-in pthreads on MacOS

# [build:link]
#   could also use (and test with)
#   target_link_system_libraries(...)

# [dist:pkg:todo]
#   TODO
#package_project(
#  TARGETS "${PROJECT_NAME}"
#  # FIXME: this does not work! ED
#  # CONFIG_EXPORT_DESTINATION "${CMAKE_INSTALL_DATADIR}/${NAME}_cmake_package"
#  # CONFIG_INSTALL_DESTINATION "${CMAKE_INSTALL_DATADIR}/${NAME}_cmake_package"
#  # FIXME: this does not work! CK
#  # PRIVATE_DEPENDENCIES_CONFIGURED
#  # project_options project_warnings
#)

