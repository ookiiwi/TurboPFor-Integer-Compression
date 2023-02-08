cmake_minimum_required(VERSION 3.20)

project(ic C)

set(CFLAGS "")
set(OPT -fstrict-aliasing)
set(DEBUG -DNDEBUG -s)
set(SSE "")
set(AVX2 "")
set(OBJ "")

if (CMAKE_C_COMPILER_ID STREQUAL "Clang")
    set(OPT ${OPT} -falign-loops)
endif()

if (CMAKE_SYSTEM_PROCESSOR STREQUAL "arm64")
    set(CFLAGS -march=armv8-a)
    set(SSE -march=armv8-a)

    if (CMAKE_C_COMPILER_ID STREQUAL "Clang")
        set(OPT ${OPT} -fomit-frame-pointer)
    endif()
elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    set(SSE -march=corei7-avx -mtune=corei7-avx)
    set(CFLAGS ${SSE})
    set(AVX2 -march=haswell)
endif()

if (BUILD_SHARED_LIBS)
    set(CMAKE_POSITION_INDEPENDENT_CODE ON)
endif()

set(CFLAGS ${CFLAGS} -w -Wall ${DEBUG} ${OPT} -Iext)

set(SOURCES vp4c.c
            vp4d.c
            bitpack.c
            bitunpack.c
            transpose.c)

set(SOURCES_EXTRA   eliasfano.c 
                    bitutil.c 
                    fp.c
                    v8.c
                    vint.c
                    trlec.c
                    trled.c
                    vsimple.c)

            
add_library(obj OBJECT ${SOURCES} ${SOURCES_EXTRA})
target_compile_options(obj PUBLIC -O3 ${CFLAGS})

add_library(obj_sse OBJECT ${SOURCES})
target_compile_options(obj_sse PUBLIC -O3 -w ${SSE} -DSSE2_ON ${OPT})

set(OBJ obj obj_sse)

if (CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    add_library(obj_avx2 OBJECT ${SOURCES})
    target_compile_options(obj_avx2 PUBLIC -O3 -w ${AVX2} -DAVX2_ON ${OPT})
    set(OBJ ${OBJ} obj_avx2)
endif()

add_library(${PROJECT_NAME})
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

target_include_directories(${PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(${PROJECT_NAME} PUBLIC ${OBJ} m)
