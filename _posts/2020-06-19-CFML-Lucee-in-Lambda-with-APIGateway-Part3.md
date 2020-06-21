---
layout: post
title:  "Lucee - Extentions running cfimage in lambda"
date:   2020-06-20 19:18:01 +1000
categories: aws lambda apigateway cfml lucee java coretto cfimage extentions
---


In the [previouse article on CFML](hhttps://blog.kukiel.dev/posts/CFML-Lucee-in-Lambda-with-APIGateway-Part2.html) I provided a more indepth look at CFML in lambda.  In this post we will explore how to install extentions so we can use functions such as cfimage of the AWS S3 extentions in cfml.

This was actually much easier then expected, I thought at first I'd need to drop in some extra jars, or open the lucee lite jar and add the lex files to the extentions folder then zip it back up, but it was easier then that.  Lucee extentions will be picked up and installed via envrioment variables.  We easily have this in the SAM template so literally 1 line I had the extention installed:

```bash
      Environment:
        Variables:
          FELIX_CACHE_BUFSIZE: 16384
          LUCEE_EXTENSIONS: 'B737ABC4-D43F-4D91-8E8E973E37C40D1B;name=cfimage;version=1.0.0.35,17AB52DE-B300-A94B-E058BD978511E39E;name=cfs3;version=0.9.4.122'
```
In the example abouve 2 extention are installed: Image extension and S3 Resource Extension

The ID comes from:  https://download.lucee.org, this is simple the ID of the extention, name is a name ( can be anything ) version is wel the version.

![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part3/lucee.png "Function")

Here is some example code:

```javascript
<cfscript>

    cfheader( name="Content-Type", value="text/html" );

    if(structKeyExists(url,'img')){
        imgUrl = url['img']
    }else{
        imgUrl="https://avatars1.githubusercontent.com/u/10973141?s=280&v=4"
    }
    
    // Download the image into a CFML image variable.
    cfimage(action="read",name="sourceImage",source=imgUrl);
    // Get the mete data of the file
    cfimage(action="info",structname="original",source=sourceImage);
   
    // Maintain spect ratio.
    newWidth = 320;
    newHeight = (original.height/original.width) * newWidth;

    // resize the image
    cfimage(action="resize",source=sourceImage,name="resized",
        height=newHeight,width=newWidth,quality="1");
    
    // Deep copy the image, cfimage rotate rotates the original aswell as the output?  Meh, seams like an issue?
    copy = Duplicate(resized);
    writeOutput("<strong>Original Image:</strong><br/>" & resized & "<br/>");

    // Turn the image upside down
    cfimage(action="rotate",source=resized,angle="180",name="upsideDownImage");
    writeOutput("<strong>Rotated Image:</strong><br/>" & upsideDownImage & "<br/>");

    // Covert to greyscale
    imageGrayscale(copy);
    writeOutput("<strong>GreyScale Image:</strong><br/>" & copy);

</cfscript>
```

Example output:
![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway-Part3/example.png "Function")

Loading the extention measn the startup time will take a bit of a hit.  Perhaps there is a better way, I suspect the future would be a custom layer for lucee and that shoud enable us to pre-package extentions.