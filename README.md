Documentation Author: Alexander Amling

# Unity Indirect Instanced Rendering

[part 1](https://github.com/IGME-RIT/unity-instanced-rendering)

* Unity Compute Shaders Part 2*

As previously mentioned, one of the main issues with instanced rendering in Unity is that there is a limit of 1023 instances.


Using buffers gives us another added benefit. Data no longer has to be passed to the graphics card ever render call. It can read from the buffer directly.

This also means that we can modify the buffer directly from compute shaders, and it never has to leave the graphics card. See part 3 for more

[part 3]()
