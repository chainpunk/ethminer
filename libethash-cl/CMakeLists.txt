# A custom command and target to turn the OpenCL kernel into a byte array header
# The normal build depends on it properly and if the kernel file is changed, then
# a rebuild of libethash-cl should be triggered

#TODO: clean up the copy&pasting here
add_custom_command(
    OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/ethash_miner_kernel.cpp"
    COMMAND ${CMAKE_COMMAND} ARGS
    -DBIN2H_SOURCE_FILE="${CMAKE_CURRENT_SOURCE_DIR}/ethash_miner_kernel.cl"
    -DBIN2H_VARIABLE_NAME="cl_ethash_miner_kernel"
    -DBIN2H_HEADER_FILE="${CMAKE_CURRENT_BINARY_DIR}/ethash_miner_kernel.cpp"
    -P "${CMAKE_CURRENT_SOURCE_DIR}/bin2h.cmake"
    COMMENT "Generating OpenCL Ethash Kernel Byte Array"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/ethash_miner_kernel.cl"
)

add_custom_command(
    OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/progpow_miner_kernel.cpp"
    COMMAND ${CMAKE_COMMAND} ARGS
    -DBIN2H_SOURCE_FILE="${CMAKE_CURRENT_SOURCE_DIR}/progpow_miner_kernel.cl"
    -DBIN2H_VARIABLE_NAME="cl_progpow_miner_kernel"
    -DBIN2H_HEADER_FILE="${CMAKE_CURRENT_BINARY_DIR}/progpow_miner_kernel.cpp"
    -P "${CMAKE_CURRENT_SOURCE_DIR}/bin2h.cmake"
    COMMENT "Generating OpenCL ProgPoW Kernel Byte Array"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/progpow_miner_kernel.cl"
)

set(SOURCES
    CLMiner.cpp
    "${CMAKE_CURRENT_BINARY_DIR}/ethash_miner_kernel.cpp"
    "${CMAKE_CURRENT_BINARY_DIR}/progpow_miner_kernel.cpp"
)

if(APPLE)
    # On macOS use system OpenCL library.
    find_package(OpenCL REQUIRED)
else()
    hunter_add_package(OpenCL)
    find_package(OpenCL CONFIG REQUIRED)
endif()

include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(..)

add_library(ethash-cl ${SOURCES})
target_link_libraries(ethash-cl PUBLIC ethcore progpow ethash::ethash)
target_link_libraries(ethash-cl PRIVATE OpenCL::OpenCL)
target_link_libraries(ethash-cl PRIVATE Boost::filesystem Boost::thread)
