A bunch of guides for Smoothie-RS (Windows only)

> [!IMPORTANT]  
> Please read [Troubleshooting](#troubleshooting) if you encounter any issues. If you are unable to address your issue on your own, please create a support post in [CTT's support forum](https://discord.com/channels/774315187183288411/1020030329790148678).

## Table of contents

- [Installation](#installation)  
- [RIFE/pre-interp](#rifepre-interp)   
- [Explaining the config (recipe)](#explaining-the-config-recipe)
- [Making a mask](#making-a-mask)
- [Encoding arguments](#encoding-arguments)
- [Coldchrome LUT download](#coldchrome-lut-download)
- [Troubleshooting](#troubleshooting)

## Installation

#### You can follow [this tutorial I made](https://youtu.be/RfPDgoMuSWg).

If you don't want to watch the tutorial, follow these steps:

- Go to https://github.com/couleur-tweak-tips/smoothie-rs/releases/latest and download `smoothie-rs-nightly.zip`.
- Unzip the file.
- Running `launch.cmd` will start the program, the config file is called `recipe.ini`.
- Adding to Send to:
  - Go to `...\smoothie-rs-nightly\smoothie-rs\bin` and make a shortcut to `smoothie-rs.exe`.
  - Right click the shortcut and type `-v -i` at the end of `Target`. Should be like this:
   `...\smoothie-rs\bin\smoothie-rs.exe -v -i`.
  - Do Win + R, type `Shell:SendTo` in the run dialog and hit enter.
  - Move the smoothie-rs.exe shortcut you made earlier into the folder that opened.
  - Now you can send videos to Smoothie by Right click > Send to > `smoothie-rs.exe - shortcut` (you can rename the shortcut if you would like).

## RIFE/pre-interp

#### What is RIFE?

RIFE is a more accurate interpolation model, it is very slow compared to SVPFlow (the interpolation algorithm that Blur/Smoothie's interpolation uses), but its way more accurate. Its generally only worth using if your input video is <240fps or a weird color format (such as I444).

### Installation:

> This is also covered in [my installation video](https://youtu.be/RfPDgoMuSWg)
- Go to https://github.com/nihui/rife-ncnn-vulkan/releases/latest and download the Windows version.    
- Unzip the file.
- Navigate to the folder with all the different models, and copy the path to the one you want to use. (If you're unsure, just use rife-v4.6)
- Go to `...\smoothie-rs-nightly\smoothie-rs` and open `recipe.ini`
- Scroll to the bottom of the config, and replace `model: rife-v4.4` with `model: {rife model path here}`
- To use RIFE, enable pre-interp.

## Explaining the config (recipe)

Smoothie-RS's config ***MUST STAY IN THAT FOLDER***, moving it to the same folder as your video wont do anything.

- Heres the config with explanations of what everything does:

```ini
[interpolation]
enabled: yes # Enables interpolation (SVPFlow algorithm).
masking: no # Masking let's you block out certain areas of the video to avoid HUD artifacts. This toggles masking for interpolation only.
fps: 1920 # What framerate you would like to interpolate to (a factor such as 2x or 3x will also work).
speed: medium # A faster speed will result in more artifacts and less accurate interpolation, only worth changing if your input video is over 300fps.
tuning: weak # A different tuning will slightly change how the interpolation looks, weak is considered the best for Minecraft.
algorithm: 23 # Different algorithms change the interpolation a bit, 23 looks the smoothest, 13 has the least artifacts. Others are not recommended.
use gpu: yes # Uses your GPU to interpolate.

[frame blending]
enabled: yes # Enables frame blending.
fps: 60 # Desired output framerate.
intensity: 1.0 # A higher intensity will reuse blur frames from the previous frame, 1.0-1.5 is considered the sweet spot for 60fps output.
weighting: equal # A different weighting will change the opacity of the blur frames, such as ascending or gaussian.
bright blend: no # Overlays the frames in a different way, the same way as Premiere Pro's frame blending.

[flowblur] # RSMB-like motion blur.
enabled: no # Enables flowblur.
masking: yes # Masking toggle for flowblur.
intensity: 50 # Intensity of the flowblur.
do blending: after # Choose if you want to do frame blending before or after flowblur is applied.

[output]
process: ffmpeg # What process is used for the encoding.
enc args: # Put your own encoding arguments here.
file format: %FILENAME% ~ %FRUIT% # How the file name is formatted.
container: .MP4 # Output video container.

[preview window]
enabled: yes # Enables the preview window.
process: ffplay # Process used for the preview window.
output args: -f yuv4mpegpipe - # Preview window arguments.

[artifact masking] # Tutorial for mask-making later in the guide.
enabled: no # Global toggle for masking.
feathering: yes # Adds feathering to the mask.
folder path: # Put the path to the folder that your mask is in.
file name: # Put the name of your mask here.

[miscellaneous]
play ding: no # Plays a ding sound when the video is done rendering (it's either broken or im incorrect).
always verbose: no # The Send to shortcut takes care of this.
dedup threshold: 0 # Uses interpolation to guess duplicated frames (set to around 0.01 - 0.005).
global output folder: # Seems to be broken.
source indexing: no # People don't seem to know, i'll update this if I ever get an answer.
ffmpeg options: -loglevel error -i - -hide_banner -stats -stats_period 0.15 # Arguments for ffmpeg.
ffplay options: -loglevel quiet -i - -autoexit -window_title smoothie.preview # Argumentss for ffplay (preview window).

[console] # These seem to be ignored when using Send to.
stay on top: no
borderless: yes
position: top left
width: 900
height: 350

[timescale] # Changes the speed of the video.
in: 1.0
out: 1.0

[color grading]
enabled: no # Enables color grading.
brightness: 1.0
saturation: 1.0
contrast: 1.0

[lut]
enabled: no # Enables the LUT filter.
path: # Put a path to a .cube file here (coldchrome download later in the guide).
opacity: 0.1 # Opacity of the lut.

[pre-interp]
enabled: no # Enables pre-interp.
masking: no # Enables masking for pre-interp.
factor: 2x # Fps factor (2x is probably all you need).
model: # Put your RIFE path here.
```

## Making a mask

It's worth noting that masking is only really needed for low-fps inputs such as 120-180fps, or if you are using Flowblur.
- Download Paint.net from https://getpaint.net.
- Take a screenshot of your video (make sure the resolution matches and the HUD is visible).
- Open the screenshot in Paint.net.
- Make a new layer.
- Make sure Anti-Aliasing is DISABLED.
- Start coloring the parts of the video you want to mask in *black*.
- Once you are done, fill the rest of the image with *white*.
- Do Ctrl + Shift + S and save as a PNG.
- Copy the path and use it in Smoothie-RS.

## Encoding arguments

### What is an encoding argument?

Encoding arguments change how your video is encoded, you can customize bitrate, encoder, codec, even add some visual effects like sharpening or upscaling.

### GPU-specific recommendations:
  - `-c:v h264_nvenc -rc constqp -preset p7 -qp 15` - H264 (Nvidia)
  - `-c:v hevc_nvenc -rc constqp -preset p7 -qp 15` - HEVC (Nvidia)
  - `-c:v h264_amf -quality quality -qp_i 16 -qp_p 18 -qp_b 22` - H264 (AMD)
  - `-c:v hevc_amf -quality quality -qp_i 18 -qp_p 20 -qp_b 24` - HEVC (AMD)
  - `-c:v libx264 -preset slow -aq-mode 3 -crf 16` - H264 (CPU)
  - `-c:v libx265 -preset medium -x265-params aq-mode=3:no-sao=1 -crf 20` - HEVC (CPU)
> *Tweak the quality level to match your recording settings.*
- You can edit `encoding_presets.ini` to add your own custom presets!

## Coldchrome LUT download

Coldchrome should be used with around 0.1-0.2 opacity depending on the clip. You can download it [here](https://files.catbox.moe/d5jvto.cube)

## Troubleshooting

Here are some common problems and their solutions:
#### Smoothie crashing before any error can be read
- **Solution:** Run Smoothie by using `launch.cmd` and select your video. Smoothie will now stay open and you can read the error. <ins>If there's no error message, it's likely a problem with your Send to shortcut</ins>.

#### Recipe: Setting 'X' has no parent category
- **Solution:** Double check the syntax of your recipe config file. Make sure you're not using the config from the old [smoothie-py](https://github.com/couleur-tweak-tips/Smoothie) or [blur](https://github.com/f0e/blur).

#### ffmpeg.exe is not installed/in PATH, ensure FFmpeg is installed
- **Solution:** You need to have FFmpeg installed <ins>and in path</ins> in for Smoothie to work. The easiest way to install it is by running an automated installer:  
`powershell -noe "iex(irm tl.ctt.cx); Get FFmpeg"`  
This will install [Scoop](https://scoop.sh) and FFmpeg.

#### No valid videos were passed to smoothie
- **Solution:** Make sure there's nothng wrong with your video (try reencoding), and make sure FFmpeg is installed.

#### 'X' is not an official yuv4mpegpipe pixel format
- **Solution:** Your video is recorded in a color format that is unsuppored by SVPFlow (interpolation algorithm). Convert your video to a supported color format (NV12) or don't use interpolation (or use pre-interp instead).

#### Driver does not support the required nvenc API version.
- **Solution:** Update your Nvidia GPU drivers. 

#### Python exception: Source: The index does not match the source file
- **Solution:** Ensure your video path doesn't have any non-english, special characters. The name of your file might be the problem, try renaming it.
