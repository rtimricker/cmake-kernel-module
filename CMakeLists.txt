cmake_minimum_required(VERSION 3.0.0 FATAL_ERROR)

project(driver VERSION 0.1.0 LANGUAGES C)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)


# Module info
add_definitions(-D__KERNEL__ -DMODULE)

set (KERNEL_HEADERS_DIR linux-headers-5.10.0-18-common)
# Find the kernel release
execute_process(
        COMMAND echo ${KERNEL_HEADERS_DIR}
        OUTPUT_VARIABLE KERNEL_RELEASE
        OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Find the headers
find_path(
        KERNEL_HEADERS_DIR
        include/linux/user.h
        PATHS /usr/src/linux-headers-${KERNEL_RELEASE}
)

message(STATUS "===> Kernel release: ${KERNEL_RELEASE}")
message(STATUS "===> Kernel headers: ${KERNEL_HEADERS_DIR}")

include_directories(
        ${KERNEL_HEADERS_DIR}/include
        ${KERNEL_HEADERS_DIR}
)


set(DRIVER_FILE hello.ko)
set(KBUILD_CMD $(MAKE) -C ${KERNEL_HEADERS_DIR} modules M=${CMAKE_CURRENT_BINARY_DIR} src=${CMAKE_CURRENT_SOURCE_DIR})

# Generate the Kbuild file through cmake.
file(WRITE ${CMAKE_CURRENT_SOURCE_DIR}/Kbuild "obj-m := hello.o")

add_custom_command(OUTPUT ${DRIVER_FILE}
        COMMAND ${KBUILD_CMD}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        DEPENDS hello.c VERBATIM)
#
add_custom_target(driver ALL DEPENDS ${DRIVER_FILE})

