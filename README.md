Documentation Author: Alexander Amling

# Unity Indirect Instanced Rendering

[part 1](https://github.com/IGME-RIT/unity-instanced-rendering)

*Unity Compute Shaders Part 2*

*Developed with modified code from [Unity Documentation](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html)*

As previously mentioned, one of the main issues with instanced rendering in Unity is that there is a limit of 1023 instances.

To overcome this, we can work with a different draw call: Graphics.DrawMeshInstancedIndirect(), but this complicates the process of passing in transforms a little.

Take a look at the parameters for DrawMeshInstancedIndirect [in the documentation](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstancedIndirect.html). Notice anything missing? There is no longer an paramter for the array of transform matricies. This is actually a good thing. The process of passing an array to the graphics card is a (relativley) slow process. Instead we are going to use GPU data buffers, which allows to leave the data in the graphics card between render calls.

To do this, we'll need to create a ComputeBuffer and a array of matricies
```
Matrix4x4[] instanceTransforms;
ComputeBuffer transformBuffer;
...
transformBuffer = new ComputeBuffer(instanceCount, sizeof(float) * 16);
instanceTransforms = new Matrix4x4[instanceCount];
```
Populate the array with data like we did previously, then set the data to the buffer and then set the buffer within the shader
```
for (int i = 0; i < instanceCount; i++)
{
    pos = Random.insideUnitSphere * 50;
    rot = Quaternion.Euler(Random.insideUnitSphere * 180);
    scale = Random.insideUnitSphere;
    instanceTransforms[i] = Matrix4x4.TRS(pos, rot, scale);
}
transformBuffer.SetData(instanceTransforms);
instanceMaterial.SetBuffer("transformBuffer", transformBuffer);
```
If you haven't worked with shaders before, that last line may seem a little strange, but it is simply setting the value of "transformBuffer" inside of the shader that instanceMaterial is using.

Within this shader, there a few lines to pay attention to:
```
#pragma instancing_options procedural:setup
...
StructuredBuffer<float4x4> transformBuffer;
...
void setup()
{
  unity_ObjectToWorld = transformBuffer[unity_InstanceID];
}
```
[Unity Documentation](https://docs.unity3d.com/Manual/GPUInstancing.html) explains the first line under the section "#pragma instancing_options":

> procedural:FunctionName	
>> Use this to instruct Unity to generate an additional variant for use with Graphics.DrawMeshInstancedIndirect. At the beginning of the vertex Shader stage, Unity calls the function specified after the colon. To set up the instance data manually, add per-instance data to this function in the same way you would normally add per-instance data to a Shader. Unity also calls this  function at the beginning of a fragment Shader if any of the fetched instance properties are included in the fragment Shader.

So what these lines of code achieve is setting the object to world matrix of each of the instances as they are drawn effectively achieving what we did in part 1, but entirely within the graphics card.

Using buffers gives us another added benefit. Since the data we are working with is held in the graphics card, it's just as easy to modify it from within the graphics card. See part 3 for more.

[part 3](https://github.com/IGME-RIT/unity-compute-shaders)
