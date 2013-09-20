-------------------------------------------------------------------------------
CIS565 Project 1: CUDA Raytracer
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------
Ricky Arietta Fall 2013
-------------------------------------------------------------------------------

![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/006_final.bmp)

-------------------------------------------------------------------------------
Initial Ray Casting From Camera & Geometry Intersection
-------------------------------------------------------------------------------
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/001_flat_shading.bmp)

-------------------------------------------------------------------------------
Addition of Basic Diffuse Shading with Point Lighting
-------------------------------------------------------------------------------
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/002_diffuse_illumination.bmp)

-------------------------------------------------------------------------------
Addition of Hard Shadows with Point Lighting
-------------------------------------------------------------------------------
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/003_diffuse_illumination_with_hard_shadows.bmp)

-------------------------------------------------------------------------------
Addition of Soft Shadows with Area Lighting
-------------------------------------------------------------------------------
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/004_diffuse_illumination_with_soft_shadows.bmp)

-------------------------------------------------------------------------------
Addition of Phong Illumination and Reflective Ray Tracing
-------------------------------------------------------------------------------
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/005_phong_illumination_with_soft_shadows_and_reflections.bmp)

-------------------------------------------------------------------------------
Addition of Supersampled AntiAliasing
-------------------------------------------------------------------------------
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/005_phong_illumination_with_soft_shadows_and_reflections_and_supersampled_antialiasing.bmp)

-------------------------------------------------------------------------------
PERFORMANCE EVALUATION
-------------------------------------------------------------------------------
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/block_data_chart.bmp)
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/block_data_graph.bmp)

-------------------------------------------------------------------------------
BASE CODE TOUR:
-------------------------------------------------------------------------------
The main files of interest in this prooject, which handle the ray-tracing
algorithm and image generation, are the following:

* raytraceKernel.cu contains the core raytracing CUDA kernel. You will need to
  complete:
    * cudaRaytraceCore() handles kernel launches and memory management; this
      function already contains example code for launching kernels,
      transferring geometry and cameras from the host to the device, and transferring
      image buffers from the host to the device and back. You will have to complete
      this function to support passing materials and lights to CUDA.
    * raycastFromCameraKernel() is a function that you need to implement. This
      function once correctly implemented should handle camera raycasting. 
    * raytraceRay() is the core raytracing CUDA kernel; all of your raytracing
      logic should be implemented in this CUDA kernel. raytraceRay() should
      take in a camera, image buffer, geometry, materials, and lights, and should
      trace a ray through the scene and write the resultant color to a pixel in the
      image buffer.

* intersections.h contains functions for geometry intersection testing and
  point generation. You will need to complete:
    * boxIntersectionTest(), which takes in a box and a ray and performs an
      intersection test. This function should work in the same way as
      sphereIntersectionTest().
    * getRandomPointOnSphere(), which takes in a sphere and returns a random
      point on the surface of the sphere with an even probability distribution.
      This function should work in the same way as getRandomPointOnCube(). You can
      (although do not necessarily have to) use this to generate points on a sphere
      to use a point lights, or can use this for area lighting.

* interactions.h contains functions for ray-object interactions that define how
  rays behave upon hitting materials and objects. You will need to complete:
    * getRandomDirectionInSphere(), which generates a random direction in a
      sphere with a uniform probability. This function works in a fashion
      similar to that of calculateRandomDirectionInHemisphere(), which generates a
      random cosine-weighted direction in a hemisphere.
    * calculateBSDF(), which takes in an incoming ray, normal, material, and
      other information, and returns an outgoing ray. You can either implement
      this function for ray-surface interactions, or you can replace it with your own
      function(s).
	  
* sceneStructs.h, which contains definitions for how geometry, materials,
  lights, cameras, and animation frames are stored in the renderer.
