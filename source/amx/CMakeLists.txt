#build file for CMake, see http://www.cmake.org/

PROJECT(pawnamx)

CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

# check for optional include files
INCLUDE(${CMAKE_ROOT}/Modules/CheckIncludeFile.cmake)
CHECK_INCLUDE_FILE("unistd.h" HAVE_UNISTD_H)
IF(HAVE_UNISTD_H)
  ADD_DEFINITIONS(-DHAVE_UNISTD_H)
ENDIF(HAVE_UNISTD_H)
CHECK_INCLUDE_FILE("inttypes.h" HAVE_INTTYPES_H)
IF(HAVE_INTTYPES_H)
  ADD_DEFINITIONS(-DHAVE_INTTYPES_H)
ENDIF(HAVE_INTTYPES_H)
CHECK_INCLUDE_FILE("stdint.h" HAVE_STDINT_H)
IF(HAVE_STDINT_H)
  ADD_DEFINITIONS(-DHAVE_STDINT_H)
ENDIF(HAVE_STDINT_H)
CHECK_INCLUDE_FILE("alloca.h" HAVE_ALLOCA_H)
IF(HAVE_ALLOCA_H)
  ADD_DEFINITIONS(-DHAVE_ALLOCA_H)
ENDIF(HAVE_ALLOCA_H)

ADD_DEFINITIONS(-DFLOATPOINT -DFIXEDPOINT)
IF (UNIX)
  ADD_DEFINITIONS(-DLINUX)
  CHECK_INCLUDE_FILE("ffi.h" HAVE_FFI_H)
  INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/../linux)
ENDIF (UNIX)
IF(WIN32)
  ADD_DEFINITIONS(-DAMXEXPORT=__stdcall -DAMX_NATIVE_CALL=__stdcall -DSTDECL)
  IF(NOT BORLAND)
    LINK_LIBRARIES(winmm)
  ENDIF(NOT BORLAND)
ENDIF(WIN32)

# --------------------------------------------------------------------------
# Extension modules

# amxArgs
SET(ARGS_SRCS amxargs.c amx.c)
ADD_LIBRARY(amxArgs SHARED ${ARGS_SRCS})
SET_TARGET_PROPERTIES(amxArgs PROPERTIES PREFIX "")
IF(WIN32)
  SET(ARGS_SRCS ${ARGS_SRCS} dllmain.c amxargs.rc)
  IF(BORLAND)
    # Borland linker uses a DEF file if one is in the output directory
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/amxargs.def ${CMAKE_BINARY_DIR}/amxargs.def COPY_ONLY)
  ELSE(BORLAND)
    # For Microsoft Visual C/C++ we can set explicit flags for exports
    SET_TARGET_PROPERTIES(amxArgs PROPERTIES LINK_FLAGS "/export:amx_ArgsInit /export:amx_ArgsCleanup /export:amx_ArgsSetCmdLine")
  ENDIF(BORLAND)
ENDIF(WIN32)
IF(UNIX)
  ADD_CUSTOM_COMMAND(TARGET amxArgs POST_BUILD COMMAND strip ARGS -K amx_ArgsInit -K amx_ArgsCleanup -K amx_ArgsSetCmdLine ${CMAKE_BINARY_DIR}/amxArgs.so)
ENDIF(UNIX)

# amxDGram
SET(DGRAM_SRCS amxdgram.c amx.c)
ADD_LIBRARY(amxDGram SHARED ${DGRAM_SRCS})
SET_TARGET_PROPERTIES(amxDGram PROPERTIES PREFIX "")
IF(WIN32)
  SET(DGRAM_SRCS ${DGRAM_SRCS} dllmain.c amxargs.rc)
  IF(BORLAND)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/amxdgram.def ${CMAKE_BINARY_DIR}/amxdgram.def COPY_ONLY)
  ELSE(BORLAND)
    SET_TARGET_PROPERTIES(amxDGram PROPERTIES LINK_FLAGS "/export:amx_DGramInit /export:amx_DGramCleanup")
  ENDIF(BORLAND)
ENDIF(WIN32)
IF(UNIX)
  ADD_CUSTOM_COMMAND(TARGET amxDGram POST_BUILD COMMAND strip ARGS -K amx_DGramInit -K amx_DGramCleanup ${CMAKE_BINARY_DIR}/amxDGram.so)
ENDIF(UNIX)

# amxFile
SET(FILE_SRCS amxfile.c amx.c)
ADD_LIBRARY(amxFile SHARED ${FILE_SRCS})
SET_TARGET_PROPERTIES(amxFile PROPERTIES PREFIX "")
IF(WIN32)
  SET(FILE_SRCS ${FILE_SRCS} dllmain.c amxfile.rc)
  IF(BORLAND)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/amxfile.def ${CMAKE_BINARY_DIR}/amxfile.def COPY_ONLY)
  ELSE(BORLAND)
    SET_TARGET_PROPERTIES(amxFile PROPERTIES LINK_FLAGS "/export:amx_FileInit /export:amx_FileCleanup")
  ENDIF(BORLAND)
ENDIF(WIN32)
IF(UNIX)
  ADD_CUSTOM_COMMAND(TARGET amxFile POST_BUILD COMMAND strip ARGS -K amx_FileInit -K amx_FileCleanup ${CMAKE_BINARY_DIR}/amxFile.so)
ENDIF(UNIX)

# amxFixed
SET(FIXED_SRCS fixed.c amx.c)
ADD_LIBRARY(amxFixed SHARED ${FIXED_SRCS})
SET_TARGET_PROPERTIES(amxFixed PROPERTIES PREFIX "")
IF(WIN32)
  SET(FIXED_SRCS ${FIXED_SRCS} dllmain.c amxfixed.rc)
  IF(BORLAND)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/amxfixed.def ${CMAKE_BINARY_DIR}/amxfixed.def COPY_ONLY)
  ELSE(BORLAND)
    SET_TARGET_PROPERTIES(amxFixed PROPERTIES LINK_FLAGS "/export:amx_FixedInit /export:amx_FixedCleanup")
  ENDIF(BORLAND)
ENDIF(WIN32)
IF(UNIX)
  TARGET_LINK_LIBRARIES(amxFixed m)
  ADD_CUSTOM_COMMAND(TARGET amxFixed POST_BUILD COMMAND strip ARGS -K amx_FixedInit -K amx_FixedCleanup ${CMAKE_BINARY_DIR}/amxFixed.so)
ENDIF(UNIX)

# amxFloat
SET(FLOAT_SRCS float.c amx.c)
ADD_LIBRARY(amxFloat SHARED ${FLOAT_SRCS})
SET_TARGET_PROPERTIES(amxFloat PROPERTIES PREFIX "")
IF(WIN32)
  SET(FLOAT_SRCS ${FLOAT_SRCS} dllmain.c amxfloat.rc)
  IF(BORLAND)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/amxfloat.def ${CMAKE_BINARY_DIR}/amxfloat.def COPY_ONLY)
  ELSE(BORLAND)
    SET_TARGET_PROPERTIES(amxFloat PROPERTIES LINK_FLAGS "/export:amx_FloatInit /export:amx_FloatCleanup")
  ENDIF(BORLAND)
ENDIF(WIN32)
IF(UNIX)
  TARGET_LINK_LIBRARIES(amxFloat m)
  ADD_CUSTOM_COMMAND(TARGET amxFloat POST_BUILD COMMAND strip ARGS -K amx_FloatInit -K amx_FloatInit ${CMAKE_BINARY_DIR}/amxFloat.so)
ENDIF(UNIX)

# amxProcess
SET(PROCESS_SRCS amxprocess.c amx.c)
IF(WIN32)
  ADD_LIBRARY(amxProcess SHARED ${PROCESS_SRCS})
ELSE(WIN32)
  IF(HAVE_FFI_H)
    ADD_LIBRARY(amxProcess SHARED ${PROCESS_SRCS})
  ELSE(HAVE_FFI_H)
    MESSAGE(SEND_ERROR "amxProcess requires libffi; see http://sources.redhat.com/libffi/")
    MESSAGE(SEND_ERROR "libffi is not available (foreign function interface)")
  ENDIF(HAVE_FFI_H)
ENDIF(WIN32)
SET_TARGET_PROPERTIES(amxProcess PROPERTIES PREFIX "")
IF(WIN32)
  SET(PROCESS_SRCS ${PROCESS_SRCS} dllmain.c amxprocess.rc)
  IF(BORLAND)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/amxprocess.def ${CMAKE_BINARY_DIR}/amxprocess.def COPY_ONLY)
  ELSE(BORLAND)
    SET_TARGET_PROPERTIES(amxProcess PROPERTIES LINK_FLAGS "/export:amx_ProcessInit /export:amx_ProcessCleanup")
  ENDIF(BORLAND)
ENDIF(WIN32)
IF(UNIX)
  TARGET_LINK_LIBRARIES(amxProcess dl)
  ADD_CUSTOM_COMMAND(TARGET amxProcess POST_BUILD COMMAND strip ARGS -K amx_ProcessInit -K amx_ProcessCleanup ${CMAKE_BINARY_DIR}/amxProcess.so)
ENDIF(UNIX)

# amxString
SET(STRING_SRCS amxstring.c amx.c amxcons.c)
ADD_LIBRARY(amxString SHARED ${STRING_SRCS})
SET_TARGET_PROPERTIES(amxString PROPERTIES PREFIX "")
IF(WIN32)
  SET(STRING_SRCS ${STRING_SRCS} dllmain.c amxstring.rc)
  IF(BORLAND)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/amxstring.def ${CMAKE_BINARY_DIR}/amxstring.def COPY_ONLY)
  ELSE(BORLAND)
    SET_TARGET_PROPERTIES(amxString PROPERTIES LINK_FLAGS "/export:amx_StringInit /export:amx_StringCleanup")
  ENDIF(BORLAND)
ENDIF(WIN32)
IF(UNIX)
  ADD_CUSTOM_COMMAND(TARGET amxString POST_BUILD COMMAND strip ARGS -K amx_StringInit -K amx_StringCleanup ${CMAKE_BINARY_DIR}/amxString.so)
ENDIF(UNIX)

# amxTime
SET(TIME_SRCS amxtime.c amx.c)
ADD_LIBRARY(amxTime SHARED ${TIME_SRCS})
SET_TARGET_PROPERTIES(amxTime PROPERTIES PREFIX "")
IF(WIN32)
  SET(TIME_SRCS ${TIME_SRCS} dllmain.c amxtime.rc)
  IF(BORLAND)
    CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/amxtime.def ${CMAKE_BINARY_DIR}/amxtime.def COPY_ONLY)
  ELSE(BORLAND)
    SET_TARGET_PROPERTIES(amxTime PROPERTIES LINK_FLAGS "/export:amx_TimeInit /export:amx_TimeCleanup")
  ENDIF(BORLAND)
ENDIF(WIN32)
IF(UNIX)
  ADD_CUSTOM_COMMAND(TARGET amxTime POST_BUILD COMMAND strip ARGS -K amx_TimeInit -K amx_TimeCleanup ${CMAKE_BINARY_DIR}/amxTime.so)
ENDIF(UNIX)

# --------------------------------------------------------------------------
# Simple run-time (example program)

SET(PAWNRUN_SRCS pawnrun.c amx.c amxcore.c amxcons.c amxdbg.c)
IF (UNIX)
  SET(PAWNRUN_SRCS ${PAWNRUN_SRCS} ${CMAKE_CURRENT_SOURCE_DIR}/../linux/getch.c ${CMAKE_CURRENT_SOURCE_DIR}/../linux/binreloc.c)
ENDIF (UNIX)
ADD_EXECUTABLE(pawnrun ${PAWNRUN_SRCS})
SET_TARGET_PROPERTIES(pawnrun PROPERTIES COMPILE_FLAGS -DAMXDBG COMPILE_FLAGS -DENABLE_BINRELOC)
IF (UNIX)
  TARGET_LINK_LIBRARIES(pawnrun dl)
ENDIF (UNIX)

# --------------------------------------------------------------------------
# Simple console debugger

SET(PAWNDBG_SRCS pawndbg.c amx.c amxcore.c amxcons.c amxdbg.c)
IF (UNIX)
  SET(PAWNDBG_SRCS ${PAWNDBG_SRCS} ${CMAKE_CURRENT_SOURCE_DIR}/../linux/getch.c ${CMAKE_CURRENT_SOURCE_DIR}/../linux/binreloc.c)
ENDIF (UNIX)
ADD_EXECUTABLE(pawndbg ${PAWNDBG_SRCS})
SET_TARGET_PROPERTIES(pawndbg PROPERTIES COMPILE_FLAGS -DENABLE_BINRELOC)
IF (UNIX)
  TARGET_LINK_LIBRARIES(pawndbg dl)
ENDIF (UNIX)