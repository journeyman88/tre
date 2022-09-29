cmake_minimum_required(VERSION 3.10.0)
project(tre)
set(TRE_VERSION 0.8.0)

set(libtre_src lib/tre-ast.c lib/tre-compile.c lib/tre-match-backtrack.c lib/tre-match-parallel.c lib/tre-mem.c lib/tre-parse.c lib/tre-stack.c lib/regcomp.c lib/regexec.c lib/regerror.c)
set(agrep_src src/agrep.c)



configure_file(lib/tre-config.h.in include/tre-config.h)
configure_file(config.h.in include/config.h)

add_library(libtre_obj OBJECT ${libtre_src})
if(TRE_APPROX)
#    tre-match-approx.c
endif()
set_property(TARGET libtre_obj PROPERTY POSITION_INDEPENDENT_CODE 1)
target_compile_definitions(libtre_obj PRIVATE HAVE_CONFIG_H)


add_library(libtre_shared SHARED $<TARGET_OBJECTS:libtre_obj>)
set_target_properties(libtre_shared PROPERTIES OUTPUT_NAME tre)
add_library(libtre_static STATIC $<TARGET_OBJECTS:libtre_obj>)
set_target_properties(libtre_static PROPERTIES OUTPUT_NAME tre)

add_executable(agrep ${agrep_src})