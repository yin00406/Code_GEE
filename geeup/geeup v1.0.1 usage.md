# geeup v1.0.1 usage

*<u>Using Colab with geeup often results in errors</u>*

## Install geeup

```bash
pip install git+https://github.com/samapriya/geeup.git
```

## GEE authentication

```bash
earthengine authenticate
```

## Cookie set up

- Install the Chrome Extension: [Copy cookies](https://chrome.google.com/webstore/detail/copy-cookies/jcbpglbplpblnagieibnemmkiamekcdg/related)
-  Log into [code.earthengine.google](https://code.earthengine.google.com/). Use the extension to copy the necessary cookies (click the extension after opening the page). The extension will capture the required cookies for Earth Engine.
- Run the `cookie_setup` method and parse and save cookies

```bash
geeup cookie_setup
```

## Check quota

```bash
geeup quota # legacy asset
```

## Check task status

```bash
geeup tasks
```

## Renaming before data uploading

It ensures that filenames adhere to GEE guidelines, which include allowing only hyphens, underscores, letters, and numbers, with no spaces.

```bash
geeup rename --input PATH-UPLOAD
```

## zip shapefiles before uploading

```bash
geeup zipshape --input PATH-SHAPEFILE --output PATH-ZIPPEDFILE
```

## Batch upload tables

```bash
geeup tabup --source PATH-ZIPPEDFILE --dest PATH-GEE -u GMAIL
# --dest: Full path for upload to Google Earth Engine folder, e.g., users/pinkiepie/myfolder
```

