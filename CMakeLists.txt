cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Download and unpack googletest at configure time
configure_file(CMakeLists.txt.in googletest-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
  RESULT_VARIABLE result
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/googletest-download )
if(result)
  message(FATAL_ERROR "Build step for googletest failed: ${result}")
endif()

# Prevent overriding the parent project's compiler/linker
# settings on Windows
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

# Add googletest directly to our build. This defines
# the gtest and gtest_main targets.
add_subdirectory(${CMAKE_CURRENT_BINARY_DIR}/googletest-src
                 ${CMAKE_CURRENT_BINARY_DIR}/googletest-build
                 EXCLUDE_FROM_ALL)

# The gtest/gtest_main targets carry header search path
# dependencies automatically when using CMake 2.8.11 or
# later. Otherwise we have to add them here ourselves.
if (CMAKE_VERSION VERSION_LESS 2.8.11)
  include_directories("${gtest_SOURCE_DIR}/include")
endif()

# Now simply link against gtest or gtest_main as needed. Eg

project(serverSQL)

SET(SRC_LIST_SERVER
  src/Parser.cpp
  src/Server.cpp
  src/ServerSQL.cpp
)

SET(SRC_LIST_CLIENT
  src/Parser.cpp
)

SET(SRC_LIST_TESTS
  tests/ParserTests.cpp
  tests/ServerSQLTests.cpp
  tests/ServerTests.cpp
)

include_directories(include
  src
  /usr/include/mariadb
)

add_compile_options(-Wall -Wextra -Werror -pedantic)

add_executable(server src/main-server.cpp ${SRC_LIST_SERVER})
target_link_libraries(server PRIVATE -ljsoncpp -I/usr/include/mysql -lsoci_core -lsoci_mysql -ldl -lmysqlclient -lpthread)

add_executable(client src/main-client.cpp ${SRC_LIST_CLIENT})
target_link_libraries(client PRIVATE -ljsoncpp -lpthread)

add_executable(${PROJECT_NAME}-ut ${SRC_LIST_TESTS} ${SRC_LIST_SERVER})
target_link_libraries(${PROJECT_NAME}-ut PRIVATE gtest_main -ljsoncpp -I/usr/include/mysql -lsoci_core -lsoci_mysql -ldl -lmysqlclient -lpthread)

enable_testing()
add_test(NAME Tests COMMAND ${PROJECT_NAME}-ut)