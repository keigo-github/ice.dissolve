# ice.dissolve

// Upgrade NOTE: upgraded instancing buffer 'Props' to new syntax.

Shader "Custom/disolve" {
	Properties{
		_Color("Color", Color) = (1,1,1,1)
		_MainTex("Albedo (RGB)", 2D) = "white" {}
		_DisolveTex("DisolveTex (RGB)", 2D) = "white" {}
		_Glossiness("Smoothness", Range(0,1)) = 0.5
		_Metallic("Metallic", Range(0,1)) = 0.0
		_Threshold("Threshold", Range(0,1)) = 0.0
	}
		SubShader{
			Tags {"RenderType" = "Transparent"  "Queue" = "Transparent" }
			LOD 200
			Cull off

			CGPROGRAM
			#pragma surface surf Standard alpha:fade
			#pragma target 3.0

			sampler2D _MainTex;
			sampler2D _DisolveTex;

			struct Input {
				float3 worldNormal;
				float3 viewDir;
				float2 uv_MainTex;
			};

			half _Glossiness;
			half _Metallic;
			half _Threshold;
			fixed4 _Color;

			UNITY_INSTANCING_BUFFER_START(Props)
			UNITY_INSTANCING_BUFFER_END(Props)

			void surf(Input IN, inout SurfaceOutputStandard o) {
				// Albedo comes from a texture tinted by color
				fixed4 m = tex2D(_DisolveTex, IN.uv_MainTex);
				_Threshold += 3 * _Time;

				half g = m.r * 0.2 + m.g * 0.7 + m.b * 0.1;
				if (g < _Threshold) {
					discard;
				}

				fixed4 c = tex2D(_MainTex, IN.uv_MainTex) * _Color;
				o.Albedo = c.rgb;
				// Metallic and smoothness come from slider variables
				o.Metallic = _Metallic;
				o.Smoothness = _Glossiness;
				float alpha = 1 - (abs(dot(IN.viewDir, IN.worldNormal)));
				o.Alpha = alpha * 1.5f;
			}
			ENDCG
		}
			FallBack "Diffuse"
}
