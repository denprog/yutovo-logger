cmake_minimum_required(VERSION 3.17)

set(CMAKE_DEBUG_POSTFIX d)

find_package(PkgConfig REQUIRED)

add_library(${PROJECT_NAME}
    logger.cpp
)

set_target_properties(${PROJECT_NAME} PROPERTIES
	CXX_STANDARD 20
	CXX_STANDARD_REQUIRED ON
	CXX_EXTENSIONS OFF
	POSITION_INDEPENDENT_CODE ON
    CMAKE_INCLUDE_CURRENT_DIR ON
)

target_link_libraries(${PROJECT_NAME} PRIVATE 
    stdc++
    pthread
)

target_include_directories(${PROJECT_NAME} PRIVATE
    ../third_party
)

install(
    TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}
    DESTINATION ${INSTALL_PATH}/lib
    COMPONENT ${PROJECT_NAME}
)

install(
    FILES 
        logger.h
    DESTINATION ${INSTALL_PATH}/include/${PROJECT_NAME}
    COMPONENT ${PROJECT_NAME}
)

install(
    DIRECTORY 
        ../third_party/spdlog
    DESTINATION ${INSTALL_PATH}/include
    COMPONENT ${PROJECT_NAME}
)

install(
    EXPORT ${PROJECT_NAME}
    DESTINATION ${INSTALL_PATH}/lib/cmake/${PROJECT_NAME}
    COMPONENT ${PROJECT_NAME}
    FILE ${PROJECT_NAME}.cmake
)

install(
    FILES ${PROJECT_NAME}-config.cmake
    DESTINATION ${INSTALL_PATH}/lib/cmake/${PROJECT_NAME}
    COMPONENT ${PROJECT_NAME}
)
