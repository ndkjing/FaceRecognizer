project(PlugSeeta)
find_package(SeetaFace)
message("== Plugs SeetaFac:${SeetaFace_FOUND}")
if(NOT SeetaFace_FOUND)
    return()
endif()

set(LIBS ${SeetaFace_LIBRARIES})
foreach(componet_seetaFace ${SeetaFace_LIBRARIES})
    if(ANDROID)
        INSTALL(FILES $<TARGET_FILE:${componet_seetaFace}>
            DESTINATION "libs/${ANDROID_ABI}"
            COMPONENT Runtime)
    else()
        INSTALL(FILES $<TARGET_FILE:${componet_seetaFace}>
            DESTINATION "${CMAKE_INSTALL_BINDIR}"
            COMPONENT Runtime)
    endif()
endforeach()

set(SOURCES_FILES
    FaceSeeta.cpp
    DetectorSeeta.cpp
    TrackerSeeta.cpp
    LandmarkerSeeta.cpp
    RecognizerSeeta.cpp
    FaceToolsSeeta.cpp
    )

set(HEADER_FILES
    FaceSeeta.h
    DetectorSeeta.h
    TrackerSeeta.h
    LandmarkerSeeta.h
    RecognizerSeeta.h
    FaceToolsSeeta.h
    )

set(SOURCE_UI_FILES
    ${SOURCE_UI_FILES}
    )

# Install model files
if(NOT EXISTS "${CMAKE_SOURCE_DIR}/model/Seeta")
    if(ENABLE_DOWNLOAD)
        DOWNLOAD(FILENAME "fd_2_00.dat"
            HASH e88669e5f1301ca56162de8aef1fd5d5
            URL "https://github.com/KangLin/SeetaFace2/releases/download/model/"
            DESTINATION_DIR "${CMAKE_SOURCE_DIR}/model/Seeta"
            STATUS retval RELATIVE_URL)
        DOWNLOAD(FILENAME "fr_2_10.dat"
            HASH 2d637aad8b1b7ae62154a877ec291c99
            URL "https://github.com/KangLin/SeetaFace2/releases/download/model/"
            DESTINATION_DIR "${CMAKE_SOURCE_DIR}/model/Seeta"
            STATUS retval RELATIVE_URL)
        DOWNLOAD(FILENAME "pd_2_00_pts5.dat"
            HASH 877a44aa6f07cb3064ad2828f50f261a
            URL "https://github.com/KangLin/SeetaFace2/releases/download/model/"
            DESTINATION_DIR "${CMAKE_SOURCE_DIR}/model/Seeta"
            STATUS retval RELATIVE_URL)
        DOWNLOAD(FILENAME "pd_2_00_pts81.dat"
            HASH f3f812f01121b5a80384af3c35211bdd
            URL "https://github.com/KangLin/SeetaFace2/releases/download/model/"
            DESTINATION_DIR "${CMAKE_SOURCE_DIR}/model/Seeta"
            STATUS retval RELATIVE_URL)
    else(ENABLE_DOWNLOAD)
        message(AUTHOR_WARNING "Please set ENABLE_DOWNLOAD to ON "
            " to automation download, or manual download Seeta model files from"
            " https://github.com/KangLin/SeetaFace2/releases/tag/model"
            " to ${CMAKE_SOURCE_DIR}/model/Seeta")
    endif(ENABLE_DOWNLOAD)
endif()

if(EXISTS "${CMAKE_SOURCE_DIR}/model/Seeta")
    INSTALL(FILES ${CMAKE_SOURCE_DIR}/model/Seeta/fd_2_00.dat
        ${CMAKE_SOURCE_DIR}/model/Seeta/fr_2_10.dat
        ${CMAKE_SOURCE_DIR}/model/Seeta/pd_2_00_pts5.dat
        ${CMAKE_SOURCE_DIR}/model/Seeta/pd_2_00_pts81.dat
        DESTINATION ${INSTALL_DATA_PREFIX}/model/Seeta
        COMPONENT Runtime)
endif()

add_library(${PROJECT_NAME} ${SOURCES_FILES} ${HEADER_FILES})
set_target_properties(${PROJECT_NAME} PROPERTIES
    LIBRARY_OUTPUT_DIRECTORY ${FACE_RECOGNIZER_PLUGS_BINARY_DIR}
    ARCHIVE_OUTPUT_DIRECTORY ${FACE_RECOGNIZER_PLUGS_BINARY_DIR}
    RUNTIME_OUTPUT_DIRECTORY ${FACE_RECOGNIZER_PLUGS_BINARY_DIR}
    VERSION ${BUILD_VERSION})
target_link_libraries(${PROJECT_NAME} PUBLIC FaceRecognizer RabbitCommon ${QT_LIBRARIES} ${LIBS})
target_include_directories(${PROJECT_NAME}
    PRIVATE ${RabbitCommon_DIR}/Src
    ${CMAKE_SOURCE_DIR}/Src
    ${CMAKE_BINARY_DIR}
    ${INCLUDE_DIRS})

if(BUILD_VERSION)
    target_compile_definitions(${PROJECT_NAME} PRIVATE -DBUILD_VERSION="${BUILD_VERSION}")
endif()

# Install target
INSTALL(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}Config
    RUNTIME DESTINATION "${INSTALL_FACE_RECOGNIZER_PLUGS_DIR}"
        COMPONENT Runtime
    LIBRARY DESTINATION "${INSTALL_FACE_RECOGNIZER_PLUGS_DIR}"
        COMPONENT Runtime
    ARCHIVE DESTINATION "${INSTALL_FACE_RECOGNIZER_PLUGS_DIR}"
    )

export(TARGETS ${PROJECT_NAME}
       APPEND FILE ${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake
)

# Install cmake configure files
install(EXPORT ${PROJECT_NAME}Config
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake"
        )
write_basic_package_version_file(
    "${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${BUILD_VERSION}
    COMPATIBILITY AnyNewerVersion)
install(FILES "${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake" DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake")
