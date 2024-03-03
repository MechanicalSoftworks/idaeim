set(HEADER_LIST Types.hh STRINGIFY.hh Checker.hh Exception.hh Internet_Checksum.hh)

add_library(objUtility OBJECT
${HEADER_LIST}
Checker.cc Exception.cc Internet_Checksum.cc)

set_target_properties(objUtility PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_include_directories(objUtility PUBLIC ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/..)

add_library(Utility SHARED $<TARGET_OBJECTS:objUtility>)
add_library(sllUtility STATIC $<TARGET_OBJECTS:objUtility>)

if (WIN32)
   set_target_properties(sllUtility PROPERTIES OUTPUT_NAME libUtility)
	set_target_properties(Utility PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)
	if(${CMAKE_VERSION} VERSION_GREATER_EQUAL "3.13")
		target_link_options(Utility PRIVATE "/MANIFEST:NO")
	else()
		set(CMAKE_SHARED_LINKER_FLAGS "/MANIFEST:NO")
	endif()
	#
	# TODO this hack copies the static lib over the import lib
	# because the latter doesn't have symbols for static data fields
	add_custom_command(TARGET Utility POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:sllUtility> $<TARGET_LINKER_FILE:Utility>)

else()
	set_target_properties(sllUtility PROPERTIES OUTPUT_NAME Utility)
endif()

set_target_properties(sllUtility PROPERTIES PUBLIC_HEADER "${HEADER_LIST}")
set_target_properties(   Utility PROPERTIES PUBLIC_HEADER "${HEADER_LIST}")

install(TARGETS sllUtility Utility
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
	PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/Utility
)