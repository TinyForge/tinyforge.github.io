---
title: A Surprising Chat with ChatGPT - Learning Shader Language Made Easy
author: doodums
date: 2023-03-28 10:46:00 +0800
categories: [Tutorials, AI]
tags: [Shaders, URP]
render_with_liquid: false
image:
  path: /tiny_forge_post_landing_2.png
---

I wanted to share an interesting experience I had with ChatGPT 3.5. I wanted to learn shader language for Unity's Universal Render Pipeline (URP), and instead of sifting through numerous tutorials, I tried something different. I had a casual conversation with ChatGPT, and to my surprise, it taught me exactly what I needed to know, in the manner I asked it too. Instead of attempting to explain our interaction, I decided to share the raw chat between us, as I believe it will be more beneficial for you to see how the learning unfolded. Read on to discover how this engaging chat made learning shader code in Unity a much more enjoyable process, and experience the conversation for yourself. This is the future of learning.

---

**Me:**

I'd like to learn to write my own shaders for URP in Unity. I learn best by looking at functional code. Could you show me how I could write a shader that uses an Emission map to fake subsurface scattering for URP?

---

**ChatGPT:**

Sure! Here's an example of a shader that uses an emission map to simulate subsurface scattering in Unity's Universal Render Pipeline (URP).
```
Shader "Custom/Surface Scattering"
{
    Properties
    {
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _EmissionTex ("Emission (RGB)", 2D) = "black" {}
        _SubsurfaceStrength ("Subsurface Strength", Range(0, 1)) = 0.5
        _SubsurfaceRadius ("Subsurface Radius", Range(0, 1)) = 0.5
        _SubsurfaceColor ("Subsurface Color", Color) = (1,1,1,1)
    }

    SubShader
    {
        Tags { "RenderType"="Opaque" }

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float4 vertex : SV_POSITION;
            };

            sampler2D _MainTex;
            sampler2D _EmissionTex;
            float _SubsurfaceStrength;
            float _SubsurfaceRadius;
            float4 _SubsurfaceColor;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = v.uv;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                // Sample the albedo texture
                fixed4 col = tex2D(_MainTex, i.uv);

                // Sample the emission texture
                fixed4 emission = tex2D(_EmissionTex, i.uv);

                // Calculate the subsurface scattering color
                fixed3 subsurface = (1.0 - col.rgb) * _SubsurfaceColor.rgb * _SubsurfaceStrength;

                // Calculate the subsurface scattering distance
                float distance = length(i.uv - 0.5);

                // Calculate the subsurface scattering radius
                float radius = pow(distance, _SubsurfaceRadius) * 2.0;

                // Add the subsurface scattering to the color
                col.rgb += subsurface * emission.rgb * radius;

                return col;
            }
            ENDCG
        }
    }
}
```
This shader defines two textures: **'_MainTex'** for the albedo color and **'_EmissionTex'** for the emission map. It also defines three properties that control the strength, radius, and color of the subsurface scattering effect.

In the vertex shader, we simply pass through the vertex position and UV coordinates to the fragment shader. In the fragment shader, we sample the **'_MainTex'** and **'_EmissionTex'** textures and calculate the subsurface scattering color using the difference between the albedo color and white, multiplied by the subsurface color and strength. We then calculate the subsurface scattering distance from the center of the texture using the UV coordinates and apply a power function to control the radius. Finally, we add the subsurface scattering to the color based on the emission map and return the final color.

You can use this shader in your URP project by creating a new material and assigning this shader to it. Then, you can set the **'_MainTex'** and **'_EmissionTex'** textures and adjust the subsurface properties to achieve the desired effect.

---

**Me:**

Could you explain to me in more detail what this method does in the above example?
```
v2f vert (appdata v)
{
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex);
    o.uv = v.uv;
    return o;
}
```

---

**ChatGPT:**

Sure! The **'vert'** function is the vertex shader in the above example. It takes an **'appdata'** input structure that contains the vertex position and texture coordinates of the input mesh, and outputs a **'v2f'** structure that contains the transformed vertex position and texture coordinates.

The **'v2f'** structure is an output structure used to pass data from the vertex shader to the fragment shader. In this example, it contains only the texture coordinates **'uv'** and the transformed vertex position **'vertex'**.

The **'UnityObjectToClipPos'** function is a built-in Unity function that transforms the vertex position from object space to clip space. Clip space is the coordinate space where the final output image is generated, and it is defined as a 3D space where the camera frustum is mapped to a cube with coordinates ranging from -1 to 1 in all dimensions.

The output structure **'o'** is then populated with the transformed vertex position and texture coordinates, and returned to the calling function.

In summary, the **'vert'** function transforms the input mesh vertex position from object space to clip space and passes the transformed position and texture coordinates to the fragment shader for further processing.

---

**Me:**

Could you explain to me in more detail what this function is doing?
```
fixed4 frag (v2f i) : SV_Target
{
    // Sample the albedo texture
    fixed4 col = tex2D(_MainTex, i.uv);

    // Sample the emission texture
    fixed4 emission = tex2D(_EmissionTex, i.uv);

    // Calculate the subsurface scattering color
    fixed3 subsurface = (1.0 - col.rgb) * _SubsurfaceColor.rgb * _SubsurfaceStrength;

    // Calculate the subsurface scattering distance
    float distance = length(i.uv - 0.5);

    // Calculate the subsurface scattering radius
    float radius = pow(distance, _SubsurfaceRadius) * 2.0;

    // Add the subsurface scattering to the color
    col.rgb += subsurface * emission.rgb * radius;

    return col;
}
```

---

**ChatGPT:**

Certainly! The **'frag'** function is the fragment shader in the above example. It takes a **'v2f'** input structure that contains the transformed vertex position and texture coordinates, and outputs a **'fixed4'** color value that represents the final pixel color.

The first two lines of the **'frag'** function sample the **'_MainTex'** and **'_EmissionTex'** textures using the input texture coordinates **'i.uv'**, and store the resulting colors in **'col'** and **'emission'**, respectively.

The next three lines calculate the subsurface scattering color, distance, and radius based on the input parameters **'_SubsurfaceColor'**, **'_SubsurfaceStrength'**, and **'_SubsurfaceRadius'**. Specifically, **'subsurface'** is calculated as the difference between white and the albedo color, multiplied by the subsurface color and strength. **'distance'** is calculated as the distance between the current pixel's texture coordinate and the center of the texture (0.5, 0.5). Finally, **'radius'** is calculated as the subsurface radius exponentiated to the power of the distance, multiplied by 2.0.

The last line adds the subsurface scattering to the color by multiplying the emission color with the subsurface color, radius, and strength, and adding the result to the original color. The **'rgb'** property of the **'fixed4'** color value is used to access the red, green, and blue components of the color. Finally, the modified **'col'** value is returned as the output of the function.

In summary, the **'frag'** function samples the albedo and emission textures, calculates the subsurface scattering color, distance, and radius, and applies the subsurface scattering effect to the pixel color based on the emission map and the subsurface properties.

---

**Me:**

So vert manages the vertices' position and frag controls the colour that gets rendered?

---

**ChatGPT:**

That's right! In a typical rendering pipeline, the vertex shader (in this case, the **'vert'** function) is responsible for transforming the vertices of a mesh from their local space to a common coordinate system (such as world or camera space), and passing on any per-vertex data (such as normals, tangents, or texture coordinates) to the fragment shader.

The fragment shader (in this case, the **'frag'** function) is responsible for determining the final color of each individual pixel that makes up the rendered object. It does this by interpolating the per-vertex data passed on by the vertex shader, and using it to sample textures, perform lighting calculations, or any other operations needed to generate the desired output color.

So while the vertex shader mainly deals with geometry and position, the fragment shader is primarily responsible for determining the appearance of the rendered object, including its color, texture, and lighting.

---

**Me:**

So what if I changed the above method into the following method. What would be the expected outcome compared to the one you shared?

```
v2f vert (appdata v)
{
    v2f o;
    o.vertex = v.vertex;
    o.uv = v.uv;
    return o;
}
```

---

**ChatGPT:**

If you change the **'vert'** function in the above example to simply pass through the input vertex position and texture coordinates without transforming them, as shown below:

```
v2f vert (appdata v)
{
    v2f o;
    o.vertex = v.vertex;
    o.uv = v.uv;
    return o;
}
```

then the resulting shader would render the mesh with its vertices in their original object space positions. This means that the mesh would not be affected by any transformations applied to it in the scene (such as rotation or scaling), and would appear as if it is at its default position and orientation.

This could be useful in certain situations, such as when you want to create a shader effect that specifically targets the mesh's original vertex positions, or when you want to manually apply any transformations to the mesh in the shader code itself.

However, in most cases, you would want to transform the vertices to a common coordinate system (such as camera space or world space) in the vertex shader, as this allows you to apply lighting and other effects correctly based on the position and orientation of the mesh in the scene.

---

**Me:**

Does the appdata struct always have to have those variables, or is it customizable? How do those values get populated?

---

**ChatGPT:**

The **'appdata'** struct in a Unity shader is customizable to some extent. It defines the input data that is passed from the CPU to the GPU for each vertex in the mesh being rendered. However, the specific fields that it contains are determined by the type of mesh being rendered, and are generated automatically by Unity based on the mesh's vertex format.

In the case of the **'appdata'** struct in the example shader, it contains two fields: **'vertex'**, which represents the position of the vertex in object space, and **'uv'**, which represents the texture coordinate of the vertex. These fields are commonly used in many Unity shaders, but other fields such as **'normal'**, **'tangent'**, **'color'**, or custom vertex attributes can also be included in the **'appdata'** struct depending on the requirements of the shader.

The values in the **'appdata'** struct are populated automatically by Unity at runtime, based on the vertex data contained in the mesh being rendered. When Unity renders a mesh, it sends the vertex data to the GPU as a buffer, which the shader can then access through the **'appdata'** struct.

The **'appdata'** struct is a built-in Unity struct that is used in both built-in and custom shaders. By default, Unity provides a basic definition for **'appdata'** that includes only the **'vertex'** and **'uv'** fields, but this can be extended or modified as needed for different shader effects.

---

**Me:**

And the struct v2f is custom to our implementation and we could make that anything we want?

---

**ChatGPT:**

Yes, that's correct! The **'v2f'** struct in a Unity shader is a custom output struct that defines the data passed from the vertex shader to the fragment shader. You can define the structure of the **'v2f'** struct as needed, based on the data that needs to be passed between the vertex and fragment shaders.

In the example shader, the **'v2f'** struct contains only two fields: **'vertex'**, which represents the transformed position of the vertex, and **'uv'**, which represents the texture coordinate of the vertex. However, you can include additional fields in the **'v2f'** struct as needed, such as surface normals, tangents, or any other per-vertex data that needs to be interpolated across the surface of the mesh.

When you define the **'v2f'** struct in the vertex shader, the same structure must be defined in the fragment shader as well, so that the interpolated data can be correctly passed on to the fragment shader. The **'v2f'** struct in the fragment shader should have the same fields and data types as the **'v2f'** struct in the vertex shader, so that the compiler can properly match up the data during the rendering process.

So while the specific fields and data types in the **'v2f'** struct are customizable, it's important to ensure that both the vertex and fragment shaders use the same structure for the **'v2f'** data to be correctly passed between them.

---

**Me:**

Could you explain to me the purpose of the following lines?
```
CGPROGRAM
#pragma vertex vert
#pragma fragment frag
#include "UnityCG.cginc"
```

---

**ChatGPT:**

Certainly!

The **'CGPROGRAM'** directive is used in Unity to indicate the start of a shader program written in the Cg/HLSL language. Everything between **'CGPROGRAM'** and **'ENDCG'** is treated as shader code by Unity's shader compiler.

The **'#pragma vertex'** and **'#pragma fragment'** directives specify the vertex and fragment shader functions, respectively, that Unity should use to render the object. In the example shader, **'#pragma vertex vert'** indicates that the vertex shader function is **'vert'**, and **'#pragma fragment frag'** indicates that the fragment shader function is **'frag'**.

The **'#include'** directive is used to include external code files into the shader program. In the example shader, **'"UnityCG.cginc"'** is a built-in Unity code file that contains common utility functions and macros used in Unity shaders. This file includes useful functions such as **'UnityObjectToClipPos'**, which is used in the **'vert'** function to transform the vertex position from object space to clip space.

By including **'"UnityCG.cginc"'**, we gain access to these utility functions without having to write them ourselves, which can save a lot of time and effort when writing custom shaders.

So in summary, these lines set up the shader program, specify the vertex and fragment shader functions, and include a utility code file to help with shader development.

---

**Me:**

Thank you, you helped me understand shaders really quickly.

---

**ChatGPT:**

You're welcome! I'm glad I could help you understand shaders better. If you have any more questions, feel free to ask!

---

**Me:**

Could you help me update the above shader you wrote to take into account the light direction of the main light?

---

**ChatGPT:**

Certainly! Here's an updated version of the shader that takes into account the direction of the main light:
```
Shader "Custom/Surface Scattering"
{
    Properties
    {
        _MainTex ("Albedo (RGB)", 2D) = "white" {}
        _EmissionTex ("Emission (RGB)", 2D) = "black" {}
        _SubsurfaceStrength ("Subsurface Strength", Range(0, 1)) = 0.5
        _SubsurfaceRadius ("Subsurface Radius", Range(0, 1)) = 0.5
        _SubsurfaceColor ("Subsurface Color", Color) = (1,1,1,1)
    }

    SubShader
    {
        Tags { "RenderType"="Opaque" }

        Pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #include "UnityCG.cginc"

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;
                float3 worldNormal : TEXCOORD1;
                float3 worldPos : TEXCOORD2;
                float4 vertex : SV_POSITION;
            };

            sampler2D _MainTex;
            sampler2D _EmissionTex;
            float _SubsurfaceStrength;
            float _SubsurfaceRadius;
            float4 _SubsurfaceColor;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = v.uv;
                o.worldNormal = mul((float3x3)unity_WorldToObject, v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                return o;
            }

            fixed4 frag (v2f i) : SV_Target
            {
                // Sample the albedo texture
                fixed4 col = tex2D(_MainTex, i.uv);

                // Sample the emission texture
                fixed4 emission = tex2D(_EmissionTex, i.uv);

                // Calculate the subsurface scattering color
                fixed3 subsurface = (1.0 - col.rgb) * _SubsurfaceColor.rgb * _SubsurfaceStrength;

                // Calculate the subsurface scattering distance
                float distance = length(i.uv - 0.5);

                // Calculate the subsurface scattering radius
                float radius = pow(distance, _SubsurfaceRadius) * 2.0;

                // Calculate the lighting direction
                float3 lightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));

                // Calculate the diffuse lighting term
                float diffuse = max(0.0, dot(i.worldNormal, lightDir));

                // Add the subsurface scattering to the color based on the emission map and the diffuse lighting
                col.rgb += subsurface * emission.rgb * radius * diffuse;

                return col;
            }
            ENDCG
        }
    }
}
```
The main change in this version is the addition of a lighting calculation in the **'frag'** function that takes into account the direction of the main light. This is done by first calculating the world space lighting direction using the **'UnityWorldSpaceLightDir'** function provided by Unity. We then use this lighting direction to calculate the diffuse lighting term using the dot product of the surface normal and the lighting direction. This diffuse term is multiplied with the subsurface scattering effect to produce the final color.

To support the lighting calculation, we also modify the **'appdata'** and **'v2f'** structs to include the surface normal and

---

**Me:**

What are some common methods that the #include "UnityCG.cginc" line gives me access too?

---

**ChatGPT:**

The **'#include "UnityCG.cginc"'** directive provides access to a variety of common utility functions, macros, and constants used in Unity shaders. Some of the most commonly used features include:

- Transform functions: These functions are used to convert between different coordinate spaces, such as object space, world space, and camera space. Examples include **'UnityObjectToWorldDir'** and **'UnityWorldToObjectPos'**.

- Lighting functions: These functions are used to calculate lighting in shaders, such as the direction and color of the main directional light. Examples include **'UnityWorldSpaceLightDir'** and **'UnityGI'**.

- Texture sampling functions: These functions are used to sample textures in shaders. Examples include **'tex2D'** and **'tex2Dproj'**.

- Mathematical functions: These functions are used to perform various mathematical operations in shaders, such as clamping, normalization, and interpolation. Examples include **'lerp'**, **'saturate'**, and **'max'**.

- Shader constants: These constants are predefined values that can be used in shaders, such as the screen resolution or the PI constant. Examples include **'_ScreenParams'** and **'UNITY_PI'**.

These are just a few examples of the many functions and constants provided by **'UnityCG.cginc'**. You can find a complete list of all the available features in the Unity documentation. By including this file, you can save a lot of time and effort when writing custom shaders, as you don't have to reinvent the wheel and can leverage Unity's built-in functionality.

---

**Me:**

What are common native functions I have access too?

---

**ChatGPT:**

In addition to the functions and features provided by **'UnityCG.cginc'**, Unity shaders also have access to a range of native functions and data types that are part of the underlying shading language being used (such as Cg or HLSL). Some of the most commonly used native functions and data types in Unity shaders include:

- Vector math functions: These functions perform various mathematical operations on vectors, such as addition, subtraction, multiplication, and division. Examples include **'normalize'**, **'dot'**, **'cross'**, and **'lerp'**.

- Matrix math functions: These functions perform various mathematical operations on matrices, such as multiplication, inversion, and transposition. Examples include **'mul'**, **'transpose'**, and **'inverse'**.

- Trigonometric functions: These functions perform various trigonometric operations, such as sine, cosine, and tangent. Examples include **'sin'**, **'cos'**, and **'tan'**.

- Color manipulation functions: These functions perform various operations on colors, such as blending, saturating, and converting between color spaces. Examples include **'lerp'**, **'saturate'**, and **'sRGBtoLinear'**.

- Texture sampling functions: These functions are used to sample textures in shaders. Examples include **'tex2D'**, **'tex3D'**, and **'texCUBE'**.

- Flow control statements: These statements are used to control the flow of execution in shaders, such as conditional statements (if/else), loops (for/while), and function calls.

These are just a few examples of the many native functions and data types available in Unity shaders. The specific set of functions and data types available depends on the shading language being used (Cg, HLSL, etc.), and you can find more information on the specific syntax and functionality of each language in the corresponding documentation.

---

**Me:**

It feels quite overwhelming to try think of how to use all those parameters, while I don't know most of them.

---

**ChatGPT:**

It's understandable to feel overwhelmed by the many functions and parameters available in Unity shaders, especially when you're first starting out. However, it's important to remember that you don't need to use every single feature in order to create effective shaders.

Instead, it's often more useful to focus on the specific features that are most relevant to your project or shader effect, and learn those thoroughly. As you gain more experience and familiarity with Unity shaders, you can gradually expand your knowledge and try out more advanced features.

One way to approach learning Unity shaders is to start with simple examples and gradually build up to more complex effects. You can find many tutorials and resources online that cover the basics of shader development, and provide step-by-step instructions for creating different effects.

Another helpful approach is to experiment with the different parameters and functions in the shader code, and observe the effects they have on the rendered output. Unity's real-time rendering allows you to quickly see the results of any changes you make, which can be a powerful tool for learning and experimentation.

Finally, don't be afraid to ask for help or seek out resources when you're stuck. There are many active communities and forums online where you can ask questions and get feedback on your shader code, and these can be valuable resources for improving your skills and knowledge.

---

