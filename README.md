

Procedural Terrain and Multithreading -- Lily Brenner

Efficient Terrain Rendering and Chunking -- Raymond Feng

Game Engine Tick Function and Player Physics -- Jillian Rayca

Texture and Texture Animation -- Jillian Rayca

Procedural Sky with Day and Night cycle, Sound, Far Clip Plane, Debugging -- Jillian Rayca


Link to video (Milestone 1): https://drive.google.com/file/d/14h5yqFYx37QcZJlt942V2-XDh0ZI5c-f/view?usp=sharing

# Jillian Rayca
Milestone 1: Game Engine Tick Function and Player Physics 
Milestone 2: Texturing and Texture Animation
For the player physics, we implemented those by calculating the collision and reducing speed repeatedly like in the lecture slides. It was difficult because it didn't work cross-platform, so it ultimately wasn't usable on mac. 

For the rendering, we rendered face-by-face, only choosing faces that were visible. We had a big issue where the VBOs were continuously being regenerated, causing a huge memory leak, but resolved that by using a boolean to ensure that it was only generated once. 

For the procedural terrain, we used domain-warped FBM to generate the terrain, interpolating between them using perlin noise. It had major issues with scaling, but by fiddling around with constants we were able to make it work.

For the texture animation, we used fragment shaders and a separate texture class to import and implement the textures onto the multithreaded generated terrain. We ran into some issues with compatibility on Mac vs Windows, as well as just understanding the functions of other group members. The biggest issue was chunking in relation to the VBOs and the time function in mygl, which caused the Windows implementation to crash immediately. This, however, was fixed with a 1 character change (thanks Adam!!!! :) )

For the Procedural Sky, we used the demo code, but altered some of the worley noise functions to add a northern lights effect that spans across the whole screen, as well as adding another sun to make a Tatooine-like sky. The biggest issue was more or less just messing around with the functions and color palettes of the dawn and dusk vectors to get the exact design we wanted.

For the Sound, we used the OpenGL Qt Sound Effects to implement a walking noise when the player moves on the ground. This was rather straight forward, the only issue was with the actual name of the Sound being changed in the OpenGL software.

We also debugged some of the removeblock, addblock, and collisions to be compatible with our new more complex multithreaded implementation

# Raymond Feng
Milestone 1: Efficient Terrain Rendering and Chunking
- I found that the part I had the most trouble with was storing interleaved data in one VBO. I started out by trying to store all the data in a single vector rather than 3 separate vec4s.
- As a group, we had problems with deciding when a chunk's VBO data should be generated, and we solved this using a boolean called VBOSet.

Milestone 2: Cave Systems
- Cave systems were implemented using 3d perlin noise, which sampled the 8 corners of a block rather than the 4 corners of 2d perlin noise. Then I ran a loop to check if the y value was below 25. If so, a lava block was placed. Otherwise, I set the block to EMPTY in order to carve out caves. This was done after the biome block stacks were generated.
- The collision with water was implemented by calling a function in computePhysics called isPlayerInWater that would check a box around the player, setting the inWater member variable of our InputBundle class if the player was in water or lava. If inWater was true, pressing up would increase the upward velocity by a constant amount. 
- The post-process overlays were created by using a quad class that would span the entire screen, then rendering a new frame buffer's texture as that quad. I also wrote several new shader programs (postprocess.vert.glsl, nothing.frag.glsl, lava.frag.glsl, water.frag.glsl) that would vary the post-process shader based on the player's camera position. If the camera position was not in water or lava, the post process shader would be nothing.frag.glsl, which didn't change the base color whatsoever. Otherwise, based on the block at the camera position, the post process shader would either tint the screen red or blue.

Problems I ran into:
- I found that when writing the post-process overlays, I had to do a lot of fiddling around with values to get what I wanted.
- When doing collisions, I also had to take time to understand the code already written for collisions.

Milestone 3: Extra Credit
- Additional Biomes: I created a "snow", "swamp", and "desert" biome, as well as two noise functions that decided the temperature and erosion of blocks. I used smoothstep to clamp the noise functions to the values 0.0 - 1.0, then interpolated between the biome heights using the noise function values as the amounts. To decide which biome was placed in a specific spot, I wrote an if statement with a result that varied depending on the temperature and erosion values.
- Procedurally placed assets: I used a 2d noise function to generate patches of land. Iterating over the x and z values of the terrain, if the terrain's noise value was below a certain threshold, I would create an object based on the terrain (i.e. cacti for sand, icicles for snow) and offset the x and z value to create some distance.
- Water waves: I created a separate vertex shader and fragment shader for any liquid blocks, which I would use instead of lambert shading. In my vertex shader, I offseted the y value of each vertex by an amount dictated by 4 different sin functions, which would be my waves. I also added blinn-phong shading and altered the normals of each vertex based on the sin wave using the tangent vectors of each sin wave.
- Post-process camera overlay: I added some fractal-based worley noise to the water and lava post process shaders. 
- Ambient occlusion: Using the 4th value of the vec4 representing color, I stored a value between 0 to 3 to represent the amount of shading a block got, with 3 being the most and 0 being the least. I performed this calculation when VBO data was being set, using a function based on the vertex (represented as an integer in a for loop) and the block face's direction. This function would check the surrounding 3 blocks to determine the ambient occlusion value of a particular point. Based on the ambient occlusion value, the resulting color would be mixed with a differing amount of dark overlay. Unfortunately I couldn't get the "crossing over chunks" bit to work...
- Distance fog: I copied the sky shader code over to my lambert fragment shader and liquid fragment shader, then adjusted a few values to add it to the base color in an aesthetically pleasing way.
- "Spaghetti & cheese" caves: I created an additional 3D noise function to get a different "seed". I then layered two differently seeded 3D noise functions, then found their values close to 0 in order to get a nice spaghetti-like cave structure.


# Lily Brenner
Milestone 1: Terrain Generation
- I had a lot of trouble getting the perlin noise to work. For some reason, it wouldn't give anything other than 0. In the end, we were able to resolve it by dividing the coordinates so they were fractional.
- I separated the world into terrain generation zones and that into chunks. Each chunk was created column by column, and it interpolated smoothly between biomes.

Milestone 2: Multithreading
- I created BlockTypeWorkers and VBOWorkers to generate terrain and VBOs respectively. I added a bunch of mutexes inside the Chunk class so as to lock them for individual calls but not the entirety of terrain generation. 
- I had a lot of trouble getting the threads to generate but it turned out to be an issue with using the "new" keyword to initialize a runnable. It worked really well, though it's slowed down noticeably since more terrain generation features were added. 

Milestone 3: Extra Credit
- Shadow Mapping: I created a shadow map using a second depth-based framebuffer. This was logistically challenging, as I had a lot of trouble with tutorials that used outdated approaches that were really frustrating to debug. It worked eventually, but it was frustrating. Using two framebuffers and activating and deactivating them has been extremely challenging. I also added additional logic to add an adaptive bias that removed shadow acne from the scene. 
