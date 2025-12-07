#     ___
#    / _/    ___
#   / /   ——  \ \ ,—,__ _____ ____ ___  _____  __———,——____
#  | (    )(   \ / / \ || ___)  _ ' _ \(  ___)/ / \ ||  __ \
#  |  \__/  \__/ \ \_) ||(   | ( | | ) )\__ \( (__) || /  ) )
#   \_____/\____/ \_  / |_\  /_| /_|/_/\_____)\___/\_\_\ /_/
#                 /__/

cmake_minimum_required(VERSION 3.7.1)
project(wyrmsun VERSION 5.4.0 LANGUAGES NONE)

# Platform detection
if(APPLE)
    set(PLATFORM_MACOS TRUE)
elseif(UNIX)
    set(PLATFORM_LINUX TRUE)
elseif(WIN32)
    set(PLATFORM_WINDOWS TRUE)
endif()

# Platform-specific directory defaults
if(PLATFORM_MACOS)
    set(BIN_DIR bin CACHE STRING "Where to install the Wyrmsun launch script")
    set(DATA_DIR share/wyrmsun CACHE STRING "Where to install the Wyrmsun data files")
    set(DOC_DIR share/doc/wyrmsun CACHE STRING "Where to install the Wyrmsun documentation")

    # App bundle option
    option(BUILD_APP_BUNDLE "Build macOS application bundle" ON)
else()
    set(BIN_DIR games CACHE STRING "Where to install the Wyrmsun launch script")
    set(DATA_DIR share/wyrmsun CACHE STRING "Where to install the Wyrmsun data files")
    set(ICONS_DIR share/icons/hicolor CACHE STRING "The root of icon theme")
    set(DOC_DIR share/doc/wyrmsun CACHE STRING "Where to install the Wyrmsun documentation")
endif()

### Linux-specific installation ###################################################
if(PLATFORM_LINUX)
    configure_file(linux/wyrmsun.sh.in wyrmsun.sh @ONLY)

    install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/wyrmsun.sh DESTINATION ${BIN_DIR} RENAME wyrmsun)
    install(FILES linux/wyrmsun.desktop DESTINATION share/applications)
    install(FILES linux/wyrmsun.appdata.xml DESTINATION share/metainfo)
    install(FILES graphics/interface/icons/wyrmsun_icon_32.png DESTINATION ${ICONS_DIR}/32x32/apps RENAME wyrmsun.png)
    install(FILES graphics/interface/icons/wyrmsun_icon_64.png DESTINATION ${ICONS_DIR}/64x64/apps RENAME wyrmsun.png)
    install(FILES graphics/interface/icons/wyrmsun_icon_128.png DESTINATION ${ICONS_DIR}/128x128/apps RENAME wyrmsun.png)

    install(DIRECTORY
        data
        fonts
        graphics
        interface
        maps
        modules
        music
        scripts
        sounds
        translations
        DESTINATION ${DATA_DIR}
    )

    install(FILES readme.txt DESTINATION ${DOC_DIR})
    install(DIRECTORY documents DESTINATION ${DOC_DIR})
endif()

### macOS-specific installation ###################################################
if(PLATFORM_MACOS)
    if(BUILD_APP_BUNDLE)
        # Configure Info.plist
        configure_file(macos/Info.plist.in ${CMAKE_CURRENT_BINARY_DIR}/Info.plist @ONLY)

        # Configure launch script
        configure_file(macos/wyrmsun.sh.in ${CMAKE_CURRENT_BINARY_DIR}/wyrmsun @ONLY)

        # Set up app bundle structure
        set(APP_BUNDLE_NAME "Wyrmsun.app")
        set(APP_BUNDLE_DIR "${CMAKE_CURRENT_BINARY_DIR}/${APP_BUNDLE_NAME}")
        set(APP_CONTENTS_DIR "${APP_BUNDLE_DIR}/Contents")
        set(APP_MACOS_DIR "${APP_CONTENTS_DIR}/MacOS")
        set(APP_RESOURCES_DIR "${APP_CONTENTS_DIR}/Resources")

        # Create app bundle directories
        file(MAKE_DIRECTORY ${APP_MACOS_DIR})
        file(MAKE_DIRECTORY ${APP_RESOURCES_DIR})

        # Custom target to build the app bundle
        add_custom_target(app_bundle ALL
            COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_BINARY_DIR}/Info.plist ${APP_CONTENTS_DIR}/
            COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_BINARY_DIR}/wyrmsun ${APP_MACOS_DIR}/
            COMMAND chmod +x ${APP_MACOS_DIR}/wyrmsun
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/data ${APP_RESOURCES_DIR}/data
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/fonts ${APP_RESOURCES_DIR}/fonts
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/graphics ${APP_RESOURCES_DIR}/graphics
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/interface ${APP_RESOURCES_DIR}/interface
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/maps ${APP_RESOURCES_DIR}/maps
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/modules ${APP_RESOURCES_DIR}/modules
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/music ${APP_RESOURCES_DIR}/music
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/scripts ${APP_RESOURCES_DIR}/scripts
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/sounds ${APP_RESOURCES_DIR}/sounds
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/translations ${APP_RESOURCES_DIR}/translations
            COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/readme.txt ${APP_RESOURCES_DIR}/
            COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/documents ${APP_RESOURCES_DIR}/documents
            COMMENT "Building macOS app bundle"
        )

        # Install the app bundle
        install(DIRECTORY ${APP_BUNDLE_DIR} DESTINATION "." USE_SOURCE_PERMISSIONS)
    else()
        # Non-bundle installation for macOS (similar to Linux)
        configure_file(macos/wyrmsun.sh.in wyrmsun.sh @ONLY)

        install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/wyrmsun.sh DESTINATION ${BIN_DIR} RENAME wyrmsun)

        install(DIRECTORY
            data
            fonts
            graphics
            interface
            maps
            modules
            music
            scripts
            sounds
            translations
            DESTINATION ${DATA_DIR}
        )

        install(FILES readme.txt DESTINATION ${DOC_DIR})
        install(DIRECTORY documents DESTINATION ${DOC_DIR})
    endif()
endif()

### PACKING ###################################################################

if(PLATFORM_LINUX)
    set(CPACK_GENERATOR DEB)
    set(CPACK_PACKAGING_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
    set(CPACK_PACKAGE_CONTACT "Anton Karmanov <techie@bastardi.net>")
    set(CPACK_PACKAGE_DESCRIPTION
      "Strategy game data files
Wyrmsun is an open-source real-time strategy game which features elements of
mythology, history and fiction.

In the Wyrmsun universe a myriad of inhabited planets exist. Humans dwell on
Earth, while dwarves inhabit Nidavellir and elves nourish the world of Alfheim.
These peoples struggle to carve a place for themselves with their tools of
stone, bronze and iron. And perhaps one day they will meet one another, beyond
the stars...")

    set(CPACK_DEBIAN_PACKAGE_DEPENDS "wyrmgus (=${CMAKE_PROJECT_VERSION})")
    set(CPACK_DEBIAN_PACKAGE_SECTION games)
endif()

if(PLATFORM_MACOS)
    set(CPACK_GENERATOR DragNDrop)
    set(CPACK_DMG_VOLUME_NAME "Wyrmsun ${CMAKE_PROJECT_VERSION}")
    set(CPACK_DMG_FORMAT "UDBZ")
    set(CPACK_PACKAGE_FILE_NAME "Wyrmsun-${CMAKE_PROJECT_VERSION}-macOS")
    set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Wyrmsun - Real-time strategy game with mythology and history")
endif()

include(CPack)
