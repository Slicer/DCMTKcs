# declare project
project(dcmjp2kcs)

# Find required dependencies before importing targets
# OpenJPEG is required for JPEG 2000 support
if(DCMTK_WITH_OPENJPEG)
  if(NOT TARGET openjp2)
    find_package(OpenJPEG QUIET)
    if(NOT OPENJPEG_FOUND AND NOT TARGET openjp2)
      message(WARNING "DCMTK was built with OpenJPEG support but OpenJPEG was not found. "
                      "You may need to set OpenJPEG_DIR to the location of OpenJPEGConfig.cmake")
    endif()
  endif()
  if(TARGET openjp2 AND DEFINED OpenJPEG_DIR)
    # Re-export the openjp2 target into DCMTKTargets.cmake so downstream consumers
    # of DCMTK get the openjp2 target automatically without needing to find OpenJPEG.
    # GenerateCMakeExports.cmake runs via dcmtk_create_install_exports() after all
    # subdirectories are processed, so we defer this append to run after it.
    cmake_language(DEFER DIRECTORY "${CMAKE_SOURCE_DIR}" CALL
      file APPEND "${CMAKE_BINARY_DIR}/DCMTKTargets.cmake"
      "\n# OpenJPEG target re-exported by dcmjp2kcs\nif(NOT TARGET openjp2)\n  include(\"${OpenJPEG_DIR}/OpenJPEGTargets.cmake\")\nendif()\n"
    )
  endif()
endif()

# External modules include directories
list(APPEND DCMTK_INCLUDE_DIR "${DCMTK_SOURCE_DIR}/ExternalModules/dcmjp2kcs/include")

# recurse into subdirectories
foreach(SUBDIR libsrc include apps tests)
  add_subdirectory(${SUBDIR})
endforeach()
