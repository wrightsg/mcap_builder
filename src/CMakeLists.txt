option(MCAP_BUILDER_BUILD_SHARED_LIB "force building MCAP library as shared library" ON)

if (MCAP_BUILDER_BUILD_SHARED_LIB)
  add_library(mcap SHARED lib.cpp)
else()
  add_library(mcap lib.cpp)
  get_target_property(target_type mcap TYPE)
  if (target_type STREQUAL STATIC_LIBRARY)
    target_compile_definitions(mcap PUBLIC MCAP_STATIC_DEFINE)
  endif()
endif()

target_include_directories(mcap PUBLIC
  $<BUILD_INTERFACE:${MCAP_INCLUDE_DIR}>
  $<INSTALL_INTERFACE:include/>
)

target_compile_features(mcap PUBLIC cxx_std_17)

set(DEPENDENCIES "")

find_package(lz4)
if (${lz4_FOUND})
  list(APPEND DEPENDENCIES LZ4::lz4)
else()
  message(WARNING "Dependency LZ4 not found, disabling")
  target_compile_definitions(mcap PUBLIC "MCAP_COMPRESSION_NO_LZ4")
endif()

find_package(zstd)
if (${zstd_FOUND})
  list(APPEND DEPENDENCIES zstd::libzstd_static)
else()
  message(WARNING "Dependency zstd not found, disabling")
  target_compile_definitions(mcap PUBLIC "MCAP_COMPRESSION_NO_ZSTD")
endif()

target_link_libraries(mcap PUBLIC ${DEPENDENCIES})
