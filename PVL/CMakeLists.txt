set(HEADER_LIST Lister.hh Parameter.hh Parser.hh PVL_Exceptions.hh PVL.hh Value.hh Vectal.hh)

add_library(objPVL OBJECT
   ${HEADER_LIST}
   Lister.cc
   PVL_Exceptions.cc
   Parameter.cc
   Parser.cc
   Value.cc
   Vectal.cc
)

set_target_properties(objPVL PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_include_directories(objPVL PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/..)

get_target_property(PVL_INCLUDE_DIRS objPVL INCLUDE_DIRECTORIES)
add_library(PVL SHARED $<TARGET_OBJECTS:objPVL>)
add_library(sllPVL STATIC $<TARGET_OBJECTS:objPVL>)

target_include_directories(PVL PUBLIC ${PVL_INCLUDE_DIRS})
target_include_directories(sllPVL PUBLIC ${PVL_INCLUDE_DIRS})

if (WIN32)
	set_target_properties(PVL PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)
	set_target_properties(sllPVL PROPERTIES OUTPUT_NAME libPVL)
	if(${CMAKE_VERSION} VERSION_GREATER_EQUAL "3.13")
		target_link_options(PVL PRIVATE "/MANIFEST:NO")
	else()
		set(CMAKE_SHARED_LINKER_FLAGS "/MANIFEST:NO")
	endif()
	#
	# TODO this hack copies the static lib over the import lib
	# because the latter doesn't have symbols for static data fields
	add_custom_command(TARGET PVL POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:sllPVL> $<TARGET_LINKER_FILE:PVL>)
else()
	set_target_properties(sllPVL PROPERTIES OUTPUT_NAME PVL)
endif()

target_link_libraries(PVL Utility Strings)

set_target_properties(sllPVL PROPERTIES PUBLIC_HEADER "${HEADER_LIST}")
set_target_properties(   PVL PROPERTIES PUBLIC_HEADER "${HEADER_LIST}")

install(TARGETS sllPVL PVL
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/PVL
)