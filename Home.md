# Welcome to the Open Location Code wiki!

The wiki is where you can find out information about using the software, the codes, or the API.

## Plus codes for the codes, Open Location Code for the technology

Since the codes always have a plus sign in them (that's how you can recognise them), we use **plus codes** as the term for the codes themselves, and when talking about the project generally. See the [Naming Guidelines](https://github.com/google/open-location-code/wiki/Naming-guidelines) article for more information.

Open Location Code is how we refer to the technology and the code libraries.

## Displaying plus codes

The first four digits of the code are termed the area code, the remaining digits the local code. When displaying a full code, the local code part should be bolded to make this distinction plain: 796R**WF8Q+WF**

When displaying the local code on it's own or in combination with a locality, this is not necessary: WF8Q+WF Praia, Cape Verde

## Supporting plus codes in your app or site

### Online support

If you want to support plus codes in a website or app, the big question is whether you can rely on being online? If the answer is yes, by far the easiest way to add support is to integrate with the [plus codes API](https://github.com/google/open-location-code/wiki/Plus-code-API).

> Note that the API is experimental and may be turned off. Including an email address in your requests will ensure you can be contacted.

Using the API, in addition to the global code ("796R**WF8Q+WF**"), you can obtain the local code and address ("WF8Q+WF Praia, Cape Verde") from a latitude/longitude ("14.917313,-23.511313") with a single request. It also supports searching for mixed addresses, so if someone enters "9G8F+6X Zürich, Switzerland" you can make a single call to the API and discover they mean the Google office in Zürich.

These short codes can be used on Google search and in Maps and are more likely to be accepted by your users because they are easier to use and remember than the long ones.

Calls to the API with addresses, or to generate the short codes and addresses, use Google's Geocoding API and require a Google API key. This is free to use up to a [daily quota](https://developers.google.com/maps/documentation/geocoding/usage-limits). If you need more requests, paid plans are available.

### Offline support

If you need to create and decode the codes offline, you will need to take one of the implementations and integrate it into your site or app. For example, if you're working on an [Android](https://github.com/google/open-location-code/tree/master/android_demo) app, and you want to be able to convert from a plus code to a location or vice versa without a data connection, you're going to need the [Java library](https://github.com/google/open-location-code/blob/master/java/com/google/openlocationcode/OpenLocationCode.java). See [Support OLC in your app](https://github.com/google/open-location-code/wiki/Supporting-OLC-in-your-app) for more information on working with the offline libraries.

If you're working on a website, there is a [Javascript](https://github.com/google/open-location-code/tree/master/js) library to convert from latitude/longitude to codes, and back again. But if you are working on a website, you are probably online, and instead of the library you can (and should) use the API.
