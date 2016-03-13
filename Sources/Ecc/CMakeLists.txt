CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

IF(WIN32)
	SET(CMAKE_PROGRAM_PATH ${PROJECT_SOURCE_DIR}/../Tools.Win32)
ENDIF()
FIND_PACKAGE(BISON REQUIRED)
FIND_PACKAGE(FLEX REQUIRED)

BISON_TARGET(Parser Parser.y ${CMAKE_CURRENT_BINARY_DIR}/Parser.cpp)
FLEX_TARGET(Scanner Scanner.l ${CMAKE_CURRENT_BINARY_DIR}/Scanner.cpp)


FILE(GLOB_RECURSE ECC_INCLUDES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "*.h")
# no recursing, as we then also add CMAKE-cpp stuff which we don't want.
# this also would break the build as we'd have multiple main symbols.
FILE(GLOB ECC_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "*.cpp")

INCLUDE_DIRECTORIES ("../")
# we need that for Windows as Flex is auto-generating a system-<unistd.h> include.
INCLUDE_DIRECTORIES (AFTER SYSTEM "./")
# add Parser stuff manually as it is generated and therefor possibly not in glob
ADD_EXECUTABLE (Ecc
	${ECC_SOURCES} ${ECC_INCLUDES} ${BISON_Parser_OUTPUTS} ${FLEX_Parser_OUTPUTS})

