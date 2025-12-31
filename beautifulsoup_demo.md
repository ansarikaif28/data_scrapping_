# BeautifulSoup Demo Notebook Documentation

## Overview
This notebook demonstrates web scraping using BeautifulSoup to extract population data from a Wikipedia page. It covers fetching HTML content, parsing it, extracting table data, cleaning it, and saving to a CSV file.

## Cell 1: Installing Required Libraries
```python
!pip install requests beautifulsoup4 pandas
```
This cell installs the necessary Python libraries: `requests` for HTTP requests, `beautifulsoup4` for HTML parsing, and `pandas` for data manipulation.

## Cell 2: Importing Libraries
```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import re
```
Imports the required modules for making web requests, parsing HTML, handling data frames, and using regular expressions for text cleaning.

## Cell 3: Defining the Target URL
```python
url = "https://en.wikipedia.org/wiki/List_of_countries_and_dependencies_by_population"
print(f"Target URL selected: {url}")
```
Sets the URL of the Wikipedia page containing the population data and prints a confirmation message.

## Cell 4: Setting Up Request Headers
```python
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"
}
```
Defines a User-Agent header to mimic a browser request, helping to avoid potential blocking by the server.

## Cell 5: Fetching the Webpage
```python
response = requests.get(url, headers=headers)
if response.status_code == 200:
    print("Success! Webpage fetched.")
    print(f"Server responded with status code: {response.status_code}")
    print("\n--- Raw HTML Snippet ---")
    print(response.text[:500])
else:
    print(f"Error: Failed to fetch page. Status code: {response.status_code}")
```
Makes a GET request to the URL with headers. Checks the response status and prints success or error messages, along with a snippet of the HTML content.

## Cell 6: Parsing HTML with BeautifulSoup
```python
soup = BeautifulSoup(response.text, 'html.parser')
print("HTML parsed into a BeautifulSoup object.")
print(f"Page Title: {soup.title.string}")
```
Creates a BeautifulSoup object from the response text using the HTML parser and prints the page title.

## Cell 7: Finding the Target Table
```python
table = soup.find('table', {'class': 'wikitable'})
if table:
    print("Table found.")
else:
    print("Table not found. Check the class name or URL.")
```
Locates the table with the class 'wikitable' on the page and confirms its existence.

## Cell 8: Extracting Table Headers
```python
headers = []
for th in table.find_all('th'):
    headers.append(th.text.strip())
print(f"\nHeaders detected: {headers}")
```
Iterates through table header elements, extracts their text, and stores them in a list.

## Cell 9: Extracting Table Data Rows
```python
table_data = []
rows = table.find_all('tr')
for row in rows[1:]:
    cells = row.find_all(['td', 'th'])
    row_data = [cell.text.strip() for cell in cells]
    if len(row_data) > 0:
        table_data.append(row_data)
print(f"Extracted {len(table_data)} rows of data.")
print("First row example:", table_data[0])
```
Collects data from table rows (skipping the header row), extracts text from cells, and stores in a list of lists.

## Cell 10: Creating a Pandas DataFrame
```python
df = pd.DataFrame(table_data)
if len(df.columns) == len(headers):
    df.columns = headers
else:
    df.columns = headers[:len(df.columns)]
print("--- Raw DataFrame Preview ---")
```
Creates a DataFrame from the extracted data and assigns column headers if possible.

## Cell 11: Dropping Unnecessary Columns
```python
df.drop(columns=['Source (official or fromthe United Nations)', 'Notes'], inplace=True)
display(df.head())
```
Removes specified columns from the DataFrame and displays the first few rows.

## Cell 12: Identifying the Country Column
```python
print(f"Current Columns: {df.columns.tolist()}")
country_col = df.columns[1]
```
Prints the current column names and selects the second column as the country column.

## Cell 13: Defining a Text Cleaning Function
```python
def clean_text(text):
    text = re.sub(r'\[.*?\]', '', str(text))
    return text.strip()
```
Defines a function to remove square brackets and their contents (typically references) from text.

## Cell 14: Applying Text Cleaning
```python
df[country_col] = df[country_col].apply(clean_text)
display(df.head())
```
Applies the cleaning function to the country column and displays the updated DataFrame.

## Cell 15: Defining Output Filename
```python
filename = "wikipedia_population_data.csv"
```
Sets the filename for the output CSV file.

## Cell 16: Saving Data to CSV
```python
df.to_csv(filename, index=False, encoding='utf-8')
print(f"Data saved successfully to '{filename}'")
```
Exports the DataFrame to a CSV file without the index and using UTF-8 encoding.
