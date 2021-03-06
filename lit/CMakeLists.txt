set(LLVM_SOURCE_DIR "${LLVM_MAIN_SRC_DIR}")
set(LLVM_BINARY_DIR "${LLVM_BINARY_DIR}")
set(LLVM_BUILD_MODE "%(build_mode)s")
set(LLVM_TOOLS_DIR "${LLVM_TOOLS_BINARY_DIR}/%(build_config)s")
set(LLVM_LIBS_DIR "${LLVM_BINARY_DIR}/lib${LLVM_LIBDIR_SUFFIX}/%(build_config)s")
set(CLANG_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/..")
set(CLANG_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/..")
if(BUILD_SHARED_LIBS)
  set(ENABLE_SHARED 1)
else()
  set(ENABLE_SHARED 0)
endif(BUILD_SHARED_LIBS)

# the value is not canonicalized within LLVM
llvm_canonicalize_cmake_booleans(
  LLVM_ENABLE_ZLIB)

option(LLDB_TEST_CLANG "Use in-tree clang when testing lldb" Off)
option(LLDB_TEST_SWIFT "Use in-tree swift when testing lldb" Off)
set(LLDB_TEST_C_COMPILER "" CACHE STRING "C compiler to use when testing LLDB")
set(LLDB_TEST_CXX_COMPILER "" CACHE STRING "C++ compiler to use when testing LLDB")

# This should be inherited from the `check-lldb` target. We currently don't run
# it for swift, but we might consider removing this code once we're done.
set(LIT_ARGS "-sv")

configure_lit_site_cfg(
  ${CMAKE_CURRENT_SOURCE_DIR}/lit.site.cfg.in
  ${CMAKE_CURRENT_BINARY_DIR}/lit.site.cfg)
configure_lit_site_cfg(
  ${CMAKE_CURRENT_SOURCE_DIR}/Unit/lit.site.cfg.in
  ${CMAKE_CURRENT_BINARY_DIR}/Unit/lit.site.cfg
  )

set(LLDB_TEST_DEPS
  LLDBUnitTests
  lldb
  lldb-test
  )

if(NOT LLDB_BUILT_STANDALONE)
  list(APPEND LLDB_TEST_DEPS FileCheck not yaml2obj)
endif()
  
# lldb-server is not built on every platform.
if (TARGET lldb-server)
  list(APPEND LLDB_TEST_DEPS lldb-server)
endif()
  
if(APPLE)
  list(APPEND LLDB_TEST_DEPS debugserver)
endif()

if(TARGET clang)
  list(APPEND LLDB_TEST_DEPS clang)
endif()

set(LLDB_TEST_PARAMS
  lldb_site_config=${CMAKE_CURRENT_BINARY_DIR}/lit.site.cfg
  )

add_lit_testsuite(check-lldb-lit "Running lldb lit test suite"
  ${CMAKE_CURRENT_BINARY_DIR}
  ${LIT_ARGS}
  PARAMS lldb_site_config=${CMAKE_CURRENT_BINARY_DIR}/lit.site.cfg
       lldb_unit_site_config=${CMAKE_CURRENT_BINARY_DIR}/Unit/lit.site.cfg
  DEPENDS ${LLDB_TEST_DEPS}
  )

set_target_properties(check-lldb-lit PROPERTIES FOLDER "LLDB tests")

# If we're building with an in-tree clang, then list clang as a dependency
# to run tests.
if (TARGET clang)
  add_dependencies(check-lldb-lit clang)
endif()

add_lit_testsuites(LLDB ${CMAKE_CURRENT_SOURCE_DIR}
  ${LIT_ARGS}
  PARAMS lldb_site_config=${CMAKE_CURRENT_BINARY_DIR}/lit.site.cfg
         lldb_unit_site_config=${CMAKE_CURRENT_BINARY_DIR}/Unit/lit.site.cfg
  DEPENDS ${LLDB_TEST_DEPS}
  )
