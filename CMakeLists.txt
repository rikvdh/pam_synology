cmake_minimum_required(VERSION 2.8.7)

project(pam_synology C)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(PROJECT_VERSION 0.1.0)

option(ENABLE_CODECOVERAGE "ENABLE_CODECOVERAGE" OFF)

include(CTest)
include(CheckIncludeFiles)
include("cmake/add_cflag_if_supported.cmake")
include("cmake/lint.cmake")
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")

if(${ENABLE_CODECOVERAGE} MATCHES "ON")
	message(STATUS "Code-coverage enabled")
	set(CMAKE_C_FLAGS "--coverage -fprofile-arcs -ftest-coverage")
	set(CMAKE_CXX_FLAGS "--coverage -fprofile-arcs -ftest-coverage")
endif()

add_cflag_if_supported("-std=c99")
add_cflag_if_supported("-Wall")
add_cflag_if_supported("-Wextra")
add_cflag_if_supported("-Wshadow")
add_cflag_if_supported("-D_FORTIFY_SOURCE=2")
add_cflag_if_supported("-fstrict-aliasing=1")
add_cflag_if_supported("-Wformat")
add_cflag_if_supported("-Wformat-security")
add_cflag_if_supported("-Wno-sign-conversion")
add_cflag_if_supported("-Wconversion")
add_cflag_if_supported("-Wmisleading-indentation")
add_cflag_if_supported("-Wmaybe-uninitialized")
add_cflag_if_supported("-Wmissing-variable-declarations")
add_cflag_if_supported("-Wshorten-64-to-32")
add_cflag_if_supported("-Wimplicit-function-declaration")
add_cflag_if_supported("-Wredundant-decls")
add_cflag_if_supported("-Wundef")
add_cflag_if_supported("-Wswitch")
add_cflag_if_supported("-Wfloat-equal")
add_cflag_if_supported("-Werror")
add_cflag_if_supported("-Wl,-z,relro")
add_cflag_if_supported("-Wl,-z,now")
add_cflag_if_supported("-fPIC")

if(${CMAKE_BUILD_TYPE} MATCHES "Debug")
    include(CTest)
    add_cflag_if_supported("-ggdb")
    add_cflag_if_supported("-O0")
    add_definitions(-DDEBUG_BUILD)
    enable_linting()
elseif()
    add_cflag_if_supported("-O2")
endif()

find_package(CURL REQUIRED)
find_package(ZLIB REQUIRED)
find_package(OpenSSL REQUIRED)

check_include_files(pam_appl.h HAVE_PAM_APPL_H)
check_include_files(security/pam_appl.h HAVE_SECURITY_PAM_APPL_H)
check_include_files(pam_modules.h HAVE_PAM_MODULES_H)
check_include_files(security/pam_modules.h HAVE_SECURITY_PAM_MODULES_H)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/include/config.h.in ${CMAKE_CURRENT_BINARY_DIR}/config.h)

include_directories(
	${OPENSSL_INCLUDE_DIR}
	${CMAKE_CURRENT_BINARY_DIR})

set(CFILES
	src/pam_synology.c
	src/stdlib_wrapper.c)

add_library(${PROJECT_NAME} SHARED ${CFILES})
target_link_libraries(${PROJECT_NAME} ${CURL_LIBRARIES} ${ZLIB_LIBRARIES} ${OPENSSL_CRYPTO_LIBRARY})

install(TARGETS ${PROJECT_NAME} DESTINATION /lib/security)
