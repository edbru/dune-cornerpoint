# -*- mode: cmake; tab-width: 2; indent-tabs-mode: t; truncate-lines: t; compile-command: "cmake -Wdev" -*-
# vim: set filetype=cmake autoindent tabstop=2 shiftwidth=2 noexpandtab softtabstop=2 nowrap:

###########################################################################
#                                                                         #
# Note: The bulk of the build system is located in the cmake/ directory.  #
#       This file only contains the specializations for this particular   #
#       project. Most likely you are interested in editing one of these   #
#       files instead:                                                    #
#                                                                         #
#       dune.module                              Name and version number  #
#       CMakeLists_files.cmake                   Path of source files     #
#       cmake/Modules/${project}-prereqs.cmake   Dependencies             #
#                                                                         #
###########################################################################

cmake_minimum_required (VERSION 2.8)

# additional search modules
find_path (OPM_MACROS_ROOT cmake/Modules/OpmInit.cmake
	PATHS ${opm-macros_ROOT}
	      ${CMAKE_CURRENT_SOURCE_DIR}
	DOC "Path to build system macros and scripts"
	NO_DEFAULT_PATH
	)
list (APPEND CMAKE_MODULE_PATH "${OPM_MACROS_ROOT}/cmake/Modules")

# not the same location as most of the other projects? this hook overrides
macro (dir_hook)
endmacro (dir_hook)

# project information is in dune.module. Read this file and set variables.
# we cannot generate dune.module since it is read by dunecontrol before
# the build starts, so it makes sense to keep the data there then.
include (OpmInit)

# list of prerequisites for this particular project; this is in a
# separate file (in cmake/Modules sub-directory) because it is shared
# with the find module
include (${project}-prereqs)

# read the list of components from this file (in the project directory);
# it should set various lists with the names of the files to include
include (CMakeLists_files.cmake)

macro (config_hook)
	opm_need_version_of ("dune-common")
	opm_need_version_of ("dune-grid")
	find_file(_HAVE_DUNE_GRID_CHECKS checkpartition.cc HINTS ${dune-grid_INCLUDE_DIR}
		PATH_SUFFIXES dune/grid/test)
	if(_HAVE_DUNE_GRID_CHECKS)
		set(HAVE_DUNE_GRID_CHECKS 1)
	else(_HAVE_DUNE_GRID_CHECK)
		set(HAVE_DUNE_GRID_CHECKS 0)
  endif(_HAVE_DUNE_GRID_CHECKS)
	list (APPEND ${project}_CONFIG_IMPL_VARS
		HAVE_DUNE_GRID_CHECKS
		)
endmacro (config_hook)

macro (prereqs_hook)
endmacro (prereqs_hook)

macro (sources_hook)
endmacro (sources_hook)

macro (fortran_hook)
endmacro (fortran_hook)

macro (files_hook)
endmacro (files_hook)

macro (tests_hook)
endmacro (tests_hook)

# all setup common to the OPM library modules is done here
include (OpmLibMain)

add_custom_target(check-commits
									COMMAND ${CMAKE_COMMAND}
													-DPROJECT_SOURCE_DIR=${PROJECT_SOURCE_DIR}
													-DCMAKE_BINARY_DIR=${CMAKE_BINARY_DIR}
													-P ${PROJECT_SOURCE_DIR}/cmake/Scripts/CheckCommits.cmake)
