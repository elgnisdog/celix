# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
# 
#   http://www.apache.org/licenses/LICENSE-2.0
# 
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.
celix_subproject(LAUNCHER "Option to build the launcher" "ON" DEPS UTILS FRAMEWORK)
if (LAUNCHER) 
    find_package(CURL REQUIRED)

    include_directories(public/include)

    #celix launcher linked to libc
    add_executable(celix
        private/src/main.c
    )

    set_target_properties(celix     PROPERTIES "INSTALL_RPATH" "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}")
    
    target_link_libraries(celix     celix_framework ${CURL_LIBRARIES})
    if (NOT ANDROID)
        target_link_libraries(celix celix_dfi) #note not strictly needed, but ensure libdfi is a dep for the framework, useful when create docker images
    endif()

    include_directories("${PROJECT_SOURCE_DIR}/utils/public/include")
    include_directories("${PROJECT_SOURCE_DIR}/framework/public/include")
    include_directories("${CURL_INCLUDE_DIRS}")
    
    install(TARGETS celix RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT framework)

    find_package(CppUTest QUIET)
    if (CPPUTEST_FOUND)
        #Test running which start celix and run CppUTest RUN_ALL_TESTS.
        #Using this test running it is possible to create bundles containing CppUTests.
	include_directories(celix_test_runner)
	include_directories(SYSTEM ${CPPUTEST_INCLUDE_DIRS})
        add_executable(celix_test_runner
                private/src/celix_test_runner.cpp
        )
        set_target_properties(celix_test_runner PROPERTIES "INSTALL_RPATH" "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}")
        target_link_libraries(celix_test_runner
            celix_framework
            ${CURL_LIBRARIES}
            ${CPPUTEST_LIBRARIES}
            ${CPPUTEST_EXT_LIBRARIES}
        )
        install(TARGETS celix_test_runner RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT framework)
    endif ()
endif (LAUNCHER) 
