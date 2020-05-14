# General Notes: Applications

[![CC BY-SA 4.0][cc-by-sa-shield]][cc-by-sa]

by Matthew A. Clapp

[cc-by-sa]: http://creativecommons.org/licenses/by-sa/4.0/
[cc-by-sa-shield]: https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg

## Helpful Links

https://github.com/Overboard/discoverhue

https://github.com/quentinsf/qhue

https://developers.meethue.com/develop/hue-api/

## Notes

### Colors

First we get an RGB value.  If this RGB value is from a computer, it probably
has a Gamma function applied to it, which means that every R,G,B value has had
a nonlinear exponential function applied to it.  Thus we need to undo this
function to get linearized R,G,B values before proceeding with future
computations.

From a sensor, presumably we have linear outputs for R,G,B and may omit the
linearization step.

The [Hue api documentation on
colors](https://developers.meethue.com/develop/application-design-guidance/color-conversion-formulas-rgb-to-xy-and-back/)
has a missing group of equations for the "Wide RGB D65 conversion formula".
Luckily the values can be found further down the page in the example code.

```
// Wide gamut conversion D65
float X = r * 0.664511f + g * 0.154324f + b * 0.162028f;
float Y = r * 0.283881f + g * 0.668433f + b * 0.047685f;
float Z = r * 0.000088f + g * 0.072310f + b * 0.986039f;
```

Then from the  the Hue xy values (little-x, little-y) are as follows.  Little-x
and little-y are X and Y normalized to sum(X,Y,Z).

* `float x = X / (X + Y + Z);`
* `float y = Y / (X + Y + Z);`

Hue also says that `brightness = Y` which presumably means that `bri = 254*Y`.
(??)

#### Correlated Color Temperature

There is a locus of points within the color space that corresponds to the
spectrums at various temperatures of a black body.

To find the Correlated Color Temperature (equivalent Color Temperature for
colors possibly not on the locus) we must find the perceptually closest color
on the locus to a given color.  To make perceptual distance equal to coordiante
distance, we need to measure distance on a **perceptually uniform** coordinate
space.  The color space invented for this purpose is the (u,v) color space.
Converting to the (u,v) coordinate space and finding the closest point on the
black body locus will be the most accurate CCT possible.

The typically used approximation to find CCT in the (x,y) color space is the
McCamy approximation.

CCT(x,y) = -449n^3 +3525n^2 -6823.3n + 5520.33

n = (x − x\_e)/(y - y\_e) where  (x\_e = 0.3320, y\_e = 0.1858)

There is a maximum distance in (u,v) space that CIE recommends as an acceptable
CCT (an acceptable perceptual match to the Color Temperature locus.)  It is
0.05 distance in the (u,v) space.

### Links
["Planckian locus (Wikipedia)"](https://en.wikipedia.org/wiki/Planckian_locus)

["Correlated Color Temperature (Wikipedia)"](https://en.wikipedia.org/wiki/Color_temperature#Correlated_color_temperature)

["How to convert between sRGB and
CIEXYZ"](https://www.image-engineering.de/library/technotes/958-how-to-convert-between-srgb-and-ciexyz)

["RGB/XYZ
Matrices"](http://www.brucelindbloom.com/index.html?Eqn_RGB_XYZ_Matrix.html)
* This seems excellent.  What we need to do is to solve for the matrix M that maps between sensor RGB and Hue XY by changing the Hue bulbs color and noting the RGB values.

[Wikipedia "CIE 1931 color
space"](https://en.wikipedia.org/wiki/CIE_1931_color_space)
