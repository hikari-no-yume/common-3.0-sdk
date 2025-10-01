cmake_minimum_required(VERSION 3.20)

set(CMAKE_ARCHITECTURES "armv6;armv7")

project(libsystem VERSION 1.0.0 LANGUAGES C)

set(LIBSYSTEM_NAME "libSystem.B.dylib")
set(LIBOBJC_NAME "libobjc.A.dylib")
set(INSTALL_PATH /usr/lib)
set(VERSION 1.0.0)
set(COMPAT_VERSION 1.0.0)

set(LIBSYSTEM_SOURCES
    libc_stub.c
)
set(LIBOBJC_SOURCES
    libobjc_stub.c
)

add_library(${LIBSYSTEM_NAME} SHARED ${LIBSYSTEM_SOURCES})
add_library(${LIBOBJC_NAME} SHARED ${LIBSYSTEM_SOURCES})

set_target_properties(${LIBSYSTEM_NAME} PROPERTIES
    OUTPUT_NAME "System.B"
    VERSION ${VERSION}
    MACHO_COMPATIBILITY_VERSION ${COMPAT_VERSION}
    INSTALL_NAME_DIR "${INSTALL_PATH}"
    POSITION_INDEPENDENT_CODE OFF
    C_STANDARD 99
)
set_target_properties(${LIBOBJC_NAME} PROPERTIES
    OUTPUT_NAME "objc.A"
    VERSION ${VERSION}
    MACHO_COMPATIBILITY_VERSION ${COMPAT_VERSION}
    INSTALL_NAME_DIR "${INSTALL_PATH}"
    POSITION_INDEPENDENT_CODE OFF
    C_STANDARD 99
)

target_compile_options(${LIBSYSTEM_NAME} 
    PRIVATE
        -mthumb
        -Xclang
        -mframe-pointer=none
        -nostdlib
        -fno-builtin
        -fno-stack-protector
        -Wno-deprecated-non-prototype
        -Wno-expansion-to-defined 
)
target_compile_options(${LIBOBJC_NAME} 
    PRIVATE
        -mthumb
        -Xclang
        -mframe-pointer=none
        -nostdlib
        -fno-builtin
        -fno-stack-protector
        -Wno-deprecated-non-prototype
        -Wno-expansion-to-defined 
)

target_link_options(${LIBSYSTEM_NAME}
    PRIVATE
        -dead_strip
        -dead_strip_dylibs
        -dylib
        -single_module
        -dynamic
        -twolevel_namespace
        -twolevel_namespace_hints
        -install_name
        ${INSTALL_PATH}/${LIBSYSTEM_NAME}
        -nostdlib
)
target_link_options(${LIBOBJC_NAME}
    PRIVATE
        -dead_strip
        -dead_strip_dylibs
        -dylib
        -single_module
        -dynamic
        -twolevel_namespace
        -twolevel_namespace_hints
        -install_name
        ${INSTALL_PATH}/${LIBOBJC_NAME}
        -nostdlib
)

install(TARGETS ${LIBSYSTEM_NAME}
    LIBRARY DESTINATION lib
)
install(TARGETS ${LIBOBJC_NAME}
    LIBRARY DESTINATION lib
)

# Also install as libSystem.dylib.
# Ideally, this would be symlinked but that's flaky on windows.
install(FILES $<TARGET_FILE:${LIBSYSTEM_NAME}>
    DESTINATION lib
    RENAME libSystem.dylib
)
install(FILES $<TARGET_FILE:${LIBOBJC_NAME}>
    DESTINATION lib
    RENAME libobjc.dylib
)
