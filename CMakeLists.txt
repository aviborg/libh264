cmake_minimum_required(VERSION 3.15)
project(h264 VERSION 1.0)


set(LIB_SOURCES
      src/h264.c
      src/h264_bac.c
      src/h264_bitstream.c
      src/h264_cabac.c
      src/h264_cabac_ctx_tables.c
      src/h264_ctx.c
      src/h264_dump.c
      src/h264_fmo.c
      src/h264_macroblock.c
      src/h264_reader.c
      src/h264_slice_data.c
      src/h264_types.c
      src/h264_writer.c
)

add_library(${PROJECT_NAME} SHARED ${LIB_SOURCES})

# checks if set up rpath exists for install
if(COMMAND set_up_rpath)
    set_up_rpath()
else()
    message("Set up rpath not defined!")
endif()

option(BUILD_SHARED_LIBS "Build using shared libraries" ON)
set_target_properties(${PROJECT_NAME} PROPERTIES
                    POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS}
)

target_include_directories(${PROJECT_NAME} PUBLIC
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                            $<INSTALL_INTERFACE:include>
)

target_compile_definitions(${PROJECT_NAME} PRIVATE "H264_API_EXPORTS")
target_compile_definitions(${PROJECT_NAME} PRIVATE "_GNU_SOURCE")
target_compile_options(${PROJECT_NAME} PRIVATE "-fvisibility=hidden")
target_compile_options(${PROJECT_NAME} PRIVATE "-std=gnu99")


find_package(json-c CONFIG REQUIRED)
if(WIN32)
    target_link_libraries(${PROJECT_NAME} 
                      PRIVATE
                      json-c::json-c 
                      ulog 
                      ws2_32)

    install(CODE "file(GLOB dll_files \"${CMAKE_CURRENT_BINARY_DIR}/*.dll\")")
    install(CODE "file(INSTALL DESTINATION \"\$\{CMAKE_INSTALL_PREFIX\}/bin\" TYPE SHARED_LIBRARY FILES \$\{dll_files\})")

else()
  target_link_libraries(${PROJECT_NAME} 
                      PRIVATE
                      json-c::json-c
                      ulog)

    install(CODE [[
        file(GET_RUNTIME_DEPENDENCIES
            LIBRARIES $<TARGET_FILE:h264>
            RESOLVED_DEPENDENCIES_VAR _r_deps
            UNRESOLVED_DEPENDENCIES_VAR _u_deps
            POST_EXCLUDE_REGEXES "^/lib" "^/usr/lib"
        )
        foreach(_file ${_r_deps})
            file(INSTALL
            DESTINATION "${CMAKE_INSTALL_PREFIX}/lib"
            TYPE SHARED_LIBRARY
            FOLLOW_SYMLINK_CHAIN
            FILES "${_file}"
            )
        endforeach()
    ]])

endif()



set(${PROJECT_NAME}-headers
    ${PROJECT_SOURCE_DIR}/include/${PROJECT_NAME}/
)


install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}-targets
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

# to get all header files correctly
install(
    DIRECTORY ${${PROJECT_NAME}-headers} DESTINATION include/${PROJECT_NAME}
)

