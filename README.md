-------------------------------------------------------------------------------
CIS565: Project 1: CUDA Raytracer
-------------------------------------------------------------------------------
Fall 2013
-------------------------------------------------------------------------------
Due Thursday, 09/19/2013
-------------------------------------------------------------------------------

![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/006_final.bmp)
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/001_flat_shading.bmp)
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/002_diffuse_illumination.bmp)
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/003_diffuse_illumination_with_hard_shadows.bmp)
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/004_diffuse_illumination_with_soft_shadows.bmp)
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/005_phong_illumination_with_soft_shadows_and_reflections.bmp)
![Flat Shading](https://raw.github.com/rarietta/Project1-RayTracer/master/PROJ1_WIN/565Raytracer/README_images/005_phong_illumination_with_soft_shadows_and_reflections_and_supersampled_antialiasing.bmp)


-------------------------------------------------------------------------------
REQUIREMENTS:
-------------------------------------------------------------------------------
In this project, you are given code for:

* Loading, reading, and storing the TAKUAscene scene description format
* Example functions that can run on both the CPU and GPU for generating random
  numbers, spherical intersection testing, and surface point sampling on cubes
* A class for handling image operations and saving images
* Working code for CUDA-GL interop

You will need to implement the following features:

* Raycasting from a camera into a scene through a pixel grid
* Phong lighting for one point light source
* Diffuse lambertian surfaces
* Raytraced shadows
* Cube intersection testing
* Sphere surface point sampling

You are also required to implement at least 2 of the following features:

* Specular reflection 
* Soft shadows and area lights 
* Texture mapping 
* Bump mapping
* Depth of field
* Supersampled antialiasing
* Refraction, i.e. glass
* OBJ Mesh loading and renderin
* Interactive camera

-------------------------------------------------------------------------------
BASE CODE TOUR:
-------------------------------------------------------------------------------
You will be working in three files: raytraceKernel.cu, intersections.h, and
interactions.h. Within these files, areas that you need to complete are marked
with a TODO comment. Areas that are useful to and serve as hints for optional
features are marked with TODO (Optional). Functions that are useful for
reference are marked with the comment LOOK.

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

You will also want to familiarize yourself with:

* sceneStructs.h, which contains definitions for how geometry, materials,
  lights, cameras, and animation frames are stored in the renderer. 
* utilities.h, which serves as a kitchen-sink of useful functions

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

Check the Google group for some sample .obj files of varying complexity.

-------------------------------------------------------------------------------
README
-------------------------------------------------------------------------------
All students must replace or augment the contents of this Readme.md in a clear 
manner with the following:

* A brief description of the project and the specific features you implemented.
* At least one screenshot of your project running.
* A 30 second or longer video of your project running.  To create the video you
  can use http://www.microsoft.com/expression/products/Encoder4_Overview.aspx 
* A performance evaluation (described in detail below).

-------------------------------------------------------------------------------
PERFORMANCE EVALUATION
-------------------------------------------------------------------------------
The performance evaluation is where you will investigate how to make your CUDA
programs more efficient using the skills you've learned in class. You must have
perform at least one experiment on your code to investigate the positive or
negative effects on performance. 

One such experiment would be to investigate the performance increase involved 
with adding a spatial data-structure to your scene data.

Another idea could be looking at the change in timing between various block
sizes.

A good metric to track would be number of rays per second, or frames per 
second, or number of objects displayable at 60fps.

We encourage you to get creative with your tweaks. Consider places in your code
that could be considered bottlenecks and try to improve them. 

Each student should provide no more than a one page summary of their
optimizations along with tables and or graphs to visually explain and
performance differences.

-------------------------------------------------------------------------------
SELF-GRADING
-------------------------------------------------------------------------------
* On the submission date, email your grade, on a scale of 0 to 100, to Liam,
  liamboone+cis565@gmail.com, with a one paragraph explanation.  Be concise and
  realistic.  Recall that we reserve 30 points as a sanity check to adjust your
  grade.  Your actual grade will be (0.7 * your grade) + (0.3 * our grade).  We
  hope to only use this in extreme cases when your grade does not realistically
  reflect your work - it is either too high or too low.  In most cases, we plan
  to give you the exact grade you suggest.
* Projects are not weighted evenly, e.g., Project 0 doesn't count as much as
  the path tracer.  We will determine the weighting at the end of the semester
  based on the size of each project.

-------------------------------------------------------------------------------
SUBMISSION
-------------------------------------------------------------------------------
As with the previous project, you should fork this project and work inside of
your fork. Upon completion, commit your finished project back to your fork, and
make a pull request to the master repository.  You should include a README.md
file in the root directory detailing the following

* A brief description of the project and specific features you implemented
* At least one screenshot of your project running, and at least one screenshot
  of the final rendered output of your raytracer
* A link to a video of your raytracer running.
* Instructions for building and running your project if they differ from the
  base code
* A performance writeup as detailed above.
* A list of all third-party code used.
* This Readme file, augmented or replaced as described above in the README section.
