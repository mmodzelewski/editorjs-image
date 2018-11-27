![](https://badgen.net/badge/CodeX%20Editor/v2.0/blue)

# Image Tool

Image Block for the [CodeX Editor](https://ifmo.su/editor).

![](https://capella.pics/63a03d04-3816-45b2-87b2-d85e556f0066.jpg)

## Features

- Uploading file from the device
- Pasting copied content from the web
- Pasting images by drag-n-drop
- Allows to add border, background
- Allows to stretch image to the container's full-width 

**Note** This Tool requires server-side implementation for file uploading. See [backend response format](#server-format) for more details.
  
## Installation

### Install via NPM

Get the package

```shell
npm i --save-dev codex.editor.image
```

Include module at your application

```javascript
const ImageTool = require('codex.editor.image');
```

### Download to your project's source dir

1. Upload folder `dist` from repository
2. Add `dist/bundle.js` file to your page.

### Load from CDN

You can load specific version of package from [jsDelivr CDN](https://www.jsdelivr.com/package/npm/codex.editor.image).

`https://cdn.jsdelivr.net/npm/codex.editor.image@1.0.0`

Then require this script on page with CodeX Editor through the `<script src=""></script>` tag.

## Usage

Add a new Tool to the `tools` property of the CodeX Editor initial config.

```javascript
var editor = CodexEditor({
  ...
  
  tools: {
    ...
    image: {
      class: ImageTool,
      config: {
        endpoints: {
          byFile: 'http://localhost:8008/uploadFile', // Your backend file uploader endpoint 
          byUrl: 'http://localhost:8008/fetchUrl', // Your endpoint that provides uploading by Url 
        }
      }
    }
  }
  
  ...
});
```

## Config Params 

Image Tool supports these configuration parameters:

| Field | Type     | Description        |
| ----- | -------- | ------------------ |
| endpoints | `{byFile: string, byUrl: string}` | **Required** Endpoints for file uploading. <br> 
Contains 2 fields: <br> __byFile__ - for file uploading <br> __byUrl__ - for uploading by URL |
| field | `string` | (default: `image`) Name of uploaded image field in POST request |
| types | `string` | (default: `image/*`) Mime-types of files that can be [accepted with file selection](https://github.com/codex-team/ajax#accept-string).|
| additionalRequestData | `object` | Object with any data you want to send with uploading requests |
| captionPlaceholder | `string` | (default: `Caption`) Placeholder for Caption input |
| buttonContent | `string` | Allows to override HTML content of «Select file» button |

## Tool's settings

![](https://capella.pics/c74cdeec-3405-48ac-a960-f784188cf9b4.jpg)

1. Add border

2. Stretch to full-width

3. Add background

## Output data

This Tool returns `data` with following format

| Field          | Type      | Description                     |
| -------------- | --------- | ------------------------------- |
| file           | `object`  | Uploaded file data. Any data got from backend uploader. Always contain the `url` property |
| caption        | `string`  | image's caption                 |
| withBorder     | `boolean` | add border to image             |
| withBackground | `boolean` | need to add background          |
| stretched      | `boolean` | stretch image to screen's width |


```json
{
    "type" : "image",
    "data" : {
        "file": {
            "url" : "https://www.tesla.com/tesla_theme/assets/img/_vehicle_redesign/roadster_and_semi/roadster/hero.jpg"
        },
        "caption" : "Roadster // tesla.com",
        "withBorder" : false,
        "withBackground" : false,
        "stretched" : true
    }
}
```

## Backend response format <a name="server-format"></a>

This Tool works by one of following schemes:

1. Uploading files from the device
2. Uploading by URL (handle image-like URL's pasting)
3. Uploading by drag-n-drop file 

### Uploading files from device <a name="from-device"></a>

Scenario:

1. User select file from the device
2. Tool sends it to **your** backend (on `config.endpoint.byFile` route) 
3. Your backend should save file and return file data with JSON at specified format.
4. Image tool shows saved image and stores server answer

So, you can implement backend for file saving by your own way. It is a specific and trivial task depending on your
environment and stack.

Response of your uploader **should** cover following format:

```json5
{
    "success" : 1,
    "file": {
        "url" : "https://www.tesla.com/tesla_theme/assets/img/_vehicle_redesign/roadster_and_semi/roadster/hero.jpg",
        // ... and any additional fields you want
    }
}
```

**success** - uploading status. 1 for successful, 0 for failed

**file** - uploaded file data. **Must** contain an `url` field with full public path to the uploaded image. 
Also, can contain any additional fields you want to store. For example, width, height, id etc. 
All additional fields will be saved at the `file` object of output data.    

### Uploading by pasted URL

Scenario:

1. User pastes and URL for image file to the Editor
2. Editor pass pasted string to the Image Tool
3. Tool sends it to **your** backend (on `config.endpoint.byUrl` route) via 'url' POST-parameter 
3. Your backend should accept URL, **download and save the original file by passed URL** and return file data with JSON at specified format.
4. Image tool shows saved image and stores server answer

Response of your uploader should be at the same format as described at «[Uploading files from device](#from-device)» section
 
