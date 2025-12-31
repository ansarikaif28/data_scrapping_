# reCAPTCHA Solver Demo

[![Python](https://img.shields.io/badge/Python-3.7%2B-blue.svg)](https://www.python.org/)
[![Selenium](https://img.shields.io/badge/Selenium-4.0%2B-green.svg)](https://www.selenium.dev/)
[![Transformers](https://img.shields.io/badge/Transformers-4.0%2B-orange.svg)](https://huggingface.co/docs/transformers/index)

An automated reCAPTCHA solver using machine learning and web automation. This demo showcases how to bypass reCAPTCHA challenges by leveraging the CLIP (Contrastive Language-Image Pretraining) model for image recognition and Selenium for browser interaction.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Setup](#setup)
- [Usage](#usage)
- [How It Works](#how-it-works)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Disclaimer](#disclaimer)
- [Contributing](#contributing)
- [License](#license)

## Features

- **Automated reCAPTCHA Solving**: Intelligently solves image-based reCAPTCHA challenges
- **Machine Learning Powered**: Uses CLIP model for accurate image classification
- **Prompt Engineering**: Custom prompts for different object types (traffic lights, crosswalks, etc.)
- **Browser Automation**: Selenium-based interaction with undetected Chrome driver
- **Multi-round Support**: Handles multiple rounds of CAPTCHA challenges
- **Local Model Caching**: Downloads and caches CLIP model locally for faster subsequent runs
- **Grid Detection**: Automatically detects 3x3 or 4x4 CAPTCHA grids

## Prerequisites

- **Python**: 3.7 or higher
- **Operating System**: Windows, macOS, or Linux
- **RAM**: At least 4GB (8GB recommended for optimal performance)
- **Storage**: ~2GB free space for model and temporary files
- **Internet Connection**: Required for initial model download and CAPTCHA solving

## Installation

### Step 1: Clone or Download the Repository

Ensure you have the `recapcha_demo.ipynb` file in your working directory.

### Step 2: Install Python Dependencies

Run the following commands in your terminal or Jupyter notebook cell:

```bash
pip install transformers torch pillow numpy requests undetected-chromedriver selenium
```

Or execute in a Jupyter cell:

```python
!pip install transformers
!pip install torch
!pip install pillow
!pip install numpy
!pip install requests
!pip install undetected-chromedriver
```

### Step 3: Install Chrome Browser

Ensure Google Chrome is installed on your system. The script uses Chrome for automation.

- Download from: https://www.google.com/chrome/

## Setup

### Step 1: Prepare the Environment

1. Open the `recapcha_demo.ipynb` file in Jupyter Notebook or JupyterLab.

2. Ensure you're in the correct working directory:

```python
import os
print(os.getcwd())  # Should show the path containing recapcha_demo.ipynb
```

### Step 2: Configure Model Path (Optional)

By default, the model will be downloaded to `./local_clip_model_optimized`. You can change this in the configuration cell:

```python
LOCAL_MODEL_PATH = "./local_clip_model_optimized"  # Modify this path if needed
```

### Step 3: Set Up Data Folder

The script automatically creates an `image_data` folder for temporary CAPTCHA images. No manual setup required.

### Step 4: Run Initial Setup Cells

Execute the first few cells in order:

1. **Installations Cell**: Install required libraries
2. **Imports Cell**: Import all necessary modules
3. **Configuration Cell**: Set model and data paths
4. **CAPTCHA_CONFIG Cell**: Review prompt configurations

## Usage

### Step 1: Load the Model

Execute the model loading cell. This will either load from local cache or download the CLIP model:

```python
# This cell loads the CLIP model
if os.path.exists(LOCAL_MODEL_PATH):
    print(f"Loading optimized model from {LOCAL_MODEL_PATH}...")
    # ... model loading code
```

### Step 2: Run the Solver

Execute the main function:

```python
if __name__ == "__main__":
    solve_optimized()
```

The script will:

1. Launch an undetected Chrome browser
2. Navigate to the reCAPTCHA demo page
3. Attempt to solve the CAPTCHA automatically
4. Report progress and results

### Step 3: Monitor Progress

Watch the console output for:

- Model loading status
- CAPTCHA detection and grid size
- Tile analysis results
- Success/failure messages

### Step 4: Review Results

After completion, check the `image_data` folder for:

- Screenshots of CAPTCHA grids
- Individual tile images
- Analysis logs

## How It Works

### 1. Browser Automation Setup

- Launches an undetected Chrome browser using Selenium
- Navigates to a reCAPTCHA demo page
- Interacts with the reCAPTCHA iframe

### 2. Challenge Detection

- Detects when a reCAPTCHA challenge is presented
- Identifies the object type (e.g., "Select all images with traffic lights")
- Determines grid size (3x3 or 4x4)

### 3. Image Capture

- Takes a screenshot of the entire CAPTCHA grid
- Splits the image into individual tiles
- Saves tiles for analysis

### 4. AI-Powered Analysis

- Uses CLIP model to analyze each tile
- Compares tile content against positive and negative prompts
- Calculates confidence scores for each object type

### 5. Intelligent Selection

- Applies threshold-based filtering to identify matching tiles
- Uses prompt engineering for improved accuracy:
  - Positive prompts: Descriptions of target objects
  - Negative prompts: Descriptions of non-target objects

### 6. Interaction Simulation

- Clicks identified tiles in sequence
- Submits the solution
- Handles multiple rounds if required

### 7. Success Verification

- Checks for CAPTCHA completion
- Reports success or continues to next round

## Configuration

### Adjusting Thresholds

Modify confidence thresholds in `CAPTCHA_CONFIG` for different object types:

```python
CAPTCHA_CONFIG = {
    "traffic light": {
        "pos": ["a traffic light", "traffic signal"],
        "neg": ["street light pole", "building window"],
        "thresh": 0.65  # Adjust this value (0.0-1.0)
    }
}
```

- Lower thresholds: More sensitive (may select incorrect tiles)
- Higher thresholds: More strict (may miss correct tiles)

### Custom Prompts

Add or modify prompts for new object types:

```python
"new_object": {
    "pos": ["description of object", "alternative description"],
    "neg": ["things that are not the object"],
    "thresh": 0.60
}
```

## Troubleshooting

### Common Issues

1. **Model Download Fails**
   - Check internet connection
   - Ensure sufficient disk space (~2GB)
   - Try running in a new session

2. **Chrome Launch Fails**
   - Update Chrome to latest version
   - Check Chrome installation path
   - Try running with administrator privileges

3. **Low Accuracy**
   - Adjust thresholds in CAPTCHA_CONFIG
   - Improve prompts for specific object types
   - Ensure good lighting/contrast in CAPTCHA images

4. **Script Hangs**
   - Check for network timeouts
   - Verify reCAPTCHA page is accessible
   - Restart kernel and try again

### Debug Mode

Enable debug prints by uncommenting debug lines in `get_smart_prediction()`:

```python
# print(f"   Tile {tile_index+1}: {int(pos_score*100)}% (Req: {int(threshold*100)}%)")
```

## Disclaimer

This project is for educational and research purposes only. Automated CAPTCHA solving may violate the terms of service of websites using reCAPTCHA. Use responsibly and only on sites you own or have permission to test.

reCAPTCHA is a trademark of Google LLC. This project is not affiliated with or endorsed by Google.

## Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

Areas for improvement:
- Support for more CAPTCHA types
- Improved prompt engineering
- Performance optimizations
- Additional model support

## License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**Note**: This is a demonstration of AI capabilities and should not be used for malicious purposes. Always respect website terms of service and robots.txt files.
