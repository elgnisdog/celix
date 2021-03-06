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

celix_subproject(SHELL_BONJOUR "Option to enable building the Bonjour Shell (shell access by chat clients)" OFF DEPS LAUNCHER shell)
if (SHELL_BONJOUR)
	find_package(LibXml2 REQUIRED)
	
	#TODO create/add FindDNS_SD.cmake and use it (with required)
	find_library(DNS_SD_LIB NAMES dns_sd dns_services)
	
	set(BUNDLE_SYMBOLICNAME "bonjour_shell")
	set(BUNDLE_VERSION "0.1.0")
	set(BUNDLE_NAME "bonjour_shell")

	include_directories("${PROJECT_SOURCE_DIR}/utils/public/include")
	include_directories("${PROJECT_SOURCE_DIR}/shell/public/include")
	include_directories("${LIBXML2_INCLUDE_DIR}")
	include_directories("private/include")

	set(MEMSTREAM_SOURCES )
	if (APPLE OR ANDROID)
		set(MEMSTREAM_SOURCES ${PROJECT_SOURCE_DIR}/utils/private/src/memstream/open_memstream.c  ${PROJECT_SOURCE_DIR}/utils/private/src/memstream/fmemopen.c)
		include_directories(${PROJECT_SOURCE_DIR}/utils/public/include/memstream)
	endif()
	add_celix_bundle(bonjour_shell
                VERSION "1.0.0"
		SOURCES
		 	private/src/activator.c
		 	private/src/bonjour_shell.c
			${MEMSTREAM_SOURCES}
	)
	
	target_link_libraries(bonjour_shell celix_framework celix_utils ${LIBXML2_LIBRARIES} ${DNS_SD_LIB})

	add_celix_container("bonjour_shell_deploy" BUNDLES
		shell
		bonjour_shell
		PROPERTIES "bonjour.shell.id=Apache Celix"
	)
endif (SHELL_BONJOUR)

