# Car Number Plate Extraction System

This project implements the three-step number plate extraction pipeline from *Car Number Plate Extraction in Three Steps: Detection, Alignment, and OCR* by Gabriel Baziramwabo.

The live pipeline follows the book closely:

1. Capture frames from a webcam
2. Detect rectangular plate candidates with OpenCV contours
3. Align the best candidate with perspective correction
4. Extract characters with Tesseract OCR
5. Validate the text with a regex pattern
6. Confirm the plate over multiple frames
7. Save confirmed plates to a CSV file

## Project Structure

```text
plate-rec/
├── README.md
├── requirements.txt
├── data/
│   └── plates.csv
├── screenshots/
├── src/
│   ├── common.py
│   ├── camera.py
│   ├── detect.py
│   ├── align.py
│   ├── ocr.py
│   ├── validate.py
│   ├── temporal.py
│   └── main.py
├── Y3 Practical Assignment.pdf
└── CarNumberPlateExtractioninThreeSteps.pdf
```

## How the Pipeline Works

### 1. Detection

`src/detect.py` converts each frame to grayscale, applies Gaussian blur, runs Canny edge detection, and finds contours. It keeps only contours that look like number plates based on minimum area and aspect ratio.

### 2. Alignment

`src/align.py` selects the best plate candidate, orders the four corner points, and applies a perspective transform to warp the plate into a normalized `450 x 140` image.

### 3. OCR

`src/ocr.py` preprocesses the aligned plate using grayscale conversion, blur, and Otsu thresholding. The processed image is passed to Tesseract with a whitelist of uppercase letters and digits.

### 4. Validation

`src/validate.py` validates OCR output with this expected plate format:

```text
[A-Z]{3}[0-9]{3}[A-Z]
```

This matches the book's rule of three letters, three digits, and one final letter.

### 5. Temporal Confirmation

`src/temporal.py` keeps a short buffer of recent valid reads, chooses the most frequent value, and only logs the result once the plate is stable across frames.

### 6. CSV Logging

Confirmed plates are saved to `data/plates.csv` with a timestamp. Duplicate logging is reduced with a cooldown period.

## Installation

### 1. Create and activate a virtual environment

Windows PowerShell:

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

### 2. Install Python dependencies

```powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 3. Install Tesseract OCR

Install Tesseract separately because `pytesseract` is only the Python wrapper.

Windows:

- Download Tesseract from the UB Mannheim build:
[https://github.com/UB-Mannheim/tesseract/wiki](https://github.com/UB-Mannheim/tesseract/wiki)
- Make sure the Tesseract executable is available on your `PATH`

Verify the install:

```powershell
tesseract --version
```

## Running the Scripts

Run commands from the project root:

```powershell
python src/camera.py
python src/detect.py
python src/align.py
python src/ocr.py
python src/validate.py
python src/temporal.py
```

You can also run the full pipeline with:

```powershell
python src/main.py
```

If your webcam is not index `0`, pass a camera index:

```powershell
python src/main.py --camera-index 1
```

## Screenshots and Evidence

The scripts automatically save example images here when they first succeed:

- `screenshots/detection.png`
- `screenshots/alignment.png`
- `screenshots/ocr.png`

You should still test on real vehicles and keep the best screenshots for submission.

## Real-World Testing Checklist

- Ask the vehicle owner for permission before testing
- Explain that the test is for a school project
- Use only the plate for testing, not personal information
- Test on multiple vehicles in the school parking area
- Capture proof of detection, alignment, OCR, and final confirmation

## Output File

Confirmed plates are stored in:

```text
data/plates.csv
```

The CSV contains:

```text
Plate Number,Timestamp
```

## Notes

- This implementation follows the book's CPU-oriented classical OpenCV approach instead of using a deep learning detector.
- The regex pattern may need to be adjusted if your local test plates follow a different format.
- Good camera angle and lighting improve OCR accuracy significantly.

