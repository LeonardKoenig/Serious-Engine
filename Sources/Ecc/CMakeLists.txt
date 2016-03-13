CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
PROJECT(Ecc)

FILE(GLOB_RECURSE ECC_INCLUDES "*.h")
# no recursing, as we then also add CMAKE-cpp stuff which we don't want.
# this also would break the build as we'd have multiple main symbols.
FILE(GLOB ECC_SOURCES "*.cpp")

INCLUDE_DIRECTORIES ("../")
# add Parser stuff manually as it is generated and therefor possibly not in glob
ADD_EXECUTABLE (Ecc ${ECC_SOURCES} Parser.cpp Scanner.cpp ${ECC_INCLUDES} Parser.h)




# use bison to generate Parser.h/.cpp

ADD_CUSTOM_COMMAND(OUTPUT Parser.cpp Scanner.cpp Parser.h
	COMMAND ${BISON_EXECUTABLE} -o ${CMAKE_CURRENT_SOURCE_DIR}/Parser.c ${CMAKE_CURRENT_SOURCE_DIR}/Parser.y -d
	COMMAND mv ${CMAKE_CURRENT_SOURCE_DIR}/Parser.c ${CMAKE_CURRENT_SOURCE_DIR}/Parser.cpp
	COMMAND cp ${CMAKE_CURRENT_SOURCE_DIR}/Parser.cpp ${CMAKE_CURRENT_BINARY_DIR}/Parser.cpp
	COMMAND ${FLEX_EXECUTABLE} -o ${CMAKE_CURRENT_SOURCE_DIR}/Scanner.cpp ${CMAKE_CURRENT_SOURCE_DIR}/Scanner.l)
