---
layout: page
title: ""
permalink: /swirl-effect
---


# Swirl Effect

A swirling effect used on items to give them appeal.

<video width="320" height="320" controls>
  <source src="vfx-swirl.mov" type="video/mp4">
</video>

### Process
I created a mesh in Blender and edited the UVs to stretch the texture across the entire mesh. Then I exported the FBX and imported that into Unity.

In Unity, I brought the mesh into the scene and created a material with a custom shader written in HLSL.  The shader code is available here.

I repeated this process to create 4 unique curves and created multiple materials so I could change their color and scrolling speed individually.
```
Shader "Unlit/SH_MovingTexture_Additive"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        _ScrollSpeed ("Scroll Speed", Range(0.1, 2.0)) = 0.5
        _Color ("Color", Color) = (1, 1, 1, 1)
        _Alpha ("Alpha", Range(0.0, 1.0)) = 0.1
        _Delay ("Time Delay", float) = 0.1
    }

    SubShader
    {
        Tags { "RenderType"="Opaque"  }
        Blend SrcAlpha OneMinusSrcAlpha

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
            float4 _MainTex_ST;
            float _ScrollSpeed;
            float _Alpha;
            float4 _Color;
            float _Delay;

            v2f vert (appdata v)
            {
                v2f o;
                o.vertex = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                return o;
            }

            float4 frag (v2f i) : SV_Target
            {
                // determine the X offset
                float x = frac(_Delay + i.uv.x + _Time.y * _ScrollSpeed);

				// sample the texture at this offeset
                float4 tex = tex2D(_MainTex, float2(x, i.uv.y));

                // update the alpha based on the texture
                _Color.w = tex.w;

                return _Color;
            }
            ENDCG
        }
    }
}
```

