# API Developer's Guide

### Table of Contents
* [Functionality](#functionality)
* [API Request Format](#api-request-format)
* [Example Requests](#example-requests)
* [Locality Requests](#locality-requests)

> This API is *experimental*. As of December 2016, we are soliciting feedback on the functionality, in order to inform proposals to geocoding API providers such as Google. You can discuss the API in the [public mailing list](https://groups.google.com/forum/#!forum/open-location-code) or create an issue in the [issue tracker](https://github.com/google/open-location-code/issues/new?labels=api&assignee=drinckes).

> If the API needs to be turned off, or there are other important messages, they will be returned in the JSON result in the field `error_message` or described on this page.

## Functionality

The API provides the following functions:

*  Conversion of a location (lat/lng or address) to a plus code (including the bounding box and the center);
*  Conversion of a plus code to the bounding box and center.

Additionally, it will use Google's locality information to:

*  Generate short codes and addresses (such as generating "WF8Q+WF Praia, Cape Verde" for "796RWF8Q+WF");
*  Generate codes for addresses ("1600 Amphitheatre Parkway, Mountain View, CA") or features ("Paris").

The results are provided in JSON format. The API is loosely modeled on the [Google Geocoding
API](https://developers.google.com/maps/documentation/geocoding/intro).

## API Request Format

A Plus codes API request takes the following form:

`https://plus.codes/api?parameters`

**Note**: URLs must be [properly encoded](https://developers.google.com/maps/web-services/overview#BuildingURLs) (specifically, `+` characters must be encoded to `%2B`).

**Required parameter:**

* `address` — The address to encode. This can be any of the following:
  * A latitude/longitude
  * A street address
  * A global code
  * A local code and locality
  * A local code and latitude/longitude

**Optional parameter:**

* `language` — The language in which to return results. This won't affect the global code, but it will affect the names of any features generated, as well as the address used for the local code.
 * If `language` is not supplied, results will be provided in English.

## Example Requests

The following is a simple request with a latitude and longitude:

```
https://plus.codes/api?address=14.917313,-23.511313
```

Or with a global code

```
https://plus.codes/api?address=796RWF8Q%2BWF
```

Or with a local code and locality:

```
https://plus.codes/api?address=WF8Q%2BWF%20Praia%20Cape%20Verde
```

The result from all the above requests is the same, namely:

```javascript
{
  "plus_code": {
    "global_code": "796RWF9Q+4X",
    "geometry": {
      "bounds": {
        "northeast": {
          "lat": 14.917874999999995,
          "lng": -23.51000000000002
        },
        "southwest": {
          "lat": 14.917749999999998,
          "lng": -23.510125000000016
        }
      },
      "location": {
        "lat": 14.917812499999997,
        "lng": -23.510062500000018
      }
    },
    "local_code": "WF9Q+4X",
    "locality": {
      "local_address": "Praia, Cape Verde",
      "locality_place_id": "ChIJV_GnJLaQNQkR8sSzx_UCrKA"
    },
    "best_street_address": "Ave Machado Santos, Praia, Cape Verde"
  },
  "status": "OK"
}
```

The JSON response contains two root elements:

*  `"status"` contains metadata on the request. Since an API key was not provided, it has returned `"NO_API_KEY"`. Other status values are documented [here](https://developers.google.com/maps/documentation/geocoding/intro#StatusCodes).
*  `"plus_code"` contains the plus code information for the location specified in `address`.

> There may be an additional `error_message` field within the response object. This may contain additional background information for the status code.

The `plus_code` structure has the following fields:
*  `global_code` gives the global code for the latitude/longitude
*  `bounds` provides the bounding box of the code, with the north east and south west coordinates
*  `location` provides the center of the bounding box.
*  `best_street_address` is Google's best guess of the street address. This might be a long way away, up to hundreds of meters and shouldn't be relied upon. It may not always be present.

If a locality feature near enough and large enough to be used to shorten the code was found, the following fields will also be returned:
*  `local_code` gives the local code relative to the locality
*  `locality` provides the address of the locality and Google's unique identifier for the feature. See the [place ID overview](https://developers.google.com/places/place-id).

The locality tends to be larger rather than smaller, to make it more likely that it will work on other systems.

### Locality Requests

The `address` parameter may match a large feature, such as:

```
https://plus.codes/api?address=Paris
```

In this case, the returned code may not have full precision. This is because for large features, the returned code will be the **largest** code that fits entirely **within** the bounding box of the feature:

```javascript
{
  "plus_code": {
    "global_code": "8FW4V900+",
    "geometry": {
      "bounds": {
        "northeast": {
          "lat": 48.900000000000006,
          "lng": 2.4000000000000057
        },
        "southwest": {
          "lat": 48.849999999999994,
          "lng": 2.3499999999999943
        }
      },
      "location": {
        "lat": 48.875,
        "lng": 2.375
      }
    }
  },
  "status": "OK"
}
```