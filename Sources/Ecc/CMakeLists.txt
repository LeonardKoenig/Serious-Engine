CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
PROJECT(Ecc)

FILE(GLOB_RECURSE ECC_INCLUDES "*.h")
# no recursing, as we then also add CMAKE-cpp stuff which we don't want.
# this also would break the build as we'd have multiple main symbols.
FILE(GLOB ECC_SOURCES "*.cpp")

INCLUDE_DIRECTORIES ("../")
# add Parser stuff manually as it is generated and therefor possibly not in glob
ADD_EXECUTABLE (Ecc ${ECC_SOURCES} Parser.cpp Scanner.cpp ${ECC_INCLUDES} Parser.h)

# Create a temporary blacklist of non-functional ES files
SET(BLACKLISTED_ES "ParticleCloudsMarker.es;ParticleCloudsHolder.es")

FILE(GLOB_RECURSE ES_SOURCES "../*.es")


FOREACH(FILE ${ES_SOURCES})
    GET_FILENAME_COMPONENT(FILE ${FILE} ABSOLUTE)

    # Check the black list
    GET_FILENAME_COMPONENT(FILENAME ${FILE} NAME)
    LIST(FIND BLACKLISTED_ES ${FILENAME} FILELOCATION)

    IF (${FILELOCATION} EQUAL -1)
        ADD_CUSTOM_COMMAND(TARGET Ecc POST_BUILD
            COMMAND $<TARGET_FILE:Ecc> ${FILE})
    ENDIF()
ENDFOREACH()


# use bison to generate Parser.h/.cpp

ADD_CUSTOM_COMMAND(OUTPUT Parser.cpp Scanner.cpp Parser.h
	COMMAND ${BISON_EXECUTABLE} -o ${CMAKE_CURRENT_SOURCE_DIR}/Parser.c ${CMAKE_CURRENT_SOURCE_DIR}/Parser.y -d
	COMMAND mv ${CMAKE_CURRENT_SOURCE_DIR}/Parser.c ${CMAKE_CURRENT_SOURCE_DIR}/Parser.cpp
	COMMAND ${FLEX_EXECUTABLE} -o ${CMAKE_CURRENT_SOURCE_DIR}/Scanner.cpp ${CMAKE_CURRENT_SOURCE_DIR}/Scanner.l)
