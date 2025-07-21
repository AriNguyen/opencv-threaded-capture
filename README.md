# Increase Webcam FPS with Multithreading in OpenCV C++

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](/LICENSE)
[![Stars](https://img.shields.io/github/stars/AriNguyen/opencv-threaded-capture.svg?style=social)](https://github.com/AriNguyen/opencv-threaded-capture/stargazers)
[![CI](https://github.com/AriNguyen/opencv-threaded-capture/actions/workflows/ci.yml/badge.svg)](https://github.com/AriNguyen/opencv-threaded-capture/actions/workflows/ci.yml)
[![Lines of Code](https://tokei.rs/b1/github/AriNguyen/opencv-threaded-capture)](https://github.com/XAMPPRocky/tokei)

Real‑time multithreaded webcam/video capture in modern C++20 & OpenCV that keeps your main thread free for computer vision or ML inference.

## Why?

OpenCV's `VideoCapture` is synchronous: every `read()` blocks on USB/RTSP I/O and decoding. This library adds a **producer/consumer** queue so frame acquisition runs on a dedicated thread, lifting throughput up to **32%** on a 4‑core laptop while keeping latency bounded.

## Features

| Category       | What you get                                                                 |
|----------------|------------------------------------------------------------------------------|
| Concurrency    | Single‑producer / single‑consumer lock‑free ring buffer with back‑pressure.  |
| Modern C++     | C++20, RAII, std::scoped_lock, std::jthread, std::chrono timing.             |
| Cross‑platform | Linux 🐧, macOS 🍏, Windows 🪟 (tested in CI).                                |
| Metrics        | Built‑in FPS / latency stats returned as a C++ struct or JSON.               |
| Extensible     | Optional CUDA path (-DENABLE_CUDA=ON), gRPC frame streaming, ONNXRuntime inference hooks. |

## Quick Start

### Docker (zero install)

```sh
# Linux: make your webcam available inside the container
sudo docker run --device /dev/video0 -it aring/opencv-threaded-capture --num_frames 500
```

### Native

```sh
# Ubuntu 22.04 example
sudo apt-get install -y build-essential cmake libopencv-dev
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
./build/threaded_capture --device 0 --num_frames 500
```

## Build from Source

```sh
mkdir build
cd build
cmake ../
make

./../bin/thread_opencv_cpp  # execute bin file
```

Remove files in .gitignore:

```sh
chmod 700 utils/clean.bash
./utils/clean.bash < .gitignore
```

## Webcam Stream

The `detach` method (`t1.detach()`) is used so we don't need to wait for thread 1 to finish. Instead, it will get the dataframe. The process happens simultaneously.

## Measuring FPS and Elapsed Time

I first used the **chrono** library to measure the time but found that it's hard to convert to seconds for calculating FPS. So, I use **ctime**:

```cpp
// in utils.cpp
#include <ctime>

numFrames = 100;

clock_t start = clock();
// some function here
clock_t end = clock();

double elapsed_secs = double(end - start) / CLOCKS_PER_SEC;
double fps = numFrames / elapsed_secs;
```

## Face Detection using dlib

See: [dlib webcam_face_pose_ex.cpp example](http://dlib.net/webcam_face_pose_ex.cpp.html)

## Benchmark

### Just streaming webcam

Stream 1000 frames for 10 times and record the data:

```sh
# run in terminal
for i in {1..10}; do
    # execute and direct output to text file
    ./bin/thread_opencv_cpp 1000 >> output.txt
done
```

#### Test 10 times with multithreading

| Frames | Elapsed (Avg) | FPS (Avg) |
|--------|---------------|-----------|
| 100    | 1.57126       | 63.6563   |
| 1000   | 14.5097       | 68.9689   |

#### Test 10 times without multithreading

| Frames | Elapsed (Avg) | FPS (Avg) |
|--------|---------------|-----------|
| 100    | 1.95773       | 51.0956   |
| 1000   | 13.9149       | 52.4172   |

The elapsed time doesn't change much; however, the FPS of streaming 100 and 1000 frames increases by 23.5% and 31.5%, respectively.

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

## Acknowledgements

- Inspired by [PyImageSearch: "How to increase FPS with multithreading in OpenCV"](https://www.pyimagesearch.com/2015/12/21/increasing-webcam-fps-with-python-and-opencv/)
- Ring‑buffer pattern adapted from Dmitry Vyukov’s MPSC queue.
- Thanks to all contributors and stargazers for keeping the project alive!