-------------------------------------------------------------------------------
CIS565 Project 1: CUDA Raytracer
-------------------------------------------------------------------------------
Ricky Arietta Fall 2013
-------------------------------------------------------------------------------
-------------------------------------------------------------------------------

This project is a highly parallel version of the traditional ray tracing
algorithm on the implemented on the GPU, augmented from a template provided by
Karl Yi and Liam Boone. It works by casting out virtual rays through an image plane 
from a user-defined camera and assigning pixel values based on intersections 
with the defined geometry and lights. The final implementation, when built, 
is capable of rendering high-quality images composed of physically realistic 
area light(s) and geometry primitives (spheres, cubes). An example of a final
render can be seen immediately below this description. As you can see, the final
implementation accounts for Phong Illumination, soft shadows from area lights,
recursive reflections of light within the scene, and supersampled antialiasing. The
implementation of each feature is described briefly below with a rendered image
demonstrating the development of the code. Finally, there is some performance
analysis included regarding the size and number of blocks requested on the GPU
during runtime.

(A brief tour of the base code and a description of the scene file format
used during implementation are also included at the end of this file, adapted
from the project description provided by Patrick Cozzi and Liam Boone.)

![Final Rendered Image](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/006_final.bmp)

-------------------------------------------------------------------------------
Initial Ray Casting From Camera & Geometry Primitive Intersection
-------------------------------------------------------------------------------

In order to render images using ray tracing, we needed to define the set of rays
being sent out from the camera. The camera, defined by the user input file,
includes a resolution, field of view angle, viewing direction, up direction, etc.
Using this information, I was able to define an arbitrary image plane some distance
from the camera, orthogonal to the viewing direction vector. Then, using the 
up direction and a computed third orthogonal "right" vector, I was able to
define a grid on the image plane with the same resolution as the desired image.
Then, from the camera, and given an x-index and y-index for the pixel in question,
I could compute a single ray from the camera position to the center of the corresponding
pixel in the image plane (adapted for supersampled antialiasing; see below). These
rays served as the initial rays for tracing paths through the scene.

Additionally, when following these rays, I needed to be able to determine any
geometry intersections along the ray direction vector, since these intersections
define the illumination color value returned to the image pixel. Thus, in addition
to the provided sphereIntersectionTest(), I created a boxIntersectionTest() for
computing ray collisions with cube primitives. This tested for an intersection with
the plane defined by each face, tested it against the size of the cube to see if
the intersection was bounded by the edges, and returned the minimum-distance intersection
from this set.

A very early render is seen below, proving the successful implementation of ray-casting
and primitive intersection into a multi-colored Cornell box with 3 spheres of. This
render only displays raycasting and intersection, and thus no lighting model was implemented.
All the surfaces are rendered according to their flat diffuse RGB color value.  

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

To analyze the performance of the program on the GPU hardware, I decided to run
timing tests on the renders with varying block sizes/counts. To do this, I altered
the tile size within the rendered image, increasing or decreasing the block size
and count when invoking the kernel.

*Caveat*: Unfortunately, the hardware to which I had access provided certain limitations
to my testing. The following block sizes were the only ones that the hardware in
the Moore 100 Lab could handle. I intended to test the code with more tile sizes
in [1,20], but they all caused the hardware to crash. Furthermore, I will
add that my program seemed to run at a generally slower rate than the demonstrations
provided in class, and I believe this was an artifact of the hardware as well. Regardless,
the following data shows certain trends in hardware utilization, it just lacks the 
completeness I would have desired. 

![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/block_data_chart.bmp)

As you can see, a tile size of 8 when rendering these images seemed to be the most efficient.
Increasing the tile size to 10 slowed down the iteration renders dramatically, indicating
an inefficient utilization of GPU memory and processing power. Interestingly enough,
doubling the size of the tile, while increasing the runtime, did not nearly have as much
of a negative impact on the runtime as an increase in tile size from 8 to 10.

We can also see that the runtime per iteration increases with an increase in the 
traceDepth of the rays, or how many times the rays are recursively followed through
a scene when calculating reflective contributions. This is to be expected, since more
instructions and memory accesses are required for deeper paths. But we can see that
the increase is not linear, having the greatest jump from traceDepth=1 (no reflection)
to traceDepth=2.

The above data is visualized in a graph below, plotting the average render time per
iteration for each tileSize for each traceDepth:

![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/block_data_graph.bmp)

-------------------------------------------------------------------------------
BASE CODE TOUR:
-------------------------------------------------------------------------------
The main files of interest in this prooject, which handle the ray-tracing
algorithm and image generation, are the following:

* raytraceKernel.cu contains the core raytracing CUDA kernel. You will need to
  complete:
    * cudaRaytraceCore() handles kernel launches and memory management
    * raycastFromCameraKernel() handles camera raycasting. 
    * raytraceRay() is the core raytracing CUDA kernel; raytraceRay() should
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
      This function should work in the same way as getRandomPointOnCube().

* interactions.h contains functions for ray-object interactions that define how
  rays behave upon hitting materials and objects. You will need to complete:
    * getRandomDirectionInSphere(), which generates a random direction in a
      sphere with a uniform probability. This function works in a fashion
      similar to that of calculateRandomDirectionInHemisphere(), which generates a
      random cosine-weighted direction in a hemisphere.
	  
* sceneStructs.h, which contains definitions for how geometry, materials,
  lights, cameras, and animation frames are stored in the renderer.

-------------------------------------------------------------------------------
TAKUAscene FORMAT:
-------------------------------------------------------------------------------
This project uses a custom scene description format, called TAKUAscene.
TAKUAscene files are flat text files that describe all geometry, materials,
lights, cameras, render settings, and animation frames inside of the scene.
Items in the format are delimited by new lines, and comments can be added at
the end of each line preceded with a double-slash.

Materials are defined in the following fashion:

* MATERIAL (material ID)								//material header
* RGB (float r) (float g) (float b)					//diffuse color
* SPECX (float specx)									//specular exponent
* SPECRGB (float r) (float g) (float b)				//specular color
* REFL (bool refl)									//reflectivity flag, 0 for
  no, 1 for yes
* REFR (bool refr)									//refractivity flag, 0 for
  no, 1 for yes
* REFRIOR (float ior)									//index of refraction
  for Fresnel effects
* SCATTER (float scatter)								//scatter flag, 0 for
  no, 1 for yes
* ABSCOEFF (float r) (float b) (float g)				//absorption
  coefficient for scattering
* RSCTCOEFF (float rsctcoeff)							//reduced scattering
  coefficient
* EMITTANCE (float emittance)							//the emittance of the
  material. Anything >0 makes the material a light source.

Cameras are defined in the following fashion:

* CAMERA 												//camera header
* RES (float x) (float y)								//resolution
* FOVY (float fovy)										//vertical field of
  view half-angle. the horizonal angle is calculated from this and the
  reslution
* ITERATIONS (float interations)							//how many
  iterations to refine the image, only relevant for supersampled antialiasing,
  depth of field, area lights, and other distributed raytracing applications
* FILE (string filename)									//file to output
  render to upon completion
* frame (frame number)									//start of a frame
* EYE (float x) (float y) (float z)						//camera's position in
  worldspace
* VIEW (float x) (float y) (float z)						//camera's view
  direction
* UP (float x) (float y) (float z)						//camera's up vector

Objects are defined in the following fashion:
* OBJECT (object ID)										//object header
* (cube OR sphere OR mesh)								//type of object, can
  be either "cube", "sphere", or "mesh". Note that cubes and spheres are unit
  sized and centered at the origin.
* material (material ID)									//material to
  assign this object
* frame (frame number)									//start of a frame
* TRANS (float transx) (float transy) (float transz)		//translation
* ROTAT (float rotationx) (float rotationy) (float rotationz)		//rotation
* SCALE (float scalex) (float scaley) (float scalez)		//scale

An example TAKUAscene file setting up two frames inside of a Cornell Box can be
found in the scenes/ directory.

For meshes, note that the base code will only read in .obj files. For more 
information on the .obj specification see http://en.wikipedia.org/wiki/Wavefront_.obj_file.

An example of a mesh object is as follows:

OBJECT 0
mesh tetra.obj
material 0
frame 0
TRANS       0 5 -5
ROTAT       0 90 0
SCALE       .01 10 10 
