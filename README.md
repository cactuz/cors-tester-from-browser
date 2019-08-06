# cors-tester-from-browser

__Project Title__
A single page html to test if an API is CORS enabled.

__Prerequisites__

Download the html file and edit the html file using any text editor.

Edit the script in the html body, guides are provided:<br>
```
      dataType: "html",           //edit datatype: json, html, etc.
      url: "http://google.com",   //edit host and port, example: http://198.0.1.20:8443/path
```      

__Set-up and Launching the test__

1) In my experience running this in a browser will not work. The server receiving the ajax 
request will only receive null in the request origin resulting in CORS rejection. I recommended that you create a nodeJS static server on your machine to execute the file. 
    
    1.1) Install connect and serve-static with NPM
    
    ```$ npm install connect serve-static```

    1.2) Create server.js file with this content:
    ```var connect = require('connect');
       var serveStatic = require('serve-static');
       connect().use(serveStatic(__dirname)).listen(8080, function(){
           console.log('Server running on 8080...');
       });
    ```
    
    1.3 Run with Node.js
    ```
    $ node server.js
    ```

2) Open the included html using Chrome or Safari (will not work with Firefox), make sure the 
port is as set in step 1.2.
    ```
    http://localhost:8080/yourfile.html.
    ```


__Checking the Test Result__

From your browser the reply should display if the HTTP request is successful. Otherwise, right 
click the page and open > Inspect Element > Console where you will view the error messages.<br>
If CORS is not enabled you should see these messages in red font. The message will change as you 
make adjustments to your applications CORS handling.

Chrome:
Access to XMLHttpRequest at "http://google.com/" from origin 'null' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

Safari:
Cross-origin redirection to "http://www.google.com/" denied by Cross-Origin Resource Sharing 
policy: Origin null is not allowed by Access-Control-Allow-Origin.

3) _(optional) If you don't see anything_, the jquery library may not have loaded fast enough . If so, please download query locally and refer to it by editing the head  <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.4.0/jquery.min.js"></script>

__Implementing CORS__

On the requested resource, the backend will need to implement CORS handling. 

I had a Vertx server and I was able to implement it in 2 ways:
1) I added it on the Http OPTIONS method request handler. 
A request for a resource on a different origin will make a pre-flight request to the server using Http OPTIONS method.
```
Here's a sample using Vertx:
        router.options().handler(HTTPRequestValidationHandler.create()
                .addCustomValidatorFunction(new CorsValidator())))
                .handler(CorsHandler.create(".*.")
                            .allowedMethods(getAllowedMethods())
                            .allowedHeaders(getAllowedHeaders())
                            .allowCredentials(true));
```
After the pre flight Http Options call completes, the actual Http call will be made and the server should return 2 fields in the header:
Access-Control-Allow-Origin and (optional) Access-Control-Allow-Credentials depending on the request
```
        context.response()
                .setStatusCode(statusCode.orElse(HttpResponseStatus.OK.code()))
                .putHeader("content-type", "application/json; charset=utf-8")
                .putHeader("Access-Control-Allow-Origin", "set value here"))
                .putHeader("Access-Control-Allow-Credentials", "true")
                .end(body);
```
2) I also used this one:
```
router.route().handler(CorsHandler.create("((http://)|(https://))localhost\:\d+")  
.allowedMethod(HttpMethod.GET)
.allowedMethod(HttpMethod.POST)
.allowedMethod(HttpMethod.OPTIONS)
.allowCredentials(true)
.allowedHeader("Access-Control-Allow-Method")
.allowedHeader("Access-Control-Allow-Origin")
.allowedHeader("Access-Control-Allow-Credentials")
.allowedHeader("Content-Type"));  //makes sure you add other headers expected in the request
```
You will need to provide a regex String on CorsHandler.create("Regex String Here"). That Regex String will need to be a valid Java regex that will work on Pattern.compile("Regex String Here"). So if you want to allow any protocol:host:port, aka "*", through CORS handling you can use.
```
router.route().handler(CorsHandler.create(".*.")  //note the "." surrounding "*"
```
If you want a fine-grained control of allowed protocol:host:port, you have flexibility with the Regex String. Example: I want to set CORS handling for either http:// or https:// from localhost and any port. It will look like the one shown.


_Acknowledgments_
Thanks to Nick Gibbon for the original html test material
https://medium.com/pareture/simple-local-cors-test-tool-544f108311c5

For the static server:
https://stackoverflow.com/questions/6084360/using-node-js-as-a-simple-web-server
