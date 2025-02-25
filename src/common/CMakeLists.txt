file(GLOB data_files "${CMAKE_SOURCE_DIR}/data/*")
add_custom_command(OUTPUT _non_existant_file_so_we_always_run.txt "${CMAKE_SOURCE_DIR}/out/data.zip"
    WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/data"
    COMMAND ${CMAKE_COMMAND} -E tar "cvf" "${CMAKE_SOURCE_DIR}/out/data.zip" --format=zip .)
add_custom_target(data_zip
    DEPENDS _non_existant_file_so_we_always_run.txt)
if(CMAKE_SYSTEM_NAME STREQUAL Android OR CMAKE_SYSTEM_NAME STREQUAL iOS OR CMAKE_SYSTEM_NAME STREQUAL Emscripten)
    if (CMAKE_HOST_WIN32)
        add_custom_target(embedded_data_zip_header
            DEPENDS data_zip
            COMMAND ${CMAKE_SOURCE_DIR}/tools/cembed/win64/cembed.exe ${CMAKE_SOURCE_DIR}/out/data.zip > ${CMAKE_SOURCE_DIR}/src/common/data_zip.h)
    elseif(CMAKE_HOST_APPLE)
        add_custom_target(embedded_data_zip_header
            DEPENDS data_zip
            COMMAND ${CMAKE_SOURCE_DIR}/tools/cembed/osx/cembed ${CMAKE_SOURCE_DIR}/out/data.zip > ${CMAKE_SOURCE_DIR}/src/common/data_zip.h)
    elseif(CMAKE_HOST_UNIX)
        add_custom_target(embedded_data_zip_header
            DEPENDS data_zip
            COMMAND ${CMAKE_SOURCE_DIR}/tools/cembed/linux/cembed ${CMAKE_SOURCE_DIR}/out/data.zip > ${CMAKE_SOURCE_DIR}/src/common/data_zip.h)
    endif()
endif()

add_library(common STATIC
    alloc.c
    audio.c
    base64.c
    bvh.c
    data.c
    debug_console.c
    file.c
    graphics.c
    inputs.c
    json.c
    level.c
    log.c
    level.c
    map.c
    maths.c
    script.c
    storage.c
    string.c
    thread.c
    vec.c)


if(CMAKE_SYSTEM_NAME STREQUAL Windows)
    target_compile_options(common PRIVATE /W3)
elseif(CMAKE_SYSTEM_NAME STREQUAL Linux)
    target_compile_options(common PRIVATE -Wall -Wextra -Wpedantic)
elseif(CMAKE_SYSTEM_NAME STREQUAL iOS)
    target_compile_options(common PRIVATE -x objective-c)
endif()

if(CMAKE_SYSTEM_NAME STREQUAL Android OR CMAKE_SYSTEM_NAME STREQUAL iOS OR CMAKE_SYSTEM_NAME STREQUAL Emscripten)
    add_dependencies(common embedded_data_zip_header)
endif()
