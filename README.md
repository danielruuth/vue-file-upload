# vue-file-upload
File upload component

* drag and drop / file chooser
* multiple upload (except for IE9)
* add more files during upload
* overall progress bar
* no dependency

Original fileuploader by Frank Freiburger, modified for Laravel by Daniel Ruuth


## Browser support
Same browser support as [Vue.js 2](https://github.com/vuejs/vue/blob/dev/README.md)


## Example
```
<template>
  <div>
    <upload url='/upload'></upload>
  </div>
</template>
```

## Install
```
npm install --save danielruuth-vue-file-upload
```

## API

### Properties

#### :url <sup>string<sup>

Target url for the uploaded files (post multipart/form-data).


#### :multiple <sup>boolean, default: false<sup>

Allow multiple files to be uploaded simultaneously.


#### :image <sup>boolean, default: false<sup>

Indicate that you wish to upload images.


#### :capture <sup>boolean, default: false<sup>

Indicates that the capture of media directly from the device's environment using a media capture mechanism is preferred.
See [capture attribute](https://www.w3.org/TR/html-media-capture/#the-capture-attribute)


#### :accept <sup>function(filename) returns boolean<sup>

Called before a file is about to be uploaded. Return `false` to reject the upload, otherwise return `true`.

`filename`: The filename (without path) of the file.  

By default any file is accepted.


#### :done <sup>function(status, responseText, feedback)<sup>

Called when a file or a set of files has been uploaded.

`status`: HTTP status of the upload or `undefined` if no status is available (IE9).  
`responseText`: reponse of the server.  
`feedback`: function you can call to give a positive or negative (`true`/`false`) UI feedback about the upload.  

By default, if the property is not defined, a positive feedback is send for HTTP status 2xx and 3xx


#### :data <sup>string<sup>

Extra data sent with files (name=data).


## Credits
* [<img src="https://www.franck-freiburger.com/FF.png" width="16"> Franck Freiburger](https://www.franck-freiburger.com)
* [Daniel Ruuth](http://www.mondayhero.com)
