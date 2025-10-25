.. _doc_godot_cpp_build_system_cmake_new_project:


Writing a CMakeLists file for a New GDExtension Project
=======================================================

It's highly recommended to become familiar with the official
documentation and tutorials_

.. _tutorials: https://cmake.org/cmake/help/latest/guide/tutorial/Getting%20Started%20with%20CMake.html#

The Most Basic
--------------
Now that you know where to look for detailed information, lets start off with
something simple, and then move it towards a complete solution.

Below is what you might find in any beginners tutorial, or starting template:

.. code-block:: cmake

    cmake_minimum_required(VERSION 3.17)

    project( MyExtensionProject
        # VERSION <major>[.<minor>[.<patch>[.<tweak>]]]
        DESCRIPTION "This is an example cmake project"
        HOMEPAGE_URL "http://www.godotengine.org"
        LANGUAGES CXX)

    add_library( my_library SHARED )

    target_sources( my_library
        PRIVATE
            library.cpp
    )


Adding Dependencies
-------------------

Read the documentation on how to add external dependencies, this section will
show you how to add godot-cpp as a dependency.

.. _cmake_dependencies: https://cmake.org/cmake/help/latest/guide/tutorial/Finding%20Dependencies.html

.. code-block:: cmake

    # Python
    find_package(Python3 3.4 REQUIRED) # pathlib should be present

Adding the godot-cpp Library
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

setting the configuration of the godot-cpp library happens before including it
as a sub project

.. code-block:: cmake

    set( GODOTCPP_BUILD_PROFILE "${CMAKE_SOURCE_DIR}/build_profile.json" )

Because this is C++ There are innumerable ways to consume an externally
developed library, in the SCons example we are recommended to add godot-cpp as
a git submodule. Here is a list of alternatives:

- git submodules
- Use CMake's FetchContent features
- Use CMake's External Project features

It is not recommended is to use a pre-built library from a package manager,
because the godot-cpp extension library is highly targeted to the specific
godot binary that will be shipped as your game, there is no expectation of
broad compatibility between different binaries though a great deal of thought
and effort is spent to achieve as good as possible.

.. tabs::

    .. tab:: Git submodules

        This section is pulled directly from modern_cmake_

        .. _modern_cmake: https://cliutils.gitlab.io/modern-cmake/chapters/projects/submodule.html

        .. code-block:: cmake

            find_package(Git QUIET)
            if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
            # Update submodules as needed
                option(GIT_SUBMODULE "Check submodules during build" ON)
                if(GIT_SUBMODULE)
                    message(STATUS "Submodule update")
                    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                                    RESULT_VARIABLE GIT_SUBMOD_RESULT)
                    if(NOT GIT_SUBMOD_RESULT EQUAL "0")
                        message(FATAL_ERROR "git submodule update --init --recursive failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
                    endif()
                endif()
            endif()

            if(NOT EXISTS "${PROJECT_SOURCE_DIR}/extern/repo/CMakeLists.txt")
                message(FATAL_ERROR "The submodules were not downloaded! GIT_SUBMODULE was turned off or failed. Please update submodules and try again.")
            endif()

            add_subdirectory(extern/repo)

    .. tab:: FetchContent

        Better CMake Part 10 -- When to use FetchContent
        https://youtu.be/GIGHalVqSBE

        .. code-block:: cmake

            include( FetchContent )

            # Godot-cpp
            set( GODOTCPP_GIT_URL "http://github.com/godotengine/godot-cpp.git" CACHE STRING "The git url of godot-cpp to fetch" )
            set( GODOTCPP_GIT_BRANCH "master" CACHE STRING "The git branch of godot-cpp to fetch" )
            FetchContent_Declare( godot-cpp
                    GIT_REPOSITORY ${GODOTCPP_GIT_URL}
                    GIT_TAG ${GODOTCPP_GIT_BRANCH}
                    GIT_TAG godot-4.5-stable
                    GIT_PROGRESS ON
            )
            FetchContent_MakeAvailable( godot-cpp )

    .. tab:: External Project

        Better CMake Part 6 -- Superbuilds w/ ExternalProject
        https://youtu.be/nBptg3SHPGU

.. code-block:: cmake

    # This is a cmake comment.

Adding Documentation
~~~~~~~~~~~~~~~~~~~~

I need to add examples of including documentation in here, and links to the documentation sections.

Adding Platform Compatibility
-----------------------------

.. tabs::

   .. tab:: Windows

        Windows Specific things

   .. tab:: Linux

        Linux Specific things

   .. tab:: macOS

        macOS Specific things

   .. tab:: iOS

        iOS Specific things

   .. tab:: visionOS

        visionOS Specific things

   .. tab:: Web

        The web build has one very specific requirement, and that's due to
        the CMake toolchain file for emscripten disallowing support for shared
        libraries. Emscripten itself does support `dynamic linking`_, and there are
        long standing issues_, and pull requests to fix the toolchain.

        We can work around the toolchain problems by using CMake's code injection_
        feature.

        .. _`dynamic linking`: https://emscripten.org/docs/compiling/Dynamic-Linking.html

        .. _issues: https://github.com/emscripten-core/emscripten/issues/20340

        .. _injection: https://cmake.org/cmake/help/latest/command/project.html#code-injection

        Within the ``godot-cpp/cmake/`` folder there is a ``emsdkHack.cmake`` file which
        provides the necessary functionality and can be used for the code injection.

        .. code-block:: cmake

            set(CMAKE_PROJECT_<my project name goes here>_INCLUDE ${godot-cpp_SOURCE_DIR}/cmake/emsdkHack.cmake)

   .. tab:: Android

        Android Specific things
