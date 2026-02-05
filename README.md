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
- `TOP_K` (Optional): Number of top predictions to return (default is 3, maximum is 10).
- `COUNTRY_CODE` (Optional): 2-letter country code to limit search to a specific country (e.g., "US", "FR", "DE").
- `ADMIN1` (Optional): Admin1 region name (e.g., "California", "Niedersachsen", "Toscana"). Must be used with `COUNTRY_CODE`.
- `Center_LATITUDE` (Optional): Latitude of the center of the search area.
- `Center_LONGITUDE` (Optional): Longitude of the center of the search area.
- `RADIUS` (Optional): Radius of the search area around the center point in kilometers (maximum 25km).

#### Search Priority

When using location filters, the API searches in the following priority order:
1. **Admin1 + Country** (if both provided and admin1 is supported)
2. **Country** (if only country provided, or admin1 not supported)
3. **Worldwide** (if no location filters provided)

#### Example Request using the `picarta` Package

```python
from picarta import Picarta

api_token = "YOUR_API_TOKEN"
localizer = Picarta(api_token)

# Geolocate a local image worldwide
result = localizer.localize(img_path="/path/to/local/image.jpg")

print(result)

# Geolocate an image within a specific admin1 region
result = localizer.localize(
    img_path="https://upload.wikimedia.org/wikipedia/commons/8/83/San_Gimignano_03.jpg",
    top_k=3,
    country_code="IT",
    admin1="Toscana"
)

print(result)

# Geolocate with zone search (center point + radius)
result = localizer.localize(
    img_path="https://upload.wikimedia.org/wikipedia/commons/8/83/San_Gimignano_03.jpg",
    top_k=3,
    country_code="IT",
    admin1="Toscana",
    center_latitude=43.464, 
    center_longitude=11.038,
    radius=25
)

print(result)
```

#### Get Supported Admin1 Regions

Using the `picarta` package:

```python
from picarta import Picarta

localizer = Picarta("YOUR_API_TOKEN")
result = localizer.get_admin1_regions("IT")
print(result)
```

Or via GET request:

```
GET https://picarta.ai/admin1/{country_code}
```

Example response for `GET /admin1/IT`:
```json
{
  "admin1_regions": ["Abruzzo", "Basilicata", "Calabria", "Campania", "Emilia-Romagna", "Friuli Venezia Giulia", "Lazio", "Liguria", "Lombardia", "Marche", "Molise", "Piemonte", "Puglia", "Sardegna", "Sicilia", "Toscana", "Trentino-Alto Adige", "Umbria", "Valle d'Aosta", "Veneto"],
  "country_code": "IT"
}
```

If admin1 is not supported for a country:
```json
{
  "admin1_regions": [],
  "country_code": "AD",
  "message": "Admin1 search not supported for this country. Use country-level search."
}
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
country_code = "IT"
admin1 = "Toscana"
center_latitude, center_longitude, radius = None, None, None 

payload = {"TOKEN": api_token,
           "IMAGE": img_path,
           "TOP_K": top_k,
           "COUNTRY_CODE": country_code,
           "ADMIN1": admin1,
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
  "topk_predictions_dict": {
    "1": {
      "address": {"city": "San Gimignano", "country": "Italy", "province": "Tuscany"},
      "confidence": 0.9429214000701904,
      "gps": [43.46722412109375, 11.04349136352539]
    },
    "2": {
      "address": {"city": "San Gimignano", "country": "Italy", "province": "Tuscany"},
      "confidence": 0.8665691614151001,
      "gps": [43.4670295715332, 11.040340423583984]
    },
    "3": {
      "address": {"city": "San Gimignano", "country": "Italy", "province": "Tuscany"},
      "confidence": 0.8580218553543091,
      "gps": [43.4670295715332, 11.040340423583984]
    }
  },
  "ai_confidence": 0.9429214000701904,
  "ai_country": "Italy",
  "ai_lat": 43.46722412109375,
  "ai_lon": 11.04349136352539,
  "camera_maker": "NIKON CORPORATION",
  "camera_model": "NIKON D200",
  "city": "San Gimignano",
  "province": "Tuscany",
  "timestamp": "2010:09:21 12:04:46"
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
