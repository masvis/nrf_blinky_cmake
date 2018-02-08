set(NRF_SDK_ROOT $ENV{HOME}/Nordic/sdk)
set(NRFJPROG_PATH $ENV{HOME}/Nordic/cli/nrfjprog)

cmake_minimum_required(VERSION 3.6)
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
        ${NRF_SDK_ROOT}/components/drivers_nrf/nrf_soc_nosd
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

set(LDFLAGS "-O3 -g3 \
-mthumb -mabi=aapcs -L ${TEMPLATE_PATH} -Tblinky_gcc_nrf52_blank.ld \
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

add_executable(nrf52832_xxaa ${SOURCE_FILES})

set_target_properties(
        nrf52832_xxaa
        PROPERTIES
        OUTPUT_NAME "nrf52832_xxaa"
        SUFFIX ".out"
)

add_custom_target(size
        DEPENDS nrf52832_xxaa
        COMMAND ${SIZE} nrf52832_xxaa.out
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_custom_target(createhex
        DEPENDS size nrf52832_xxaa
        COMMAND ${OBJCOPY} -O ihex nrf52832_xxaa.out nrf52832_xxaa.hex
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_custom_target(flash
        DEPENDS createhex size nrf52832_xxaa
        COMMAND ${NRFJPROG_PATH}/nrfjprog --program nrf52832_xxaa.hex -f nrf52 --sectorerase
        COMMAND ${NRFJPROG_PATH}/nrfjprog --reset -f nrf52
        WORKING_DIRECTORY ${CMAKE_BINARY_DIR})

add_custom_target(flash_softdevice_nrf52_132_2
        COMMAND ${NRFJPROG_PATH}/nrfjprog --program support/s132_nrf52_2.0.1_softdevice.hex -f nrf52 --sectorerase
        COMMAND ${NRFJPROG_PATH}/nrfjprog --reset -f nrf52
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})

add_custom_target(flash_softdevice_nrf52_132_3
        COMMAND ${NRFJPROG_PATH}/nrfjprog --program support/s132_nrf52_3.0.0_softdevice.hex -f nrf52 --sectorerase
        COMMAND ${NRFJPROG_PATH}/nrfjprog --reset -f nrf52
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})