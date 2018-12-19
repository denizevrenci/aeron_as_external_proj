cmake_minimum_required(VERSION 3.13 FATAL_ERROR)

project(aeron_test)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_EXTENSIONS OFF)

include(ExternalProject)

ExternalProject_Add(
	aeron_proj
	GIT_REPOSITORY https://github.com/real-logic/aeron.git
	GIT_TAG de2ccd4a83ef9562554f94bb86bdab1385eeda76
	CMAKE_ARGS -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER};-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER};
		-DCMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD};-DCMAKE_CXX_EXTESIONS=${CMAKE_CXX_EXTENSIONS};
		-DAERON_TESTS=OFF;-DAERON_BUILD_SAMPLES=OFF;-DAERON_BUILD_DOCUMENTATION=OFF;-DAERON_INSTALL_TARGETS=OFF
	INSTALL_COMMAND ""
)

ExternalProject_Get_property(aeron_proj SOURCE_DIR)
ExternalProject_Get_property(aeron_proj BINARY_DIR)

add_library(aeron_client STATIC IMPORTED)

add_dependencies(aeron_client aeron_proj)

set_target_properties(aeron_client PROPERTIES
	IMPORTED_LOCATION "${BINARY_DIR}/lib/libaeron_client.a"
)

# Need to create this directory in configure time to set target_include_directories.
file(MAKE_DIRECTORY "${SOURCE_DIR}/aeron-client/src/main/cpp/")
target_include_directories(aeron_client SYSTEM
	INTERFACE "${SOURCE_DIR}/aeron-client/src/main/cpp/"
)

target_compile_definitions(aeron_client
	INTERFACE "__PROJECT_SOURCE_DIR__=\"${SOURCE_DIR}\""
)

target_link_libraries(aeron_client
	INTERFACE ${CMAKE_THREAD_LIBS_INIT}
)

add_executable(BasicPublisher BasicPublisher.cpp)
target_link_libraries(BasicPublisher aeron_client)

add_executable(BasicSubscriber BasicSubscriber.cpp)
target_link_libraries(BasicSubscriber aeron_client)
