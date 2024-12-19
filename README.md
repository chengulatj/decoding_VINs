# Autonomous Vehicle Disengagement VIN Decoder

# Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Step 1: Import the Necessary Libraries](#step-1-import-the-necessary-libraries)
   - [Libraries Overview](#libraries-overview)
4. [Step 2: Read the API Link Securely from a File](#step-2-read-the-api-link-securely-from-a-file)
   - [Input](#input)
   - [Process](#process)
5. [Step 3: Define a Function to Decode a VIN Using the NHTSA API](#step-3-define-a-function-to-decode-a-vin-using-the-nhtsa-api)
   - [Code](#code)
   - [Process](#process-1)
6. [VIN Decoding and Data Processing Steps](#vin-decoding-and-data-processing-steps)
   - [Step 4: Load the Dataset Containing VINs](#step-4-load-the-dataset-containing-vins)
   - [Step 5: Create an Empty List for Decoded Information](#step-5-create-an-empty-list-for-decoded-information)
   - [Step 6: Iterate Over VINs and Decode Them](#step-6-iterate-over-vins-and-decode-them)
   - [Step 7: Convert Decoded Information to a DataFrame](#step-7-convert-decoded-information-to-a-dataframe)
   - [Step 8: Add Temporary Unique Identifiers](#step-8-add-temporary-unique-identifiers)
   - [Step 9: Merge the DataFrames](#step-9-merge-the-dataframes)
   - [Step 10: Drop the Temporary Identifier](#step-10-drop-the-temporary-identifier)
   - [Step 11: Save the Merged DataFrame to a New CSV File](#step-11-save-the-merged-dataframe-to-a-new-csv-file)


## Overview
This project uses reports from the California DMV on autonomous vehicle disengagements to analyze and decode Vehicle Identification Numbers (VINs). The decoded VIN information is fetched using the [NHTSA VIN Decoder API](https://vpic.nhtsa.dot.gov/decoder/), providing comprehensive details about the vehicle associated with the VIN. The API link is securely read from a configuration file stored in Google Drive, ensuring sensitive information is not exposed in the script. The decoded data is then saved to a new CSV file.

## Features

- **Secure API Link Handling**: The API link is stored in a file (`VINLINK.txt`) and loaded dynamically during execution.
- **Batch Processing**: Processes multiple VINs from a CSV file in one go.
- **Output Merging**: Combines the decoded information with the original dataset.
- **Google Drive Integration**: Reads input files and saves the output to Google Drive for easy access.

# Step 1: Import the Necessary Libraries 

This section of the code imports essential libraries and mounts Google Drive to access files stored there. It is specifically designed for use in Google Colab.

## Code

```python
import pandas as pd
import requests
```
## Libraries Overview

### **Pandas**
- A powerful Python library for data manipulation and analysis.
- Commonly used to:
  - Load datasets in CSV (and other formats)
  - Process and clean data efficiently.
  - Perform operations like filtering, grouping, merging, and aggregation.

### **Requests**
- A popular HTTP library in Python for making HTTP requests.
- Simplifies sending and handling requests such as:
  - `GET`: Retrieve data from a specified URL.
  - `POST`: Send data to a server.
- Provides easy-to-use methods to handle API responses, including JSON data and HTTP status codes.

# Step 2: Read the API Link Securely from a File

This code snippet securely reads an API link from a file (`VINLINK.txt`) stored in Google Drive. The file contains the API link in a `key=value` format.

## Code

```python
# Step 2: Read the API link securely from a file
with open('/content/drive/MyDrive/VINLINK.txt') as f:
    for line in f:
        if '=' in line:
            parts = line.strip().split('=')  # Split the line by '='
            # Safely access the key and value, handling potential extra '='
            key = parts[0] if parts else None
            value = parts[1] if len(parts) > 1 else None
            if key == 'VINLINK':
                VINLINK = value
                break  # Exit loop after finding VINLINK
```

## Description

### **Input**
A file (`VINLINK.txt`) containing the API link in the following format:
```plaintext
VINLINK= https://xxxx
```
### **Process**
1. Opens the file and reads it line by line.
2. Checks if the line contains an `=` character.
3. Splits the line into a key-value pair:
   - **`key`**: The text before the `=` character.
   - **`value`**: The text after the `=` character.
4. Assigns the `value` to the variable `VINLINK` if the `key` matches `"VINLINK"`.
5. Breaks the loop once the API link is found.

### **Step 3: Define a Function to Decode a VIN Using the NHTSA API**

This function decodes a Vehicle Identification Number (VIN) using the secure API link read from the file.

---

### **Code**
```python
def decode_vin(vin):
    # Use the secure API link read from the file
    url = f"{VINLINK}{vin}?format=json"
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        return None
```
### **Process**
1. Constructs the API URL by appending the `vin` to the `VINLINK` with the `?format=json` query string.
2. Sends a `GET` request to the constructed URL using the `requests` library.
3. Checks if the response status code is `200` (successful request).
4. If successful:
   - Parses the JSON response and returns it.
5. If unsuccessful:
   - Returns `None`.
### **VIN Decoding and Data Processing Steps**

---

### **Step 4: Load the Dataset Containing VINs**
```python
df = pd.read_csv("/content/drive/MyDrive/vin_nos.csv")
```
- **Loads the input dataset** from Google Drive containing Vehicle Identification Numbers (VINs).  
- **Requirement**: The file must have at least one column named `VIN`.

- ### **Step 5: Create an Empty List for Decoded Information**

```python
decoded_vins = []
```
**Purpose**: Initializes an empty list to store the decoded VIN information retrieved from the API.

### **Step 6: Iterate Over VINs and Decode Them**

```python
for vin in df['VIN']:
    decoded_info = decode_vin(vin)
    if decoded_info and 'Results' in decoded_info:
        # Append the decoded information to the list
        decoded_vins.append(decoded_info['Results'][0])
```
**Purpose**: Iterates over each VIN in the dataset.

**Process**:
1. Retrieves decoded vehicle information from the API using the `decode_vin` function.
2. Checks if the response contains valid data (`'Results'` key).
3. Appends the decoded information to the `decoded_vins` list if the response is valid.

### **Step 7: Convert Decoded Information to a DataFrame**

```python
decoded_vins_df = pd.DataFrame(decoded_vins)
```
**Purpose**: Converts the list of decoded VIN information into a Pandas DataFrame for easier processing and merging.

### **Step 8: Add Temporary Unique Identifiers**

```python
df['temp_id'] = range(1, len(df) + 1)
decoded_vins_df['temp_id'] = range(1, len(decoded_vins_df) + 1)
```
**Purpose**: Adds a temporary unique identifier (temp_id) to both DataFrames to facilitate accurate merging.

### **Step 9: Merge the DataFrames**

```python
merged_df = pd.merge(df, decoded_vins_df, on=['VIN', 'temp_id'], how='inner')
```
**Purpose**: Merges the original VIN dataset (`df`) with the decoded information (`decoded_vins_df`) using `VIN` and `temp_id` as keys.

**Outcome**: Ensures that the original data and decoded information align correctly.

### **Step 10: Drop the Temporary Identifier**

```python
merged_df.drop('temp_id', axis=1, inplace=True)
```
**Purpose**: Removes the temp_id column from the merged DataFrame as it is no longer needed.

### **Step 11: Save the Merged DataFrame to a New CSV File**

```python
merged_df.to_csv("/content/drive/MyDrive/decoded_vins.csv", index=False)
print("Decoded VINs saved to 'decoded_vins.csv'")
```
**Purpose**: Saves the final merged DataFrame, which includes both the original VINs and the decoded information, to a new CSV file in Google Drive.

**Output File**: The new file is named `decoded_vins.csv` and will be stored in the specified Google Drive path.

## Conclusion

This project provides a streamlined approach to decoding Vehicle Identification Numbers (VINs) using the NHTSA VIN Decoder API. By automating the decoding process, the code efficiently processes multiple VINs, retrieves detailed vehicle information, and merges it with the original dataset. The final result is a comprehensive CSV file that combines the original VINs with their corresponding decoded details, enabling further analysis or reporting. This solution is designed to be robust, secure, and scalable for larger datasets.

