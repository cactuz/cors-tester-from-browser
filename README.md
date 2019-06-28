# cors-tester-from-browser

__Project Title__
A single page html to test if an API is CORS enabled.

__Prerequisites__

Download the html file and edit the html file using any text editor.

Edit the script in the html body, guides are provided:
      dataType: "html",           //edit datatype: json, html, etc.
      url: "http://google.com",   //edit host and port, example: http://198.0.1.20:8443/path

__Running the test__

1) Open the edited html file using google chrome. The script does not work in Firefox. The script will run instantly.
It is recommended that you create a nodeJS static server on your machine to execute the file, you can follow the instructions here:
https://stackoverflow.com/questions/6084360/using-node-js-as-a-simple-web-server

2) The test result will be viewed from the browser console, right click the html page > inspect element or inspect > console.

If CORS is not enabled you should see this message in red font.

Chrome:
Access to XMLHttpRequest at 'http://google.com/' from origin 'null' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

Safari:
Cross-origin redirection to http://www.google.com/ denied by Cross-Origin Resource Sharing policy: Origin null is not allowed by Access-Control-Allow-Origin.

3) _(optional) If you don't see anything_, the jquery library may not have loaded fast enough . If so, please download query locally and refer to it by editing the head  <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.0/jquery.min.js"></script>

__Implementing CORS__

On the requested resource, the backend will need to implement CORS logic. In my case, I added it on the Http OPTIONS method request handler. 
A request for a resource on a different origin will make a pre-flight request to the server using Http OPTIONS method.

Here's a sample using Vertx:
        router.options().handler(HTTPRequestValidationHandler.create()
                .addCustomValidatorFunction(new CorsValidator())))
                .handler(CorsHandler.create("*")
                            .allowedMethods(getAllowedMethods())
                            .allowedHeaders(getAllowedHeaders())
                            .allowCredentials(true));

After the pre flight Http Options call completes, the actual Http call will be made and the server should return 2 fields in the header:
Access-Control-Allow-Origin and (optional) Access-Control-Allow-Credentials depending on the request

        context.response()
                .setStatusCode(statusCode.orElse(HttpResponseStatus.OK.code()))
                .putHeader("content-type", "application/json; charset=utf-8")
                .putHeader("Access-Control-Allow-Origin", "set value here"))
                .putHeader("Access-Control-Allow-Credentials", "true")
                .end(body);

_Acknowledgments_
Thanks to Nick Gibbon for the original material
https://medium.com/pareture/simple-local-cors-test-tool-544f108311c5