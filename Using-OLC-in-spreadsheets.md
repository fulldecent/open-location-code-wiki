Being able to work with Open Location Code in spreadsheets is, for most people, probably the easiest method to work with them in bulk.

This page explains how you can access the Open Location Code functions from Excel, LibreOffice or Google Spreadsheets.

## Excel/LibreOffice

VBA script and instructions are checked into github [here](https://github.com/google/open-location-code/tree/master/visualbasic).

LibreOffice does sometimes have problems. This may be due to slightly unreliable VBA support in some versions.

## Google Spreadsheets

### Here's one we made earlier

[Here is a spreadsheet](https://docs.google.com/spreadsheets/d/1WN-j2cmAA05AAumpnhowcuAhUJZsAwpfNCy25wpfKL8/edit?usp=sharing) that uses the Open Location Code library and has functions to encode and decode a range of latitude and longitudes. Open the spreadsheet, then choose "File -> Make a copy" to get a copy that you can edit.

### Adding the Open Location Code Library to an existing spreadsheet

1. Open a Google Spreadsheet.
1. Open the script editor (Tools > Script Editor). It will open a new browser tab called _Untitled project_.
1. Click on _Untitled project_, give your project a name in the popup, and click **Save**.
1. Open the libraries panel (Resources > Libraries)
1. Where it says **Add a library** enter
    ```
    1tJz-3SxbCtp8kwaeDyUKFUs3Ui27aOW6c_7uErZ3xZNLGsGPJ3_W2ad7
    ```
    and press **Add**. (This identifies the spreadsheet library that will provide Open Location Code functions.)
1. When the _OpenLocationCode_ library appears, select the highest available version and press **Save**.

### Using the library from your spreadsheet

The Open Location Code spreadsheet library is available for you to use in your `Code.gs` **script**, but you can't use it directly from the spreadsheet yet. You need to add a function in the `Code.gs` panel.

Let's say you have a spreadsheet with some latitudes and longitudes, and you want to encode them into Open Location Codes:

|  | A | B | C |
|--|---|---|---|
| **1** | latitude | longitude | OLC Code|
| **2** | 47.3654605 | 8.5247277 | |
| **3** | 47.376998 | 8.535091 | |
| **4** | 47.379318 | 8.530906 | |

The Open Location Code library has a method to create codes that takes a *range*. The range can have many rows, as long as it has two columns, and you will get an Open Location Code for every row.

To do this, create a function in the `Code.gs` panel that calls the Open Location Code library. Edit your `Code.gs` to be:

```javascript
function encode(range, len) {
  return OpenLocationCode.encoderange(range, len);
}

function decode(range, len) {
  return OpenLocationCode.decoderange(range, len);
}
```

Now, in your spreadsheet, in cell C2, enter `=encode(A2:B4)`. After a couple of seconds, your sheet should contain:

|  | A | B | C |
|--|---|---|---|
| **1** | latitude | longitude | OLC Code|
| **2** | 47.3654605 | 8.5247277 | 8FVC9G8F+5V |
| **3** | 47.376998 | 8.535091 | 8FVC9GGP+Q2 |
| **4** | 47.379318 | 8.530906 | 8FVC9GHJ+P9 |

Because we used a range, we get codes for rows 3 and 4, even though our formula is **only** in row 2.

### Why use the App Script library?

Google Spreadsheets uses [App Script](https://developers.google.com/apps-script/) - a Javascript-like language to add new functions. Calls to App Script functions are sent over the network to Google - this avoids scripts running in your browser (for security), but calling hundreds or thousands of functions may be blocked.

It is possible to just cut and paste the [Javascript](https://github.com/google/open-location-code/blob/master/js/src/openlocationcode.js) implementation into your `Code.gs`, but the implementation of the Open Location Code library that we use above is more efficient because it uses ranges, rather than individual latitude and longitudes.

Note to maintainers: The library that we use above is defined in this [spreadsheet](https://docs.google.com/spreadsheets/d/1Yvew4fPhqG1UmCkGj28dM6G-0vzst_VuT7_8d048in8/edit). You can see the `encoderange()` function here and any other functions provided.

### Feedback

If you have feedback, requests or suggestions, please [open an issue](https://github.com/google/open-location-code/issues/new).