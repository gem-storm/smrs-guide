Welcome to my _unofficial guide_ for Couleur's [Smoothie-RS](https://github.com/couleur-tweak-tips/smoothie-rs).

I personally recommend using f0e's [blur](https://github.com/f0e/blur) if you would rather have a more simple and user-friendly program.

> [!IMPORTANT]  
> Please read [Troubleshooting](#troubleshooting) if you encounter any issues. If you are unable to address your issue on your own, please create a support post in [CTT's support forum](https://discord.com/channels/774315187183288411/1020030329790148678).

## Table of contents

- [Installation](#installation)  
- [RIFE/pre-interp](#pre-interp-rife)   
- [Explaining the config (recipe)](#explaining-the-config-recipe)
- [Making a mask](#making-a-mask)
- [Encoding arguments](#encoding-arguments)
- [Coldchrome LUT download](#coldchrome-lut-download)
- [More SendTo/command line arguments](#more-sendtocommand-line-arguments)
- [Troubleshooting](#troubleshooting)

## Installation
- Go to [github.com/couleur-tweak-tips/smoothie-rs/releases/latest](https://github.com/couleur-tweak-tips/smoothie-rs/releases/latest), download `smoothie-rs-nightly.zip`, and unzip it.
- Running `launch.cmd` will start the program, you can change the settings by editing `recipe.ini` (all settings are explained later in this guide).
#### Adding to Send to (recommended):
- Go to `...\smoothie-rs-nightly\smoothie-rs\bin` and make a shortcut to `smoothie-rs.exe`.
- Open the shortcut's properties and type `-v -i` at the end of `Target`. Should be like this:
```
...\path\to\smoothie-rs.exe -v -i
```

You can name this shortcut to whatever you prefer, I just call it `Smoothie-RS`
- Do Win + R, type `shell:sendto` in the run dialog and hit enter.
- Move the smoothie-rs.exe shortcut you made earlier into the folder that opened.
- Now you can send videos to Smoothie by Right click > Send to > `Smoothie-RS`.

## pre-interp (RIFE)

#### What is RIFE?

RIFE is an interpolation model, it is very slow compared to SVPFlow (the interpolation algorithm that Blur/Smoothie's interpolation uses), but it's way more accurate. It's only worth installing and using if your input video is <240fps or a color format that SVPFlow doesn't support (such as I444).

### Installation:
- Go to [github.com/nihui/rife-ncnn-vulkan/releases/latest](https://github.com/nihui/rife-ncnn-vulkan/releases/latest) and download the Windows version.    
- Unzip the file.
- Navigate to the folder with all the models and copy the path of the desired model. (If you're unsure, just use rife-v4.6)
- Go to `...\smoothie-rs-nightly\smoothie-rs` and open `recipe.ini`
- Scroll to the bottom of the config, and replace `model: rife-v4.4` with `model: "path\to\rife-model"`
- To use RIFE, enable pre-interp.

## Explaining the config (recipe)

Smoothie-RS's config ***MUST STAY IN THAT FOLDER*** (unlike f0e's blur).

- Here's a config I made with comments explaining the settings:
```ini
[interpolation]
fps: 1920
	# Target interpolation framerate (a factor such as 2x or 3x will also work).
speed: medium
	# Faster speeds = less accurate interpolation but faster render times. Options: medium, fast, faster, and fastest.
tuning: weak
	# Slightly changes how the interpolation frames look. Options: animation, film, smooth, and weak.
algorithm: 23
	# Algorithms also change the interpolation a bit. Options: 13 and 23 (there are more, but they aren't recommended).

[frame blending]
fps: 60
	# Output video's framerate.
intensity: 1.0
	# Higher intensity will give more ghosting, recommended to start at 1.0 and adjust to your liking.
weighting: equal
	# Weighting will change the opacity of the blur frames. Options: ascending, equal, gaussian, gaussian_sym, pyramid, and vegas.
bright blend: no
	# Only blends the bright parts of the frames, the same as Adobe Premiere Pro's blending.

[flowblur]
	# RSMB-like motion blur.
do blending: after
	# Choose whether frame blending before or after flowblur.

[output]
process: ffmpeg
	# Process used for encoding.
enc args: H264 CPU
	# Encoding arguments (explained later).
file format: %FILENAME% ~ %FRUIT%
	# File name format.

[artifact masking]
	# Covers areas of your video to prevent artifacts. Tutorial for mask-making later in the guide.

[miscellaneous]
dedup threshold: 0
	# Uses interpolation to replace duplicated frames (set to around 0.01 - 0.005).
global output folder:
	# Seems to be broken

[lut]
path:
	# Put a path to a .cube file here (coldchrome download later in the guide).

[pre-interp]
factor: 2x
	# Fps factor.
```

## Making a mask

It's worth noting that masking is only really needed for low-fps inputs such as 120-180fps, or if you are using Flowblur.
> [Video guide](https://www.youtube.com/watch?v=5GW2TUx78WY), shoutout to `@ixtf`.
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
  - `-c:v h264_nvenc -preset p7 -qp 15` - H264 (Nvidia)
  - `-c:v hevc_nvenc -preset p7 -qp 15` - HEVC (Nvidia)
  - `-c:v h264_amf -quality quality -qp_i 16 -qp_p 18 -qp_b 22` - H264 (AMD)
  - `-c:v hevc_amf -quality quality -qp_i 18 -qp_p 20 -qp_b 24` - HEVC (AMD)
  - `-c:v libx264 -preset slow -aq-mode 3 -crf 16` - H264 (CPU)
  - `-c:v libx265 -preset medium -x265-params aq-mode=3:no-sao=1 -crf 20` - HEVC (CPU)
> *Tweak the quality level to match your recording settings.*
- You can edit `encoding_presets.ini` to add your own custom presets!

## Coldchrome LUT download

Coldchrome should be used with around 0.1-0.2 opacity. You can download it [here](https://files.catbox.moe/d5jvto.cube)

## More SendTo/command line arguments

Remember when we added `-v` and `-i` in the SendTo shortcut? There's actually a lot more we can do with the target box:

```
-i/--input      Specify input video (you need to do '-i "path\to\input"' if you're using the command line.).
-o/--output     Specify output video's path.
-t/--tui        Makes smoothie pause before exiting.
--outdir        Specify output directory.
--stripaudio    Removes audio.
-v/--verbose    Prints verbose information.
--debug         Prints all the nerdy stuff to find bugs.
-r/--recipe     Specify a recipe path.
--override      Override any recipe setting(s), eg --override "flowblur;amount;40".
```
> I've removed the ones that I don't recommend. If you want to see them all; drag smoothie-rs.exe onto a command prompt window (to paste the path) and hit enter.
#### Here's my personal configuration (I have multiple SendTo shortcuts that use specific recipes):
```
path\to\smoothie-rs.exe -r "path\to\recipe" --outdir "D:\Smoothied" -v -i
```

## Troubleshooting

Here are some common problems and their solutions:
#### Smoothie crashing before any error can be read
- **Solution:** Run Smoothie by using `launch.cmd` and select your video. Smoothie will now stay open and you can read the error. <ins>If there's no error message, it's likely a problem with your Send to shortcut</ins>.

#### Recipe: Setting 'X' has no parent category
- **Solution:** Double check the syntax of your recipe config file. Make sure you're not using the config from the old [smoothie-py](https://github.com/couleur-tweak-tips/Smoothie) or [blur](https://github.com/f0e/blur).

#### ffmpeg.exe is not installed/in PATH, ensure FFmpeg is installed
- **Solution:** You need to have FFmpeg installed <ins>and in path</ins> in for Smoothie to work. The easiest way to install it is by running an automated installer:  
`powershell -noe "iex(irm tl.ctt.cx); Get FFmpeg"`  
This will install [Scoop](https://scoop.sh) and use it to install FFmpeg (and add it to path).

#### No valid videos were passed to smoothie
- **Solution:** Make sure there's nothng wrong with your video (try reencoding), and make sure FFmpeg is installed.

#### 'X' is not an official yuv4mpegpipe pixel format
- **Solution:** Your video is recorded in a color format that is unsuppored by SVPFlow (interpolation algorithm). Convert your video to a supported color format (NV12) or don't use interpolation (or use pre-interp instead).

#### Driver does not support the required nvenc API version.
- **Solution:** Update your Nvidia GPU drivers. 

#### Python exception: Source: The index does not match the source file
- **Solution:** Ensure your video path doesn't have any non-english, special characters. The name of your file might be the problem, try renaming it.
