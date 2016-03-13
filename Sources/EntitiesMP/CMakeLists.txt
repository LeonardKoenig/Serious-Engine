CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
#PROJECT(Engine)

# we want the relative dir because
#  a) this is the way it's done in the vcxproj
#  b) this way we don't get nasty C:/D: #defines in the generated files for
#     full paths in Windows
FILE(GLOB_RECURSE ENTITIESMP_ES_SOURCES RELATIVE ${PROJECT_SOURCE_DIR} "*.es")

# these will include the generated sources if Ecc was run before.
# ie.: Due to the nature of (most?) buildsystems we need to rerun
#      CMake if we modify/add *.es files so that additional or less
#      *.h/*.cpp files are generated.
FILE(GLOB_RECURSE ENTITIESMP_INCLUDES "*.h")
FILE(GLOB_RECURSE ENTITIESMP_SOURCES "*.cpp")


INCLUDE_DIRECTORIES ("../" ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})
ADD_LIBRARY (EntitiesMP SHARED ${ENTITIESMP_SOURCES} ${ENTITIESMP_INCLUDES})


ADD_CUSTOM_TARGET(EntitiesMP_Ecc_Headers
	ALL
	DEPENDS Ecc)
ADD_DEPENDENCIES(EntitiesMP EntitiesMP_Ecc_Headers)


SET(BLACKLISTED_ES "ParticleCloudsMarker.es;ParticleCloudsHolder.es")

FOREACH(FILE ${ENTITIESMP_ES_SOURCES})
    #GET_FILENAME_COMPONENT(FILE ${FILE} ABSOLUTE)

    # Check the black list
    GET_FILENAME_COMPONENT(FILENAME ${FILE} NAME)
    LIST(FIND BLACKLISTED_ES ${FILENAME} FILELOCATION)

    IF (${FILELOCATION} EQUAL -1)
		# This generates the code *in-tree*.
		# This is where code is expected by any compiler.
	    ADD_CUSTOM_COMMAND(TARGET EntitiesMP_Ecc_Headers POST_BUILD
			COMMAND echo Ecc: ${FILE}
            COMMAND $<TARGET_FILE:Ecc> ${FILE}
			WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})
    ENDIF()
ENDFOREACH()
