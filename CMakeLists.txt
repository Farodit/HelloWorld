cmake_minimum_required(VERSION 3.2)

if($ENV{TRAVIS_BUILD_NUMBER})
	project(helloworld VERSION 0.0.$ENV{TRAVIS_BUILD_NUMBER})
else()
	project(helloworld VERSION 0.0.124)
endif()

find_package(Boost COMPONENTS unit_test_framework REQUIRED)
find_package(GTest)

configure_file(version.h.in version.h)

add_executable(helloworld main.cpp)
add_library(helloworld_lib lib.cpp)
add_executable(boost_test_version test_version.cpp)
add_executable(google_test_version gtest_version.cpp)

set_target_properties(helloworld helloworld_lib boost_test_version google_test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)
target_include_directories(helloworld_lib
    PRIVATE "${CMAKE_BINARY_DIR}"
)
target_include_directories(google_test_version
    PRIVATE "${GTEST_INCLUDE_DIRS}"
)
set_target_properties(boost_test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
)

target_link_libraries(helloworld PRIVATE
    helloworld_lib
)
target_link_libraries(boost_test_version
    ${Boost_LIBRARIES}
    helloworld_lib
)
target_link_libraries(google_test_version
    ${GTEST_BOTH_LIBRARIES}
    pthread
    helloworld_lib
)

if (MSVC)
    target_compile_options(helloworld PRIVATE
        /W4
    )
    target_compile_options(helloworld_lib PRIVATE
        /W4
    )
    target_compile_options(boost_test_version PRIVATE
        /W4
    )
    target_compile_options(google_test_version PRIVATE
	/W4
    )
else ()
    target_compile_options(helloworld PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(helloworld_lib PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(boost_test_version PRIVATE
        -Wall -Wextra -pedantic -Werror
    )
    target_compile_options(google_test_version PRIVATE
	-Wall -Wextra -pedantic -Werror
    )
endif()



install(TARGETS helloworld RUNTIME DESTINATION bin)

set(CPACK_GENERATOR DEB)

set(CPACK_PACKAGE_VERSION_MAJOR "${PROJECT_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${PROJECT_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${PROJECT_VERSION_PATCH}")

set(CPACK_PACKAGE_CONTACT farodit@gmail.com)

include(CPack)

enable_testing()

add_test(boost_test_version boost_test_version)
add_test(google_test_version google_test_version)
