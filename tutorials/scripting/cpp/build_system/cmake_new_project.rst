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
        HOMEPAGE_URL "http://www.my-awesome-extension.org"
        LANGUAGES CXX)

    add_library( my_library SHARED )

    target_sources( my_library
        PRIVATE
            library.cpp
    )

Incorporating the godot-cpp library
-----------------------------------

The CMake script needs to know where the godot-cpp library is so it can link the
extension target to it.

godot-cpp is unlikely to be found in your system libraries, or be available to
install by your package manager because it's not intended to exist as a
pre-compiled thing. For finding such libraries look to the CMake documentation
on finding dependencies_.

.. _dependencies: https://cmake.org/cmake/help/latest/guide/tutorial/Finding%20Dependencies.html

Instead it is intended to have the godot-cpp source code available to
incorporate into your build system, customised to your specific needs,
compiled, and linked to your project.

The :ref:`SCons document <doc_godot_cpp_build_system>` provide the example
of using a git submodule approach. We can do the same here, and also take
advantage of some of CMake's features.

Finding the godot-cpp Library
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

        Official Documentation for FetchContent.
        https://cmake.org/cmake/help/latest/module/FetchContent.html

        .. code-block:: cmake

            include( FetchContent )
            FetchContent_Declare( godot-cpp
                    GIT_REPOSITORY http://github.com/godotengine/godot-cpp.git
                    GIT_TAG godot-4.5-stable
                    GIT_PROGRESS ON
            )
            FetchContent_MakeAvailable( godot-cpp )

    .. tab:: External Project

        Better CMake Part 6 -- Superbuilds w/ ExternalProject
        https://youtu.be/nBptg3SHPGU

.. code-block:: cmake

    # This is a cmake comment.

Configuring the godot-cpp Library
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Setting the configuration of the godot-cpp library happens before including it
into the project. I know the order I am explaining things might seem a little
backwards, but bear with me.

You can check what options added libraries expose by looking at the CMake cache

The one's were most interested in are :code:`GODOTCPP_BUILD_PROFILE`,
:code:`GODOTCPP_TARGET`



.. code-block:: cmake

    set( GODOTCPP_BUILD_PROFILE "${CMAKE_SOURCE_DIR}/build_profile.json" )

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
