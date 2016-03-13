CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
#PROJECT(Engine)

# we want the relative dir because
#  a) this is the way it's done in the vcxproj
#  b) this way we don't get nasty C:/D: #defines in the generated files for
#     full paths in Windows
FILE(GLOB_RECURSE ENGINE_ES_SOURCES RELATIVE ${PROJECT_SOURCE_DIR} "*.es")

# these will include the generated sources if Ecc was run before.
# ie.: Due to the nature of (most?) buildsystems we need to rerun
#      CMake if we modify/add *.es files so that additional or less
#      *.h/*.cpp files are generated.
FILE(GLOB_RECURSE ENGINE_INCLUDES "*.h")
FILE(GLOB_RECURSE ENGINE_SOURCES "*.cpp")

BISON_TARGET(smcPars Ska/smcPars.y ${CMAKE_CURRENT_BINARY_DIR}/Ska/smcPars.cpp COMPILE_FLAGS "-psyy")

INCLUDE_DIRECTORIES ("../" ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})
ADD_LIBRARY (Engine SHARED ${ENGINE_SOURCES} ${ENGINE_INCLUDES} ${BISON_smcPars_OUTPUTS})


ADD_CUSTOM_TARGET(Engine_Ecc_Headers
	DEPENDS Ecc)
ADD_DEPENDENCIES(Engine Engine_Ecc_Headers)


FOREACH(FILE ${ENGINE_ES_SOURCES})
    #GET_FILENAME_COMPONENT(FILE ${FILE} ABSOLUTE)

    # Check the black list
    GET_FILENAME_COMPONENT(FILENAME ${FILE} NAME)
    LIST(FIND BLACKLISTED_ES ${FILENAME} FILELOCATION)

    IF (${FILELOCATION} EQUAL -1)
		# This generates the code *in-tree*.
		# This is where code is expected by any compiler.
	    ADD_CUSTOM_COMMAND(TARGET Engine_Ecc_Headers POST_BUILD
			COMMAND echo Ecc: ${FILE}
            COMMAND $<TARGET_FILE:Ecc> ${FILE}
			WORKING_DIRECTORY ${PROJECT_SOURCE_DIR})
    ENDIF()
ENDFOREACH()





















# - Try to find precompiled headers support for GCC 3.4 and 4.x
# Once done this will define:
#
# Variable:
#   PCHSupport_FOUND
#
# Macro:
#   ADD_PRECOMPILED_HEADER

IF(CMAKE_COMPILER_IS_GNUCXX)
    EXEC_PROGRAM(
    	${CMAKE_CXX_COMPILER} 
        ARGS 			--version 
        OUTPUT_VARIABLE _compiler_output)
    STRING(REGEX REPLACE ".* ([0-9]\\.[0-9]\\.[0-9]) .*" "\\1" 
           gcc_compiler_version ${_compiler_output})
    #MESSAGE("GCC Version: ${gcc_compiler_version}")
    IF(gcc_compiler_version MATCHES "4\\.[0-9]\\.[0-9]")
        SET(PCHSupport_FOUND TRUE)
    ELSE(gcc_compiler_version MATCHES "4\\.[0-9]\\.[0-9]")
        IF(gcc_compiler_version MATCHES "3\\.4\\.[0-9]")
            SET(PCHSupport_FOUND TRUE)
        ENDIF(gcc_compiler_version MATCHES "3\\.4\\.[0-9]")
    ENDIF(gcc_compiler_version MATCHES "4\\.[0-9]\\.[0-9]")
ENDIF(CMAKE_COMPILER_IS_GNUCXX)

MACRO(ADD_PRECOMPILED_HEADER _targetName _input )

    GET_FILENAME_COMPONENT(_name ${_input} NAME)
    SET(_source "${CMAKE_CURRENT_SOURCE_DIR}/${_input}")
    SET(_outdir "${CMAKE_CURRENT_BINARY_DIR}/${_name}.gch")
    MAKE_DIRECTORY(${_outdir})
    SET(_output "${_outdir}/${CMAKE_BUILD_TYPE}.c++")
    STRING(TOUPPER "CMAKE_CXX_FLAGS_${CMAKE_BUILD_TYPE}" _flags_var_name)
    SET(_compiler_FLAGS ${${_flags_var_name}})
    
    GET_DIRECTORY_PROPERTY(_directory_flags INCLUDE_DIRECTORIES)
    FOREACH(item ${_directory_flags})
    LIST(APPEND _compiler_FLAGS "-I${item}")
    ENDFOREACH(item)

	GET_DIRECTORY_PROPERTY(_directory_flags DEFINITIONS)
    LIST(APPEND _compiler_FLAGS ${_directory_flags})

    SEPARATE_ARGUMENTS(_compiler_FLAGS)
    #MESSAGE("_compiler_FLAGS: ${_compiler_FLAGS}")
    message("${CMAKE_CXX_COMPILER} ${_compiler_FLAGS} -x c++-header -o ${_output} ${_source}")
    ADD_CUSTOM_COMMAND(
        OUTPUT ${_output}
        COMMAND ${CMAKE_CXX_COMPILER}
				${_compiler_FLAGS}
				-x c++-header
				-o ${_output} ${_source}
        DEPENDS ${_source} )
   	ADD_CUSTOM_TARGET(${_targetName}_gch DEPENDS ${_output})
    ADD_DEPENDENCIES(${_targetName} ${_targetName}_gch)
    #SET(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} "-include ${_name} -Winvalid-pch -H")
    #SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -include ${_name} -Winvalid-pch")
    SET_TARGET_PROPERTIES(${_targetName} PROPERTIES
    	COMPILE_FLAGS "-include ${_name} -Winvalid-pch"
    )
	
ENDMACRO(ADD_PRECOMPILED_HEADER)

if(NOT WIN32)
ADD_PRECOMPILED_HEADER (Engine StdH.h)
endif(NOT WIN32)
