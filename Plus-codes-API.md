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
* [Securing Your API Key](#securing-your-api-key)
  * [Allowing Multiple Referrers](#allowing-multiple-referrers)

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
human-readable address.

This API provides **encoding** and **decoding** functions:
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
 * The API key **must** not have any restrictions
 * The API key **must** have the Google Maps Geocoding API enabled
 * See [Securing Your API Key](#securing-your-api-key) for how to prevent others from scraping your key from your site and using it.

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

*  `"status"` contains metadata on the request. Since an API key was not provided, it has returned `"NO_API_KEY"`. Other status values are documented [here](https://developers.google.com/maps/documentation/geocoding/intro#StatusCodes).
*  `"results"` contains an array with a single item.

There may be an additional `error_message` field within the response object. This may contain additional background information for the status code.

The `plus_code` structure has the following fields:
*  `global_code` gives the global code for the latitude/longitude
*  `bounds` provides the bounding box of the code, with the north east and south west coordinates
*  `location` provides the center of the bounding box.

### Encoding With An API Key

If a valid Google API key is provided in the `key` parameter of the request, the results for a latitude/longitude request will be the same as a [reverse geocode request](https://developers.google.com/maps/documentation/geocoding/intro#ReverseGeocoding), but with a `plus_code` structure added to *every* result.

The results for an address-based request will be the same as a [forward geocode request](https://developers.google.com/maps/documentation/geocoding/intro#geocoding), but with a `plus_code` structure added to every result.

> To encode addresses, such as "1600 Amphitheatre Parkway, Mountain View, CA" the `key` parameter **must** be supplied.

Below is shown the first two entries of the `results` list for the latitude/longitude encoding request above, this time with an API key:

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
* `api_cost` — This is the number of requests sent to the Google Geocoding API. In this case, only one call is required.
* `local_code` — This is the shortened version of the code. Codes are shortened relative to a containing or nearby locality.
* `local_address` — This address can be used together with `local_code`. For example, "WF8Q+WF Praia, Cape Verde". The address should geocode to a nearby location.
* `locality_place_id` — This is the unique identifier for the feature that provided `local_address`. See the [place ID overview](https://developers.google.com/places/place-id).

> These fields are only populated on the **first** result.

The locality used to shorten the code is either the **largest** feature that allows
shortening the code to just 4+2 digits, or the **smallest** feature that allows
shortening the code to 6+2 digits. This is to select more prominent features
(e.g., cities) that have more chance of being known in other geocoding platforms, rather than 
obscure features that may not be widely known (e.g., neighbourhoods).

The second structure doesn't have those fields, and the `global_code` is "796RWF8P+". This code represents a larger area (roughly 250x250 meters), and is the largest code that fits entirely within the bounding box of the result.

### Encoding Features

If the `address` parameter does not contain a specific address, but the name of a locality or town (e.g., "Paris"), the `global_code` will be the largest code that fits entirely within the bounding box of the feature. That means it might not have 10 digits.

If the code does not have 10 digits, the `local_code`, `local_address` and `locality_place_id` fields will not be provided.

## Decoding Requests

Decoding requests are used to decode a plus code. You may pass a global code (e.g. "796RWF8Q+WF"), a local code with an address (e.g., "WF8Q+WF Praia, Cape Verde") or a local code and a latitude/longitude.

**Required parameters in an decoding request:**

* `address` — The plus code to encode. This can be any of:
 * a global code ("796RWF8Q+WF")
   * Any other text in the address will be ignored
 * a local code with a street address ("WF8Q+WF Praia Cape Verde")
   * The `key` parameter **must** be specified so the address can be located
 * a local code ("WF8Q+WF")
   * The `latlng` parameter **must** be specified to provide a reference location.

**Optional parameters in an encoding request:**

* `language` — The language in which to return results. This won't affect the global code, but it will affect the names of any features generated, as well as the address used for the local code.
 * If `language` is not supplied, results will be provided in English.
* `key` — Your Google API key. This allows requests to be passed to the Google Geocoding API. To get a key refer [here](https://developers.google.com/maps/documentation/geocoding/start#get-a-key).
 * The API key **must** not have any restrictions
 * The API key **must** have the Google Maps Geocoding API enabled
 * See [Securing Your API Key](#securing-your-api-key) for how to prevent others from scraping your key from your site and using it.

## Decoding Responses

Responses are returned as JSON objects, consisting of `results` and `status`. `results` will contain a single entry, containing the `plus_code` structure.

### Decoding Without An API Key

Without an API key, only global codes or local codes with latitude/longitude can be decoded. The following is a simple request to decode a global code:

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
* The second request reverse geocodes the center of the global code, to obtain the `local_code`, `local_address`, and `locality_place_id` fields.

## Securing Your API Key

> Securing your Google API key is important if you are using it from a website, where it can be easily detected.

Google API keys provide access to different APIs, and most have a free daily quota allowance. If other people obtain your key, they can consume your free quota, and if you have billing enabled, can incur charges.

The normal way of securing API keys is setting restrictions on the host calling the Google API or the IP addresses that can make requests. Because the calls to the Google API are being done from the plus codes server, not your site, these methods don't work.

As a solution, instead of the plain API key, you can encrypt it together with your host, and then use the encrypted value. When the plus codes API receives this, it decrypts the key and checks that the host making the request matches the encrypted value.

If the host matches, then the requests are done. If the host does not match, a failure message is returned.

For example, to protect the API key `my_api_key` for the site `openlocationcode.com`, encrypt it:

```
https://plus.codes/key?referer=openlocationcode.com&key=my_api_key
```

This will respond with:

```javascript
{
  "encryption_message": "Provide the encrypted key in the ekey parameter in your requests",
  "encrypted_key": "kPc3Mn9sTPp1qoDF9yLoR%2FStyQjK6lMRZwJz6HdJQDJVPZJqZSKIDOigxNU%3D",
  "status": "OK"
}
```

Now, instead of setting the `key` parameter in your requests, use the parameter `ekey` with the value of the `encrypted_key` field in the above response. For example:

```
https://plus.codes/api?latlng=14.917313,-23.511313&ekey=kPc3Mn9sTPp1qoDF9yLoR%2FStyQjK6lMRZwJz6HdJQDJVPZJqZSKIDOigxNU%3D
```

If this value is used from any host other than `openlocationcode.com`, the request will fail and an error message displayed:

```javascript
{
  "error_message": "Referer not valid",
  "status": "INVALID_REQUEST"
}
```
### Allowing Multiple Referrers

If you need to use the same encrypted key from multiple different hosts, say `plus.codes` and `v27.plus-codes.dev.site`, include the hosts in the referer like this:

```
https://plus.codes/key?referer=(plus.codes|.*plus-codes.dev.site)&key=my_api_key
```

>The `referer` field will be interpreted as a Go [regular expression](https://golang.org/pkg/regexp/).