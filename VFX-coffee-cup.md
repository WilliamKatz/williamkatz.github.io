---
layout: page
title: ""
permalink: /coffee-cup
---


# Coffee Cup Outline Shader

For the game Coffee Shop Clues [available on itch.io](https://sip-up-games.itch.io/coffee-shop-clues), I created a shader 
 to create simple outline effect.  Inside the cup, I added three tessellated planes and wrote a shader
to displace the vertices.

<video width="320" height="320" controls>
  <source src="./Movies/Coffee-shop-clues-mug.mov" type="video/mp4">
</video>

### Stroke Shader
The shader takes dot product between the normal of the surface and viewing direction.
Based on the dot product it returns one of two colors to crete the outline effect.


    Shader "Unlit/SH_Outline"
    {
        Properties
        {
            _MainTex ("Texture", 2D) = "white" {}
            _Threshold ("Threshold", Range(0.01, 1.0)) = 0.1 
            _Alpha ("Alpha", Range(0.1, 1.0)) = 0.1 
            _MugColor("Mug Color", Color) = (1, 1, 1, 1)
            _BorderColor("Border Color", Color) = (1, 1, 1, 1)
            _StrokeThreshold("Stroke Threshold", Range(0.01, 1)) = 0.2
        }
        SubShader
        {
            Tags { "RenderType"="Opaque" "Queue"="Transparent"}
            Blend One One
    
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
                    float3 normal: NORMAL;
                };
    
                struct v2f
                {
                    float2 uv : TEXCOORD0;
                    float4 vertex : SV_POSITION;
                    float3 normal: TEXCOORD1;
                    float3 worldPos: TEXCOORD2;
                    float3 viewDir: TEXCOORD3;
                };
    
                sampler2D _MainTex;
                float4 _MainTex_ST;
    
                float _Threshold;
                float _StrokeThreshold;
                float _Alpha;
                float4 _MugColor;
                float4 _BorderColor;
    
                v2f vert (appdata v)
                {
                    v2f o;
                    o.vertex = UnityObjectToClipPos(v.vertex);
                    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
                    o.normal = UnityObjectToWorldNormal(v.normal);
                    o.worldPos = mul(unity_ObjectToWorld, v.vertex);
                    o.viewDir = normalize(_WorldSpaceCameraPos - o.worldPos);
                    return o;
                }
    
                float4 frag (v2f i) : SV_Target
                {
                    float dotProduct = dot(i.normal, i.viewDir);
                    float step1 = step(_Threshold, dotProduct);
    
                    if (abs(dot(i.normal, i.viewDir)) < _StrokeThreshold)
                    {
                        return (_BorderColor);
                    }
                        return float4(_MugColor.xyz, _Alpha);
                    }
                }
                ENDCG
            }
        }
    }



### Liquid Shader
For the liquid, I created tessellated planes in Blender and brought in FBX files
into Unity.  I wrote a single shader and tweak values to get waves that looked different.

The code calculates a value on a wave to move an individual vertex. Then based on UV it uses
a main color or border color to create a pleasing effect.

    Shader "Unlit/SH_Drink_V2"
    {
        Properties
        {
            _BackgroundColor ("Background Color", Color) = (1, 1, 1, 1)
            _BorderColor ("Border Color", Color) = (1, 1, 1, 1)
            
            _WaveAmp ("Wave Amp", Range(0.5, 3)) = 1
            _WaveFreq("Wave Frequency", Range(0.01, 20)) = 1
            
            _ThresholdX ("Border Threshold X", Range(0.01, 0.1)) = 0.1
            _ThresholdY ("Border Threshold Y", Range(0.01, 0.1)) = 0.1
            
        }
        SubShader
        {
            Tags { "RenderType"="Opaque" }
            Blend SrcAlpha OneMinusSrcAlpha
    
            Pass
            {
                
                NAME "WAVE"
    
                CGPROGRAM
                #pragma vertex vert
                #pragma fragment frag
                
                #define TAU 6.28318530717958647692
    
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
                
    
                float4 _BackgroundColor;
                float4 _BorderColor;
                float _WaveAmp;
                float _WaveSpeed;
                float _ThresholdX;
                float _ThresholdY;
                float _WaveFreq;
    
                float GetWave(float2 uv)
                {
                    float distance = 0.5 + length(uv) * _WaveFreq;
                    float cosWave = cos(distance + 3 * TAU * (_Time.y * 0.2)) * 0.5 + 0.5;
                    float sinWave = sin(distance + 3 * TAU * (_Time.y * 0.1)) * 0.5 + 0.5;
                    return  (cosWave + sinWave) / 1000;
                }
    
                v2f vert (appdata v)
                {
                    v2f o;
    
                    float ignoreBottom = step(v.uv.x, 0.5);
                    v.vertex.x = v.vertex.x + (GetWave(v.uv) *_WaveAmp * ignoreBottom);
    
                    // convert vertex to clip space
                    o.vertex = UnityObjectToClipPos(v.vertex);
                    
                    o.uv = v.uv;
                    return o;
                }
    
                float4 frag (v2f i) : SV_Target
                {
                    // colors the pixel the border color if it is close enough to border based on the threshold
                    float isBorder = step(1-_ThresholdX, i.uv.x);
                    isBorder += step(i.uv.x, _ThresholdX);
                    isBorder += step(i.uv.y, _ThresholdY);
                    isBorder += step(1-_ThresholdY, i.uv.y);
                    
                    isBorder = clamp(isBorder, 0, 1);
                    float4 color = (_BackgroundColor * (1-isBorder)) + (_BorderColor*isBorder);
                    return + color;
                }
                ENDCG
            }
        }
    }
