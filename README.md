# A Deployment-Oriented Intelligent System for PCB Defect Identification and Localization Using Mobile–Cloud Computing

**Testing Dataset**

This repository contains the **testing image set** used to evaluate a Mobile–Cloud Computing (MCC) based intelligent inspection system for Printed Circuit Boards (PCBs).

The system allows a user to submit a PCB image from a mobile application. The image is transferred to a cloud backend, where an AI model performs inference and returns three things:

1. **Identification** – whether the board contains a defect,
2. **Classification** – which type of defect it is, and
3. **Localization** – where on the board the defect appears.

The result is sent back to the mobile application and presented to the user. The heavy computation (model inference) runs in the cloud, so the mobile device only handles image capture, upload, and result display.

---

## Table of Contents

1. [About This Testing Dataset](#1-about-this-testing-dataset)
2. [System Overview](#2-system-overview)
3. [Dataset Structure](#3-dataset-structure)
4. [Defect Categories](#4-defect-categories)
5. [Image Information](#5-image-information)
6. [How to Test the Dataset](#6-how-to-test-the-dataset)
7. [Testing Through the Mobile Application](#7-testing-through-the-mobile-application)
8. [Expected Output](#8-expected-output)
9. [Example Testing Workflow](#9-example-testing-workflow)
10. [Project Architecture](#10-project-architecture)
11. [Information To Be Completed](#11-information-to-be-completed)
12. [License](#12-license)

---

## 1. About This Testing Dataset

### What this dataset is

This is a small, curated collection of **16 PCB images** drawn from two well-known public PCB inspection benchmarks:

| Source folder | Images | Type of imagery |
|---|---|---|
| `DeepPCB/` | 12 | Binarized (black-and-white) PCB trace images, aligned and cropped |
| `HRIPCB/` | 4 | High-resolution colour photographs of a bare PCB |

The two folders deliberately represent **two very different imaging conditions**. `DeepPCB` images are small, pre-processed, single-channel images of circuit traces. `HRIPCB` images are large, full-colour photographs of a complete board captured with an industrial camera. Testing on both makes it possible to check whether the system behaves consistently across image scales, colour formats, and file sizes.

### Why it is provided

The dataset is provided so that researchers, developers, and reviewers can:

- Reproduce and verify the behaviour of the deployed system without needing access to the original training data.
- Exercise the complete end-to-end pipeline (mobile capture → upload → cloud inference → result display) with realistic inputs.
- Check how the system handles both small pre-processed images (tens of kilobytes) and large high-resolution images (over 1 MB), which is relevant for upload time, network usage, and cloud response latency in a Mobile–Cloud setting.
- Perform qualitative inspection of defect identification, classification, and localization output.

### How it should be used

These images are intended for **testing and inference only, not for model training.**

Two observations from the dataset itself support this:

- The folders contain **no annotation or label files** of any kind (no `.txt`, `.xml`, `.json`, or `.csv` files). Only image files are present, so there is no ground truth in a form a training pipeline could consume.
- The `DeepPCB` images all carry the `_test` suffix, which in the original DeepPCB convention marks the test image of a test/template pair. The corresponding defect-free template images are **not** included here.

For the `HRIPCB` images, the expected defect type is encoded in the filename (see [Defect Categories](#4-defect-categories)), so those images can be used as a light-weight qualitative check: the class predicted by the system can be compared against the class named in the filename. No bounding-box ground truth is supplied, so localization accuracy has to be judged visually rather than computed numerically from this dataset.

---

## 2. System Overview

The complete inspection workflow is as follows:

```
        PCB Image
            |
            v
    Mobile Application          (capture or select an image)
            |
            v
       Image Upload             (image sent over the network)
            |
            v
     Cloud Processing           (receive, validate, pre-process)
            |
            v
     AI Model Inference         (model runs on cloud compute)
            |
            v
   Defect Identification        (is there a defect?)
            |
            v
   Defect Classification        (which type of defect?)
            |
            v
    Defect Localization         (where is the defect?)
            |
            v
    Inspection Result           (returned to the mobile app)
```

### The role of Mobile–Cloud Computing

PCB defect detection models are computationally demanding, and PCB images — as the `HRIPCB` folder in this dataset shows — can be large. Running such a model directly on a phone would be slow, would drain the battery, and would make the app difficult to update.

In the Mobile–Cloud Computing design used here, the work is split:

| Runs on the mobile device | Runs in the cloud |
|---|---|
| Capturing or selecting the PCB image | Receiving and validating the uploaded image |
| Basic client-side handling of the image | Pre-processing (resizing, format conversion, normalization) |
| Uploading the image to the backend | AI model inference |
| Displaying the returned result | Defect identification, classification, and localization |
| Showing past inspection records | Storing images and results |

This gives three practical benefits:

- **Lightweight client.** The mobile app stays small and runs on ordinary devices, because it never loads the model.
- **Centralized model management.** The model can be retrained, replaced, or upgraded in the cloud without shipping a new app version to users.
- **Scalability.** Multiple users and multiple inspections can be served concurrently by cloud resources.

The trade-off is a dependency on network connectivity, which is why upload size matters and why this dataset intentionally contains images at two very different sizes.

---

## 3. Dataset Structure

The dataset has a flat, two-folder structure. Both folders contain image files only.

```
Testing Dataset/
├── DeepPCB/
│   ├── 00041000_test.jpg
│   ├── 00041001_test.jpg
│   ├── 00041002_test.jpg
│   ├── 00041003_test.jpg
│   ├── 12000001_test.jpg
│   ├── 12000017_test.jpg
│   ├── 12000038_test.jpg
│   ├── 12000052_test.jpg
│   ├── 12100000_test.jpg
│   ├── 12100001_test.jpg
│   ├── 12100002_test.jpg
│   └── 12100003_test.jpg
├── HRIPCB/
│   ├── 01_missing_hole_01.jpg
│   ├── 01_missing_hole_02.jpg
│   ├── 01_missing_hole_03.jpg
│   └── 01_missing_hole_04.jpg
└── README.md
```

**Notes on the structure**

- There are **no sub-folders inside `DeepPCB/` or `HRIPCB/`**. The images sit directly in each folder.
- There are **no label, annotation, or metadata files** anywhere in the dataset.
- The folders are **not** organised into per-class sub-directories. In `HRIPCB/`, the defect class is carried by the filename instead.
- Total: 16 images, approximately 6 MB (`DeepPCB/` ≈ 468 KB, `HRIPCB/` ≈ 5.5 MB).

---

## 4. Defect Categories

### Categories confirmed in `HRIPCB/`

The `HRIPCB` filenames explicitly name the defect type. **One category is present in this testing set**, represented by four images:

| Defect category | Filename token | Images | What the defect represents |
|---|---|---|---|
| **Missing hole** | `missing_hole` | 4 | A drilled hole (a via or a component mounting hole) that should exist on the board is absent. Components cannot be mounted correctly and intended connections between layers may not be formed. |

> **Note:** The full HRIPCB benchmark defines additional defect classes beyond this one. Only *missing hole* appears in this testing subset. If the deployed model supports further classes, they are simply not exercised by these particular images, and testing against those classes would require additional images from the source benchmark.

### Categories in `DeepPCB/`

The `DeepPCB` filenames contain only a numeric identifier and the `_test` suffix — they do **not** encode a defect type — and no annotation files are provided. Therefore:

- **The defect category of each individual `DeepPCB` image cannot be confirmed from the contents of this dataset.**
- These images should be treated as unlabelled test inputs. They are useful for observing how the system responds to binarized trace imagery, but the prediction cannot be scored against ground truth using only what is included here.

Defect classes supported by the deployed model: `[Add information here]`

---

## 5. Image Information

### Summary

| Property | `DeepPCB/` | `HRIPCB/` |
|---|---|---|
| File format | JPEG (`.jpg`) | JPEG (`.jpg`) |
| Number of images | 12 | 4 |
| Resolution | 640 × 640 pixels (all images) | 3034 × 1586 pixels (all images) |
| Colour | Single-channel grayscale, except `00041003_test.jpg`, which is stored as 3-channel | 3-channel RGB colour |
| Visual content | Binarized black-and-white circuit traces | Photograph of a bare green PCB with copper tracks, pads, and drilled holes |
| File size range | ≈ 16 KB – 74 KB | ≈ 1.42 MB per image (uniform) |
| Folder size | ≈ 468 KB | ≈ 5.5 MB |
| Colour depth | 8-bit precision | 8-bit precision |
| Camera metadata | None (JFIF, 96 × 96 DPI density) | EXIF present; captured with an OSEE H1600 camera |
| Annotation files | None | None |

### Naming convention

**`DeepPCB/` — `<8-digit image ID>_test.jpg`**

```
00041000_test.jpg
└──┬───┘ └─┬─┘
   │       └── "test" suffix: this is the inspected (test) image
   └────────── 8-digit image identifier
```

The identifiers fall into three numeric groups — `00041xxx`, `12000xxx`, and `12100xxx` — corresponding to different board series in the source benchmark. In the original DeepPCB convention each `_test` image is paired with a defect-free `_temp` (template) image; **the template images are not included in this testing set**, so any comparison-based inspection method would need them supplied separately.

**`HRIPCB/` — `<board ID>_<defect type>_<sample number>.jpg`**

```
01_missing_hole_01.jpg
└┬┘ └─────┬────┘ └┬┘
 │        │       └── sample number for that board/defect combination (01–04)
 │        └────────── defect type (missing_hole in this testing set)
 └─────────────────── board identifier (all images here are board 01)
```

All four `HRIPCB` images come from the same board (`01`) and show the same defect type (*missing hole*), as four separate samples numbered `01` to `04`.

### Per-file listing

**`DeepPCB/` — 640 × 640 px**

| Filename | Size |
|---|---|
| `00041000_test.jpg` | 17.0 KB |
| `00041001_test.jpg` | 18.7 KB |
| `00041002_test.jpg` | 15.9 KB |
| `00041003_test.jpg` | 25.2 KB |
| `12000001_test.jpg` | 40.9 KB |
| `12000017_test.jpg` | 63.5 KB |
| `12000038_test.jpg` | 74.0 KB |
| `12000052_test.jpg` | 68.1 KB |
| `12100000_test.jpg` | 28.2 KB |
| `12100001_test.jpg` | 34.1 KB |
| `12100002_test.jpg` | 34.5 KB |
| `12100003_test.jpg` | 30.3 KB |

**`HRIPCB/` — 3034 × 1586 px**

| Filename | Size | Expected defect (from filename) |
|---|---|---|
| `01_missing_hole_01.jpg` | 1.42 MB | Missing hole |
| `01_missing_hole_02.jpg` | 1.42 MB | Missing hole |
| `01_missing_hole_03.jpg` | 1.42 MB | Missing hole |
| `01_missing_hole_04.jpg` | 1.42 MB | Missing hole |

### Annotation information

**No annotation files are included in this dataset.** There are no bounding-box coordinate files, no segmentation masks, and no class-index files.

The only ground-truth signal available is the defect type embedded in the `HRIPCB` filenames. Consequently:

- **Classification** can be qualitatively verified for `HRIPCB` images by comparing the predicted class with the filename.
- **Localization** can only be verified visually, by looking at where the system draws the detected region on the returned image.
- Quantitative metrics such as mAP or IoU **cannot** be computed from this dataset alone; they require the annotated evaluation split of the source benchmarks.

---

## 6. How to Test the Dataset

The steps below describe how to run one image through the full system. Repeat for as many images as needed.

### Before you start

- Install the mobile application on a device or emulator: `[Add installation instructions here]`
- Make sure the cloud backend is running and reachable: `[Add backend URL / deployment information here]`
- Copy the testing images onto the device (or onto the emulator's storage) so they can be selected from the gallery. Alternatively, display an image on a monitor and photograph it with the app's camera.
- Confirm the device has a working network connection — the system depends on it, because inference happens in the cloud.

### Step-by-step procedure

**Step 1 — Select a testing PCB image**

Pick one image from either folder. A good starting point is `HRIPCB/01_missing_hole_01.jpg`, because the expected defect type is known from its filename.

If you want to test the system's tolerance to different input scales, run one image from each folder — a 640 × 640 binarized image from `DeepPCB/` and a 3034 × 1586 colour photograph from `HRIPCB/`.

**Step 2 — Upload or capture the image in the mobile application**

Open the app and provide the image using one of the supported input methods:

- Choose the file from the device gallery/storage, or
- Point the device camera at a physical PCB (or at the displayed test image) and capture it.

The app should show a preview of the selected image before submission.

**Step 3 — Send the image to the cloud backend**

Confirm the submission in the app. The image is uploaded over the network to the cloud service, which receives it and prepares it for the model.

Expected backend request: `[Add API endpoint and request format here]`

**Step 4 — Cloud pre-processing and model inference**

On the server side the image is pre-processed (for example resized to the model's input resolution, converted to the expected colour format, and normalized) and then passed to the AI model.

- Model used for inference: `[Add model name/version here]`
- Input resolution expected by the model: `[Add information here]`

Nothing is required from the user at this step; the app typically shows a progress indicator while waiting.

**Step 5 — Receive the prediction**

The cloud service returns the inference result to the mobile application as a structured response.

Response format: `[Add response schema here]`

**Step 6 — View the detected defect**

The app displays whether a defect was found on the submitted board. If the model finds nothing, the board is reported as defect-free.

**Step 7 — View the defect class**

For each detection, the predicted defect category is shown. For every `HRIPCB` image in this testing set the expected category is *missing hole*.

Compare the predicted class against the defect type in the filename. For example, `01_missing_hole_03.jpg` should be classified as a missing hole.

**Step 8 — View the localized defect region**

The app shows where on the board the defect was found, typically by overlaying a marked region or bounding box on the image.

Check that the marked region actually sits on a plausible defect. For the `HRIPCB` images this means a pad that lacks its drilled hole, while the surrounding pads are correctly drilled. Because no ground-truth coordinates are supplied with this dataset, this check is visual.

**Step 9 — Interpret the final inspection result**

Read the result as a whole:

| What you see | How to interpret it |
|---|---|
| No detections returned | The system considers the board free of the defects it was trained to find. |
| One or more detections with a class label | The system identified a defect and assigned it to that category. |
| A marked region on the image | The system's estimate of where the defect is located on the board. |
| A confidence value (if provided) | How certain the model is about that detection. Low values indicate an uncertain prediction that deserves manual review. |

Remember that for `DeepPCB` images there is no ground truth in this dataset, so those results can only be assessed by visual judgement.

---

## 7. Testing Through the Mobile Application

The dataset is designed to be consumed through the developed mobile application, which is the intended entry point of the deployed system.

```
   Open Application
          |
          v
 Select / Capture PCB Image      (from gallery, or with the camera)
          |
          v
  Submit for Inspection          (image uploaded to the cloud)
          |
          v
   Cloud Processing              (validation + pre-processing)
          |
          v
     AI Inference                (identification, classification, localization)
          |
          v
     View Result                 (annotated image + defect details)
```

### Practical tips for testing with these images

- **Loading the images onto a device.** Transfer the `DeepPCB/` and `HRIPCB/` folders to the device's internal storage so they appear in the gallery picker. On an Android emulator, images can be dragged onto the emulator window or pushed with `adb push`.
- **Testing upload behaviour.** The `HRIPCB` images are around 1.42 MB each while `DeepPCB` images are under 75 KB. Submitting one of each is a simple way to observe how upload time and end-to-end latency change with image size — a directly relevant property of a Mobile–Cloud design.
- **Testing the camera path.** To exercise the capture flow rather than the gallery flow, display an image full-screen on a monitor and photograph it with the app. Note that this introduces glare, perspective distortion, and re-compression, so results may differ from uploading the original file.
- **Repeatability.** Submitting the same image twice should produce the same prediction. This is a quick way to confirm the backend and model are behaving deterministically.

Application screen names and in-app navigation steps: `[Add the actual screen names and steps from the mobile application here]`

---

## 8. Expected Output

The specific fields returned by this deployment must be filled in from the project implementation. The list below describes the output categories the system is designed to produce, as stated in the project description.

| Output | Description | Status |
|---|---|---|
| Defect / No Defect | Whether any defect was identified on the submitted board | Described by the project |
| Defect class | The category assigned to each detected defect | Described by the project |
| Defect location | Where on the board the defect was found | Described by the project |
| Localized region | The marked region or bounding box drawn on the image | Described by the project |
| Confidence score | The model's certainty for each detection | `[Confirm whether this is returned]` |
| Processed image | The returned image with detections overlaid | `[Confirm whether this is returned]` |
| Inspection history | Stored record of past inspections viewable in the app | `[Confirm whether this is implemented]` |

- Exact response fields, data types, and units: `[Add the actual API response schema here]`
- Example of a real response from this system: `[Add a real sample response here]`

> This section intentionally does not show an invented example response. Replace the placeholders above with the actual output produced by the deployed backend, so that testers can compare their results against the real format.

---

## 9. Example Testing Workflow

The following walkthrough uses a single image from this dataset. It shows the path an image takes through the system; values that depend on the deployment are marked as placeholders.

- **Input image:** `HRIPCB/01_missing_hole_01.jpg`
- **Image properties:** JPEG, 3034 × 1586 px, RGB, ≈ 1.42 MB
- **Expected defect type (from the filename):** missing hole

```
  Input PCB Image
  HRIPCB/01_missing_hole_01.jpg  (3034 x 1586, RGB, ~1.42 MB)
            |
            v
  Upload
  Image selected in the mobile app and submitted to the cloud backend
            |
            v
  Preprocessing
  Cloud service resizes / normalizes the image for the model
  Target input size: [Add information here]
            |
            v
  Model Inference
  Model: [Add model name/version here]
            |
            v
  Defect Detection
  Result: defect found / not found
            |
            v
  Classification
  Predicted class: [Add actual predicted class here]
  Expected class from filename: missing hole
            |
            v
  Localization
  Marked region drawn over the affected area of the board
  Coordinates: [Add actual output here]
            |
            v
  Final Result
  Annotated image and defect details shown in the mobile application
```

### How to judge the outcome

1. **Classification check.** The predicted class should read as *missing hole*. If the system returns a different class, record the image name and the returned class.
2. **Localization check.** Open the returned annotated image and confirm the marked region sits on a pad or hole location that visibly differs from the surrounding, correctly drilled pads.
3. **Cross-format check.** Repeat the same procedure with `DeepPCB/00041000_test.jpg` (640 × 640, binarized, ≈ 17 KB) to confirm the system also accepts small single-channel images. Note that the expected class for this image is unknown, so only the system's ability to process the input can be verified.

Reference results and annotated example outputs: `[Add example result images here, if available]`

---

## 10. Project Architecture

The architecture below reflects the system as described in the project. Component names should be confirmed against the actual deployment.

```
        Mobile Application
   (image capture / selection, result display)
                 |
                 v
             FastAPI
   (cloud API: receives the uploaded image)
                 |
                 v
       Image Preprocessing
   (resize, format conversion, normalization)
                 |
                 v
        Cloud AI / Vertex AI
   (model serving and inference)
                 |
                 v
  Defect Detection & Classification
   (is there a defect, and of what type)
                 |
                 v
       Defect Localization
   (region / bounding box on the board)
                 |
                 v
    Firestore / Cloud Storage
   (result records and image storage)
                 |
                 v
        Inspection Result
   (structured response)
                 |
                 v
        Mobile Application
   (annotated image and defect details shown to the user)
```

| Layer | Responsibility | Details to confirm |
|---|---|---|
| Mobile Application | Image capture/selection, submission, result display | `[Add platform and framework here]` |
| API layer (FastAPI) | Receives uploads, orchestrates the pipeline, returns results | `[Add endpoint list and deployment target here]` |
| Pre-processing | Prepares the image for the model | `[Add pre-processing steps and parameters here]` |
| Model serving | Runs inference | `[Add model name, version, and serving platform here]` |
| Storage | Persists images and inspection records | `[Add storage/database configuration here]` |

---

## 11. Information To Be Completed

The following details could not be determined from the contents of this dataset repository and should be filled in by the project author before publication:

- Mobile application installation and setup instructions
- Backend/cloud service URL and API endpoint specification
- Request and response formats
- Model name, version, and expected input resolution
- The complete list of defect classes supported by the deployed model
- Whether confidence scores, annotated images, and inspection history are returned
- Application screen names and in-app navigation steps
- Example results for the images in this dataset
- Citation details for the source benchmarks the images were drawn from

---

## 12. License

License information will be added here.
