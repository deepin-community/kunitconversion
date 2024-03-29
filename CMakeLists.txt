cmake_minimum_required(VERSION 3.16)

set(KF_VERSION "5.103.0") # handled by release scripts
set(KF_DEP_VERSION "5.103.0") # handled by release scripts
project(KUnitConversion VERSION ${KF_VERSION})

include(FeatureSummary)
find_package(ECM 5.103.0  NO_MODULE)
set_package_properties(ECM PROPERTIES TYPE REQUIRED DESCRIPTION "Extra CMake Modules." URL "https://commits.kde.org/extra-cmake-modules")
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH})

include(KDEInstallDirs)
include(KDEFrameworkCompilerSettings NO_POLICY_SCOPE)
include(KDECMakeSettings)
include(ECMGenerateExportHeader)
include(ECMSetupVersion)
include(ECMGenerateHeaders)
include(ECMQtDeclareLoggingCategory)
include(ECMDeprecationSettings)

set(EXCLUDE_DEPRECATED_BEFORE_AND_AT 0 CACHE STRING "Control the range of deprecated API excluded from the build [default=0].")

set(REQUIRED_QT_VERSION 5.15.2)
find_package(Qt${QT_MAJOR_VERSION} ${REQUIRED_QT_VERSION} CONFIG REQUIRED Core Network)

set(CMAKECONFIG_INSTALL_DIR "${KDE_INSTALL_CMAKEPACKAGEDIR}/KF5UnitConversion")

ecm_setup_version(PROJECT VARIABLE_PREFIX KUNITCONVERSION
                        VERSION_HEADER "${CMAKE_CURRENT_BINARY_DIR}/kunitconversion_version.h"
                        PACKAGE_VERSION_FILE "${CMAKE_CURRENT_BINARY_DIR}/KF5UnitConversionConfigVersion.cmake"
                        SOVERSION 5)

# QCH
include(ECMAddQch)
option(BUILD_QCH "Build API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)" OFF)
add_feature_info(QCH ${BUILD_QCH} "API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)")
if (BUILD_QCH)
    ecm_install_qch_export(
        TARGETS KF5UnitConversion_QCH
        FILE KF5UnitConversionQchTargets.cmake
        DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
        COMPONENT Devel
    )
    set(PACKAGE_INCLUDE_QCHTARGETS "include(\"\${CMAKE_CURRENT_LIST_DIR}/KF5UnitConversionQchTargets.cmake\")")
endif()

# I18n
find_package(KF5I18n ${KF_DEP_VERSION} REQUIRED)
add_definitions(-DTRANSLATION_DOMAIN=\"kunitconversion5\")
ki18n_install(po)

ecm_set_disabled_deprecation_versions(
    QT 5.15.2
    KF 5.97
)

add_subdirectory(src)
if (BUILD_TESTING)
    add_subdirectory(autotests)
endif()
# create a Config.cmake and a ConfigVersion.cmake file and install them
include(CMakePackageConfigHelpers)
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/KF5UnitConversionConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/KF5UnitConversionConfig.cmake"
    INSTALL_DESTINATION  ${CMAKECONFIG_INSTALL_DIR}
)
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/KF5UnitConversionConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/KF5UnitConversionConfigVersion.cmake"
    DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
    COMPONENT Devel
)
install(EXPORT KF5UnitConversionTargets DESTINATION "${CMAKECONFIG_INSTALL_DIR}" FILE KF5UnitConversionTargets.cmake NAMESPACE KF5:: )
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/kunitconversion_version.h
    DESTINATION ${KDE_INSTALL_INCLUDEDIR_KF}/KUnitConversion COMPONENT Devel
)

feature_summary(WHAT ALL FATAL_ON_MISSING_REQUIRED_PACKAGES)
