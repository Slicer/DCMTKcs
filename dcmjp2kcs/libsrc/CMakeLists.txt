# declare additional include directories
include_directories("${dcmjp2kcs_SOURCE_DIR}/include" "${ofstd_SOURCE_DIR}/include" "${oflog_SOURCE_DIR}/include" "${dcmdata_SOURCE_DIR}/include" "${dcmimgle_SOURCE_DIR}/include" "${dcmimage_SOURCE_DIR}/include" ${ZLIB_INCDIR})

if(DCMTK_WITH_OPENJPEG)
  include_directories(${OPENJPEG_INCLUDE_DIRS})
endif()

# create library from source files
DCMTK_ADD_LIBRARY(dcmjp2kcs
  dj2kparam.cc
  dj2krparam.cc
  dj2kdecode.cc
  dj2kcodecd.cc
  dj2kencode.cc
  dj2kcodece.cc
  dj2kutils.cc
)

DCMTK_TARGET_LINK_MODULES(dcmjp2kcs ofstd oflog dcmdata dcmimgle dcmimage)
target_compile_definitions(dcmjp2kcs INTERFACE DCMTK_WITH_DCMJP2KCS)

if(DCMTK_WITH_OPENJPEG)
  # Resolve how OpenJPEG should be referenced on the link line.
  # OPENJPEG_LIBS is set by DCMTK's 3rdparty.cmake for Windows support libraries;
  # otherwise OpenJPEG is provided either as the imported "openjp2" target (when
  # found via its CMake config package) or through the OPENJPEG_LIBRARIES variable.
  if(OPENJPEG_LIBS)
    set(DCMJP2KCS_OPENJPEG_LIBRARIES ${OPENJPEG_LIBS})
  elseif(TARGET openjp2)
    set(DCMJP2KCS_OPENJPEG_LIBRARIES openjp2)
  elseif(OPENJPEG_LIBRARIES)
    set(DCMJP2KCS_OPENJPEG_LIBRARIES ${OPENJPEG_LIBRARIES})
  else()
    set(DCMJP2KCS_OPENJPEG_LIBRARIES)
  endif()

  if(DCMJP2KCS_OPENJPEG_LIBRARIES)
    if(BUILD_SHARED_LIBS)
      # OpenJPEG is an internal implementation detail of dcmjp2kcs: only the codec
      # registration API (defined in this module) is public, while the OpenJPEG
      # (opj_*) calls live entirely inside this module's own sources. When dcmjp2kcs
      # is built as a shared library those symbols are fully resolved into
      # libdcmjp2kcs, so consumers must NOT link OpenJPEG themselves. Add it as a
      # private link dependency only (LINK_LIBRARIES, not INTERFACE_LINK_LIBRARIES).
      #
      # Propagating OpenJPEG through the link interface would make every consumer of
      # the DCMTK umbrella target (which re-exports dcmjp2kcs) transitively link
      # libopenjp2 even when it never uses JPEG 2000 - which, on macOS, breaks
      # downstream packaging that bundles each referenced library.
      set_property(TARGET dcmjp2kcs APPEND PROPERTY LINK_LIBRARIES ${DCMJP2KCS_OPENJPEG_LIBRARIES})
    else()
      # Static build: OpenJPEG symbols are not resolved until the final link, so
      # consumers of dcmjp2kcs must also link OpenJPEG. Propagate it through the
      # link interface (plain signature populates both link and interface).
      target_link_libraries(dcmjp2kcs ${DCMJP2KCS_OPENJPEG_LIBRARIES})
    endif()
  endif()
endif()
