# [cmake] deploying ONNX models from pure OpenCV on C++ using ppocrv5v6

## Windows C++ console OCR recognition program based on PaddleOCRv5/v6

![[cmake] deploying ONNX models from ppocrv5v6 using pure OpenCV on C++.](01-efb8234e.jpg)

Insert image description here.

### Project Overview

This project is a **C++ console implementation**, which dynamically loads the `PPOCR.dll` to call PaddleOCR v5/v6 series ONNX models, and uses OpenCV DNN inference engine for local text recognition on Windows platforms. The program runs in a console interactive manner, supporting model switching and batch recognition in loops, with clear structure and simple dependencies.

---

### Algorithm Principles

#### Overall OCR Pipeline

Adopt the classic **three-stage OCR pipeline** architecture:

```
Input image → Text Detection (Det) → Orientation Classification (Cls, optional) → Text Recognition (Rec) → Output results
```

##### 1. Text Detection (Detection)

Detect text regions using the **DB (Differentiable Binarization)** algorithm:

1.

1. Scale the input image to a ratio of `limit_side_len = 960` while maintaining the aspect ratio.

1.

2. Extract multi-scale features through lightweight CNN backbone networks (MobileNetV3 / ResNet).

1.

3. Utilize FPN (Feature Pyramid Network) to fuse multi-scale feature maps

1.

4. Output the probability map, generate a binary map after thresholding (`det_db_thresh = 0.3`)

1.

5. Contour detection on binary images, using the Vatti clipping algorithm for dilation (`det_db_unclip_ratio = 1.6`) to obtain the final polygonal frame of the text area.

1.

6. Filter out boxes with confidence below `det_db_box_thresh = 0.6`

##### 2. Direction Classification (Optional)

-

• Only the PPOCRv5 Mobile model is enabled by default.

-

• Use lightweight classification models to determine whether text areas need to be rotated 180°.

-

• Classification threshold `cls_thresh = 0.9`

##### 3. Text Recognition

Text recognition using the **CRNN + CTC** architecture:

1.

1. Crop and resize the detected text region to a uniform height (`rec_img_h = 48`)

1.

2. Extracting visual feature sequences through CNN

1.

3. Use BiLSTM to encode the context of a sequence

1.

4. Decode the output text through CTC (Connectionist Temporal Classification)

1.

5. Supports batch inference (`rec_batch_num = 4`) to improve efficiency

#### Inference Engine

The program dynamically loads the `PPOCR.dll` (C interface), and uses the **Intel OpenVINO** inference engine, supporting:

-

• Direct loading of ONNX model format

-

• CPU reasoning

-

• Multi-threaded inference (`predictor_num = 4`)

#### Supported Models

|

Model Name

 ||

Detection model

 ||

Recognition model

 ||

Directional classification

 ||

Note

 |

|

PPOCRv5 Mobile

 ||

PP-OCRv5_mobile_det

 ||

PP-OCRv5_mobile_rec

 ||

Support

 ||

Lightweight, suitable for embedded scenarios

 |
|

PPOCRv5 Server

 ||

PP-OCRv5_server_det

 ||

PP-OCRv5_server_rec

 ||

Not Supported

 ||

High precision, suitable for server deployment

 |
|

PPOCRv6 Tiny

 ||

PP-OCRv6_tiny_det

 ||

PP-OCRv6_tiny_rec

 ||

Not Supported

 ||

Minimum volume, fastest speed

 |
|

PPOCRv6 Small

 ||

PP-OCRv6_small_det

 ||

PP-OCRv6_small_rec

 ||

Not Supported

 ||

Balanced precision and speed (recommended)

 |

---

### Environmental Requirements

|

Project

 ||

Requirements

 |

|

Operating system

 ||

Windows 10 / 11 x64

 |
|

Compiler toolchain

 ||

Visual Studio 2019 and above (MSVC v142+)

 |
|

CMake version

 ||

3.20 and above

 |
|

C++ Standard

 ||

C++17

 |
|

Platform architecture

 ||

x64 (required)

 |
|

OpenCV version

 ||

5.0 (world build version)

 |
|

Inference engine

 ||

OpenCV 5.0 (encapsulated in PPOCR.dll)

 |

#### Dependency Files

|

File name

 ||

Note

 |

| `PPOCR.dll` ||

OCR inference core DLL (encapsulates OpenCV + PPOCR algorithm)

 |
| `opencv_world500.dll` ||

OpenCV 5.0 runtime library

 |
| `inference/`

Contents

 ||

ONNX model file and dictionary file

 |

---

### Installation Steps

1. Preparing the source code

```
cd C:\Users\Administrator\Downloads\code
# Ensure that the FIRC_CPP directory exists.
```

#### 2. Preparing lib directory

Create a `lib/` folder in the root directory of the project, and place the following files inside:

```
FIRC_CPP/
└── lib/
    ├── PPOCR.dll
    └── inference/
        ├── PP-OCRv5_mobile_det_onnx.onnx
        ├── PP-OCRv5_mobile_rec_onnx.onnx
        ├── PP-OCRv5_mobile_cls_onnx.onnx
        ├── PP-OCRv5_server_det_infer.onnx
        ├── PP-OCRv5_server_rec_infer.onnx
        ├── PP-OCRv6_small_det.onnx
        ├── PP-OCRv6_small_rec.onnx
        ├── PP-OCRv6_small_rec_dict.txt
        ├── PP-OCRv6_tiny_det.onnx
        ├── PP-OCRv6_tiny_rec.onnx
        ├── PP-OCRv6_tiny_rec_dict.txt
        └── ppocrv5_dict.txt
```

#### 3. Configure OpenCV Path

Open `CMakeLists.txt`, confirm the following two paths are consistent with the actual path on your machine:

```
set(OpenCV_DIR "D:\\lufiles\\opencv500\\build\\x64\\vc16\\lib")   # OpenCV .lib Location in the directory
Set the OPENCV_BIN_DIR variable to "D:/lufiles/opencv500/build/x64/vc16/bin" ... # OpenCV .dll directory
```

#### 4. CMake generates the project

```
cd C:\Users\Administrator\Downloads\code\FIRC_CPP
cmake -B build -G "Visual Studio 16 2019" -A x64
```

>

If using VS2022, replace the generator with "Visual Studio 17 2022".

5. Compilation

```
cmake --build build --config Release
```

After compilation, the CMake post-build step will automatically copy the following files to the output directory `build\Release\`.

-

• PPOCR.dll (from lib/)

-

• `opencv_world500.dll` (from OpenCV bin directory)

-

• `inference/` entire directory (from `lib/inference/`)

---

### Running Steps

1. Launching the program

```
cd build\Release
# Interactive Mode (Manually Enter the Image Path After Starting)
.\FIRC_CPP.exe
### 2. Choose a model
After the program starts, display the model menu:

```

======== Selection of OCR Model ========
[1] v5 mobile
[2] v5 server
[3] v6 small
[4] v6 tiny
Please choose (1-4, default 4):

```
Enter the number to select the model, and press Enter to default to **v6 tiny**.
### 3. Input image path

```

Please enter the image path (press Enter to use 3.jpg): test.jpg

```
The program automatically identifies and outputs the results:

```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Time taken: 186.32 ms | Identified 5 text regions
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[1] Hello World (score=0.97) box: (10,20) (200,20) (200,60) (10,60)
[2] Hello OCR (score=0.95) box: …
...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

```
### 4. Cycle Recognition
After the recognition is completed, the program does not exit. You can continue to input new image paths for recognition:

```

Please enter the image path (press Enter to recognize the current image, q to exit): another.jpg

```
- **Press Enter**: Re-identify the current image
- **Input new path**: Recognize new images
- **Input `q`**: Exit the program, automatically release model resources
---
## Notes
1. **Must use x64 platform**: `PPOCR.dll` and `opencv_world500.dll` are both 64-bit, not supporting x86.
2. **CMake Cache Issues**: After modifying the `CACHE` variable in `CMakeLists.txt` (such as `PPOCR_DIR`, `OPENCV_BIN_DIR`), you need to delete the `build/` directory and regenerate, otherwise the old value will be cached:
   ```powershell
   Remove-Item build -Recurse -Force
   cmake -B build -G "Visual Studio 16 2019" -A x64
```

1.

1. **Support for Chinese Paths**: The program has built-in support for compatibility with Chinese paths (first try `cv::imread`, then fallback to binary reading + `cv::imdecode`). However, it is recommended to use purely English paths to avoid potential issues.

1.

2. **Console Chinese Display**: When the program starts, call `SetConsoleOutputCP(65001)` to switch the console to UTF-8 encoding. It is recommended to use **Windows Terminal** for running, as some fonts in older versions of cmd.exe may display abnormally.

1.

3. **Model File Integrity**: The ONNX model and dictionary files in the `inference/` directory must be complete; any missing file will cause the corresponding model initialization to fail.

1.

4. **Memory Usage**: The Server model (v5 server) file is relatively large (approximately 170MB), resulting in a higher memory usage during loading. When experiencing memory shortages, it is recommended to use the Tiny or Small models.

1.

5. **CPU Inference Performance**: Default inference is used for CPU, and the recognition time of large-size images is relatively long. Performance can be adjusted by modifying parameters such as `limit_side_len`, `rec_batch_num`, and `predictor_num` in `main.cpp`.

1.

6. **DLL Loading Failure Troubleshooting**: If the "Cannot load PPOCR.dll" error is encountered when starting, please confirm:

-

• `PPOCR.dll` has been correctly copied to the `build\Release\` directory

-

• `opencv_world500.dll` and `PPOCR.dll` are in the same directory

-

• There are no conflicting old versions of OpenCV DLL in the system PATH.

---

### Folder Structure

```
FIRC_CPP/
├── CMakeLists.txt  # CMake build configuration
├── src/
├── main.cpp  # Control console main program entry point
├── ocr_engine.h  # OCREngine wrapper class declaration
├── ocr_engine.cpp  # OCREngine implementation (dynamically loading PPOCR.dll)
└── json_parser.h # Minimal JSON Parser (Parses OCR Results)
├── lib/ # Runtime dependencies (needs to be manually prepared)
├── PPOCR.dll # OCR inference core DLL
└── inference/ # ONNX model and dictionary files
│       ├── PP-OCRv5_mobile_det_onnx.onnx
│       ├── PP-OCRv5_mobile_rec_onnx.onnx
│       ├── PP-OCRv5_mobile_cls_onnx.onnx
│       ├── PP-OCRv5_server_det_infer.onnx
│       ├── PP-OCRv5_server_rec_infer.onnx
│       ├── PP-OCRv6_small_det.onnx
│       ├── PP-OCRv6_small_rec.onnx
│       ├── PP-OCRv6_small_rec_dict.txt
│       ├── PP-OCRv6_tiny_det.onnx
│       ├── PP-OCRv6_tiny_rec.onnx
│       ├── PP-OCRv6_tiny_rec_dict.txt
│       └── ppocrv5_dict.txt
└── build/ # CMake build output (automatically generated)
    └── Release/
├── FIRC_CPP.exe # Compile and output an executable file
├── PPOCR.dll # Automatically copied from lib/
├── opencv_world500.dll # Automatically copied from OpenCV bin directory
└── inference/ # Automatically copy from lib/inference/
```


## Payment

Here is a pay link on Stripe ( https://buy.stripe.com/3cs8yP7sY87d0vu9AB ). Please contact me lonlonago@foxmail.com after funding $89, and I will send you a complete data files , thank you!

![111.png](111.png)
