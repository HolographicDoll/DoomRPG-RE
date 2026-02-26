file(GLOB_RECURSE HEADER_FILES
	"*.hpp"
	"*.h")

file(GLOB_RECURSE SOURCE_FILES
	"*.cpp"
	"*.c")

file(GLOB_RECURSE RESOURCES_FILES
	"*.rc"
	"*.ico")

source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} PREFIX "Header Files" FILES ${HEADER_FILES})
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} PREFIX "Source Files" FILES ${SOURCE_FILES})
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} PREFIX "Resources Files" FILES ${RESOURCES_FILES})

# macOS Bundle Configuration
if(APPLE)
	set(MACOSX_BUNDLE_ICON_FILE appicon.icns)
	set(MACOSX_BUNDLE_BUNDLE_NAME ${PROJECT_NAME})
	set(MACOSX_BUNDLE_BUNDLE_VERSION "${PROJECT_VERSION}")
	set(MACOSX_BUNDLE_SHORT_VERSION_STRING "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}")
	set(MACOSX_BUNDLE_LONG_VERSION_STRING "${PROJECT_VERSION}")
	set(MACOSX_BUNDLE_GUI_IDENTIFIER "com.erick194.${PROJECT_NAME}")
	
	# Set icon file path
	set(APP_ICON_MACOSX "${CMAKE_CURRENT_SOURCE_DIR}/appicon.icns")
	if(EXISTS ${APP_ICON_MACOSX})
		set_source_files_properties(${APP_ICON_MACOSX} PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")
		set(ICON_FILES ${APP_ICON_MACOSX})
	endif()
endif()

add_executable(${PROJECT_NAME}
	${HEADER_FILES}
	${SOURCE_FILES}
	${RESOURCES_FILES}
	${ICON_FILES}
)

if(APPLE)
	set_target_properties(${PROJECT_NAME} PROPERTIES
		MACOSX_BUNDLE TRUE
		MACOSX_BUNDLE_INFO_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/Info.plist.in"
	)
endif()

target_link_libraries(${PROJECT_NAME} SDL2::SDL2 SDL2::mixer ZLIB::ZLIB FluidSynth::libfluidsynth)

# Copy DLL dependencies on Windows
if(WIN32 AND NOT MSVC)
	# For MSYS2/MinGW, DLLs are in bin/ not lib/
	# Get the prefix path (e.g., C:/msys64/mingw64)
	get_target_property(SDL2_LIB SDL2::SDL2 IMPORTED_LOCATION)
	get_filename_component(SDL2_LIB_DIR "${SDL2_LIB}" DIRECTORY)
	get_filename_component(MINGW_PREFIX "${SDL2_LIB_DIR}/.." ABSOLUTE)
	set(MINGW_BIN_DIR "${MINGW_PREFIX}/bin")
	
# List of DLL files to copy
	set(REQUIRED_DLLS
		"SDL2.dll"
		"SDL2_mixer.dll"
		# To avoid SDL3 dependency you can use sideload and install libfluidsynth package from here:
		# https://repo.msys2.org/mingw/mingw64/mingw-w64-x86_64-fluidsynth-2.4.0-1-any.pkg.tar.zst
		"libfluidsynth-3.dll"
		"libFLAC.dll"
		"libogg-0.dll"
		"libvorbis-0.dll"
		"libvorbisfile-3.dll"
		"libglib-2.0-0.dll"
		"libmpg123-0.dll"
		"libopusfile-0.dll"
		"libwavpack-1.dll"
		"libxmp.dll"
		"libportaudio.dll"
		"libreadline8.dll"
		"libgmodule-2.0-0.dll"
		"libsndfile-1.dll"
        "libopus-0.dll"
		"libtermcap-0.dll"
		"libmp3lame-0.dll"
		"libvorbisenc-2.dll"
		"libpcre2-8-0.dll"
	)

	
	# Copy each DLL if it exists
	foreach(DLL_NAME ${REQUIRED_DLLS})
		if(EXISTS "${MINGW_BIN_DIR}/${DLL_NAME}")
			add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
				COMMAND ${CMAKE_COMMAND} -E copy_if_different
				"${MINGW_BIN_DIR}/${DLL_NAME}"
				"$<TARGET_FILE_DIR:${PROJECT_NAME}>"
			)
		endif()
	endforeach()
endif()