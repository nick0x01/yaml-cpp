cmake_minimum_required(VERSION 2.6)

include(CheckCXXCompilerFlag)

project(YAML_CPP)

set(YAML_CPP_VERSION_MAJOR "0")
set(YAML_CPP_VERSION_MINOR "6")
set(YAML_CPP_VERSION_PATCH "2")
set(YAML_CPP_VERSION "${YAML_CPP_VERSION_MAJOR}.${YAML_CPP_VERSION_MINOR}.${YAML_CPP_VERSION_PATCH}")

# From http://www.cmake.org/pipermail/cmake/2010-March/035992.html:
# function to collect all the sources from sub-directories
# into a single list
function(add_sources)
  get_property(is_defined GLOBAL PROPERTY SRCS_LIST DEFINED)
  if(NOT is_defined)
    define_property(GLOBAL PROPERTY SRCS_LIST
      BRIEF_DOCS "List of source files"
      FULL_DOCS "List of all source files in the entire project")
  endif()
  # make absolute paths
  set(SRCS)
  foreach(s IN LISTS ARGN)
    if(NOT IS_ABSOLUTE "${s}")
      get_filename_component(s "${s}" ABSOLUTE)
    endif()
    list(APPEND SRCS "${s}")
  endforeach()
  # append to global list
  set_property(GLOBAL APPEND PROPERTY SRCS_LIST "${SRCS}")
endfunction(add_sources)

set(header_directory "include/yaml-cpp/")

file(GLOB sources "src/[a-zA-Z]*.cpp")
file(GLOB_RECURSE public_headers "include/yaml-cpp/[a-zA-Z]*.h")
file(GLOB private_headers "src/[a-zA-Z]*.h")

add_definitions(-DYAML_CPP_NO_CONTRIB)

set(library_sources
  ${sources}
  ${public_headers}
  ${private_headers}
  ${contrib_sources}
  ${contrib_public_headers}
  ${contrib_private_headers}
)
add_sources(${library_sources})

if(VERBOSE)
	message(STATUS "sources: ${sources}")
	message(STATUS "public_headers: ${public_headers}")
	message(STATUS "private_headers: ${private_headers}")
	message(STATUS "contrib_sources: ${contrib_sources}")
	message(STATUS "contrib_public_headers: ${contrib_public_headers}")
	message(STATUS "contrib_private_headers: ${contrib_private_headers}")
endif()

include_directories(${YAML_CPP_SOURCE_DIR}/src)
include_directories(${YAML_CPP_SOURCE_DIR}/include)

SET(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")

set(yaml_cxx_flags ${CMAKE_CXX_FLAGS})

set(CMAKE_STATIC_LIBRARY_PREFIX "lib")	# to distinguish static libraries from DLL import libs

### Project stuff
# /W3 = set warning level; see http://msdn.microsoft.com/en-us/library/thxezb7y.aspx
# /wd4127 = disable warning C4127 "conditional expression is constant"; see http://msdn.microsoft.com/en-us/library/6t66728h.aspx
# /wd4355 = disable warning C4355 "'this' : used in base member initializer list"; http://msdn.microsoft.com/en-us/library/3c594ae3.aspx
set(yaml_cxx_flags "/W3 /wd4127 /wd4355 ${yaml_cxx_flags}")

add_library(yaml-cpp ${library_sources})

set_target_properties(yaml-cpp PROPERTIES
	VERSION "${YAML_CPP_VERSION}"
	SOVERSION "${YAML_CPP_VERSION_MAJOR}.${YAML_CPP_VERSION_MINOR}"
	PROJECT_LABEL "yaml-cpp"
)
