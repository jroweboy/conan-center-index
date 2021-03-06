PROJECT(MyHello)
cmake_minimum_required(VERSION 2.8)
enable_testing()

set(Boost_DEBUG 1)
include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)

if(CONAN_SETTINGS_OS STREQUAL "Android")
    set(CONAN_LIBCXX "") # NDK fails when specified: https://github.com/android-ndk/ndk/issues/541
endif()

CONAN_BASIC_SETUP()

IF(NOT HEADER_ONLY)
    set(boost_components)
    if (WITH_COROUTINE)
        list(APPEND boost_components coroutine)
        list(APPEND boost_components thread)
    endif()
    if (WITH_PYTHON)
        list(APPEND boost_components python)
    endif()
    if (WITH_RANDOM)
        list(APPEND boost_components random)
    endif()
    if (WITH_REGEX)
        list(APPEND boost_components regex)
    endif()
    if (WITH_CHRONO)
        list(APPEND boost_components chrono)
    endif()

    find_package(Boost COMPONENTS ${boost_components} REQUIRED)

    include_directories(${Boost_INCLUDE_DIRS})

    MESSAGE("LIBS=> ${CONAN_LIBS}")

    ADD_EXECUTABLE(lambda lambda.cpp)

    if (WITH_RANDOM)
        ADD_EXECUTABLE(random_exe random.cpp)
        TARGET_LINK_LIBRARIES(random_exe ${Boost_LIBRARIES})
    endif()
    if (WITH_REGEX)
        ADD_EXECUTABLE(regex_exe regex.cpp)
        TARGET_LINK_LIBRARIES(regex_exe ${Boost_LIBRARIES})
    endif()

    if (WITH_TEST)
        ADD_EXECUTABLE(test test.cpp)
        TARGET_LINK_LIBRARIES(test ${CONAN_LIBS})
    endif()

    if (WITH_COROUTINE)
        ADD_EXECUTABLE(coroutine_exe coroutine.cpp)
        TARGET_LINK_LIBRARIES(coroutine_exe ${Boost_LIBRARIES})
    endif()

    if (WITH_CHRONO)
        ADD_EXECUTABLE(chrono_exe chrono.cpp)
        TARGET_LINK_LIBRARIES(chrono_exe ${CONAN_LIBS})
    endif()

    if(WITH_PYTHON)
        add_library(hello_ext SHARED python.cpp)
        if(WIN32)
            set_target_properties(hello_ext PROPERTIES SUFFIX ".pyd")
            target_include_directories(hello_ext PRIVATE C:/Python27/include)
            target_link_libraries(hello_ext C:/Python27/libs/python27.lib ${CONAN_LIBS})
        endif()
    endif()

    if (WITH_REGEX)
        ADD_TEST(NAME TestRegex
                 COMMAND ${CMAKE_CROSSCOMPILING_EMULATOR} "$<TARGET_FILE:regex_exe>")
    endif()
ELSE()
    ADD_EXECUTABLE(lambda lambda.cpp)
ENDIF()

ADD_TEST(NAME TestLambda
         COMMAND ${CMAKE_CROSSCOMPILING_EMULATOR} "$<TARGET_FILE:lambda>" ${TEST_ARGS})

IF(NOT HEADER_ONLY AND WITH_REGEX)
    # Test a different exe linking with the CONAN_LIBS to actually test the package_info
    ADD_EXECUTABLE(newregex regex.cpp)
    TARGET_LINK_LIBRARIES(newregex ${CONAN_LIBS})
    ADD_TEST(NAME TestRegexNew
             COMMAND ${CMAKE_CROSSCOMPILING_EMULATOR} "$<TARGET_FILE:newregex>" ${TEST_ARGS})
ENDIF()
