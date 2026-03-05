cmake_minimum_required(VERSION 3.15)
project(UAVLink VERSION 1.0.0 LANGUAGES C)

set(CMAKE_C_STANDARD 99)

# ---- Core source files ----
set(UAVLINK_SOURCES
    Protocol/uavlink.c
    Protocol/uavlink_fast.c
    Protocol/uavlink_compress.c
    Protocol/uavlink_hw_crypto.c
    Protocol/monocypher.c
)

set(UAVLINK_HEADERS
    Protocol/uavlink.h
    Protocol/uavlink_fast.h
    Protocol/uavlink_compress.h
    Protocol/uavlink_hw_crypto.h
    Protocol/monocypher.h
)

# ---- Static Library ----
add_library(uavlink STATIC ${UAVLINK_SOURCES})
target_include_directories(uavlink PUBLIC Protocol/)
target_compile_options(uavlink PRIVATE -Wall -O2)

# ---- Executables ----
add_executable(uav_simulator Protocol/uav_simulator.c)
target_link_libraries(uav_simulator PRIVATE uavlink)
target_compile_options(uav_simulator PRIVATE -Wall -O2)

add_executable(gcs_receiver Protocol/gcs_receiver.c)
target_link_libraries(gcs_receiver PRIVATE uavlink)
target_compile_options(gcs_receiver PRIVATE -Wall -O2)

add_executable(uavlink_benchmark Protocol/uavlink_benchmark.c)
target_link_libraries(uavlink_benchmark PRIVATE uavlink)
target_compile_options(uavlink_benchmark PRIVATE -Wall -O2)

# ---- Platform-specific linking ----
if(WIN32)
    target_link_libraries(uav_simulator PRIVATE ws2_32)
    target_link_libraries(gcs_receiver PRIVATE ws2_32)
    target_link_libraries(uavlink_benchmark PRIVATE ws2_32)
endif()

target_link_libraries(uav_simulator PRIVATE m)
target_link_libraries(gcs_receiver PRIVATE m)
target_link_libraries(uavlink_benchmark PRIVATE m)

# ---- Install targets ----
install(TARGETS uavlink
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(TARGETS uav_simulator gcs_receiver uavlink_benchmark
    RUNTIME DESTINATION bin
)

install(FILES ${UAVLINK_HEADERS}
    DESTINATION include/uavlink
)

install(FILES
    Protocol/uav_simulator.c
    Protocol/gcs_receiver.c
    Protocol/uavlink_benchmark.c
    DESTINATION examples
)

install(FILES README.md LICENSE DESTINATION .)
