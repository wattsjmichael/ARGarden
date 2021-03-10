# ARFoundationOcclusion
An example of how to change AR Foundation Occlusion Settings in Runtime

Example of how this looks can be found in Twitter at: [https://twitter.com/Dilmerv](https://twitter.com/Dilmerv/status/1277966408605786112?s=20)

**Note**: There is a URP version of this project so make sure to look at the [feature/URPOcclusion](https://github.com/dilmerv/ARFoundationOcclusion/tree/feature/URPOcclusion) branch if you like to use that option.


Credits to **bgolus** who posted the URP AR Shadow Receiver in the post below:
[Unity Forums](https://forum.unity.com/threads/water-shader-graph-transparency-and-shadows-universal-render-pipeline-order.748142/)

Shader Code:

```
Shader "URP AR Shadow Receiver"
{
    Properties
    {
        _ShadowColor ("Shadow Color", Color) = (0.35,0.4,0.45,1.0)
    }
 
    SubShader
    {
        Tags
        {
            "RenderPipeline"="UniversalPipeline"
            "RenderType"="Transparent"
            "Queue"="Transparent-1"
        }
 
        Pass
        {
            Name "ForwardLit"
            Tags { "LightMode" = "UniversalForward" }
 
            Blend DstColor Zero, Zero One
            Cull Back
            ZTest LEqual
            ZWrite Off
   
            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag
 
            #pragma prefer_hlslcc gles
            #pragma exclude_renderers d3d11_9x
            #pragma target 2.0
 
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS
            #pragma multi_compile _ _MAIN_LIGHT_SHADOWS_CASCADE
            #pragma multi_compile _ _SHADOWS_SOFT
            #pragma multi_compile_fog
 
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
 
            CBUFFER_START(UnityPerMaterial)
            float4 _ShadowColor;
            CBUFFER_END
 
            struct Attributes
            {
                float4 positionOS : POSITION;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };
 
            struct Varyings
            {
                float4 positionCS               : SV_POSITION;
                float3 positionWS               : TEXCOORD0;
                float fogCoord                  : TEXCOORD1;
                UNITY_VERTEX_INPUT_INSTANCE_ID
                UNITY_VERTEX_OUTPUT_STEREO
            };
 
            Varyings vert (Attributes input)
            {
                Varyings output = (Varyings)0;
 
                UNITY_SETUP_INSTANCE_ID(input);
                UNITY_TRANSFER_INSTANCE_ID(input, output);
                UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);
 
                VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
                output.positionCS = vertexInput.positionCS;
                output.positionWS = vertexInput.positionWS;
                output.fogCoord = ComputeFogFactor(vertexInput.positionCS.z);
 
                return output;
            }
 
            half4 frag (Varyings input) : SV_Target
            {
                UNITY_SETUP_INSTANCE_ID(input);
                UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);
 
                half4 color = half4(1,1,1,1);
 
            #ifdef _MAIN_LIGHT_SHADOWS
                VertexPositionInputs vertexInput = (VertexPositionInputs)0;
                vertexInput.positionWS = input.positionWS;
 
                float4 shadowCoord = GetShadowCoord(vertexInput);
                half shadowAttenutation = MainLightRealtimeShadow(shadowCoord);
                color = lerp(half4(1,1,1,1), _ShadowColor, (1.0 - shadowAttenutation) * _ShadowColor.a);
                color.rgb = MixFogColor(color.rgb, half3(1,1,1), input.fogCoord);
            #endif
                return color;
            }
 
            ENDHLSL
        }
    }
}
```