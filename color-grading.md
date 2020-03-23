@echo off
cls
rem ------------------------------------------------------------------------
rem Purpose: color correction, saturation, sharpening for underwater video. 
rem Function: color-grades video, inserts the original video in lower right
rem corner as a PIP (picture-in-picture) overlay, for comparison
rem Error handling: basic. 
rem                 Checks for availability of input video (%IN%)
rem                 Deletes possibly pre-existing output file (%OUTP%) 
rem Inputs: %IN%
rem Outputs: %OUTP% (with ungraded video as PIP) 
rem Parameters: To be tested with actual GoPro footage (manual 6500K setting) 
rem ------------------------------------------------------------------------

::declarations/defines::
set "IN=<path-to-movie>"
set "OUTP=<path-to-movie>"
set "FFMPG=<path-to-executable>"

::test availability of input file::
if not exist %IN% (
echo "Error: No input file (in.mp4) found. Aborting script."
goto eof
)

::remove potentially existing output file:: 
if exist %OUTP% DEL %OUTP%

::convolution (sharpening)::
set "CONV='0 -1 0 -1 5 -1 0 -1 0:0 -1 0 -1 5 -1 0 -1 0:0 -1 0 -1 5 -1 0 -1 0:0 -1 0 -1 5 -1 0 -1 0'"

::equalizer settings::
set "CONTRAST=1.0"        :: Contrast in range -1000 to 1000 (normal is 1.0)
set "BRIGHT=0.0"          :: Brightness in range -1.0 to 1.0 (normal is 0.0)
set "SATUR=1.05"          :: Saturation in range 0.0 to 3.0 (normal is 1.0; the higher, the more pure the colors will get)
set "GAMMA=1.15"          :: Gamma in range 0.1 to 10.0 (normal is 1.0; greater 1.0 is more contrast, darker shades, brighter highlights)

::colorlevels settings::
set "rmin=0.10"           :: colorlevel red minimum
set "gmin=0.05"           :: colorlevel green minimum
set "bmin=0.05"           :: colorlevel blue minimum
set "rmax=0.70"           :: colorlevel red maximum
set "gmax=1.00"           :: colorlevel green maximum
set "bmax=0.95"           :: colorlevel blue maximum

::colorbalance and hue::
set "HUE=-5"              :: Color correction (hue), negative shifts towards red and positive towards blue, normal is 0, typical is -30...+30 range
set "rs=+0.00"            ::   red - cyan    :: Adjust red, green and blue shadows (darkest pixels) 
set "gs=-0.10"            :: green - magenta ::
set "bs=-0.10"            ::  blue - yellow  ::

set "rm=+0.10"            ::   red - cyan    :: Adjust red, green and blue midtones (medium pixels)
set "gm=-0.25"            :: green - magenta ::
set "bm=-0.25"            ::  blue - yellow  ::

set "rh=+0.05"            ::   red - cyan    :: Adjust red, green and blue highlights (brightest pixels)
set "gh=-0.20"            :: green - magenta ::
set "bh=-0.20"            ::  blue - yellow  ::

::equalizer (contrast, brightness, saturation, gamma)::
set "EQUALIZER=contrast=%CONTRAST%:brightness=%BRIGHT%:saturation=%SATUR%:gamma=%GAMMA%"

::colorlevels::
set "COLORLEVELS=rimin=%rmin%:gimin=%gmin%:bimin=%bmin%:rimax=%rmax%:gimax=%gmax%:bimax=%bmax%"

::colorbalance::
set "COLORBALANCE=rs=%rs%:gs=%gs%:bs=%bs%:rm=%rm%:gm=%gm%:bm=%bm%:rh=%rh%:gh=%gh%:bh=%bh%"

::applying all of these filter settings (color-grading and PIP combined)::
::using libx264 encoding and "-preset ultrafast", "-crf 18" for high quality, as recommended (for quick preview purposes rather than smallest file sizes)::
%FFMPG% -i %IN% -filter_complex "[0]scale=iw/3:-1[pip];[0]colorbalance=%COLORBALANCE%[1],[1]hue=h=%HUE%[2],[2]colorlevels=%COLORLEVELS%[3],[3]eq=%EQUALIZER%[4],[4]convolution=%CONV%[bg];[bg][pip]overlay=main_w-overlay_w-10:main_h-overlay_h-10 [v]" -map "[v]" -map 0:a -c:a copy -c:v libx264 -preset ultrafast -crf 18 %OUTP%
goto eof

:: End
:eof

[/revised code]