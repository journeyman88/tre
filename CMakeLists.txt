cmake_minimum_required(VERSION 3.10.0)
project(tre)
set(TRE_VERSION 0.8.0)

include(CheckIncludeFile)
include(CheckFunctionExists)
include(CheckTypeSize)

CHECK_FUNCTION_EXISTS("alloca" HAVE_ALLOCA)
CHECK_FUNCTION_EXISTS("CFLocaleCopyCurrent" HAVE_CFLOCALECOPYCURRENT)
CHECK_FUNCTION_EXISTS("CFPreferencesCopyAppValue" HAVE_CFPREFERENCESCOPYAPPVALUE)
CHECK_FUNCTION_EXISTS("getopt_long" HAVE_GETOPT_LONG)
CHECK_FUNCTION_EXISTS("isascii" HAVE_ISASCII)
CHECK_FUNCTION_EXISTS("isblank" HAVE_ISBLANK)
CHECK_FUNCTION_EXISTS("iswascii" HAVE_ISWASCII)
CHECK_FUNCTION_EXISTS("iswblank" HAVE_ISWBLANK)
CHECK_FUNCTION_EXISTS("iswctype" HAVE_ISWCTYPE)
CHECK_FUNCTION_EXISTS("iswlower" HAVE_ISWLOWER)
CHECK_FUNCTION_EXISTS("iswupper" HAVE_ISWUPPER)
CHECK_FUNCTION_EXISTS("dcgettext" HAVE_DCGETTEXT)
CHECK_FUNCTION_EXISTS("gettext" HAVE_GETTEXT)
CHECK_FUNCTION_EXISTS("iconv" HAVE_ICONV)
CHECK_FUNCTION_EXISTS("mbrtowc" HAVE_MBRTOWC)
CHECK_FUNCTION_EXISTS("mbtowc" HAVE_MBTOWC)
CHECK_FUNCTION_EXISTS("towlower" HAVE_TOWLOWER)
CHECK_FUNCTION_EXISTS("towupper" HAVE_TOWUPPER)
CHECK_FUNCTION_EXISTS("wcschr" HAVE_WCSCHR)
CHECK_FUNCTION_EXISTS("wcscpy" HAVE_WCSCPY)
CHECK_FUNCTION_EXISTS("wcslen" HAVE_WCSLEN)
CHECK_FUNCTION_EXISTS("wcsncpy" HAVE_WCSNCPY)
CHECK_FUNCTION_EXISTS("wcsrtombs" HAVE_WCSRTOMBS)
CHECK_FUNCTION_EXISTS("wcstombs" HAVE_WCSTOMBS)
CHECK_FUNCTION_EXISTS("wctype" HAVE_WCTYPE)

CHECK_INCLUDE_FILE("alloca.h" HAVE_ALLOCA_H)
CHECK_INCLUDE_FILE("sys/types.h" HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILE("libutf8.h" HAVE_LIBUTF8_H)
CHECK_INCLUDE_FILE("wctype.h" HAVE_WCTYPE_H)
CHECK_INCLUDE_FILE("wchar.h" HAVE_WCHAR_H)
CHECK_INCLUDE_FILE("getopt.h" HAVE_GETOPT_H)
CHECK_INCLUDE_FILE("dlfcn.h" HAVE_DLFCN_H)
CHECK_INCLUDE_FILE("inttypes.h" HAVE_INTTYPES_H)
CHECK_INCLUDE_FILE("regex.h" HAVE_REGEX_H)
CHECK_INCLUDE_FILE("stdint.h" HAVE_STDINT_H)
CHECK_INCLUDE_FILE("stdio.h" HAVE_STDIO_H)
CHECK_INCLUDE_FILE("stdlib.h" HAVE_STDLIB_H)
CHECK_INCLUDE_FILE("strings.h" HAVE_STRINGS_H)
CHECK_INCLUDE_FILE("string.h" HAVE_STRING_H)
CHECK_INCLUDE_FILE("sys/stat.h" HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILE("unistd.h" HAVE_UNISTD_H)

CHECK_TYPE_SIZE("mbstate_t" HAVE_MBSTATE_T)
CHECK_TYPE_SIZE("reg_errcode_t" HAVE_REG_ERRCODE_T)
CHECK_TYPE_SIZE("wchar_t" HAVE_WCHAR_T)
CHECK_TYPE_SIZE("wint_t" HAVE_WINT_T)

OPTION(TRE_APPROX "Enable the approximate matching functionality" ON)
OPTION(TRE_MULTIBYTE "Enable the approximate matching functionality" ON)
OPTION(TRE_AGREP "Enable build and install the agrep tool" ON)
OPTION(TRE_USE_ALLOCA "Define if you want TRE to use alloca() instead of malloc() when allocating memory needed for regexec operations." ON)
OPTION(TRE_WCHAR "Enable wide character (wchar_t) support" OFF)

set(libtre_src lib/tre-ast.c lib/tre-compile.c lib/tre-match-backtrack.c lib/tre-match-parallel.c lib/tre-mem.c lib/tre-parse.c lib/tre-stack.c lib/regcomp.c lib/regexec.c lib/regerror.c)
set(libtre_include lib/tre-ast.h lib/tre-compile.h lib/tre-internal.h lib/tre-match-utils.h lib/tre-mem.h lib/tre-parse.h lib/tre-stack.h lib/xmalloc.h)
set(agrep_src src/agrep.c)

configure_file(include/tre/tre-config.h.in include/tre/tre-config.h)
configure_file(config.h.in config/config.h)

add_library(libtre_obj OBJECT ${libtre_src} ${libtre_include})
if(TRE_APPROX)
    target_sources(libtre_obj PRIVATE lib/tre-match-approx.c)
endif()
set_property(TARGET libtre_obj PROPERTY POSITION_INDEPENDENT_CODE 1)
target_compile_definitions(libtre_obj PRIVATE HAVE_CONFIG_H)
target_include_directories(libtre_obj PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/config)
target_include_directories(libtre_obj PRIVATE include/tre)
target_include_directories(libtre_obj PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/include)


add_library(libtre_shared SHARED $<TARGET_OBJECTS:libtre_obj>)
set_target_properties(libtre_shared PROPERTIES OUTPUT_NAME tre)
add_library(libtre_static STATIC $<TARGET_OBJECTS:libtre_obj>)
set_target_properties(libtre_static PROPERTIES OUTPUT_NAME tre)

add_executable(agrep ${agrep_src})
target_compile_definitions(agrep PRIVATE HAVE_CONFIG_H)
target_include_directories(agrep PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/config)
target_include_directories(agrep PRIVATE include)
target_include_directories(agrep PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/include)
target_compile_definitions(agrep PRIVATE LOCALEDIR ${CMAKE_INSTALL_FULL_LOCALEDIR})
target_link_libraries(agrep libtre_static)
