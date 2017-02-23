# API Developer's Guide

### Table of Contents
* [Functionality](#functionality)
* [API Request Format](#api-request-format)
* [Example Requests](#example-requests)
* [Locality Requests](#locality-requests)
* [API Keys](#api-keys)

> This API is *experimental*. As of December 2016, we are soliciting feedback on the functionality, in order to inform proposals to geocoding API providers such as Google. You can discuss the API in the [public mailing list](https://groups.google.com/forum/#!forum/open-location-code) or create an issue in the [issue tracker](https://github.com/google/open-location-code/issues/new?labels=api&assignee=drinckes).

> If the API needs to be turned off, or there are other important messages, they will be returned in the JSON result in the field `error_message` or described on this page.

> Feb/March 2017: Google API keys will be required for the generation of short codes and searching by address. See the [API Keys](#api-keys) section and the `ekey` parameter. Currently key encryption and usage can be tested. Keys will be **required** in API requests from March 6 2017.

## Functionality

The API provides the following functions:

*  Conversion of a location (lat/lng or address) to a plus code (including the bounding box and the center);
*  Conversion of a plus code to the bounding box and center.

Additionally, it can use Google's locality information to:

*  Generate short codes and addresses (such as generating "WF8Q+WF Praia, Cape Verde" for "796RWF8Q+WF");
*  Generate codes for addresses ("1600 Amphitheatre Parkway, Mountain View, CA") or features ("Paris").

The results are provided in JSON format. The API is loosely modeled on the [Google Geocoding
API](https://developers.google.com/maps/documentation/geocoding/intro).

## API Request Format

A Plus codes API request takes the following form:

`https://plus.codes/api?parameters`

**Note**: URLs must be [properly encoded](https://developers.google.com/maps/web-services/overview#BuildingURLs) (specifically, `+` characters must be encoded to `%2B`).

**Required parameter:**

* `address` — The address to encode. This can be any of the following (if the `ekey` parameter is also provided):
  * A latitude/longitude
  * A street address
  * A global code
  * A local code and locality
  * A local code and latitude/longitude

**Recommended parameters:**

* `ekey` — An encrypted Google API key. See [API Keys](#api-keys). If this parameter is omitted, only latitude/longitude and global codes can be used in the `address` parameter, and locality information will not be returned.
* `email` — Provide an email address that can be used to contact you.
* `language` — The language in which to return results. This won't affect the global code, but it will affect the names of any features generated, as well as the address used for the local code.
 * If `language` is not supplied, results will be provided in English.

## Example Requests

The following is a simple request with a latitude and longitude:

```
https://plus.codes/api?address=14.917313,-23.511313&ekey=YOUR_ENCRYPTED_KEY&email=YOUR_EMAIL_HERE
```

Or with a global code

```
https://plus.codes/api?address=796RWF8Q%2BWF&ekey=YOUR_ENCRYPTED_KEY&email=YOUR_EMAIL_HERE
```

Or with a local code and locality:

```
https://plus.codes/api?address=WF8Q%2BWF%20Praia%20Cape%20Verde&ekey=YOUR_ENCRYPTED_KEY&email=YOUR_EMAIL_HERE
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

*  `"status"` contains metadata on the request. Other status values are documented [here](https://developers.google.com/maps/documentation/geocoding/intro#StatusCodes).
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

If the `ekey` encrypted key is not provided the following fields will not be returned:
*  `local_code`
*  `locality`
*  `local_address`
*  `locality_place_id`
*  `best_street_address`

### Locality Requests

The `address` parameter may match a large feature, such as:

```
https://plus.codes/api?address=Paris&ekey=YOUR_ENCRYPTED_KEY&email=YOUR_EMAIL_HERE
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

## API Keys

The Plus Codes API uses [Google Maps Geocoding API](https://developers.google.com/maps/documentation/geocoding/intro)
to search for and return addresses. If you want to be able to search by address, or by short codes with localities, or you would like short codes and localities to be returned, you *must* provide an API key in your request.

To get a Google API key refer [here](https://developers.google.com/maps/documentation/geocoding/start#get-a-key).

Important:
*  The API key *must* not have any restrictions
*  The API key *must* have the Google Maps Geocoding API enabled

### Securing your API key

Google API keys provide access to different APIs, and most have a free daily quota allowance. If other people obtain your key, they can consume your free quota, and if you have billing enabled, can incur charges.

The normal way of securing API keys is setting restrictions on the host calling the Google API or the IP addresses that can make requests. Because the calls to the Google API are being done from the plus codes server, not your site, these methods don't work.

As a solution, the plus codes API let's you encrypt Google API key together with your host, and you use the encrypted value in the requests. When the plus codes API receives this, it decrypts the key and checks that the host making the request matches the encrypted value.

If the host matches, then the requests are done. If the host does not match, a failure message is returned.

For example, to protect the Google API key `my_google_api_key` for the site `openlocationcode.com`, encrypt it like this:

```
https://plus.codes/api?referer=openlocationcode.com&encryptkey=my_google_api_key
```

The plus codes API will respond with:

```javascript
{
  "encryption_message": "Provide the encrypted key in the ekey parameter in your requests",
  "encrypted_key": "Nn%2BzIy2LOz7sptIe4tkONei3xfO7MUPSyYdoNanqv%2F1wgDaGvUryUDt8EPXRS4xzP%2F0b04b3J6%2BzFeeu",
  "status": "OK"
}
```

The `encrypted_key` field gives the value to specify for the `ekey` argument.

If this value is used from any host other than `openlocationcode.com`, the request will fail and an error message displayed.

### Allowing Multiple Referrers

If you need to use the same encrypted key from multiple different hosts, say example1.com and example2.com, include the hosts in the referer like this:

```
https://plus.codes/key?referer=example1.com|example2.com&key=my_api_key
```
