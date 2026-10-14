cmake_minimum_required(VERSION 3.17)

set(CMAKE_DEBUG_POSTFIX d)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)
set(POSITION_INDEPENDENT_CODE ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

if(EMSCRIPTEN)
    set(CMAKE_FIND_ROOT_PATH ${INSTALL_PATH}/wasm/cmake/)
endif()

find_package(PkgConfig REQUIRED)

if(WIN32)
	set(CMAKE_PREFIX_PATH "${INSTALL_PATH}")
    find_package(spdlog REQUIRED)
elseif(NOT EMSCRIPTEN)
    find_package(spdlog REQUIRED PATHS ${INSTALL_PATH}/lib/cmake/)
endif()

add_library(${PROJECT_NAME}
    logger.cpp
    logger.h
)

if(WIN32)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /utf-8 /MP")
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -g")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -g")
else()
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2")
endif()

if(EMSCRIPTEN)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -s USE_PTHREADS=1")
    target_compile_options(${PROJECT_NAME} PUBLIC -Wno-deprecated-literal-operator)
endif()

if(NOT WIN32)
    target_link_libraries(${PROJECT_NAME} PRIVATE 
       stdc++
       pthread
    )
else()
    target_include_directories(${PROJECT_NAME} PRIVATE ${INSTALL_PATH}/include)
endif()

target_include_directories(${PROJECT_NAME} PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    ${INSTALL_PATH}/include
)

target_include_directories(${PROJECT_NAME} PUBLIC
    $<INSTALL_INTERFACE:${INSTALL_PATH}/include>
)

if(EMSCRIPTEN)
    install(
        TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}
        DESTINATION ${INSTALL_PATH}/wasm
        COMPONENT ${PROJECT_NAME}
    )
else()
    install(
        TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}
        DESTINATION ${INSTALL_PATH}/lib
        COMPONENT ${PROJECT_NAME}
    )
endif()

if(NOT EMSCRIPTEN)
    if(WIN32)
        set(YUTOVO_DEBUG_LIBRARY "${PROJECT_NAME}d.lib")
        set(YUTOVO_RELEASE_LIBRARY "${PROJECT_NAME}.lib")
    else()
        set(YUTOVO_DEBUG_LIBRARY "lib${PROJECT_NAME}d.a")
        set(YUTOVO_RELEASE_LIBRARY "lib${PROJECT_NAME}.a")
    endif()

    file(GENERATE
        OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-release.cmake"
        CONTENT "set_property(TARGET ${PROJECT_NAME} APPEND PROPERTY IMPORTED_CONFIGURATIONS RELEASE)
        set_target_properties(${PROJECT_NAME} PROPERTIES
            IMPORTED_LINK_INTERFACE_LANGUAGES_RELEASE \"CXX\"
            IMPORTED_LOCATION_RELEASE \"${INSTALL_PATH}/lib/${YUTOVO_RELEASE_LIBRARY}\")"
    )

    install(
        FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-release.cmake"
        DESTINATION ${INSTALL_PATH}/lib/cmake/${PROJECT_NAME}
        COMPONENT ${PROJECT_NAME}
    )

    if(CMAKE_BUILD_TYPE STREQUAL "Debug" OR DEFINED CMAKE_CONFIGURATION_TYPES)
        install(
            FILES $<TARGET_FILE:${PROJECT_NAME}>
            DESTINATION ${INSTALL_PATH}/lib
            RENAME ${YUTOVO_RELEASE_LIBRARY}
            COMPONENT ${PROJECT_NAME}
        )
    endif()
endif()

install(
    FILES 
        logger.h
    DESTINATION ${INSTALL_PATH}/include/${PROJECT_NAME}
    COMPONENT ${PROJECT_NAME}
)

if(EMSCRIPTEN)
    install(
        EXPORT ${PROJECT_NAME}
        DESTINATION ${INSTALL_PATH}/wasm/cmake/${PROJECT_NAME}
        COMPONENT ${PROJECT_NAME}
        FILE ${PROJECT_NAME}.cmake
    )
else()
    install(
        EXPORT ${PROJECT_NAME}
        DESTINATION ${INSTALL_PATH}/lib/cmake/${PROJECT_NAME}
        COMPONENT ${PROJECT_NAME}
        FILE ${PROJECT_NAME}.cmake
    )
endif()

if(EMSCRIPTEN)
    install(
        FILES ${PROJECT_NAME}-config.cmake
        DESTINATION ${INSTALL_PATH}/wasm/cmake/${PROJECT_NAME}
        COMPONENT ${PROJECT_NAME}
    )
else()
    install(
        FILES ${PROJECT_NAME}-config.cmake
        DESTINATION ${INSTALL_PATH}/lib/cmake/${PROJECT_NAME}
        COMPONENT ${PROJECT_NAME}
    )
endif()
