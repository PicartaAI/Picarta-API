# Picarta-API

Picarta AI Image Geolocalization API 

### Overview

The Picarta AI Image Classification API allows users to classify images and obtain predictions about their geographic location based on their content and/or embedded metadata. Users can provide an image either from a local file or via a URL and receive predictions about the location depicted in the image. The API returns information such as city, province, country, GPS coordinates, and confidence scores for each prediction.

### Authentication

To access the API, users need to provide an API token in the request headers. Users can obtain an API token by registering on the Picarta AI website.

### Usage

#### Request Format

The API accepts HTTP POST requests with a JSON payload containing the following parameters:

- `TOKEN`: User's API token.
- `IMAGE`: Base64-encoded image data or URL of the image to classify.
- `TOP_K` (Optional): Number of top predictions to return (default is 3, maximum is 10).
- `Center_LATITUDE` (Optional): Latitude of the center of the search area.
- `Center_LONGITUDE` (Optional): Longitude of the center of the search area.
- `RADIUS` (Optional): Radius of the search area around the center point in kilometers.

#### Example Request

```python
import requests
import json
import base64

url = "https://picarta.ai/classify"
api_token = "API_TOKEN"
top_k = 3
headers = {"Content-Type": "application/json"}

# Read the image from a local file or URL
with open("path/to/local/image.jpg", "rb") as image_file:
    img_path = base64.b64encode(image_file.read()).decode('utf-8')

# OR  

# img_path = "https://upload.wikimedia.org/wikipedia/commons/8/83/San_Gimignano_03.jpg"

# Optional parameters for a specific location search
center_latitude, center_longitude, radius = None, None, None 

payload = {"TOKEN": api_token,
           "IMAGE": img_path,
           "TOP_K": top_k,
           "Center_LATITUDE": center_latitude,
           "Center_LONGITUDE": center_longitude,
           "RADIUS": radius}

response = requests.post(url, headers=headers, json=payload)

if response.status_code == 200:
    result = response.json()
    print(result)
else:
    print("Request failed with status code:", response.status_code)
    print(response.json())

```
