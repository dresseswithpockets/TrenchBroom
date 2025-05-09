set(LIB_UPDATE_TEST_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)

set(LIB_UPDATE_TEST_SOURCE
  "${LIB_UPDATE_TEST_SOURCE_DIR}/RunAllTests.cpp"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/StringMakers.h"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/TestHttpClient.cpp"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/TestHttpClient.h"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/TestUtils.cpp"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/TestUtils.h"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/TestVersion.cpp"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/TestVersion.h"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/tst_GithubApi.cpp"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/tst_QtHttpClient.cpp"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/tst_Unzip.cpp"
  "${LIB_UPDATE_TEST_SOURCE_DIR}/upd/tst_UpdateController.cpp"
)

# Catch2 needs one source file to define a macro (in our case CATCH_CONFIG_RUNNER) before including
# catch.hpp. This needs to be excluded from unity builds, otherwise the macro wouldn't be defined
# before the first catch.hpp inclusion.
set_property(SOURCE "${LIB_UPDATE_TEST_SOURCE_DIR}/RunAllTests.cpp" PROPERTY SKIP_UNITY_BUILD_INCLUSION ON)

add_executable(upd-test ${LIB_UPDATE_TEST_SOURCE})
target_include_directories(upd-test PRIVATE ${LIB_UPDATE_TEST_SOURCE_DIR} ${LIB_UPDATE_SOURCE_DIR})
target_include_directories(upd-test PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/$<CONFIG>)
target_link_libraries(upd-test PRIVATE upd Catch2::Catch2 Qt6::Test)
set_target_properties(upd-test PROPERTIES AUTOMOC TRUE)

set_compiler_config(upd-test)

# By default VS launches with a CWD one level up from the .exe (which is in a "Debug" subdirectory)
# but we copy resources into the .exe's directory, and the tests expect the CWD to be the .exe's directory.
set_target_properties(upd-test PROPERTIES VS_DEBUGGER_WORKING_DIRECTORY "$<TARGET_FILE_DIR:upd-test>")

# Copy fixtures
add_custom_command(
  TARGET upd-test POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E rm -rf "$<TARGET_FILE_DIR:upd-test>/fixture"
  COMMAND ${CMAKE_COMMAND} -E copy_directory "${CMAKE_CURRENT_SOURCE_DIR}/fixture" "$<TARGET_FILE_DIR:upd-test>/fixture"
)
