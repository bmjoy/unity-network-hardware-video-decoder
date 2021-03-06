# UNHVD Unity Network Hardware Video Decoder

Example of video and point cloud streaming with hardware decoding and custom [MLSP](https://github.com/bmegli/minimal-latency-streaming-protocol) protocol:

- streaming video to UI element (RawImage)
- streaming video to scene element (anything with texture)
- streaming textured point clouds (Mesh)

This project contains native library plugin.

See [how-it-works](https://github.com/bmegli/unity-network-hardware-video-decoder/wiki/How-it-works) on wiki to understand the project.

See [benchmarks](https://github.com/bmegli/unity-network-hardware-video-decoder/wiki/Benchmarks) on wiki for glass-to-glass latency.

See [hardware-video-streaming](https://github.com/bmegli/hardware-video-streaming) for related projects.

See videos to understand point cloud streaming features:

| Point Cloud Streaming | Infrared Textured Point Cloud Streaming |
|-----------------------|-----------------------------------------|
| [![Hardware Accelerated Point Cloud Streaming](http://img.youtube.com/vi/qnTxhfNW-_4/0.jpg)](http://www.youtube.com/watch?v=qnTxhfNW-_4) | [![Hardware Accelerated Infrared Textured Point Cloud Streaming](http://img.youtube.com/vi/zVIuvWMz5mU/0.jpg)](https://www.youtube.com/watch?v=zVIuvWMz5mU) |

## Video sources

Currently Unix-like platforms only.

- [NHVE](https://github.com/bmegli/network-hardware-video-encoder) (dummy, procedurally generated video)
- [RNHVE](https://github.com/bmegli/realsense-network-hardware-video-encoder) (Realsense camera streaming)

## Platforms 

Unix-like operating systems (e.g. Linux), [more info](https://github.com/bmegli/unity-network-hardware-video-decoder/wiki/Platforms).

Tested on Ubuntu 18.04.

## Hardware

Tested on Intel Kaby Lake.

### Video

Intel VAAPI compatible hardware decoders (Quick Sync Video).

It is likely that H.264 through VAAPI will work also on AMD and NVIDIA.

[Other technologies](https://github.com/bmegli/unity-network-hardware-video-decoder/wiki/Hardware) may also work but were not tested.


### Depth/point clouds/textured point clouds

Intel VAAPI HEVC Main10 compatible hardware decoders, at least Intel Apollo Lake.

[Other technologies](https://github.com/bmegli/unity-network-hardware-video-decoder/wiki/Hardware) may also work but were not tested.

## Dependencies

All dependencies apart from FFmpeg are included as submodules, [more info](https://github.com/bmegli/unity-network-hardware-video-decoder/wiki/Dependencies).

Works with system FFmpeg on Ubuntu 18.04 and doesn't on 16.04 (outdated FFmpeg and VAAPI ecosystem).

## Building Instructions

Tested on Ubuntu 18.04.\
Requires Unity 2019.3  for [technical reasons](https://github.com/bmegli/unity-network-hardware-video-decoder/wiki/How-it-works#point-clouds).

``` bash
# update package repositories
sudo apt-get update 
# get avcodec and avutil
sudo apt-get install ffmpeg libavcodec-dev libavutil-dev
# get compilers and make 
sudo apt-get install build-essential
# get cmake - we need to specify libcurl4 for Ubuntu 18.04 dependencies problem
sudo apt-get install libcurl4 cmake
# get git
sudo apt-get install git
# clone the repository with *RECURSIVE* for submodules
git clone --recursive https://github.com/bmegli/unity-network-hardware-video-decoder.git

# build the plugin shared library
cd unity-network-hardware-video-decoder
cd PluginsSource
cd unhvd-native
mkdir build
cd build
cmake ..
make

# finally copy the native plugin library to Unity project
cp libunhvd.so ../../../Assets/Plugins/x86_64/libunhvd.so
```

## Testing

Assuming you are using VAAPI device.

1. Open the project in Unity
2. Choose and enable GameObject
4. For Script define Configuration (`Port`, `Device`)
5. For troubleshooting use programs in `PluginsSource/unhvd-native/build`

|  Element         | Video                                 |  Point Clouds                           |
|------------------|---------------------------------------|-----------------------------------------|
| GameObject       | `Canvas` -> `CameraView` -> `RawImage` (UI) <br> `VideoQuad` (scene) | `PointCloud`                            |
| Script           | `RawImageVideoRenderer` <br> `VideoRenderer` | `PointCloudRenderer`             |
| Configuration    | `Port` (network) <br> `Device` (acceleration> <br> script code | `Port` (network) <br> `Device` (acceleration) <br> script code                        |
| Troubleshooting  | `unhvd-frame-example`                 | `unhvd-cloud-example`                   |

### Sending side
|  Element         | Video                                 |  Point Clouds                           |
|------------------|---------------------------------------|-----------------------------------------|
| Sending side     | NHVE `nhve-stream-h264` <br> RNHVE `realsense-nhve-h264` | RNHVE `realsense-nhve-hevc` <br> RNHVE `realsense-nhve-depth-ir` <br> RNHVE `realsense-nhve-depth-color` |

For a quick test you may use [NHVE](https://github.com/bmegli/network-hardware-video-encoder) procedurally generated H.264 video.

If you have Realsense camera you may use [RNHVE](https://github.com/bmegli/realsense-network-hardware-video-encoder).

#### Video

```bash
# assuming you build NHVE port is 9766, VAAPI device is /dev/dri/renderD128
# in NHVE build directory
./nhve-stream-h264 127.0.0.1 9766 10 /dev/dri/renderD128
# if everything went well you will see 10 seconds video (moving through grayscale).

# assuming you build RNHVE, port is 9766, VAAPI device is /dev/dri/renderD128
# in RNHVE build directory
./realsense-nhve-h264 127.0.0.1 9766 color 640 360 30 20 /dev/dri/renderD128
# if everything went well you will see 20 seconds video streamed from Realsense camera.
```

#### Point Clouds

Assuming Realsense D435 camera and 848x480.

```bash
# assuming you build RNHVE, port is 9768, VAAPI device is /dev/dri/renderD128
# in RNHVE build directory
./realsense-nhve-hevc 127.0.0.1 9768 depth 848 480 30 500 /dev/dri/renderD128
# for infrared textured point cloud (only D435)
./realsense-nhve-depth-ir 127.0.0.1 9768 848 480 30 500 /dev/dri/renderD128 8000000 1000000 0.0001
# for color texture point cloud, depth aligned (D435 tested)
./realsense-nhve-depth-color 127.0.01 9768 depth 848 480 848 480 30 500 /dev/dri/renderD128 8000000 1000000 0.0001f
# for color textured point cloud, color aligned (D435 tested), color intrinsics required
./realsense-nhve-depth-color 127.0.01 9768 color 848 480 848 480 30 500 /dev/dri/renderD128 8000000 1000000 0.0001f
```

If you are using different Realsense device/resolution you will have to configure camera intrinsics.

See [point clouds configuration](https://github.com/bmegli/unity-network-hardware-video-decoder/wiki/Point-clouds-configuration) for the details.

## OS tweaks

If you experience [stuttering with artifacts](https://github.com/bmegli/unity-network-hardware-video-decoder/issues/10#issuecomment-633255931) increase UDP buffer size.

```bash
# here 10 x the the default value on my system
sudo sh -c "echo 2129920 > /proc/sys/net/core/rmem_max"
sudo sh -c "echo 2129920 > /proc/sys/net/core/rmem_default"
```

## License

Code in this repository and my dependencies are licensed under Mozilla Public License, v. 2.0

This is similiar to LGPL but more permissive:
- you can use it as LGPL in prioprietrary software
- unlike LGPL you may compile it statically with your code

Like in LGPL, if you modify the code, you have to make your changes available.
Making a github fork with your changes satisfies those requirements perfectly.

Since you are linking to FFmpeg libraries consider also `avcodec` and `avutil` licensing.

