cmake_minimum_required(VERSION 3.20)

# NAM Core library integration for JUCE projects
# This wraps NeuralAmpModelerCore as a static library

project(NeuralAmpModelerCore VERSION 0.4.0 LANGUAGES CXX)

set(NAM_DIR "${CMAKE_CURRENT_SOURCE_DIR}/NAM")
set(NAM_DEPS_PATH "${CMAKE_CURRENT_SOURCE_DIR}/Dependencies")

# Add Eigen
add_subdirectory("${NAM_DEPS_PATH}/eigen" "${CMAKE_CURRENT_BINARY_DIR}/eigen" EXCLUDE_FROM_ALL)

# Collect NAM source files
set(NAM_SOURCES
    ${NAM_DIR}/activations.cpp
    ${NAM_DIR}/conv1d.cpp
    ${NAM_DIR}/convnet.cpp
    ${NAM_DIR}/dsp.cpp
    ${NAM_DIR}/get_dsp.cpp
    ${NAM_DIR}/lstm.cpp
    ${NAM_DIR}/ring_buffer.cpp
    ${NAM_DIR}/util.cpp
    ${NAM_DIR}/wavenet.cpp
)

set(NAM_HEADERS
    ${NAM_DIR}/activations.h
    ${NAM_DIR}/conv1d.h
    ${NAM_DIR}/convnet.h
    ${NAM_DIR}/dsp.h
    ${NAM_DIR}/film.h
    ${NAM_DIR}/gating_activations.h
    ${NAM_DIR}/get_dsp.h
    ${NAM_DIR}/lstm.h
    ${NAM_DIR}/registry.h
    ${NAM_DIR}/ring_buffer.h
    ${NAM_DIR}/util.h
    ${NAM_DIR}/version.h
    ${NAM_DIR}/wavenet.h
)

# Create static library
add_library(NeuralAmpModelerCore STATIC ${NAM_SOURCES} ${NAM_HEADERS})

target_include_directories(NeuralAmpModelerCore
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${NAM_DIR}
        ${NAM_DEPS_PATH}/nlohmann
    SYSTEM PUBLIC
        ${NAM_DEPS_PATH}/eigen
)

target_link_libraries(NeuralAmpModelerCore
    PUBLIC
        Eigen3::Eigen
)

target_compile_features(NeuralAmpModelerCore PUBLIC cxx_std_17)

# Use float instead of double for JUCE compatibility
target_compile_definitions(NeuralAmpModelerCore PUBLIC NAM_SAMPLE_FLOAT=1)

# Platform-specific settings
if(WIN32)
    target_compile_definitions(NeuralAmpModelerCore PUBLIC NOMINMAX WIN32_LEAN_AND_MEAN)
elseif(APPLE)
    # NAM uses std::filesystem which requires macOS 10.15+
    set_target_properties(NeuralAmpModelerCore PROPERTIES
        OSX_DEPLOYMENT_TARGET "10.15"
        XCODE_ATTRIBUTE_MACOSX_DEPLOYMENT_TARGET "10.15"
    )
    # Force load entire static library to ensure factory registrations are included
    target_link_options(NeuralAmpModelerCore INTERFACE "LINKER:-force_load,$<TARGET_FILE:NeuralAmpModelerCore>")
endif()
