# node-stl-thumbnailer
Nodejs thumbnailing service for 3D STL files. Creates beautifully rendered png and jpeg output server-side with no GPU from ASCII and Binary STL's.

## Installation
```npm install --save node-stl-thumbnailer```

## Usage
The following snippet loads a file from the current directory (```./input.stl```), and creates a 500x500 png thumbnail in the current directory called ```./output.png```.

```javascript
var StlThumbnailer = require('node-stl-thumbnailer');
var fs = require('fs');

var thumbnailer = new StlThumbnailer({
	filePath: __dirname + "/input.stl",
	requestThumbnails: [
		{
			width: 500,
			height: 500
		}
	] 	
})
.then(function(thumbnails){
	// thumbnails is an array (in matching order to your requests) of Canvas objects
	// you can write them to disk, return them to web users, etc
	// see node-canvas documentation at https://github.com/Automattic/node-canvas
	thumbnails[0].toBuffer(function(err, buf){      
		fs.writeFileSync(__dirname + "/output.png", buf);
    })
})
```

## Demo web-based thumbnail service
The code below creates a simple express-based web service that accepts the url of a public-on-the-internet STL, and responds with a 500x500 PNG representation of that STL. 

```javascript
// index.js
var StlThumbnailer = require('node-stl-thumbnailer');
var app = require("express")();

app.get('/thumbnailer', function(req, res, next) {
    var thumbnailer = new StlThumbnailer({
        url: req.query.url,           // url OR filePath must be supplied, but not both
        //filePath: "...",            // load file from filesystem
        requestThumbnails: [
            {
                width: 500,
                height: 500,
            }
        ]   
    })
    .then(function(thumbnails){
          // thumbnails is an array (in matching order to your requests) of Canvas objects
          // you can write them to disk, return them to web users, etc
          thumbnails[0].toBuffer(function(err, buf){      
          res.contentType('image/png');
          res.send(buf)
        })
    })
    .catch(function(err){
        res.status(500);
        res.send("Error thumbnailing: "+err);
    });
});

app.listen(3000, function () {
  console.log('Listening on port 3000\n')
});
```

Test your thumbnailer web-app by running ```node index.js``` and navigating to this url in your browser:

http://localhost:3000/thumbnailer?url=http://www.instructables.com/files/orig/F0Q/U1DI/IY4Q5LSH/F0QU1DIIY4Q5LSH.stl

You should see this in your browser:

![Render Output](http://www.instructables.com/files/orig/FK0/HZ6E/IY4Q8PHB/FK0HZ6EIY4Q8PHB.png "Render Output")

## Thumbnail Configuration
```requestThumbnails``` is an array of thumbnail configuration options, most of which are optional. The only required parameters are ```width``` and ```height```. The STL Object will be centered in the frame, and the frame will be chosen to make the objects fit. You can specify the angle of the camera as a vector (which will be normalized), but if left as the default a "front" view from slightly above will be chosen.

Configuration options (default values shown):
```
{
	width: 500,                       // required: output width in pixels
	height: 500,                      // required: output height in pixels
    cameraAngle: [10,50,100],         // optional: specify the angle of the view for thumbnailing. This is the camera's position vector, the opposite of the direction the camera is looking.
    showMinorEdges: true,             // optional: show all edges lightly, even ones on ~flat faces
    metallicOpacity: 0,               // optional: some models, particularly those with non-flat surfaces or very high-poly models will look good with this environment map
    enhanceMajorEdges: true,          // optional: major edges will appear more boldly than minor edges
    shadeNormalsOpacity: 0.4,         // optional: faces will be shaded lightly by their normal direction
    backgroundColor: 0xffffff,        // optional: background color (RGB) for the rendered image
    baseOpacity: 0.7,                 // optional: translucency of the base material that lets you see through it
    baseColor: 0xffffff,              // optional: base color
    baseLineweight: 0.7,              // optional: lineweights will scale to image size, but this serves as a base for that scaling. Larger numbers = heavier lineweights
    lineColor: 0x000000               // optional: color of the linework
}
```

## A note on node-canvas

Note that node-canvas is used under the hood as a rendering target. Node-canvas is backed by Cairo, which can be a little tricky to install. Get started here:
https://github.com/Automattic/node-canvas
