project(bncsutil)
cmake_minimum_required(VERSION 2.6)

SET(VERSION_MAJOR "1")
SET(VERSION_MINOR "4")
SET(VERSION_PATCH "1")

file(GLOB SOURCE
    "src/bncsutil/*.cpp"
    "src/bncsutil/*.c"
)
file(GLOB HEADERS
    "src/bncsutil/*.h"
)
set(CMAKE_MODULE_PATH CMake/Modules)

add_library(${PROJECT_NAME} SHARED ${SOURCE} ${HEADERS})
add_library(${PROJECT_NAME}_static STATIC ${SOURCE} ${HEADERS})

if (BUILD_32)
         set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "-m32" LINK_FLAGS "-m32")
        MESSAGE(STATUS "Excluding 64bit library paths from search.")
         set_property(GLOBAL PROPERTY FIND_LIBRARY_USE_LIB64_PATHS OFF)
  set(ARCH_RPM i686)
        set(ARCH_DEB i386)
elseif (BUILD_64)
        set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "-m64" LINK_FLAGS "-m64")
  set(ARCH_DEB amd64)
        set(ARCH_RM x86_64)
else()
  set(ARCH_DEB amd64)
        set(ARCH_RPM x86_64)
endif()

if (USE_MPIR)
  find_package(MPIR REQUIRED)
  include_directories(src ${MPIR_INCLUDE_DIR})
  target_link_libraries(${PROJECT_NAME} ${MPIR_LIBRARIES})
  target_link_libraries(${PROJECT_NAME}_static ${MPIR_LIBRARIES})
  add_definitions(-DUSE_MPIR=1)
else()
  find_package(GMP REQUIRED)
  include_directories(src ${GMP_INCLUDE_DIR})
  target_link_libraries(${PROJECT_NAME} ${GMP_LIBRARIES})
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES OUTPUT_NAME bncsutil)

if(UNIX)
      set_target_properties(${PROJECT_NAME} PROPERTIES VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
  set_target_properties(${PROJECT_NAME} PROPERTIES SOVERSION 1)

      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3 -Wno-multichar -fPIC")
      set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3 -Wno-multichar -fPIC")
endif()

if (WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_CRT_SECURE_NO_WARNINGS")
  target_compile_definitions(${PROJECT_NAME}
      PUBLIC "-DMUTIL_LIB_BUILD=1"
      PRIVATE "-DBNCSUTIL_EXPORTS")
  target_link_libraries(${PROJECT_NAME} Version.lib)
  target_link_libraries(${PROJECT_NAME}_static Version.lib mincore)
endif()

install(TARGETS bncsutil RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
install(FILES ${HEADERS} DESTINATION include/bncsutil)

#CPack configurtion
SET(CPACK_GENERATOR "DEB" "RPM")
SET(CPACK_PACKAGE_NAME ${PROJECT_NAME})
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Battle.Net Chat Service Utility")
SET(CPACK_PACKAGE_VENDOR "bncsutil")
SET(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
SET(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/COPYING")
SET(CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
SET(CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
SET(CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}")

#DEB configuration
SET(CPACK_DEBIAN_PACKAGE_SECTION "libs")
SET(CPACK_DEBIAN_PACKAGE_HOMEPAGE "https://github.com/BNETDocs/bncsutil")
SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "imbacen@gmail.com")
SET(CPACK_DEBIAN_PACKAGE_ARCHITECTURE ${ARCH_DEB})
SET(CPACK_DEBIAN_PACKAGE_DEPENDS "libgmp10")

#RPM configuration
SET(CPACK_RPM_PACKAGE_RELEASE 1)
SET(CPACK_RPM_PACKAGE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/COPYING")
SET(CPACK_RPM_PACKAGE_GROUP "${PROJECT_NAME}")
SET(CPACK_RPM_PACKAGE_URL "https://github.com/BNETDocs/bncsutil")
SET(CPACK_RPM_PACKAGE_REQUIRES "gmp")
SET(CPACK_RPM_PACKAGE_ARCHITECTURE ${ARCH_RPM})

INCLUDE(CPack)
