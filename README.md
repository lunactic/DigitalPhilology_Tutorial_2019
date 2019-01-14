# Introduction
 Time: 30-45 min

In this first part the goal is to execute a small workflow using methods on DIVAServices.
You can find prepared code in ocr.html and the solutions in ocr_solution.html.

The workflow is as follows:

  - Upload the selected input image to DIVAServices
  - Binarize the image using OCRopus Binarization and store the result on DIVAServices
  - Segment the binary image into text lines using OCRopus page segmentation
  - Recognize the text using the default english recognition model


# Tasks

## Task 1 - Testing the application
Open the file ocr_solution with your favorite modern web browser (Edge, Firefox, Chrome).

Upload the image example.png provided in this repository and recognize the contents using the "Recognize!" button.

The application shows the intermediate outputs at every step.

## Task 2 - Read some code
Open the ocr.html file with your favourite code/text editor (JavaScript highlighting will help). All important code information can be found at the top of the file within the \<script>\</script> tags.

Locate the `binarize` method, and read its code. This method will perform the binarization from OCRopus on DIVAServices and polls for the result. The process for this is the following:

1. Generate the POST request
  
    First the request body for the POST  request is generated. The request body will look like this:

    ``` JSON
    {
      "parameters": {
        "enableSkew":true,
        "maxskew":2.0,
        "skewsteps":8.0
      },
      "data": [
        {
          "inputImage": identifier
        }
      ]
    }
    ```
    where `identifier` refers to the generated identifier in the `uploadImage` method (this upload process is automatically triggered).

2. Execute the POST request

    The POST request is sent to the URL of the OCRopus binarization method on DIVAServices with:

    ``` JavaScript
    fetch("http://divaservices.unifr.ch/api/v2/binarization/ocropusbinarization/1", {
      method: "POST",
      body: data,
      headers: new Headers({ 'content-type': 'application/json' })
    }).
    ```
    This will start the execution of the binarization on DIVAServices.

3. Process the response and poll for results

    DIVAServices will respond to the start process with data like this:

  ``` JSON
  {
    "results": [
    {
      "resultLink": "http://divaservices.unifr.ch/api/v2/results/redsugaryindianspinyloach/data_0/data_0.json"
    }
    ],
    "collection": "redsugaryindianspinyloach",
    "resultLink": "http://divaservices.unifr.ch/api/v2/results/redsugaryindianspinyloach.json",
    "message": "This url is available for 24 hours",
    "status": "done",
    "statusCode": 202
  }
  ```
  We are interested in the `resultLink` inside the `results` array. We poll this url using the `getResult(url)` method to poll every second to check if the result is available.

  The result itself will look like this:

  ``` JSON
{
  "output": [
    {
      "file": {
        "name": "ocropusBinaryImage",
        "type": "image",
        "description": "Generated Binary Image",
        "options": {
          "colorspace": "binary",
          "visualization": true
        },
        "mime-type": "image/png",
        "url": "http://divaservices.unifr.ch/api/v2/results/redsugaryindianspinyloach/data_0/0001.bin.png"
      }
    },
    {
      "file": {
        "mime-type": "text/plain",
        "type": "log",
        "url": "http://divaservices.unifr.ch/api/v2/results/redsugaryindianspinyloach/data_0/logFile.txt",
        "name": "logFile.txt",
        "options": {
          "visualization": false
        }
      }
    }
  ],
  "status": "done",
  "resultLink": "http://divaservices.unifr.ch/api/v2/results/redsugaryindianspinyloach/data_0/data_0.json",
  "collectionName": "redsugaryindianspinyloach"
}
  ```
  From this result we can extract the url to the binary image and use this to set the `src` of the `bin_image` element in the HTML page.

  At the bottom of the method we re-upload the binary image to DIVAServices, a step currently necessary to make the binary image available for further processing.
  ## Task 3 - Make some changes

  Your task is to complete the `segment()` method such that it will work properly.

  Hints:
   - The URL for the page segmentation is: http://divaservices.unifr.ch/api/v2/segmentation/ocropuspagesegmentation/1
   - The method takes no parameters
   - the `identifier` is already correctly set to be reused
   - use a similar call structure for the method as in the binarization

  ## Task 4 - Optional

  As an additional task if you feel like it. Try to make us of another binarization method and try to compare results.

  On DIVAServices there are at least two other methods available:
   - Otsu Binarization at: http://divaservices.unifr.ch/api/v2/binarization/otsubinarization/1
   - Sauvola Binarization at: http://divaservices.unifr.ch/api/v2/binarization/sauvolabinarization/1
     - Sauvola Binarization takes two parameters:
       - "radius": an integer number between 2 and 45 to define the radius of the local region
       - "thres_tune": a number between 0.1 and 0.6 to tune the threshold
    