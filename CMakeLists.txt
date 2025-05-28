cmake_minimum_required (VERSION 3.10)

project(Tutorial22_HybridRendering CXX)

include_directories(
    ${CMAKE_CURRENT_LIST_DIR}/../../ThirdParty/glfw/include
)

set(SOURCE
    src/Tutorial22_HybridRendering.cpp
)

set(INCLUDE
    src/Tutorial22_HybridRendering.hpp
)

set(SHADERS
    assets/Structures.fxh
    assets/Utils.fxh
    assets/RayQueryMtl.fxh
    assets/Rasterization.vsh
    assets/Rasterization.psh
    assets/PostProcess.vsh
    assets/PostProcess.psh
    assets/RayTracing.csh
)

set(ASSETS
    assets/DGLogo0.png
    assets/DGLogo1.png
    assets/DGLogo2.png
    assets/DGLogo3.png
    assets/Marble.jpg
)

add_sample_app("Tutorial22_HybridRendering" "DiligentSamples/Tutorials" "${SOURCE}" "${INCLUDE}" "${SHADERS}" "${ASSETS}")

target_link_libraries(Tutorial22_HybridRendering PRIVATE glfw)