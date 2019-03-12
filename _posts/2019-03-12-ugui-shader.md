---
layout: post
title: Unity 使用Shader 对 UGUI Image 进行图像处理
---

{{ page.title }}
================

# 先放效果图

![原图](/images/1.webp)

![设置色调](/images/2.webp)

![灰度化](/images/3.webp)

![对比度](/images/4.webp)

![模糊](/images/5.webp)


# Shader
我是直接在官方自带Shader UI-Default 基础上改的，怕你们找不到，我放在网盘了
链接：http://pan.baidu.com/s/1pLfWK4b 密码：jpje

## 以下是色调、灰度化、对比度设置的Shader的代码，我都写在一个Shader里面了  

``` c#
Shader "Custom/UI/Base"
{
	Properties
	{
		[PerRendererData] _MainTex ("Sprite Texture", 2D) = "white" {}
		_Color ("Tint", Color) = (1,1,1,1)

		_ToneColor("ToneColor",Color) = (1,1,1,1)
		_ToneIntensity("ToneIntensity", Range(-1,1)) = 0
		_Saturation("Saturation",Range(0,1)) = 1
		_Contrast("Contrast",Float) = 1


		_StencilComp ("Stencil Comparison", Float) = 8
		_Stencil ("Stencil ID", Float) = 0
		_StencilOp ("Stencil Operation", Float) = 0
		_StencilWriteMask ("Stencil Write Mask", Float) = 255
		_StencilReadMask ("Stencil Read Mask", Float) = 255

		_ColorMask ("Color Mask", Float) = 15

		[Toggle(UNITY_UI_ALPHACLIP)] _UseUIAlphaClip ("Use Alpha Clip", Float) = 0
	}

	SubShader
	{
		Tags
		{ 
			"Queue"="Transparent" 
			"IgnoreProjector"="True" 
			"RenderType"="Transparent" 
			"PreviewType"="Plane"
			"CanUseSpriteAtlas"="True"
		}
		
		Stencil
		{
			Ref [_Stencil]
			Comp [_StencilComp]
			Pass [_StencilOp] 
			ReadMask [_StencilReadMask]
			WriteMask [_StencilWriteMask]
		}

		Cull Off
		Lighting Off
		ZWrite Off
		ZTest [unity_GUIZTestMode]
		Blend SrcAlpha OneMinusSrcAlpha
		ColorMask [_ColorMask]

		Pass
		{
			Name "Default"
		CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag
			#pragma target 2.0

			#include "UnityCG.cginc"
			#include "UnityUI.cginc"

			#pragma multi_compile __ UNITY_UI_ALPHACLIP
			
			struct appdata_t
			{
				float4 vertex   : POSITION;
				float4 color    : COLOR;
				float2 texcoord : TEXCOORD0;
			};

			struct v2f
			{
				float4 vertex   : SV_POSITION;
				fixed4 color    : COLOR;
				half2 texcoord  : TEXCOORD0;
				float4 worldPosition : TEXCOORD1;
			};
			
			fixed4 _Color;
			fixed4 _TextureSampleAdd;
			float4 _ClipRect;

			v2f vert(appdata_t IN)
			{
				v2f OUT;
				OUT.worldPosition = IN.vertex;
				OUT.vertex = UnityObjectToClipPos(OUT.worldPosition);

				OUT.texcoord = IN.texcoord;
				
				#ifdef UNITY_HALF_TEXEL_OFFSET
				OUT.vertex.xy += (_ScreenParams.zw-1.0) * float2(-1,1) * OUT.vertex.w;
				#endif
				
				OUT.color = IN.color * _Color;
				return OUT;
			}

			sampler2D _MainTex;
			half _ToneIntensity;
			half _Saturation;
			half _Contrast;
			fixed3 _ToneColor;

			fixed4 frag(v2f IN) : SV_Target
			{
				fixed4 renderTex = tex2D(_MainTex, IN.texcoord);
				
				half4 color = (renderTex + _TextureSampleAdd) * IN.color;
				fixed3 finalColor;
				if(_ToneIntensity<0)
				{
					finalColor = color + color*_ToneIntensity;
				}
				else
				{
					finalColor = color + (_ToneColor-color)*_ToneIntensity;
				}
				fixed gray = dot(fixed3(0.2154,0.7154,0.0721),finalColor);
				color.a *= UnityGet2DClipping(IN.worldPosition.xy, _ClipRect);
				
				finalColor = lerp(gray,finalColor,_Saturation);
				finalColor = lerp(0.5,finalColor,_Contrast);
				#ifdef UNITY_UI_ALPHACLIP
				clip (color.a - 0.001);
				#endif

				return fixed4(finalColor, color.a);  
			}
		ENDCG
		}
	}
}

```

``` c#
//片段着色器
fixed4 frag(v2f IN) : SV_Target
{
	fixed4 renderTex = tex2D(_MainTex, IN.texcoord);
	half4 color = (renderTex + _TextureSampleAdd) * IN.color;
	fixed3 finalColor;
    //当_ToneIntensity越靠近-1，则颜色越接近黑色，当_ToneIntensity越靠近1，则颜色越接近_ToneColor，当_ToneIntensity为0，则为原颜色
	if(_ToneIntensity<0)
	{
		finalColor = color + color*_ToneIntensity;
	}
	else
	{
		finalColor = color + (_ToneColor-color)*_ToneIntensity;
	}
    //灰度化，经验公式
	fixed gray = dot(fixed3(0.2154,0.7154,0.0721),finalColor);
	color.a *= UnityGet2DClipping(IN.worldPosition.xy, _ClipRect);
    
	finalColor = lerp(gray,finalColor,_Saturation);
    //lerp函数用于线性插值，可以利用这个函数做出很多效果（变亮，调高对比度）
	finalColor = lerp(0.5,finalColor,_Contrast);
	#ifdef UNITY_UI_ALPHACLIP
	clip (color.a - 0.001);
	#endif
    return fixed4(finalColor, color.a);  
}
```

## 模糊效果，高斯模糊
``` c#
fixed4 frag(v2f IN) : SV_Target
{

	float distance = _Distance;
	fixed4 computedColor = tex2D(_MainTex, IN.texcoord) * IN.color*0.147761;
	computedColor += tex2D(_MainTex, half2(IN.texcoord.x + distance , IN.texcoord.y + distance )) * IN.color*0.09474;
	computedColor += tex2D(_MainTex, half2(IN.texcoord.x + distance , IN.texcoord.y)) * IN.color*0.11831;
	computedColor += tex2D(_MainTex, half2(IN.texcoord.x , IN.texcoord.y + distance )) * IN.color*0.11831;
	computedColor += tex2D(_MainTex, half2(IN.texcoord.x - distance , IN.texcoord.y - distance )) * IN.color*0.09474;
	computedColor += tex2D(_MainTex, half2(IN.texcoord.x + distance , IN.texcoord.y - distance )) * IN.color*0.09474;
	computedColor += tex2D(_MainTex, half2(IN.texcoord.x - distance , IN.texcoord.y + distance )) * IN.color*0.09474;
	computedColor += tex2D(_MainTex, half2(IN.texcoord.x - distance , IN.texcoord.y)) * IN.color*0.11831;
	computedColor += tex2D(_MainTex, half2(IN.texcoord.x , IN.texcoord.y - distance )) * IN.color*0.11831;
	return computedColor;
}
```

原理很简单：该点像素值等于周边像素值的加权平均。  
详细解析：http://www.ruanyifeng.com/blog/2012/11/gaussian_blur.html

![距离为1像素的权重矩阵](/images/6.webp)


# 在C#代码中的应用
新创建的Image，Material为null，其实并不是真正的null，而是一个公用的material，如果直接对这个material进行操作，那么所有的ui都会跟着改变。

```c#
public static class UIExtend
{
    //根据shader创建新的material
    static public void cloneSelfMaterial(this Image img)
    {
        if( img.material.name.Equals("cloneMat") )
        {

        }
        else
        {
            Material mat = new Material(Shader.Find("Custom/UI/Base"));
            mat.name = "cloneMat";
            img.material = mat;
        }
    }
    static public void setToneIntensity(this Image image, float _toneIntensity)
    {
        _toneIntensity = Mathf.Clamp(_toneIntensity, -1f, 1f);
        image.material.SetFloat("_ToneIntensity", _toneIntensity);
    }
    static public void setSaturation(this Image image, float saturation)
    {
        saturation = Mathf.Clamp(saturation, 0f, 1f);
        image.material.SetFloat("_Saturation", saturation);
    }
    static public void setContrast(this Image image, float Contrast)
    {
        image.material.SetFloat("_Contrast", Contrast);
    }
    static public void setToneColor(this Image image, Color toneColor)
    {
        image.material.SetColor("_ToneColor", toneColor);
    }
    static public void resetDefaultImageEffect(this Image image)
    {
        image.setToneIntensity(0);
        image.setContrast(1);
        image.setSaturation(1);
        image.setToneColor(new Color(1, 1, 1, 1));
    }
}
```
对Image实现灰度化的缓动  
![缓动效果图](/images/7.webp)


```c#
using UnityEngine;
using System.Collections;
using UnityEngine.UI;

public class MainImageProcess : MonoBehaviour
{
    [SerializeField]
    private Image testImage;
    void Start()
    {
        testImage.cloneSelfMaterial();
    }
    private bool isStart = false;
    void Update()
    {
        if(Input.GetKeyDown(KeyCode.A))
        {
            isStart = !isStart;
            if(isStart)
            {
                startGray();
            }
        }
        if(isStart)
        {
            t += dir * Time.deltaTime;
            if(t>=1)
            {
                dir = -1;
            }
            else if(t<=0)
            {
                dir = 1;
            }
            currentSatuarion = Mathf.Lerp(startSaturation, endSaturation, t);
            testImage.setSaturation(currentSatuarion);
        }
    }
    private float dir = 1;
    private float t = 0f;
    private float startSaturation = 1f;
    private float currentSatuarion = 1f;
    private float endSaturation = 0f;
    void startGray()
    {
        testImage.resetDefaultImageEffect();
        testImage.setSaturation(1);
        startSaturation = 1;
        currentSatuarion = 1;
        endSaturation = 0f;
        t = 0f;
        dir = 1;
    }
}
```
