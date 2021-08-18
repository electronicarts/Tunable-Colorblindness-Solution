# Tunable Colorblindness Solution
A tunable colorblindness solution that improves color-shifting and luminosity to address common colorblind issues relating to color differentiation and visibility in colorblind accessibility settings. Authored by Karen Stevens @ Electronic Arts Inc. 

## Overview 
A post process implementation of the algorithm/code is recommended, as it:

  1.	Requires no asset changes.
  2.	Covers most colorblindness scenarios.
  3.	Reduces risk of in-world heads-up-display (HUD) elements blending into the game.
  4.	Supports partial colorblindness sliders.
  5.	Fits within post-processing passes, FXAA, or UI code.

In the case where a post process implementation means applying it twice, the change is slight enough that it isn’t that noticeable.

## About the Code

The code contains the following scope variables to be passed in by the game:


> colorBlindProtanopiaFactor : presentation = 0.0; // pass in 0 or 1 to turn on support
> 
> colorBlindDeuteranopiaFactor : presentation = 0.0; // pass in 0 or 1 to turn on support
> 
> colorBlindTritanopiaFactor : presentation = 0.0; // pass in 0 or 1 to turn on support
> 
> colorBlindDaltonizeFactor : presentation = 0.0; // pass in 0 or 0.9 for best results
> 
> accessibilityBrightnessFactor : presentation = 0.0; // zero is no effect
> 
> accessibilityContrastFactor : presentation = 0.0; // zero is no effect

If you'd like to support partial colorblindness, you can expose colorBlindDaltonizeFactor as a user-facing slider.
The code includes brightness and contrast support, which are optional to use: 

* Suggested brightness factors: -0.1, -0.05, 0, 0.05, 0.11. 
* Suggested contrast factors: -0.25, -0.12, 0, 0.2, 0.4.

## Tuning 
The algorithm and code should work well for most games as-is, but since color space is being reduced, there is a chance that the algorithm will cause color overlap. So if you do notice an issue, here's where in the algorithm and code that you can shift the colors.

>// CALL THIS METHOD TO PROCESS COLOR
>
>// applies brightness, contrast, and color blind settings to passed in color
>
>float4 AccessibilityPostProcessing(float4 color)
>{
>
>//apply contrast shift for daltonization
>color.rgb = ((color.rgb - 0.5) * (1.0+colorBlindDaltonizeFactor * **0.112**)) + 0.5;
>
>//apply brightness shift for daltonization
>color.rgb -= **0.075** * colorBlindDaltonizeFactor;
>
>// apply colorblind compensation algorithm
>color = (Daltonize(color)*colorBlindDaltonizeFactor + color*(1.0-colorBlindDaltonizeFactor));
>
>// expose contrast
>color.rgb = ((color.rgb - 0.5) * (1.0+accessibilityContrastFactor)) + 0.5;
>// expose brightness & shift colors back to lighter hues
>color.rgb += accessibilityBrightnessFactor + **0.08** * colorBlindDaltonizeFactor;
>
>return color;
>}

The **bold** numbers are tunable values:

* The 0.112 represents contrast. Higher contrasts fixes issues that are mid-range. Lower contrast helps extremes in color.
* The 0.075 is a brightness modifier, it's intended to shift the colors darker so brighter colors are less likely to clash (UI is usually bright colors)
* The 0.08 compensates for the previous two numbers. Halving contrast and adding the brightness modifier gives decent results.

The image below can be used as a reference: the top row is original color, second is protanopia filter, third is deuteranopia, and bottom row is tritanopia. If your colors are approximately shifted similar to these, you're likely fine, but results will vary dependent on game lighting.

![Reference Image](https://user-images.githubusercontent.com/26971700/129812299-a549791c-cb45-4c53-9d44-bea620f5b577.png)


## License 
This project is licensed under the Apache 2.0, see [LICENSE](https://github.com/electronicarts/Tunable-Colorblindness-Solution/blob/main/LICENSE) and [NOTICE](https://github.com/electronicarts/Tunable-Colorblindness-Solution/blob/main/NOTICE.txt) for details. 
