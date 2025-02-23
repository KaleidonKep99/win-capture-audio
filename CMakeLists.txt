cmake_minimum_required(VERSION 3.20)
project(win-capture-audio VERSION 2.0.0)

set(PLUGIN_AUTHOR "bozbez")
set(RELEASE_DIR "${PROJECT_SOURCE_DIR}/release")

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(ARCH_NAME "64bit")
else()
    set(ARCH_NAME "32bit")
endif()

configure_file(
    installer/installer.iss.in
    ../installer/installer.generated.iss
)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")
find_package(LibObs REQUIRED)

set(win-capture-audio_SOURCES
    src/plugin.c
    src/audio-capture.c
    src/obfuscate.c
    src/window-helpers.c)

add_library(win-capture-audio MODULE ${win-capture-audio_SOURCES})
target_link_libraries(win-capture-audio libobs dwmapi psapi)

add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND if $<OR:$<CONFIG:Release>,$<CONFIG:RelWithDebInfo>>==1 (
        "${CMAKE_COMMAND}" -E make_directory
            "${RELEASE_DIR}/data/obs-plugins/${CMAKE_PROJECT_NAME}"
            "${RELEASE_DIR}/obs-plugins/${ARCH_NAME}"
    )

    COMMAND if $<OR:$<CONFIG:Release>,$<CONFIG:RelWithDebInfo>>==1 (
        "${CMAKE_COMMAND}" -E copy_directory
            "${PROJECT_SOURCE_DIR}/data"
            "${RELEASE_DIR}/data/obs-plugins/${CMAKE_PROJECT_NAME}"
    )

    COMMAND if $<OR:$<CONFIG:Release>,$<CONFIG:RelWithDebInfo>>==1 (
        "${CMAKE_COMMAND}" -E copy
            "$<TARGET_FILE:${CMAKE_PROJECT_NAME}>"
            "${RELEASE_DIR}/obs-plugins/${ARCH_NAME}"
    )

    COMMAND if $<CONFIG:RelWithDebInfo>==1 (
        "${CMAKE_COMMAND}" -E copy
            "$<TARGET_FILE_BASE_NAME:${CMAKE_PROJECT_NAME}>.pdb"
            "${RELEASE_DIR}/obs-plugins/${ARCH_NAME}"
    )

    VERBATIM
)

function(copy_helper_lib)
    set(DATA_DIR "${RELEASE_DIR}/data/obs-plugins/${CMAKE_PROJECT_NAME}")
    add_custom_command(TARGET ${ARGV0} POST_BUILD
        COMMAND if $<OR:$<CONFIG:Release>,$<CONFIG:RelWithDebInfo>>==1 (
            "${CMAKE_COMMAND}" -E make_directory
                "${RELEASE_DIR}/data/obs-plugins/${CMAKE_PROJECT_NAME}"
        )

        COMMAND if $<OR:$<CONFIG:Release>,$<CONFIG:RelWithDebInfo>>==1 (
            "${CMAKE_COMMAND}" -E copy
                "$<TARGET_FILE:${ARGV0}>"
                "${RELEASE_DIR}/data/obs-plugins/${CMAKE_PROJECT_NAME}"
        )

        COMMAND if $<CONFIG:RelWithDebInfo>==1 (
            "${CMAKE_COMMAND}" -E copy
                "$<TARGET_FILE_BASE_NAME:${ARGV0}>.pdb"
                "${RELEASE_DIR}/data/obs-plugins/${CMAKE_PROJECT_NAME}"
        )

        VERBATIM
    )
endfunction()

add_subdirectory(src/audio-capture-helper)