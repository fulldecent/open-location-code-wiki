# API Developer's Guide

> This API is *experimental*. As of December 2016, we are soliciting feedback on the functionality, in order to inform proposals to geocoding API providers such as Google. You can discuss the API in the [public mailing list](https://groups.google.com/forum/#!forum/open-location-code) or create an issue in the [issue tracker](https://github.com/google/open-location-code/issues/new?labels=api&assignee=drinckes).

> If the API needs to be turned off, or there are other important messages, they will be returned in the JSON result in the field `error_message` or described on this page.

## Functionality

The API provides the following functions:

*  Conversion of a location (lat/lng or address) to a plus code (including the bounding box and the center);
*  Conversion of a plus code to the bounding box and center.

Optionally, it can also use Google's locality information to:

*  Generate short codes and addresses (such as converting `796RWF8Q+WF` to `WF8Q+WF Praia, Cape Verde`);
*  Generate codes for addresses or features.

## Result format
The results are provided in JSON format only.

(The API is loosely modeled on the [Google Geocoding
API](https://developers.google.com/maps/documentation/geocoding/intro).)

## API Function Summary

### Without Google API Key

| Search term | Request Parameters | Returns | Geocoding API calls |
|-------------|--------------------|---------|:-------------------:|
| lat/lng | [`?latlng=14.917313,-23.511313`](https://plus.codes/api?latlng=14.917313,-23.511313) | Global code and bounding box | 0 |
| global code | [`?address=796RWF8Q%2BWF`](https://plus.codes/api?address=796RWF8Q%2BWF) | Global code and bounding box | 0 |
| local code with lat/lng | [`?latlng=14.9,-23.5&address=WF8Q%2BWF`](https://plus.codes/api?latlng=14.9,-23.5&address=WF8Q%2BWF) | Global code and bounding box | 0 |
| address | [`?address=Praia,Cape%20Verde`](https://plus.codes/api?address=Praia,Cape%20Verde) | Error (the address cannot be located) | 0 |
| local code with address | [`?address=WF8Q%2BWF,Praia,Cape%20Verde`](https://plus.codes/api??address=WF8Q%2BWF,Praia,Cape%20Verde) | Error (the address cannot be located) | 0 |

### With Google API Key

If an [API
key](https://developers.google.com/maps/documentation/geocoding/start#get-a-key)
is also provided in the request, the response will include geocoding results,
and depending on the locality information, may also include a short form of the
plus code (called a local code).

>The API key **must** be a server side key without restrictions, with the Google Maps Geocoding API enabled.

In this form the `results` and `status` fields will be the same as that returned from the Google Maps Geocoding API, with the addition of the `plus_code` structure. In the event that the Geocoding API returns no results, a fake result consisting solely of a `plus_code` structure will be returned. This is so that codes can be generated for locations that cannot be geocoded, such as remote wilderness or ocean areas.

| Search term | Request Parameters | Returns | Geocoding API calls |
|-------------|--------------------|---------|:-------------------:|
| lat/lng | [`?latlng=14.917313,-23.511313&key=YOUR_API_KEY`](https://plus.codes/api?latlng=14.917313,-23.511313&key=YOUR_API_KEY) | Reverse geocoded results of the lat/lng, global code, bounding box, local code and address (if possible). | 1 |
| global code | [`?address=796RWF8Q%2BWF&key=YOUR_API_KEY`](https://plus.codes/api?address=796RWF8Q%2BWF&key=YOUR_API_KEY) | Reverse geocoded results of the global code center, global code, bounding box, local code and address (if possible). | 1 |
| local code with lat/lng | [`?latlng=14.9,-23.5&address=WF8Q%2BWF&key=YOUR_API_KEY`](https://plus.codes/api?latlng=14.9,-23.5&address=WF8Q%2BWF&key=YOUR_API_KEY) | Reverse geocoded results of the gobal code center, recovered global code, bounding box, local code and address (if possible). | 1 |
| address | [`?address=Praia,Cape%20Verde&key=YOUR_API_KEY`](https://plus.codes/api?address=Praia,Cape%20Verde&key=YOUR_API_KEY) | Geocode result of the address, global code, bounding box, local code and address (if possible). | 2 |
| local code with address | [`?address=WF8Q%2BWF,Praia,Cape%20Verde&key=YOUR_API_KEY`](https://plus.codes/api?address=WF8Q%2BWF,Praia,Cape%20Verde&key=YOUR_API_KEY) | Geocode result of the address, recovered global code, bounding box, local code and address (if possible). | 2 |

### Number of geocoding requests made

Because the address and code center are not necessarily in the same place, or
even particularly close, requests that include address information may make
**two** Geocoding API requests. One request is required to geolocate the passed
address, and compute the global code, and a second request to reverse geocode
the code center to obtain locality information used to compute the local
code.

The number of calls made to the Google Geocoding API is returned in the `api_cost` field.

### Code precision for address searches

Searches using a latitude and longitude will always return a 10 digit code. Address
based searches will return the code with the largest area that fits within the
bounding box of the result. For example, a search for `address=New+Zealand`
will return the global code `4VFP0000+`, and a search for `address=Paris` will return `8FW4V900+`.

### Language selection

By default, English will be used as the language preference. If you would like
locality names and geocoding results in a different language, use the `language` parameter.

## API Results

The following is a simple request to encode a latitude and longitude:

```
https://plus.codes/api?latlng=14.917313,-23.511313
```

The result from the API is:

```javascript
{
  "results": [
    {
      "plus_code": {
        "global_code": "796RWF8Q+WF",
        "geometry": {
          "bounds": {
            "northeast": {
              "lat": 14.917375000000007,
              "lng": -23.511250000000018
            },
            "southwest": {
              "lat": 14.91725000000001,
              "lng": -23.511375000000015
            }
          },
          "location": {
            "lat": 14.917312500000008,
            "lng": -23.511312500000017
          }
        }
      }
    }
  ],
  "status": "NO_API_KEY"
}
```

The `geometry` field is modeled on the structure returned from the Google Geocoding API. `bounds` give the bounding box of the area of the code. `location` is the center of the code bounding box.

If you also provide a valid Google API key (enabled for the Geocoding API) in your request:

```
https://plus.codes/api?latlng=14.917313,-23.511313&key=YOUR_API_KEY
```

The reverse geocode results will be included in the response. The `plus_code`
structure will be populated for **every** result. The code is computed on
`result.geometry.location`, and the length of the code depends on the
size of the feature. (It will be the largest code that fits within the 
result's bounding box).

If one of the results is close enough and small enough to shorten the code,
the `plus_code` structure **in the first result** will also contain the fields
`local_code`, `local_address`, and `locality_place_id`:

```javascript
{
  "results": [
    {
      "plus_code": {
        "global_code": "796RWF8Q+WF",
        "geometry": {
          "bounds": {
            "northeast": {
              "lat": 14.917375000000007,
              "lng": -23.511250000000018
            },
            "southwest": {
              "lat": 14.91725000000001,
              "lng": -23.511375000000015
            }
          },
          "location": {
            "lat": 14.917312500000008,
            "lng": -23.511312500000017
          }
        },
        "api_cost": 1,
        "local_code": "WF8Q+WF",
        "local_address": "Praia, Cape Verde",
        "locality_place_id": "ChIJV_GnJLaQNQkR8sSzx_UCrKA"
      }
      // Other fields returned from the geocoding API
    },
    // Other results from the geocoding API.
    // All will have a plus_code structure but not the local_ fields. */
  ],
  "status": "OK"
}
```

This is only done if the global code in the first result has at least 10 digits.

> Only 10 digit codes will be shortened.

The locality used to shorten the code is either the largest feature that allows
shortening the code to just 4+2 digits, or the smallest feature that allows
shortening the code to 6+2 digits. This is to select more prominent features
(cities) that are widely known in other geocoding platforms, rather than 
obbscure features that may not be widely known (e.g., neighbourhoods).
