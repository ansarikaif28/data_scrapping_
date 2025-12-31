# Selenium Login Demo Notebook Documentation

## Overview
This notebook demonstrates automated web testing using Selenium WebDriver for login functionality. It includes positive login tests and negative test cases for invalid credentials, showcasing proper test automation practices with explicit waits and assertions.

## Cell 1: Installing Required Libraries
```python
!pip install selenium webdriver-manager
```
Installs Selenium for web automation and webdriver-manager for automatic ChromeDriver management.

## Cell 2: Importing Libraries and Modules
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.chrome.service import Service
import time
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
```
Imports Selenium components for browser control, element location strategies, and explicit waiting mechanisms.

## Cell 3: Configuring Chrome Options
```python
options = webdriver.ChromeOptions()
options.add_argument("--no-sandbox") 
options.add_argument("--disable-dev-shm-usage") 
options.add_argument("--disable-gpu") 
options.add_argument("--start-maximized")
```
Sets up Chrome browser options for headless or sandboxed execution, disabling GPU acceleration and maximizing the window.

## Cell 4: Launching the Chrome Driver
```python
try:
    print("Launching Chrome...")
    service = Service(ChromeDriverManager().install())
    driver = webdriver.Chrome(service=service, options=options)
    print("Chrome launched successfully!")
except Exception as e:
    print(f"❌ Failed to launch: {e}")
```
Attempts to launch a Chrome browser instance using the configured options and service, with error handling for launch failures.

## Cell 5: Positive Login Test Case
```python
login_url = "https://practicetestautomation.com/practice-test-login/"
print("\n RUNNING TEST CASE 1: POSITIVE LOGIN")

try:
    driver.get(login_url)
    wait = WebDriverWait(driver, 10)
    
    username_input = wait.until(EC.presence_of_element_located((By.ID, "username")))
    password_input = driver.find_element(By.ID, "password")
    
    username_input.clear()
    time.sleep(2)
    username_input.send_keys("student")
    
    password_input.clear()
    time.sleep(2)
    password_input.send_keys("Password123")
    
    submit_btn = driver.find_element(By.ID, "submit")
    time.sleep(2)
    submit_btn.click()
    
    wait.until(EC.url_contains("logged-in-successfully"))
    current_url = driver.current_url
    if "practicetestautomation.com/logged-in-successfully/" in current_url:
        print(" PASS: URL verified.")
    else:
        print(f" FAIL: URL did not match. Got: {current_url}")
    
    body_text = driver.find_element(By.TAG_NAME, "body").text
    if "Congratulations" in body_text or "successfully logged in" in body_text:
        print(" PASS: Success message found.")
    else:
        print(" FAIL: Success message NOT found.")
    
    logout_btn = driver.find_element(By.LINK_TEXT, "Log out")
    if logout_btn.is_displayed():
        print(" PASS: 'Log out' button is visible.")
    else:
        print(" FAIL: 'Log out' button hidden.")
    
    logout_btn.click()
    print(" Logged out to prepare for next test.")
    
except Exception as e:
    print(f" TEST CRASHED: {e}")
```
Executes a positive login test case, verifying successful authentication with valid credentials, checking URL changes, success messages, and logout functionality.

## Cell 6: Negative Username Test Case
```python
print("\n RUNNING TEST CASE 2: NEGATIVE USERNAME")

try:
    driver.get(login_url)
    
    wait.until(EC.presence_of_element_located((By.ID, "username"))).send_keys("incorrectUser")
    
    time.sleep(2)   
    driver.find_element(By.ID, "password").send_keys("Password123")
    
    time.sleep(2)
    driver.find_element(By.ID, "submit").click()
    
    error_element = wait.until(EC.visibility_of_element_located((By.ID, "error")))
    
    if error_element.is_displayed():
        print(" PASS: Error message element is visible.")
    else:
        print(" FAIL: Error message element not visible.")
        
    actual_error_text = error_element.text
    expected_text = "Your username is invalid!"
    
    if expected_text in actual_error_text:
        print(f" PASS: Error text matches '{expected_text}'.")
    else:
        print(f" FAIL: Expected '{expected_text}', got '{actual_error_text}'.")
        
except Exception as e:
    print(f" TEST CRASHED: {e}")
```
Tests negative login scenario with invalid username, verifying that appropriate error messages are displayed.

## Cell 7: Negative Password Test Case
```python
print("\n RUNNING TEST CASE 3: NEGATIVE PASSWORD ")

try:
    driver.get(login_url)
    
    time.sleep(10)
    wait.until(EC.presence_of_element_located((By.ID, "username"))).send_keys("student")
    
    time.sleep(10)
    driver.find_element(By.ID, "password").send_keys("incorrectPassword")
    
    time.sleep(10)
    driver.find_element(By.ID, "submit").click()
    
    error_element = wait.until(EC.visibility_of_element_located((By.ID, "error")))
    
    if error_element.is_displayed():
        print(" PASS: Error message element is visible.")
    else:
        print(" FAIL: Error message element not visible.")
        
    actual_error_text = error_element.text
    expected_text = "Your password is invalid!"
    
    if expected_text in actual_error_text:
        print(f"PASS: Error text matches '{expected_text}'.")
    else:
        print(f" FAIL: Expected '{expected_text}', got '{actual_error_text}'.")
        
except Exception as e:
    print(f" TEST CRASHED: {e}")
```
Tests negative login scenario with invalid password, ensuring error handling and messaging for authentication failures.
