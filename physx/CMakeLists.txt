cmake_minimum_required(VERSION 3.10.0)
project(physx-build)

# Force minimum policy version to 3.10 for compatibility with CMake 4.0+
set(CMAKE_POLICY_VERSION_MINIMUM 3.10)

set(PHYSX_ROOT_DIR "${CMAKE_CURRENT_LIST_DIR}/src/physx")

set(PX_OUTPUT_LIB_DIR "${CMAKE_CURRENT_LIST_DIR}")
set(PX_OUTPUT_BIN_DIR "${CMAKE_CURRENT_BINARY_DIR}/bin")

set(PX_GENERATE_STATIC_LIBRARIES ON CACHE BOOL "Enable static library generation" FORCE)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

set(PHYSX_CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/src/externals/cmakemodules)
set(ENV{PM_CMakeModules_PATH} ${PHYSX_CMAKE_MODULE_PATH})
set(ENV{PM_PxShared_PATH} ${CMAKE_CURRENT_LIST_DIR}/src/pxshared)

if(UNIX)

    set(TARGET_BUILD_PLATFORM linux)

    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "16.1.0")
            add_compile_options(-Wno-unused-but-set-variable)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "14.1.0")
            add_compile_options(-Wno-template-id-cdtor)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "11.1.0")
            add_compile_options(-Wno-mismatched-new-delete -Wno-stringop-overread)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "8.1.0")
            add_compile_options(-Wno-class-memaccess)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "7.1.0")
            add_compile_options(-Wno-restrict -Wno-stringop-overflow)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "4.7.0")
            add_compile_options(-Wno-nonnull -Wno-free-nonheap-object)
        endif()
        add_compile_options(-Wno-array-bounds)
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "21.1.0")
            add_compile_options(-Wno-nrvo)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "20.1.0")
            add_compile_options(-Wno-unsafe-buffer-usage-in-libc-call)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "16.0.0")
            add_compile_options(-Wno-unsafe-buffer-usage)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "14.0.0")
            add_compile_options(-Wno-bitwise-instead-of-logical -Wno-unaligned-access)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "13.0.0")
            add_compile_options(-Wno-unused-but-set-variable -Wno-reserved-identifier)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "12.0.0")
            add_compile_options(-Wno-suggest-override -Wno-suggest-destructor-override)
        endif()
        if(CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL "11.0.0")
            add_compile_options(-Wno-dtor-name)
        endif()
            add_compile_options(-Wno-switch-default -Wno-invalid-offsetof -Wno-missing-include-dirs)
    endif()

    if(CMAKE_BUILD_TYPE MATCHES Debug)
        # Output minimal debug info to reduce file size
        add_compile_options(-g1)
    endif()

elseif(WIN32)

    set(TARGET_BUILD_PLATFORM windows)
    set(NV_USE_STATIC_WINCRT ON CACHE BOOL "Enable PhysX static lib generation" FORCE)

    if(CMAKE_BUILD_TYPE MATCHES Debug)
        set(NV_USE_DEBUG_WINCRT TRUE CACHE BOOL "Force PhysX to use Debug WINCRT" FORCE)
    else()
        set(NV_USE_DEBUG_WINCRT FALSE CACHE BOOL "Force PhysX to use Debug WINCRT" FORCE)
    endif()

endif()

add_subdirectory(src/physx/compiler/public)


set(HEADER_GUARD_NAME "CONFIG")
set(HEADER_CONTENT "#define PX_PHYSX_STATIC_LIB\n")
# regenerate PxConfig.h header with a fixed definition so it can be the same across all platforms
configure_file(${PHYSX_CMAKE_MODULE_PATH}/template/PxIncludeTemplate.h ${PHYSX_ROOT_DIR}/include/PxConfig.h)

