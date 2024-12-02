.PHONY: all build cmake clean flash

# Quick reminder:
# ?= set if not set
# := expand at init
# = expand at eval

BUILD_DIR := build
BUILD_TYPE ?= Debug
# BUILD_TYPE ?= RelWithDebInfo 

OUTPUT_BIN_NAME := output
# OUTPUT_BIN_NAME := $(notdir $(CURDIR))

all: build

cmake:
	cmake -B${BUILD_DIR} -DCMAKE_BUILD_TYPE=${BUILD_TYPE} -DOUTPUT_BIN_NAME=${OUTPUT_BIN_NAME}

build: cmake
	$(MAKE) -C ${BUILD_DIR} --no-print-directory

clean:
	rm -rf $(BUILD_DIR)

flash: build
	st-flash --reset write ${BUILD_DIR}/${OUTPUT_BIN_NAME}.bin 0x08000000


