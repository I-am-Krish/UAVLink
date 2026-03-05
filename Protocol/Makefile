CC = gcc
CFLAGS = -Wall -O2

# Source files
CORE_SRC = uavlink.c uavlink_fast.c uavlink_compress.c uavlink_hw_crypto.c monocypher.c
CORE_OBJ = uavlink.o uavlink_fast.o uavlink_compress.o uavlink_hw_crypto.o monocypher.o

# Headers to ship
HEADERS = uavlink.h uavlink_fast.h uavlink_compress.h uavlink_hw_crypto.h monocypher.h

# Platform-specific flags
ifeq ($(OS),Windows_NT)
    LDFLAGS = -lws2_32 -lm
    EXT = .exe
    LIB_NAME = uavlink_win64.lib
    AR = ar
    MKDIR = if not exist "$(subst /,\,$1)" mkdir "$(subst /,\,$1)"
    RM_RF = if exist "$(subst /,\,$1)" rmdir /s /q "$(subst /,\,$1)"
    RM_F = del /q /f
    CP = copy /y
    XCOPY = xcopy /y /i /q
else
    LDFLAGS = -lm
    EXT =
    LIB_NAME = libuavlink_linux.a
    AR = ar
    MKDIR = mkdir -p $1
    RM_RF = rm -rf $1
    RM_F = rm -f
    CP = cp
    XCOPY = cp
endif

# SDK output directories
SDK_DIR = uavlink-sdk
RELEASE_DIR = release

# ==============================================================
#  Standard targets
# ==============================================================

all: benchmark receiver simulator

benchmark: uavlink_benchmark.c $(CORE_SRC)
	$(CC) $(CFLAGS) -o uavlink_benchmark$(EXT) uavlink_benchmark.c $(CORE_SRC) $(LDFLAGS)

receiver: gcs_receiver.c $(CORE_SRC)
	$(CC) $(CFLAGS) -o gcs_receiver$(EXT) gcs_receiver.c $(CORE_SRC) $(LDFLAGS)

simulator: uav_simulator.c $(CORE_SRC)
	$(CC) $(CFLAGS) -o uav_simulator$(EXT) uav_simulator.c $(CORE_SRC) $(LDFLAGS)

# ==============================================================
#  Static library
# ==============================================================

objects: $(CORE_SRC)
	$(CC) $(CFLAGS) -c $(CORE_SRC)

lib: objects
	$(AR) rcs $(LIB_NAME) $(CORE_OBJ)
	@echo "Built static library: $(LIB_NAME)"

# ==============================================================
#  SDK: library + headers + examples
# ==============================================================

sdk: lib
	$(call MKDIR,$(SDK_DIR)/include)
	$(call MKDIR,$(SDK_DIR)/lib)
	$(call MKDIR,$(SDK_DIR)/examples)
ifeq ($(OS),Windows_NT)
	$(XCOPY) uavlink.h "$(SDK_DIR)\include\"
	$(XCOPY) uavlink_fast.h "$(SDK_DIR)\include\"
	$(XCOPY) uavlink_compress.h "$(SDK_DIR)\include\"
	$(XCOPY) uavlink_hw_crypto.h "$(SDK_DIR)\include\"
	$(XCOPY) monocypher.h "$(SDK_DIR)\include\"
	$(XCOPY) $(LIB_NAME) "$(SDK_DIR)\lib\"
	$(XCOPY) uav_simulator.c "$(SDK_DIR)\examples\"
	$(XCOPY) gcs_receiver.c "$(SDK_DIR)\examples\"
	$(XCOPY) uavlink_benchmark.c "$(SDK_DIR)\examples\"
	$(XCOPY) ..\README.md "$(SDK_DIR)\"
	$(XCOPY) ..\LICENSE "$(SDK_DIR)\"
else
	$(CP) $(HEADERS) $(SDK_DIR)/include/
	$(CP) $(LIB_NAME) $(SDK_DIR)/lib/
	$(CP) uav_simulator.c gcs_receiver.c uavlink_benchmark.c $(SDK_DIR)/examples/
	$(CP) ../README.md ../LICENSE $(SDK_DIR)/
endif
	@echo.
	@echo === SDK packaged in $(SDK_DIR)/ ===
	@echo   include/  - Headers
	@echo   lib/      - Static library ($(LIB_NAME))
	@echo   examples/ - Example applications

# ==============================================================
#  Release: SDK + pre-built executables
# ==============================================================

release: sdk all
	$(call MKDIR,$(RELEASE_DIR)/bin)
ifeq ($(OS),Windows_NT)
	xcopy /y /i /q /e "$(SDK_DIR)\*" "$(RELEASE_DIR)\"
	$(XCOPY) uav_simulator$(EXT) "$(RELEASE_DIR)\bin\"
	$(XCOPY) gcs_receiver$(EXT) "$(RELEASE_DIR)\bin\"
	$(XCOPY) uavlink_benchmark$(EXT) "$(RELEASE_DIR)\bin\"
else
	cp -r $(SDK_DIR)/* $(RELEASE_DIR)/
	$(CP) uav_simulator$(EXT) gcs_receiver$(EXT) uavlink_benchmark$(EXT) $(RELEASE_DIR)/bin/
endif
	@echo.
	@echo === Release packaged in $(RELEASE_DIR)/ ===

# ==============================================================
#  Package: create ZIP archive
# ==============================================================

package: release
ifeq ($(OS),Windows_NT)
	powershell -Command "Compress-Archive -Path '$(RELEASE_DIR)\*' -DestinationPath 'UAVLink-v1.0.0-windows.zip' -Force"
	@echo Created UAVLink-v1.0.0-windows.zip
else
	tar -czf UAVLink-v1.0.0-linux.tar.gz -C $(RELEASE_DIR) .
	@echo Created UAVLink-v1.0.0-linux.tar.gz
endif

# ==============================================================
#  Clean
# ==============================================================

clean:
ifeq ($(OS),Windows_NT)
	$(RM_F) *.o *.exe *.lib *.a 2>nul
	$(call RM_RF,$(SDK_DIR))
	$(call RM_RF,$(RELEASE_DIR))
	$(RM_F) UAVLink-*.zip UAVLink-*.tar.gz 2>nul
else
	rm -f *.o *.exe *.lib *.a uavlink_benchmark gcs_receiver uav_simulator
	rm -rf $(SDK_DIR) $(RELEASE_DIR)
	rm -f UAVLink-*.zip UAVLink-*.tar.gz
endif

.PHONY: all benchmark receiver simulator objects lib sdk release package clean
