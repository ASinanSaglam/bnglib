cmake_minimum_required(VERSION 2.8.4)

project(libbng)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../libs/nauty ${CMAKE_CURRENT_BINARY_DIR}/nauty)

include_directories(
	"${CMAKE_CURRENT_SOURCE_DIR}"
	"${CMAKE_CURRENT_SOURCE_DIR}/.."
	"${CMAKE_CURRENT_BINARY_DIR}/deps"
	"${CMAKE_CURRENT_SOURCE_DIR}/../libs/sparsehash/src/"
	"${CMAKE_CURRENT_SOURCE_DIR}/../libs/"
)

add_definitions(-DYY_NO_UNISTD_H)

if (NOT WIN32) 
  add_definitions(-std=c++17)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-function -Wall -isystem ${CMAKE_SOURCE_DIR}/../libs/ -isystem ${CMAKE_SOURCE_DIR}/../libs/sparsehash/src")
endif()

set(SOURCE_FILES
  shared_defines.cpp
	ast.cpp
	base_flag.cpp
	bng_data.cpp
	bng_engine.cpp
	bng_config.cpp
	cplx.cpp
	elem_mol.cpp
	elem_mol_type.cpp
	parser_utils.cpp
	parser.cpp
	rxn_class.cpp
	rxn_container.cpp
	rxn_rule.cpp
	rxn_compartment_utils.cpp
	species.cpp
	species_container.cpp
	semantic_analyzer.cpp
	graph.cpp
	filesystem_utils.cpp
)


# check for bison/flex and set up code gen
find_package(BISON)
find_package(FLEX)
BISON_TARGET(bngl_parser ${CMAKE_CURRENT_SOURCE_DIR}/bngl_parser.y
  ${CMAKE_CURRENT_BINARY_DIR}/deps/bngl_parser.cpp)

# the cmake flex plugin requires the flex file to be in the 'deps' directory
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/bngl_scanner.l ${CMAKE_CURRENT_BINARY_DIR}/deps/bngl_scanner.l COPYONLY)

FLEX_TARGET(bngl_scanner ${CMAKE_CURRENT_SOURCE_DIR}/bngl_scanner.l
  ${CMAKE_CURRENT_BINARY_DIR}/deps/bngl_scanner.cpp
  COMPILE_FLAGS "--header-file=${CMAKE_CURRENT_BINARY_DIR}/deps/bngl_scanner.hpp"
)


# TODO: add bngl_scanner.l|y dependencies
  
ADD_FLEX_BISON_DEPENDENCY(bngl_scanner bngl_parser bngl_l nauty)

set(ALL_SOURCES
    ${SOURCE_FILES}
    ${BISON_bngl_parser_OUTPUTS}
    ${FLEX_bngl_scanner_OUTPUTS}    
)


if (NOT WIN32 AND NOT APPLE)
    set(STDC_FS stdc++fs)
endif()

add_library(${PROJECT_NAME} ${ALL_SOURCES})
target_link_libraries(${PROJECT_NAME}
	PRIVATE nauty ${STDC_FS}
)

add_executable(parser_tester_${PROJECT_NAME}
  ${ALL_SOURCES}
	test.cpp # only for development
)

target_link_libraries(parser_tester_${PROJECT_NAME}
	PUBLIC nauty ${STDC_FS}
)
