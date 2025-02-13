---
date:
  created: 2025-02-11
authors:
  - bruce
categories:
  - Technology
---

# Tesseract OCR
Have basic OCR needs? [Tesseract OCR](https://tesseract-ocr.github.io/tessdoc/) is an open-source engine and it might just fit the bill.

![tesseract-ocr](tesseract-ocr.png)
image by DALL-E

<!-- more -->
If you have a need to extract text from screenshots (clipboard) or saved images, there are paid solutions available, but I usually reach for open source first. 

[Tesseract OCR](https://github.com/tesseract-ocr/tesseract) is an open source OCR Engine which supports a variety of input and output formats - 
It's been around for a few decades and was open sourced by HP in 2005. It seems Google took over the project from 2006 to 2018.

Inputs 

 - PNG, JPEG, TIFF, GIF

Outputs

  - Plain text, PDF


## Windows Clipboard
On Windows, often I'll grab a screenshot and I'll need to extract text from it. This is a practical use case to give `Tesseract OCR` a whirl.

First we'll need to host the service, I prefer to stand utility services up in [Docker](https://docs.docker.com/engine/install/) to evaluate. Below is a simple Python [Flask](https://flask.palletsprojects.com/en/stable/) app and `Dockerfile` which you can use to host so you can interact with it via a HTTP endpoint -

app.py
```python
import os
import base64
import io

from flask import Flask, request, jsonify
import pytesseract
from PIL import Image

app = Flask(__name__)

@app.route("/ocr", methods=["POST"])
def ocr():
    # Check if the request has the file part
    if "file" in request.files:
        # If we received a file directly (multipart/form-data)
        file = request.files["file"]
        image = Image.open(file.stream).convert("RGB")
        text = pytesseract.image_to_string(image)
        return jsonify({"text": text.strip()})
    else:
        # Otherwise, try parsing base64 from JSON
        data = request.get_json()
        if not data or "image" not in data:
            return jsonify({"error": "No image data found"}), 400

        # Decode base64 string
        try:
            image_data = base64.b64decode(data["image"])
            image = Image.open(io.BytesIO(image_data)).convert("RGB")
            text = pytesseract.image_to_string(image)
            return jsonify({"text": text.strip()})
        except Exception as e:
            return jsonify({"error": str(e)}), 400


@app.route("/", methods=["GET"])
def index():
    return "Tesseract OCR service is running."


if __name__ == "__main__":
    # You can make Flask listen on 0.0.0.0 so it is accessible from outside the container
    app.run(host="0.0.0.0", port=5000)
```

Dockerfile
```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.11-slim

# Install Tesseract and any additional packages you need
RUN apt-get update && \
    apt-get install -y tesseract-ocr \
    libtesseract-dev \
    # Optional: add additional language packages, e.g. for German, French, etc.
    tesseract-ocr-eng \
    # (Add any more languages you need)
    # Clean up
    && rm -rf /var/lib/apt/lists/*

# Create a directory for the app
WORKDIR /usr/src/app

# Copy requirements first to leverage Docker cache
COPY requirements.txt ./

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the code
COPY . .

# Expose the service port
EXPOSE 5000

# Run the Flask app
CMD [ "python", "app.py" ]
```

## Docker Build
Go ahead and build the image and serve on localhost port 5000 -
```shell
docker build -t tesseract-ocr-service .
docker run -d -u 1001:1001 -p 5000:5000 --name tesseract_service --restart unless-stopped tesseract-ocr-service
```

To test, create a local `test.jpg` file, it can be any image/screenshot you save with text present. 

You can use `curl` on Linux or `powershell` to verify that `Tesseract OCR` is working -
Note that the response is in JSON, so with `curl` I use the [jq](https://jqlang.org/) utility to extract just the raw text result from the JSON.

## Testing Tesseract OCR

Curl
```shell
#no jq
curl -X POST -F file=@test.jpg http://localhost:5000/ocr >| test.txt
#jq
curl -X POST -F file=@test.jpg http://localhost:5000/ocr | jq -r '.text' >| test.txt
```


PowerShell
Note: the following command with the `-Form` parameter requires [Powershell 6](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.5) or later -
```powershell 
(Invoke-RestMethod `
    -Uri "http://localhost:5000/ocr" `
    -Method POST `
    -Form @{ file = Get-Item "test.jpg" }
).text | Out-File -FilePath "test.txt"
```

## Helper Script
Now we'll create a PowerShell script which we can use to scrape our clipboard or pass it an image file to OCR.

ocr.ps1
```powershell
param (
    [string]$filePath
)

# Add necessary assembly reference
Add-Type -AssemblyName System.Drawing

# Define the target URL
$URL = "http://localhost:5000/ocr"

if ($filePath) {
    # Check if the file path is relative
    if ($filePath -notmatch "^[A-Z]:\\|^\\\\|^/") {
        $currentFolderPath = Join-Path (Get-Location) $filePath

        if (-Not (Test-Path $currentFolderPath)) {
            Write-Host "File not found: $filePath"
            exit
        }
        $filePath = $currentFolderPath
    }

    # Load image from file path
    $image = [System.Drawing.Image]::FromFile($filePath)
} else {
    # Load clipboard image
    Add-Type -AssemblyName System.Windows.Forms
    $image = [System.Windows.Forms.Clipboard]::GetImage()

    if ($image -eq $null) {
        Write-Host "No image found in clipboard."
        exit
    }
}

# Convert image to a MemoryStream (PNG format)
$memoryStream = New-Object System.IO.MemoryStream
$image.Save($memoryStream, [System.Drawing.Imaging.ImageFormat]::Png)
$byteArray = $memoryStream.ToArray()

# Convert byte array to Base64 string
$base64String = [Convert]::ToBase64String($byteArray)

# Create JSON payload
$payload = @{
    image = $base64String
    filename = "image.png"
    mimetype = "image/png"
} | ConvertTo-Json -Depth 2

# Send as HTTP POST request
$headers = @{"Content-Type" = "application/json"}
$response = Invoke-RestMethod -Uri $URL -Method Post -Body $payload -Headers $headers

# Print response
Write-Output "$($response.text)"
```

## PowerShell Profile

If you haven't created a [powershell profile](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.5) previously, you can do so like this -
```powershell
# determine if you have a profile
Test-Path $PROFILE
# create a profile
New-Item -Path $PROFILE -ItemType File -Force
```

Then update your PowerShell profile, you can do this by running `notepad $PROFILE` and add the following. Just adjust the location of the `ocr.ps1` script -
```powershell
function ocr {
    param(
        [Parameter(ValueFromRemainingArguments=$true)]
        $args
    )
    & "C:\scripts\ocr.ps1" $args
}
```

Now if you take a screenshot (e.g. using the printscreen or alt-printscreen buttons), you can run the `ocr` command and it will output the text in your powershell prompt window.
```powershell
ocr
```

Also, you can pass an argument to the ocr command which is an image file and it will ocr the jpg, png, or tiff file.

```powershell
ocr test.jpg
```

That's it. You can use Tesseract OCR locally as a productivity tool or utilize for preprocessing in an ETL process.