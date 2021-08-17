Description: Accessibility support library for shaders, covering brightness, contrast, and colorblindness issues.

Copyright (c) 2015-2021 Electronic Arts Inc. 

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.


/*** Variables ************************************************************************************/

float colorBlindProtanopiaFactor     : presentation = 0.0; // pass in 0 or 1 to turn on support
float colorBlindDeuteranopiaFactor   : presentation = 0.0; // pass in 0 or 1 to turn on support
float colorBlindTritanopiaFactor     : presentation = 0.0; // pass in 0 or 1 to turn on support

float colorBlindDaltonizeFactor      : presentation = 0.0; // pass in 0 or 0.9 for best results
float accessibilityBrightnessFactor  : presentation = 0.0; // zero is no effect
float accessibilityContrastFactor    : presentation = 0.0; // zero is no effect

// suggested brightness factors: -0.1, -0.05, 0, 0.05, 0.11
// suggested contrast factors: -0.25, -0.12, 0.2, 0.4


/*** Methods ************************************************************************************/

// Shifts from rgb to luminosity color representation. The magic numbers
// are standard conversion values used to do this.
// see https://en.wikipedia.org/wiki/CIE_1931_color_space for details
float3 RgbToLms(float3 color)
{
    float l = (17.8824 * color.r) + (43.5161 * color.g) + (4.11935 * color.b);
    float m = (3.45565 * color.r) + (27.1554 * color.g) + (3.86714 * color.b);
    float s = (0.0299566 * color.r) + (0.184309 * color.g) + (1.46709 * color.b);
    return float3(l,m,s);   
}


// Shifts from luminosity to rgb color representation. The magic numbers
// are standard conversion values used to do this.
// see https://en.wikipedia.org/wiki/LMS_color_space for details
float3 LmsToRgb(float3 color)
{
    float r = (0.0809444479 * color.r) + (-0.130504409 * color.g) + (0.116721066 * color.b);
    float g = (-0.0102485335 * color.r) + (0.0540193266 * color.g) + (-0.113614708 * color.b);
    float b = (-0.000365296938 * color.r) + (-0.00412161469 * color.g) + (0.693511405 * color.b);
    return float3(r,g,b);
}


// Shifts colors based on color blind color weaknesses to areas where user can better see.
// The magic numbers model the way the human eye works when affected by different color
// deficiencies. They will never change.
// see http://www.daltonize.org/search/label/Color%20Blindness for details
float4 Daltonize(float4 color)
{
    float3 colorLMS = color.rgb;
    colorLMS = RgbToLms(colorLMS);
    
    float3 colorWeak;
    
    colorWeak.r = (2.02344*colorLMS.g - 2.5281*colorLMS.b)*colorBlindProtanopiaFactor + colorLMS.r*(1.0-colorBlindProtanopiaFactor);
    colorWeak.g = (0.494207*colorLMS.r + 1.24827*colorLMS.b)*colorBlindDeuteranopiaFactor + colorLMS.g*(1.0-colorBlindDeuteranopiaFactor);
    colorWeak.b = (-0.395913*colorLMS.r + 0.801109*colorLMS.g)*colorBlindTritanopiaFactor + colorLMS.b*(1.0-colorBlindTritanopiaFactor);
    
    colorWeak = LmsToRgb(colorWeak);
    
    colorWeak = color.rgb - colorWeak;
    
    float3  colorShift;
    colorShift.r = 0;
    colorShift.g = colorWeak.g + 0.7*colorWeak.r;
    colorShift.b = colorWeak.b + 0.7*colorWeak.r;
    
    color.rgb += colorShift.rgb;
    color = clamp(color,0.0,1.0);
    
    return color;
}


// CALL THIS METHOD TO PROCESS COLOR
// applies brightness, contrast, and color blind settings to passed in color
float4 AccessibilityPostProcessing(float4 color)
{    
    //apply contrast shift for daltonization
    color.rgb = ((color.rgb - 0.5) * (1.0+colorBlindDaltonizeFactor*0.112)) + 0.5;
    
    //apply brightness shift for daltonization
    color.rgb -= 0.075*colorBlindDaltonizeFactor;
    
    // apply colorblind compensation algorithm
    color = (Daltonize(color)*colorBlindDaltonizeFactor + color*(1.0-colorBlindDaltonizeFactor));
    
    // expose contrast
    color.rgb = ((color.rgb - 0.5) * (1.0+accessibilityContrastFactor)) + 0.5;

    // expose brightness & shift colors back to lighter hues
    color.rgb += accessibilityBrightnessFactor+0.08*colorBlindDaltonizeFactor;
    
    return color;
}
