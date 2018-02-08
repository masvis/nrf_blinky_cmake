cmake_minimum_required(VERSION 3.6)

set(NRF_SDK_ROOT $ENV{HOME}/Nordic/sdk)
set(NRFJPROG_PATH $ENV{HOME}/Nordic/cli/nrfjprog)
set(SOFTDEVICE ON)

project(blinky_pca10040_s132 C ASM)

include(CMakeForceCompiler)

set(CC /usr/bin/arm-none-eabi-gcc)
set(CXX /usr/bin/arm-none-eabi-g++)
set(OBJCOPY /usr/bin/arm-none-eabi-objcopy)
set(SIZE /usr/bin/arm-none-eabi-size)
set(TEMPLATE_PATH ${NRF_SDK_ROOT}/components/toolchain/gcc)

set(CMAKE_C_COMPILER ${CC})
set(CMAKE_CXX_COMPILER ${CXX})
set(CMAKE_ASM_COMPILER ${CC})

set(CMAKE_SYSTEM_NAME Generic)

set(CMAKE_VERBOSE_MAKEFILE ON)
set(CMAKE_C_STANDARD 99)

set(SRC_FILES
        ${NRF_SDK_ROOT}/components/toolchain/gcc/gcc_startup_nrf52.S
        ${NRF_SDK_ROOT}/components/toolchain/system_nrf52.c
        ${NRF_SDK_ROOT}/components/boards/boards.c
        ${NRF_SDK_ROOT}/components/libraries/util/app_error.c
        ${NRF_SDK_ROOT}/components/libraries/util/app_error_weak.c
        ${NRF_SDK_ROOT}/components/libraries/util/app_util_platform.c
        ${NRF_SDK_ROOT}/components/libraries/util/nrf_assert.c
        ${NRF_SDK_ROOT}/components/libraries/strerror/nrf_strerror.c)

set(INC_FOLDERS
        ${NRF_SDK_ROOT}/components
        ${NRF_SDK_ROOT}/components/libraries/experimental_memobj
        ${NRF_SDK_ROOT}/components/libraries/experimental_section_vars
        ${NRF_SDK_ROOT}/components/toolchain
        ${NRF_SDK_ROOT}/components/libraries/util
        ${NRF_SDK_ROOT}/components/libraries/balloc
        ${NRF_SDK_ROOT}/components/libraries/experimental_log
        ${NRF_SDK_ROOT}/components/libraries/experimental_log/src
        ${NRF_SDK_ROOT}/components/toolchain/gcc
        ${NRF_SDK_ROOT}/components/libraries/bsp
        ${NRF_SDK_ROOT}/components/device
        ${NRF_SDK_ROOT}/components/boards
        ${NRF_SDK_ROOT}/components/drivers_nrf/delay
        ${NRF_SDK_ROOT}/components/toolchain/cmsis/include
        ${NRF_SDK_ROOT}/components/drivers_nrf/hal
        ${NRF_SDK_ROOT}/components/libraries/strerror)

set(CFLAGS "-O3 -g3 \
-DBOARD_PCA10040 \
-DBSP_DEFINES_ONLY \
-DCONFIG_GPIO_AS_PINRESET \
-DFLOAT_ABI_HARD \
-DNRF52 \
-DNRF52832_XXAA \
-DNRF52_PAN_74 \
-mcpu=cortex-m4 \
-mthumb -mabi=aapcs \
-Wall -Werror \
-mfloat-abi=hard -mfpu=fpv4-sp-d16 \
-ffunction-sections -fdata-sections -fno-strict-aliasing \
-fno-builtin -fshort-enums")

set(ASMFLAGS "-g3 \
-mcpu=cortex-m4 \
-mthumb -mabi=aapcs \
-mfloat-abi=hard -mfpu=fpv4-sp-d16 \
-DBOARD_PCA10040 \
-DBSP_DEFINES_ONLY \
-DCONFIG_GPIO_AS_PINRESET \
-DFLOAT_ABI_HARD \
-DNRF52 \
-DNRF52832_XXAA \
-DNRF52_PAN_74")

set(FLASH_COMMANDS
        COMMAND ${NRFJPROG_PATH}/nrfjprog -f nrf52 --eraseall
        COMMAND ${NRFJPROG_PATH}/nrfjprog --program ${PROJECT_NAME}.hex -f nrf52 --sectorerase
        COMMAND ${NRFJPROG_PATH}/nrfjprog --reset -f nrf52)

if (SOFTDEVICE STREQUAL "ON")
    message("Building with softdevice")
    set(LINKER_SCRIPT blinky_gcc_nrf52_pca10040_s132.ld)
    set(SRC_FILES ${SRC_FILES}
            ${NRF_SDK_ROOT}/components/libraries/experimental_section_vars/nrf_section_iter.c
            ${NRF_SDK_ROOT}/components/softdevice/common/nrf_sdh.c
            ${NRF_SDK_ROOT}/components/softdevice/common/nrf_sdh_soc.c)

    set(INC_FOLDERS ${INC_FOLDERS}
            ${NRF_SDK_ROOT}/components/softdevice/s132/headers
            ${NRF_SDK_ROOT}/components/toolchain/cmsis/include
            ${NRF_SDK_ROOT}/components/softdevice/common
            ${NRF_SDK_ROOT}/components/softdevice/s132/headers/nrf52)
    set(CFLAGS "${CFLAGS} \
-DNRF_SD_BLE_API_VERSION=5 \
-DS132 \
-DSOFTDEVICE_PRESENT")
    set(ASMFLAGS "${ASMFLAGS} \
-DNRF_SD_BLE_API_VERSION=5 \
-DS132 \
-DSOFTDEVICE_PRESENT")
    set(FLASH_COMMANDS
            ${FLASH_COMMANDS}
            COMMAND ${NRFJPROG_PATH}/nrfjprog --program ${NRF_SDK_ROOT}/components/softdevice/s132/hex/s132_nrf52_5.0.0_softdevice.hex -f nrf52 --sectorerase
            COMMAND ${NRFJPROG_PATH}/nrfjprog --reset -f nrf52)
else ()
    message("Building without softdevice")
    set(LINKER_SCRIPT blinky_gcc_nrf52_blank.ld)
    set(INC_FOLDERS ${INC_FOLDERS}
            ${NRF_SDK_ROOT}/components/drivers_nrf/nrf_soc_nosd)
endif ()

set(LDFLAGS "-O3 -g3 \
-mthumb -mabi=aapcs -L ${TEMPLATE_PATH} -T${LINKER_SCRIPT} \
-mcpu=cortex-m4 \
-mfloat-abi=hard -mfpu=fpv4-sp-d16 \
-Wl,--gc-sections \
--specs=nano.specs")

set(SOURCE_FILES main.c
        ${SRC_FILES})

include_directories(
        .
        ${INC_FOLDERS})

set(CMAKE_C_FLAGS ${CFLAGS})
set(CMAKE_ASM_FLAGS ${ASMFLAGS})
set(CMAKE_EXE_LINKER_FLAGS ${LDFLAGS})

add_executable(${PROJECT_NAME} ${SOURCE_FILES})

set_target_properties(
        ${PROJECT_NAME}
        PROPERTIES
        OUTPUT_NAME "${PROJECT_NAME}"
        SUFFIX ".out"
)

add_custom_target(size
        DEPENDS ${PROJECT_NAME}
        COMMAND ${SIZE} ${PROJECT_NAME}.out
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_custom_target(createhex
        DEPENDS size ${PROJECT_NAME}
        COMMAND ${OBJCOPY} -O ihex ${PROJECT_NAME}.out ${PROJECT_NAME}.hex
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_custom_target(flash
        DEPENDS createhex size ${PROJECT_NAME}
        ${FLASH_COMMANDS}
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR})