# API Developer's Guide

### Table of Contents
* [Functionality](#functionality)
* [Encoding and Decoding](#encoding-and-decoding)
* [API Request Format](#api-request-format)
* [Encoding Requests](#encoding-requests)
* [Encoding Responses](#encoding-responses)
  * [Encoding Without An API Key](#encoding-without-an-api-key)
  * [Encoding With An API Key](#encoding-with-an-api-key)
* [Decoding Requests](#decoding-requests)
* [Decoding Responses](#decoding-responses)
  * [Decoding Without An API Key](#decoding-without-an-api-key)
  * [Decoding With An API Key](#decoding-with-an-api-key)

> This API is *experimental*. As of December 2016, we are soliciting feedback on the functionality, in order to inform proposals to geocoding API providers such as Google. You can discuss the API in the [public mailing list](https://groups.google.com/forum/#!forum/open-location-code) or create an issue in the [issue tracker](https://github.com/google/open-location-code/issues/new?labels=api&assignee=drinckes).

> If the API needs to be turned off, or there are other important messages, they will be returned in the JSON result in the field `error_message` or described on this page.

## Functionality

The API provides the following functions:

*  Conversion of a location (lat/lng or address) to a plus code (including the bounding box and the center);
*  Conversion of a plus code to the bounding box and center.

Optionally, it can also use Google's locality information to:

*  Generate short codes and addresses (such as generating "WF8Q+WF Praia, Cape Verde" for "796RWF8Q+WF");
*  Generate codes for addresses ("1600 Amphitheatre Parkway, Mountain View, CA") or features ("Paris").

## Encoding and Decoding

Geocoding is the process of converting addresses (like "1600 Amphitheatre
Parkway, Mountain View, CA") into geographic coordinates (like latitude
37.423021 and longitude -122.083739), which you can use to place markers on
a map, or position the map.

Reverse geocoding is the process of converting geographic coordinates into a
human-readable address. In addition to the results, this API adds plus code
information to them, including the global code, the coordinates, and if 
possible the local code and address.

This API, although based on the Google Geocoding API, provides **encoding** and **decoding* functions:
* An **encoding** request converts from a latitude/longitude, or address, to a plus code.
* A **decoding** request takes a plus code and converts it to location and address.

The results are provided in JSON format. The API is loosely modeled on the [Google Geocoding
API](https://developers.google.com/maps/documentation/geocoding/intro).

## API Request Format

A Plus codes API request takes the following form:

`https://plus.codes/api?parameters`

**Note**: URLs must be [properly encoded](https://developers.google.com/maps/web-services/overview#BuildingURLs) (specifically, `+` characters must be encoded to `%2B`).

## Encoding Requests

Encoding requests convert an address or latitude/longitude to a plus code.

**Required parameters in an encoding request:**

* `latlng` — The latitude and longitude values specifying the location for which you wish to obtain the plus code

**or**

* `address` — The address to encode. This can be a street address, or the name of a feature such as a neighbourhood, city etc. Note that the `key` parameter must be specified to support address-based encoding requests.

**Optional parameters in an encoding request:**

* `language` — The language in which to return results. This won't affect the global code, but it will affect the names of any features generated, as well as the address used for the local code.
 * If `language` is not supplied, results will be provided in English.
* `key` — Your Google API key. This allows requests to be passed to the Google Geocoding API. To get a key refer [here](https://developers.google.com/maps/documentation/geocoding/start#get-a-key).
 * The API key **must** be a server side key without restrictions, with the Google Maps Geocoding API enabled.

## Encoding Responses

All responses are returned as JSON objects.

### Encoding Without An API Key

Without an API key, the only encoding that is possible is latitude/longitude encoding. The following is a simple request to encode a latitude and longitude:

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

The JSON response contains two root elements:

*  `"status"` contains metadata on the request. Since an API key was not provided, it has returned `"NO_API_KEY"`. Other keys are documented [here](https://developers.google.com/maps/documentation/geocoding/intro#StatusCodes).
*  `"results"` contains an array with a single item.

There may be an additional `error_message` field within the response object. This may contain additional background information for the status code.

The `plus_code` structure gives the global code, the bounding box and center (`location`).

### Encoding With An API Key

If a valid API key is provided in the `key` parameter of the request, the results will be the same as a [reverse geocode request](https://developers.google.com/maps/documentation/geocoding/intro#ReverseGeocoding), but with a `plus_code` structure added to every result.

> To encode addresses, such as "1600 Amphitheatre Parkway, Mountain View, CA" it first needs to be geocoded to get the location. This requires the `key` parameter to be supplied.

Below are the first two results of the latitude/longitude encoding request above, but this time with an API key:

```javascript
{
  "results": [
    {
      "address_components": [
        {
          "long_name": "Avenue Machado Santos",
          "short_name": "Ave Machado Santos",
          "types": [
            "route"
          ]
        },
        {
          "long_name": "Praia",
          "short_name": "Praia",
          "types": [
            "locality",
            "political"
          ]
        },
        {
          "long_name": "Praia",
          "short_name": "Praia",
          "types": [
            "administrative_area_level_1",
            "political"
          ]
        },
        {
          "long_name": "Cape Verde",
          "short_name": "CV",
          "types": [
            "country",
            "political"
          ]
        }
      ],
      "formatted_address": "Ave Machado Santos, Praia, Cape Verde",
      "geometry": {
        "bounds": {
          "northeast": {
            "lat": 14.9192329,
            "lng": -23.5094639
          },
          "southwest": {
            "lat": 14.9165614,
            "lng": -23.5108249
          }
        },
        "location": {
          "lat": 14.917857,
          "lng": -23.510049
        },
        "location_type": "GEOMETRIC_CENTER",
        "viewport": {
          "northeast": {
            "lat": 14.9192461302915,
            "lng": -23.5087954197085
          },
          "southwest": {
            "lat": 14.9165481697085,
            "lng": -23.5114933802915
          }
        }
      },
      "place_id": "ChIJabhTnOOYNQkR0uozHYX4gIk",
      "types": [
        "route"
      ],
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
    },
    {
      "address_components": [
        {
          "long_name": "Várzea",
          "short_name": "Várzea",
          "types": [
            "neighborhood",
            "political"
          ]
        },
        {
          "long_name": "Praia",
          "short_name": "Praia",
          "types": [
            "locality",
            "political"
          ]
        },
        {
          "long_name": "Praia",
          "short_name": "Praia",
          "types": [
            "administrative_area_level_1",
            "political"
          ]
        },
        {
          "long_name": "Cape Verde",
          "short_name": "CV",
          "types": [
            "country",
            "political"
          ]
        }
      ],
      "formatted_address": "Várzea, Praia, Cape Verde",
      "geometry": {
        "bounds": {
          "northeast": {
            "lat": 14.9210663,
            "lng": -23.5105061
          },
          "southwest": {
            "lat": 14.9138092,
            "lng": -23.5176301
          }
        },
        "location": {
          "lat": 14.9172665,
          "lng": -23.5140708
        },
        "location_type": "APPROXIMATE",
        "viewport": {
          "northeast": {
            "lat": 14.9210663,
            "lng": -23.5105061
          },
          "southwest": {
            "lat": 14.9138092,
            "lng": -23.5176301
          }
        }
      },
      "place_id": "ChIJ6weAsPyYNQkR1hPaMKHw67s",
      "types": [
        "neighborhood",
        "political"
      ],
      "plus_code": {
        "global_code": "796RWF8P+",
        "geometry": {
          "bounds": {
            "northeast": {
              "lat": 14.917500000000004,
              "lng": -23.512500000000017
            },
            "southwest": {
              "lat": 14.915000000000006,
              "lng": -23.515000000000015
            }
          },
          "location": {
            "lat": 14.916250000000005,
            "lng": -23.513750000000016
          }
        }
      }
    },
    // additional results
```

This time, the **first** `plus_code` structure also has the following fields:
* `api_cost` — This is the number of requests sent to the Google Geocoding API.
* `local_code` — This is the shortened version of the code that works within approximately 40-50km, but beyond that needs to be combined with a nearby address or location.
* `local_address` — This address should geocode to nearby and can be used together with `local_code`. For example, "WF8Q+WF Praia, Cape Verde".
* `locality_place_id` — This is the unique identifier for the feature that provides the `local_address`. See the [place ID overview](https://developers.google.com/places/place-id).

These fields are only populated on the **first** result, and only if it has a 10 digit `global_code`.

The locality used to shorten the code is either the **largest** feature that allows
shortening the code to just 4+2 digits, or the **smallest** feature that allows
shortening the code to 6+2 digits. This is to select more prominent features
(e.g., cities) that have more chance of being known in other geocoding platforms, rather than 
obscure features that may not be widely known (e.g., neighbourhoods).

The second structure doesn't have those fields, and the `global_code` is "796RWF8P+". This code represents a larger area (roughly 250x250 meters), and is the largest code that fits entirely within the bounding box of the result.

## Decoding Requests

**Required parameters in an decoding request:**

* `address` — The plus code to encode. This can be any of:
 * a global code ("796RWF8Q+WF")
 * a local code ("WF8Q+WF"). Note that the `latlng` parameter must be specified to provide a reference location.
 * a local code with a street address ("WF8Q+WF Praia Cape Verde"). Note that the `key` parameter **must** be specified so the address can be located.

**Optional parameters in an encoding request:**

* `language` — The language in which to return results. This won't affect the global code, but it will affect the names of any features generated, as well as the address used for the local code.
 * If `language` is not supplied, results will be provided in English.
* `key` — Your Google API key. This allows requests to be passed to the Google Geocoding API. To get a key refer [here](https://developers.google.com/maps/documentation/geocoding/start#get-a-key).
 * The API key **must** be a server side key without restrictions, with the Google Maps Geocoding API enabled.

## Decoding Responses

All responses are returned as JSON objects, and are identical to the encoding responses.

### Decoding Without An API Key

Without an API key, global codes and local codes with latitude/longitude can be decoded. The following is a simple request to decode a global code:

```
https://plus.codes/api?address=796RWF8Q%2BWF
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

### Decoding With An API Key

To decode a local code with a locality, an API key must be provided. This is so the locality can be geocoded and the global code computed. An example of a request is:

```
https://plus.codes/api?address=WF8Q%2BWF+Praia,+Cape+Verde&key=YOUR_API_KEY
```

the `api_cost` field in the first result will have the value `2`. This is because this requires
**two** requests to the Google Geocoding API:
* The first request locates "Praia, Cape Verde". That location is combined with the local code "WF8Q+WF" to generate a global code, "796RWF8Q+WF".
* The second request reverse geocodes the center of that code, to provide the results and to enable the generation of the `local_code`, `local_address`, and `locality_place_id` fields.
