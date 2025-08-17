# Picarta-API

[Picarta.ai](https://picarta.ai) Image Geolocalization API 🌍🔍

### Overview

The Picarta Image Geolocalization [API](https://picarta.ai/api) allows users to localize images and obtain predictions about their geographic location based on their content and/or embedded metadata. Users can provide an image either from a local file or via a URL and receive predictions about the location depicted in the image. The API returns information such as city, province, country, GPS coordinates, and confidence scores for each prediction.

### Authentication

To access the [API](https://picarta.ai/api), users need to provide an API token in the request headers. Users can obtain an API token by registering on the [Picarta](https://picarta.ai) website.

### Installation

To install the `picarta` package, use pip:

```bash
pip install picarta
```

### Usage

#### Request Format

The API accepts HTTP POST requests with a JSON payload containing the following parameters:

- `TOKEN`: User's API token.
- `IMAGE`: image path or URL of the image to localize.
- `TOP_K` (Optional): Number of top predictions to return (default is 10, maximum is 100).
- `COUNTRY_CODE` (Optional): 2-letter country code to limit search to a specific country (e.g., "US", "FR", "DE").
- `Center_LATITUDE` (Optional): Latitude of the center of the search area.
- `Center_LONGITUDE` (Optional): Longitude of the center of the search area.
- `RADIUS` (Optional): Radius of the search area around the center point in kilometers.

#### Example Request using the `picarta` Package

```python
from picarta import Picarta

api_token = "YOUR_API_TOKEN"
localizer = Picarta(api_token)

# Geolocate a local image
result = localizer.localize(img_path="/path/to/local/image.jpg")

print(result)

# Geolocate an image from URL with optional parameters for a specific location search
result = localizer.localize(
img_path="https://upload.wikimedia.org/wikipedia/commons/8/83/San_Gimignano_03.jpg",
top_k=10,
country_code="IT",
center_latitude=43.464, 
center_longitude=11.038,
radius=100)

print(result)

```

#### Example Request without `picarta` Package

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
country_code, center_latitude, center_longitude, radius = None, None, None, None 

payload = {"TOKEN": api_token,
           "IMAGE": img_path,
           "TOP_K": top_k,
           "COUNTRY_CODE": country_code,
           "Center_LATITUDE": center_latitude,
           "Center_LONGITUDE": center_longitude,
           "RADIUS": radius}

response = requests.post(url, headers=headers, json=payload)

if response.status_code == 200:
    result = response.json()
    print(result)
else:
    print("Request failed with status code:", response.status_code)
    print(response.text)

```
#### Response Format
The API returns a JSON object containing geographic location results, including metadata about the image and a dictionary of topk predictions.

#### Example API Response

```json
{
  "ai_country": "Fiji",
  "ai_lat": -10.932661290178117,
  "ai_lon": 173.54167690802137,
  "camera_maker": "NIKON CORPORATION",
  "camera_model": "NIKON D200",
  "city": "Ahau",
  "confidence": 0.7205776784126713,
  "province": "Rotuma",
  "timestamp": "2010:09:21 12:04:46",
  "topk_predictions_dict": {
    "1": {
      "address": {"city": "Ahau", "country": "Fiji", "province": "Rotuma"},
      "confidence": 0.7205776784126713,
      "gps": [-10.932661290178117, 173.54167690802137]
    },
    "2": {
      "address": {"city": "Nghi Xuan", "country": "Vietnam", "province": "Ha Tinh"},
      "confidence": 0.13465818962254223,
      "gps": [18.831436938230198, 106.00851919090474]
    },
    "3": {
      "address": {"city": "Hanga Roa", "country": "Chile", "province": "Valparaiso"},
      "confidence": 0.03465818962254226,
      "gps": [-42.42505486943787, -118.63631306266818]
    }
  }
}
```

#### Additional Notes

- `topk_predictions_dict` is presented in the second version of the API.
- `topk_predictions_dict[1]` is equal to province, ai_country, city, ai_lat, ai_lon, and ai_confidence. (It shows the top 1 result, which was in the first version of the API).
- The API could also return the following values if the EXIF data exists in the images:
    - `exif_lat`: Latitude from EXIF metadata.
    - `exif_lon`: Longitude from EXIF metadata.
    - `exif_country`: Country name from EXIF metadata.
- For aerial and satellite images, please refer to this repository: [Aerial-Imagery](https://github.com/PicartaAI/Picarta-Aerial-Imagery.git).

### Contact Information

For any inquiries or assistance, feel free to contact us via:

- Email: [info@picarta.ai](mailto:info@picarta.ai)
- Discord: [Join our Discord channel](https://discord.gg/g5BAd2UFbs)
- Share your feedback: [API Feedback Survey](https://forms.gle/JokVe1ZRKP1hjsA49)

