# reCAPTCHA Demo Notebook Documentation

## Overview
This notebook demonstrates an automated solution for solving reCAPTCHA challenges using Selenium, CLIP model for image recognition, and prompt engineering. It uses a local CLIP model to analyze CAPTCHA images and make intelligent selections.

## Cell 1: Installing Required Libraries
```python
!pip install transformers
!pip install torch
!pip install pillow 
!pip install numpy 
!pip install requests 
!pip install undetected-chromedriver
```
Installs necessary libraries for machine learning (transformers, torch), image processing (pillow, numpy), HTTP requests, and undetected Chrome driver for Selenium.

## Cell 2: Importing Libraries and Modules
```python
import time
import os
import shutil
import torch
from PIL import Image
from transformers import CLIPProcessor, CLIPModel
import undetected_chromedriver as uc
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException
```
Imports standard libraries for time handling, file operations, and machine learning, along with Selenium components for web automation.

## Cell 3: Configuration Setup
```python
MODEL_ID = "openai/clip-vit-base-patch16" 
LOCAL_MODEL_PATH = "./local_clip_model_optimized"
BASE_DATA_FOLDER = "image_data"
```
Defines constants for the CLIP model ID, local model path, and base folder for storing image data during CAPTCHA solving.

## Cell 4: CAPTCHA Configuration Dictionary
```python
CAPTCHA_CONFIG = {
    "crosswalk": {
        "pos": ["a pedestrian crosswalk", "white zebra crossing lines", "crosswalk markings on road"],
        "neg": ["plain asphalt road", "sidewalk", "grass", "white line on side of road"],
        "thresh": 0.55
    },
    # ... (other configurations)
}
```
Defines prompt engineering configurations for different CAPTCHA object types, including positive and negative prompts, and confidence thresholds for each category.

## Cell 5: Loading the CLIP Model
```python
if os.path.exists(LOCAL_MODEL_PATH):
    print(f"Loading optimized model from {LOCAL_MODEL_PATH}...")
    model = CLIPModel.from_pretrained(LOCAL_MODEL_PATH)
    processor = CLIPProcessor.from_pretrained(LOCAL_MODEL_PATH)
else:
    print(f"Downloading optimized model ({MODEL_ID})...")
    model = CLIPModel.from_pretrained(MODEL_ID)
    processor = CLIPProcessor.from_pretrained(MODEL_ID)
    model.save_pretrained(LOCAL_MODEL_PATH)
    processor.save_pretrained(LOCAL_MODEL_PATH)
```
Loads the CLIP model and processor from local storage if available, otherwise downloads and saves them locally for future use.

## Cell 6: Data Folder Cleanup Function
```python
def clean_data_folder():
    if os.path.exists(BASE_DATA_FOLDER):
        shutil.rmtree(BASE_DATA_FOLDER)
    os.makedirs(BASE_DATA_FOLDER)
```
Defines a function to remove and recreate the base data folder for storing CAPTCHA images during each solving session.

## Cell 7: Round Folder Creation Function
```python
def create_round_folder(round_num):
    folder = os.path.join(BASE_DATA_FOLDER, f"round_{round_num}")
    if not os.path.exists(folder):
        os.makedirs(folder)
    return folder
```
Creates a subfolder within the base data folder for each round of CAPTCHA solving to organize images.

## Cell 8: Smart Prediction Function
```python
def get_smart_prediction(image, target_name, tile_index):
    # 1. Select Config
    config = CAPTCHA_CONFIG.get(target_name, CAPTCHA_CONFIG["default"])
    
    # 2. Build Prompts
    if target_name not in CAPTCHA_CONFIG:
        positive_prompts = [f"a photo of a {target_name}"]
    else:
        positive_prompts = config["pos"]
        
    negative_prompts = config["neg"]
    all_prompts = positive_prompts + negative_prompts
    
    # 3. Process
    inputs = processor(text=all_prompts, images=image, return_tensors="pt", padding=True)
    
    with torch.no_grad():
        outputs = model(**inputs)
    
    # 4. Calculate Scores
    probs = outputs.logits_per_image.softmax(dim=1)[0]
    
    pos_score = sum(probs[:len(positive_prompts)]).item()
    
    threshold = config["thresh"]
    
    return pos_score > threshold
```
Implements intelligent image classification using CLIP model with prompt engineering to determine if a CAPTCHA tile matches the target object.

## Cell 9: Main CAPTCHA Solving Function
```python
def solve_optimized():
    clean_data_folder()
    
    driver = uc.Chrome()
    driver.get("https://patrickhlauke.github.io/recaptcha/")
    wait = WebDriverWait(driver, 10)
    
    try:
        # --- OPEN CAPTCHA ---
        print("Clicking checkbox...")
        frames = driver.find_elements(By.XPATH, "//iframe[contains(@src, 'recaptcha/api2/anchor')]")
        driver.switch_to.frame(frames[0])
        driver.find_element(By.ID, "recaptcha-anchor").click()
        driver.switch_to.default_content()
        time.sleep(4)
        
        round_count = 1
        
        while True:
            print(f"\n--- ROUND {round_count} ---")
            
            # 1. Check if Solved
            try:
                challenge_frame = wait.until(EC.presence_of_element_located((By.XPATH, "//iframe[contains(@title, 'recaptcha challenge')]")))
                driver.switch_to.frame(challenge_frame)
            except TimeoutException:
                print(">>> SOLVED! <<<")
                break
            
            # 2. Get Prompt
            try:
                prompt_text = wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, "strong"))).text
                
                clean_target = prompt_text.lower()
                # Edge case naming fixes
                if "traffic light" in clean_target: clean_target = "traffic light"
                elif "fire" in clean_target: clean_target = "fire hydrant"
                # ... (other fixes)
                
                print(f"Goal: {clean_target}")
                
                # 3. Detect Grid
                tiles = driver.find_elements(By.CSS_SELECTOR, "td.rc-imageselect-tile")
                is_4x4 = (len(tiles) == 16)
                grid_dim = 4 if is_4x4 else 3
                print(f"Detected {grid_dim}x{grid_dim} Grid.")
                
            except Exception:
                print("Error reading prompt. Exiting.")
                break
            
            # 4. Capture
            current_folder = create_round_folder(round_count)
            img_wrapper = driver.find_element(By.ID, "rc-imageselect-target")
            img_wrapper.screenshot(os.path.join(current_folder, "full_grid.png"))
            full_image = Image.open(os.path.join(current_folder, "full_grid.png"))
            
            width, height = full_image.size
            tile_w = width // grid_dim
            tile_h = height // grid_dim
            
            matches = []
            
            # 5. Analyze with Smart Logic
            for i in range(len(tiles)):
                row, col = i // grid_dim, i % grid_dim
                tile_img = full_image.crop((col*tile_w, row*tile_h, (col+1)*tile_w, (row+1)*tile_h))
                tile_img.save(os.path.join(current_folder, f"tile_{i+1}.png"))
                
                if get_smart_prediction(tile_img, clean_target, i):
                    matches.append(tiles[i])
            
            # 6. Click & Verify
            print(f"Clicking {len(matches)} tiles...")
            for tile in matches:
                tile.click()
                time.sleep(0.05)
            
            print("Clicking Verify/Next...")
            driver.find_element(By.ID, "recaptcha-verify-button").click()
            
            driver.switch_to.default_content()
            time.sleep(3) 
            round_count += 1
            
            if round_count > 15:
                print("Safety Break.")
                break
    
    except Exception as e:
        print(f"Error: {e}")
    finally:
        driver.quit()
```
Implements the complete automated CAPTCHA solving workflow, including browser setup, challenge detection, image analysis, and interaction simulation.

## Cell 10: Main Execution
```python
if __name__ == "__main__":
    solve_optimized()
```
Executes the CAPTCHA solving function when the script is run directly.
