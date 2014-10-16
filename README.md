# ee-image

smart, smooth & fast image resizing and cropping with face detection

**the library is under development, it should be stable in mod june 2014**

## installation

	npm install ee-image-worker


## build status

[![Build Status](https://travis-ci.org/eventEmitter/ee-image-worker.png?branch=master)](https://travis-ci.org/eventEmitter/ee-image-worker)


## usage

Initializing the Worker Pool


	var ImageWorker = require('ee-image-worker');


	/**
	 * new ImageWorker() returns an image worker poll instance
	 *
	 * @param <Number> worker, optional, defaults to 10, the number of parallel image workers to run
	 * @param <Number> queue, optional, defaults to 1000, the number of jobs that are waiting 
	 * 				   before starting to fail
	 * @param <String> filter, options, defaults to «lanczos», the filter to use for resizing
	 */
	var workers = new ImageWorker({
		  workers: 	10
		, queue: 	1000
		, filter: 	'lanczos'
	});



Loading an Image

	/**
	 * the loadImage() method returns an new Image instance
	 * Arguments passed to this method can be in any order.
	 *
	 * @param <Buffer> image, raw jpeg / png data
	 * @param <Array> faces, optional, array conataining the faces on the image
	 * @param <String> filter, options, defaults to the value set on the ImageWorker class
	 */
	var image = workers.loadImage(image, faces, filter);


Cropping

	/**
	 * the crop() method returns the reference to the image instance( support for method chaining )
	 * the crop method does not execute immediately, the command gets executed when the «toBuffer»
	 * is called
	 *
	 * @param <Number> top, optional, defaults to 0
	 * @param <Number> left, optional, defaults to 0
	 * @param <Number> height, optional, defaults to image.height
	 * @param <Number> width, optional, defaults to image.width
	 */
	image.crop({
		  top: 70
		, left: 50
		, height: 300
		, width: 400
	});


Resizing

	/**
	 * the resize() method returns the reference to the image instance( support for method chaining )
	 * the resize method does not execute immediately, the command gets executed when the «toBuffer»
	 * is called
	 *
	 * @param <Number> height, optional, defaults to image.height
	 * @param <Number> width, optional, defaults to image.width
	 * @param <String> mode, optional, defaults to face if facedata was passed to the «loadImage»
	 * 				   or crop when there is no face data. 
	 * 				   fit: 	the image is fitted inside a frame, so that there will be transparent 
	 * 							pixels on top & the bottom or on both sides
	 * 				   crop: 	pixels are removed either on top & the bottom or on both sides of 
	 * 							the image
	 * 				   distort: the image is distorted into the box
	 * 				   face: 	first faces will be detect if no faces were passed to the «loadImage»
	 * 							method, then the image will be cropped at the optimal position so 
	 * 							that the most relevant parts of the images will be on the new image 
	 */
	image.resize({
		  height: 	1000
		, width: 	300
		, mode: 	"fit|crop|distort|face"
	});


Image Stats

	/**
	 * the stat() method returns the image dimensions and additional availabel image meta data
	 */
	image.stat();


Face Detection

	/**
	 * the faces() method returns the reference to the image instance( support for method chaining )
	 * it executes face detection on the image, but only if no face data was passed to the 
	 * «loadImage» method and no prior call to the «faces» method was done.
	 * if the faces method is called more than once all succesive calls will either return the 
	 * cached data or wait until the first call to the «faces» method was executed.
	 *
	 * @param <Function> callback, function(err, faces){}, err = Error, faces = Array
	 */
	image.faces(callback);


Encode Image

	/**
	 * the toBuffer() method returns the reference to the image instance( support for method 
	 * chaining ). it executes all cached commands and calls the callback when finished.
	 * Arguments passed to this method can be in any order.
	 *
	 * @param <String> format, optional, defaults to «jpp», can be one of «png» and «jpg» 
	 * @param <Number> format, optional, defaults to 75, the jpeg quality
	 * @param <Function> callback, function(err, newImage, faces){} err = Error, newImage = buffer, 
	 * 				     faces = Array, faces is only present if face detection was used.
	 */
	image.toBuffer(format, quality, callback);

#Appendix
The library will be rewritten to support new features, the interface will look similar:

    .pad(color, left, top, right, bottom)
    .crop(left, top, width, height)
    .resize(width, height, strategy, focus) // planned strategies are fit|crop|strict|carve
    .stat()
    .encode(format, quality, options)
    .toBuffer(callback)

Focus by default is at the center of the image and will be adjusted according to the chosen strategy.

##Supported image formats
Support depends on the installed libraries

  - jpeg
  - png
  - tiff
  - webp
  
No `.gif` support!

##Error handling
To have a consistent error handling we'll wrap `picha's` errors.

###Unsupported image file
Can be because of an unexpected Buffer format, or a missing library (such as libwebp). We'll probably have to check this
upfront. Introduce error codes to distinguish properly.

##Strategies
We differentiate two image resizing types:

  - `resize(width, height, filter)` applies a change of dimension without taking the image dimensions into account any precomputations
  - `scale(width, height, strategy, options)` applies a change of dimension following a specific strategy. Strategies can be one of the following
    - `resize`  is an ordinary resizing
    - `fit`     scales the image to fit into a bounding box
    - `fill`    scales and crops an image to fill a bounding box
    - `carve`   applies seam carving