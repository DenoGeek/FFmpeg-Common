## FFMPEG video processing commands

1. Video compression
- Just passing the video through ffmpeg performs several optimizations and a sometiomes could lead to a size reduction

```
ffmpeg -i small.mp4 output/compress-basic.mp4
```
- Reduce the video frame size
```
ffmpeg -i small.mp4 -vf "scale=iw/2:ih/2" output/half_the_frame_size.mp4

ffmpeg -i small.mp4 -vf "scale=iw/3:ih/3" output/a_third_the_frame_size.mp4

ffmpeg -i small.mp4 -vf "scale=iw/4:ih/4" output/a_fourth_the_frame_size.mp4
```
- Vary the Constant Rate Factor:
For x264, sane values are between 18 and 28. The default is 23, so you can use this as a starting point.Refer to a more
[Detailed CRF guide](https://slhck.info/video/2017/02/24/crf-guide.html)

```
ffmpeg -i small.mp4 -c:v libx264 -crf 18 output/crf.mp4

vary the -crf factor and compare output
```
Other options include modifying the bitrate and varying the profile to "base line"

2. Adding a Watermark on the video

Ffmpeg allows you to use complex filters on your videos such as the overlay

```
ffmpeg -y -i small.mp4 -i fblogo.png \
    -filter_complex "overlay=x=main_w-overlay_w-(main_w*0.01):y=main_h-overlay_h-(main_h*0.01)" \
        logo_bottom_right.mp4
```
Sample animating the overlay filter from left to right

```
ffmpeg -i small.mp4 -i fblogo.png \
-filter_complex "overlay='if(gte(t,1), -w+(t-1)*200, NAN)':(main_h-overlay_h)/2" output/logo_animate.mp4

```
Sample bouncing the logo at the bottom right
```

```

3. Applying filters
There are different techniques for applying filters

    - Grayscale black and white
        ```
        ffmpeg -i small.mp4 -vf format=gray out.mp4
        ```
    - Adjust brightness
        ```
        ffmpeg -i small.mp4 -vf eq="brightness=0.13" out.mp4

        ```
        The equalizer can be used for different tweaks in your video
        
        The equalizer settings are
        
        |Setting | Range of values|Default|
        |--|--|--|
        |contrast|-1000 - 1000|1.0|
        |brightness|-1.0 - 1.0|0|
        |saturation|0.0 - 3.0|1.0|
        |Gamma|0.1-10|1.0|


        ```
        ffmpeg -i small.mp4 -filter_complex eq=contrast=1.0:brightness=0:saturation=3:gamma=1.0 output/equalizer.mp4

        ```
    - Adjusting the colors using color levels

        |Setting | Description|
        |--|--|
        |rimin=0.10| colorlevel red minimum|
        |gimin=0.05| colorlevel green minimum|
        |bimin=0.05| colorlevel blue minimum|
        |rimax=0.70| colorlevel red maximum|
        |gimax=1.00| colorlevel green maximum|
        |bimax=0.95| colorlevel blue maximum|

        ```
        ffmpeg -i small.mp4 -filter_complex  colorlevels=rimin=0.10:bimin=0.05 output/colorlevels.mp4

        ```

    - Ajusting the hue. Hue values between -30 to +30
        ```
        ffmpeg -i small.mp4 -filter_complex hue=20 output/hue.mp4
        ```

    - Adjusting the color balance
    
        |Property|Vales
        |--|--|
        ||
        |rs=+0.00|red - cyan|
        |gs=-0.10|green - magenta|
        |bs=-0.10|blue - yellow |
        |Adjust red, green and blue midtones (medium pixels)|
        |rm=+0.10|red - cyan|
        |gm=-0.25|green - magenta|
        |bm=-0.25|blue - yellow|
        |Adjust red, green and blue highlights (brightest pixels)|
        |rh=+0.05|red - cyan|
        |gh=-0.20|green - magenta|
        |bh=-0.20| blue - yellow |

    - Sharpen the frames using convolution
        ```
        ffmpeg -i small.mp4 -filter_complex convolution='0 -1 0 -1 5 -1 0 -1 0:0 -1 0 -1 5 -1 0 -1 0:0 -1 0 -1 5 -1 0 -1 0:0 -1 0 -1 5 -1 0 -1 0' output/convolution.mp4
        ```




4. Stacking videos

This puts videos next to each other
- hstack
    example
    ```
    ffmpeg -i small.mp4 -i small.mp4 -filter_complex hstack output/hstack.mp4

    ```
- vstack
    example

    ```
    ffmpeg -i small.mp4 -i small.mp4 -filter_complex vstack output/vstack.mp4
    ```

- Experimental combination
    - Tripple stacking the videos
        ```
        ffmpeg -i small.mp4 -i small.mp4 -filter_complex "[0][1]vstack[stacked];[stacked][0]vstack[tripple]" -map "[tripple]" output/vstacktripple.mp4
        ```
    - Add gray scale
        ```
        ffmpeg -i small.mp4 -i small.mp4 -filter_complex "[0][1]vstack[stacked];[stacked]format=gray[grayed]" -map "[grayed]" output/vstackgray.mp4

        ```
    - Set a logo on the video
        ```
        ffmpeg -i small.mp4 -i small.mp4 -i fblogo.png -filter_complex "[0][1]vstack[stacked];[stacked][2]overlay=x=main_w-overlay_w-(main_w*0.01):y=main_h-overlay_h-(main_h*0.01)[branded]" -map "[branded]" output/vstacklogo.mp4

        ```


