cmake_minimum_required(VERSION 3.16)

project(
    scmrtos
    VERSION 5.2.0
    DESCRIPTION "Tiny Real-Time Preemptive Operating System intended for use with Single-Chip Microcontrollers"
    HOMEPAGE_URL "https://github.com/scmrtos/scmrtos"
    LANGUAGES NONE
)

add_library(scmrtos_scmrtos INTERFACE)
add_library(scmrtos::scmrtos ALIAS scmrtos_scmrtos)

# Get config location
if (NOT SCM_CONFIG_DIR)
    message(FATAL_ERROR "scmRTOS: SCM_CONFIG_DIR must be specified")
endif()
get_filename_component(SCM_CONFIG_DIR "${SCM_CONFIG_DIR}" REALPATH)

# Get target arch
if (NOT SCM_ARCH)
    message(FATAL_ERROR "scmRTOS: SCM_ARCH must be specified")
endif()

# Get used toolchain
if (NOT SCM_TOOLCHAIN)
    message(NOTICE "scmRTOS: SCM_TOOLCHAIN was not set, will default to \"gcc\"...")
    set(SCM_TOOLCHAIN "gcc")
endif()

# Get current dir path
set(SCM_DIR "${CMAKE_CURRENT_LIST_DIR}")

message("SCMDIR: ${SCM_DIR}")

# Kernel files
set(SCM_CORE_SOURCES
    "${SCM_DIR}/core/os_kernel.cpp"
    "${SCM_DIR}/core/os_services.cpp"
    "${SCM_DIR}/core/usrlib.cpp")

# Detect port source location
if (${SCM_ARCH} MATCHES "^cortex")
    if (${SCM_TOOLCHAIN} STREQUAL "gcc")
        # mx-gcc for all cortex-m
        set(SCM_PORT_DIR "${SCM_DIR}/port/cortex/mx-gcc")
    elseif(${SCM_TOOLCHAIN} STREQUAL "iar")
        if (${SCM_ARCH} STREQUAL "cortex/m3")
            # m3-iar for cortex-m3
            set(SCM_PORT_DIR "${SCM_DIR}/port/cortex/m3-iar")
        else ()
            # mx-iar for all cortex-m except m3
            set(SCM_PORT_DIR "${SCM_DIR}/port/cortex/mx-iar")
        endif()
    else()
        message(ERROR "scmRTOS: Invalud toolchain for Cortex-M")
    endif()
else()
    set(SCM_PORT_DIR "${SCM_DIR}/port/${SCM_ARCH}/${SCM_TOOLCHAIN}")
endif()

# Append port sources
list(APPEND SCM_CORE_SOURCES ${SCM_PORT_DIR}/os_target.cpp)

# Append os_target_asm.S if needed
set(SCM_PORT_ASM_S ${SCM_PORT_DIR}/os_target_asm.S)
message(NOTICE "scmRTOS: Check for \"${SCM_PORT_ASM_S}\"...")
if (EXISTS ${SCM_PORT_ASM_S})
    list(APPEND SCM_CORE_SOURCES ${SCM_PORT_ASM_S})
endif()

# Core include dirs
set(SCM_CORE_INCLUDES
    ${SCM_DIR}/core
    ${SCM_PORT_DIR}
    ${SCM_CONFIG_DIR}
)

# Core library
target_sources(scmrtos_scmrtos INTERFACE ${SCM_CORE_SOURCES})
target_include_directories(scmrtos_scmrtos INTERFACE ${SCM_CORE_INCLUDES})