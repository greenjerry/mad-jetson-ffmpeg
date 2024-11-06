FFmpeg README
=============

FFmpeg is a collection of libraries and tools to process multimedia content
such as audio, video, subtitles and related metadata.

## Changes in Mad Jetson Port

**WIP**: Not yet ready and fully working. Some bugs still need to be fixed!

TODO:
 - [x] Import patches
 - [x] Fix patches to work with changes apis (ffmpeg 4.2 -> 6.0)
 - [x] Test encoders and decoders exist
 - [ ] Test decoding actually working and if options are shown
 - [ ] Fix encoder error: `Error while dequeing buffer from output plane`

I compiled this together to have a kinda "ultimate ffmpeg" for the jetson.

If you just want an easy way to use the hw decoders and encoders of the jetson, I highly recommend just using [jetson-ffmpeg](https://github.com/jocover/jetson-ffmpeg) instead. I mainly created this to have a ffmpeg build that also contains the normal software encoders/decoders as jetson-ffmpeg by default builds without many expected features.

This repo is based on the official ffmpeg repo (v6.0) but with these changes:
 - 36a7650323: Added the patch from [jetson-ffmpeg](https://github.com/jocover/jetson-ffmpeg) which was slightly altered to work with this newer ffmpeg version
 - c34197c23e: Added the official nvidia hw decoder patch as described and attached in [this email](http://ffmpeg.org/pipermail/ffmpeg-devel/2020-June/263746.html) (also altered a bit for the newer version)
 - 912208f5db: Added a fix made by [CTCaer](https://gitlab.com/CTCaer) which should fix a memory issue (see commit for more details and source).

I have not yet gotten to really look into [switch-l4t-multimedia/FFmpeg](https://gitlab.com/switchroot/switch-l4t-multimedia/FFmpeg/) but this repo seems also really promising and uses a seemingly similar but improved nvidia hw decoder and it also has a corresponding encoder. So giving this a look is also worth a short as well as integrating the encoder here as well.

### Requirements

The port assumes you use Jetpack 5.1 (L4T 35.2.1) using the Ubuntu 20.04 Sample RootFS or a version very close to it. Other versions might work but will probably break at a certain point.

I tested this repo specifically on a Jetson Xavier AGX 32GB but I don't see why it wouldn't work on other Jetson models.

#### System Dependencies (required)

Whether you want to build this yourself or just attempt to use the built release files, you should install these packages to ensure it works:

```bash
sudo apt install build-essential cmake pkg-config bzip2 fontconfig libfribidi{0,-dev} gmpc{,-dev} gnutls-bin lame libass{9,-dev} libavc1394-{0,dev} libbluray{2,-dev} libdrm{2,-dev} libfreetype6{,-dev} libmodplug{1,-dev} libraw1394-{11,dev} librsvg2{-2,-dev} libsoxr{0,-dev} libtheora{0,-dev} libva{2,-dev} libva-drm2 libva-x11-2 libvdpau{1,-dev} libvorbisenc2 libvorbis{0a,-dev} libvpx{6,-dev} libwebp{6,-dev} libx11{-6,-dev} libx264-{155,dev} libx265-{179,dev} libxcb1{,-dev} libxext{6,-dev} libxml2{,-dev} libxv{1,-dev} libxvidcore{4,-dev} libopencore-amr{nb0,nb-dev,wb0,wb-dev} opus-tools libsdl2-dev speex v4l-utils zlib1g{,-dev} libopenjp2-7{,-dev} libssh-{4,dev} libspeex{1,-dev} libgmp-dev libgnutls28-dev libladspa-ocaml-dev libmp3lame{0,-dev} libopus{0,-dev} libv4l-{0,dev} cuda-nvcc-11-4
```

#### Jetson FFmpeg Library (required)

This port includes the patch from Jetson-FFmpeg. It therefore expects their library to be built and installed on your system.

So make sure you built and installed the [jetson-ffmpeg](https://github.com/jocover/jetson-ffmpeg) library part (do step 1 from their readme).

**Important:** Please also apply [this fix](https://github.com/jocover/jetson-ffmpeg/issues/115#issuecomment-1384755884) by [HTani](https://github.com/HTani) as ffmpeg will not find the library otherwise when configuring.

#### SVT-AV1 (optional)

If you use the build with AV1 included or build it yourself without ommitting the flag `--enable-libsvtav1` you need to currently build the required library yourself as Ubuntu 20.04 does not include it in its repos.

You can build and install it using the following commands:

```bash
git clone https://gitlab.com/AOMediaCodec/SVT-AV1 -b v1.5.0 --single-branch
cd SVT-AV1
mkdir build
cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)
sudo make install
sudo ldconfig
```

### How to build it yourself

To build this yourself, make sure you followed the previous requirements first.

Then clone this repo onto your jetson device using: `git clone https://github.com/LinusCDE/mad-jetson-ffmpeg.git -b release/6.0`.

Go into the newly cloned repo and configure ffmpeg using this command:

```bash
./configure \
  --prefix=/usr/local \
  --extra-cflags="-I/usr/local/include" \
  --extra-ldflags="-L/usr/local/lib" \
  --disable-debug \
  --disable-stripping \
  --enable-lto \
  --enable-fontconfig \
  --enable-gmp \
  --enable-gnutls \
  --enable-gpl \
  --enable-ladspa \
  --enable-libass \
  --enable-libbluray \
  --enable-libdrm \
  --enable-libfreetype \
  --enable-libfribidi \
  --enable-libmodplug \
  --enable-libmp3lame \
  --enable-libopencore_amrnb \
  --enable-libopencore_amrwb \
  --enable-libopenjpeg \
  --enable-libopus \
  --enable-libpulse \
  --enable-librsvg \
  --enable-libsoxr \
  --enable-libspeex \
  --enable-libssh \
  --enable-libtheora \
  --enable-libv4l2 \
  --enable-libvorbis \
  --enable-libvpx \
  --enable-libwebp \
  --enable-libx264 \
  --enable-libx265 \
  --enable-libxcb \
  --enable-libxml2 \
  --enable-libxvid \
  --enable-version3 \
  --enable-libsvtav1 \
  --enable-nonfree \
  --enable-nvmpi \
  --enable-nvv4l2dec \
  --extra-libs="-L/usr/lib/aarch64-linux-gnu/tegra -lnvbuf_utils -lnvbufsurface" \
  --extra-cflags="-I /usr/src/jetson_multimedia_api/include/" \
  --enable-shared 
```

(Make sure to remove `--enable-libsvtav1` if you didn't install SVT-AV1 before.)

Afterwards you should be able to build ffmpeg using `make -j$(nproc)`. This will take a couple of minutes or even longer depending on your model.

Now you can run the built ffmpeg in your directory using `./ffmpeg`.

To install this onto your system, first uninstall the ffmpeg installed with apt and then install the just built version by running `sudo make install`.

### Use the prebuilt release

The [release section](https://github.com/LinusCDE/mad-jetson-ffmpeg/releases) of this repo will also have two prebuilt versions of ffmpeg: One with AV1 Support and one without. If you didn't install SVT-AV1 please choose the "non AV1" version.

Download the corresponding attached ".tar.zst" file and extract it using `tar -xf <filename>`. Afterwards go into the extracted directory and you should be able to run ffmpeg using `./ffmpeg`.

In case it doesn't run, your system might not be compatible as some version might differ too greatly from the system I built it on. In this case building it yourself (see above) might help but you might need to do some additional troubleshooting to get it to build.

If it worked and you want ffmpeg installed into your system, first remove any installed ffmpeg with `sudo apt remove ffmpeg` first, then run `sudo make install` this ffmpeg version.

### New encoders and decoders

This version should have the vast majority of normal software encoders and decoders. Additionally these will be included:

Additional decoders:

```
 V..... h264_nvmpi           h264 (nvmpi) (codec h264)
 V..... h264_nvv4l2dec       h264 (nvv4l2dec) (codec h264)
 V..... hevc_nvmpi           hevc (nvmpi) (codec hevc)
 V..... hevc_nvv4l2dec       hevc (nvv4l2dec) (codec hevc)
 V..... mpeg2_nvmpi          mpeg2 (nvmpi) (codec mpeg2video)
 V..... mpeg2_nvv4l2dec      mpeg2 (nvv4l2dec) (codec mpeg2video)
 V..... mpeg4_nvmpi          mpeg4 (nvmpi) (codec mpeg4)
 V..... mpeg4_nvv4l2dec      mpeg4 (nvv4l2dec) (codec mpeg4)
 V..... vp8_nvmpi            vp8 (nvmpi) (codec vp8)
 V..... vp8_nvv4l2dec        vp8 (nvv4l2dec) (codec vp8)
 V..... vp9_nvmpi            vp9 (nvmpi) (codec vp9)
 V..... vp9_nvv4l2dec        vp9 (nvv4l2dec) (codec vp9)
```

Additional encoders:

```
 V..... h264_nvmpi           nvmpi H.264 encoder wrapper (codec h264)
 V..... hevc_nvmpi           nvmpi HEVC encoder wrapper (codec hevc)
```

To easily see all supported flags for the encoders use `ffmpeg -h encoder=<name_here>`.

### Word of advice

When using an hardware encoder (nvmpi being jetson-ffmpeg and nvv4l2.. being the offical ones), I recommend setting the minimum video bitrate to your target one.
I had horrible quality when not specifying it, as the hardware encoder seems to use a far too low bitrate by default unless forced not to do so.

This was one command I sometimes used for conversion from h264 to h264 (to reduce size). But using SW-Encoding still seems better and not that much smaller despite testing A LOT:

```sh
ffmpeg -fflags +igndts -vcodec h264_nvmpi -i "$input_file" -vcodec h264_nvmpi -profile:v baseline -level:v 4.1 -preset:v slow -rc vbr -minrate 256k -b:v 4000k -maxrate 10000k -bufsize 50M -g 360 $extra_flags "$output_file"
```
### New encoders and decoders

This version should have the vast majority of normal software encoders and decoders. Additionally these will be included:

**Additional decoders:**

```
 V..... h264_nvmpi           h264 (nvmpi) (codec h264)
 V..... h264_nvv4l2dec       h264 (nvv4l2dec) (codec h264)
 V..... hevc_nvmpi           hevc (nvmpi) (codec hevc)
 V..... hevc_nvv4l2dec       hevc (nvv4l2dec) (codec hevc)
 V..... mpeg2_nvmpi          mpeg2 (nvmpi) (codec mpeg2video)
 V..... mpeg2_nvv4l2dec      mpeg2 (nvv4l2dec) (codec mpeg2video)
 V..... mpeg4_nvmpi          mpeg4 (nvmpi) (codec mpeg4)
 V..... mpeg4_nvv4l2dec      mpeg4 (nvv4l2dec) (codec mpeg4)
 V..... vp8_nvmpi            vp8 (nvmpi) (codec vp8)
 V..... vp8_nvv4l2dec        vp8 (nvv4l2dec) (codec vp8)
 V..... vp9_nvmpi            vp9 (nvmpi) (codec vp9)
 V..... vp9_nvv4l2dec        vp9 (nvv4l2dec) (codec vp9)
```

**Additional encoders:**

```
 V..... h264_nvmpi           nvmpi H.264 encoder wrapper (codec h264)
 V..... hevc_nvmpi           nvmpi HEVC encoder wrapper (codec hevc)
```

To easily see all supported flags for the encoders use `ffmpeg -h encoder=<name_here>`.

### Word of advice

When using an hardware encoder (nvmpi being jetson-ffmpeg and nvv4l2.. being the offical ones), I recommend setting the minimum video bitrate to your target one.
I had horrible quality when not specifying it, as the hardware encoder seems to use a far too low bitrate by default unless forced not to do so.

This was one command I sometimes used for conversion from h264 to h264 (to reduce size). But using SW-Encoding still seems better and not that much smaller despite testing A LOT:

```sh
ffmpeg -fflags +igndts -vcodec h264_nvmpi -i "$input_file" -vcodec h264_nvmpi -profile:v baseline -level:v 4.1 -preset:v slow -rc vbr -minrate 256k -b:v 4000k -maxrate 10000k -bufsize 50M -g 360 $extra_flags "$output_file"
```

## Libraries

* `libavcodec` provides implementation of a wider range of codecs.
* `libavformat` implements streaming protocols, container formats and basic I/O access.
* `libavutil` includes hashers, decompressors and miscellaneous utility functions.
* `libavfilter` provides means to alter decoded audio and video through a directed graph of connected filters.
* `libavdevice` provides an abstraction to access capture and playback devices.
* `libswresample` implements audio mixing and resampling routines.
* `libswscale` implements color conversion and scaling routines.

## Tools

* [ffmpeg](https://ffmpeg.org/ffmpeg.html) is a command line toolbox to
  manipulate, convert and stream multimedia content.
* [ffplay](https://ffmpeg.org/ffplay.html) is a minimalistic multimedia player.
* [ffprobe](https://ffmpeg.org/ffprobe.html) is a simple analysis tool to inspect
  multimedia content.
* Additional small tools such as `aviocat`, `ismindex` and `qt-faststart`.

## Documentation

The offline documentation is available in the **doc/** directory.

The online documentation is available in the main [website](https://ffmpeg.org)
and in the [wiki](https://trac.ffmpeg.org).

### Examples

Coding examples are available in the **doc/examples** directory.

## License

FFmpeg codebase is mainly LGPL-licensed with optional components licensed under
GPL. Please refer to the LICENSE file for detailed information.

## Contributing

Patches should be submitted to the ffmpeg-devel mailing list using
`git format-patch` or `git send-email`. Github pull requests should be
avoided because they are not part of our review process and will be ignored.
