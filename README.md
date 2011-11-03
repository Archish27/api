# API Overview
The following denotes the HTTP-based API for [BrowserStack](http://www.browserstack.com). It provides browser-as-a-service for automated cross-browser testing. The goal is to provide a simple service which can easily be used by any browser testing framework.

### Schema
All requests are made to `http://api.browserstack.com/VERSION/` and all returned data is done so in JSON-format. The version this documentation outlines is 1.

    $ curl -i http://api.browserstack.com/1

    HTTP/1.1 200 OK
    Content-Type: application/json
    Status: 200 OK
    X-API-Version: 1
    Content-Length: 2

    {}

All date formats are given in ISO-8601 which can be processed natively with `Date.parse` in compatible JavaScript environments.

### Error Handling
All requests are pre-processed and validated. This section outlines how we handle errors within the API and respond to them.

1. All API requests are validated. The following is an example output for a required parameter that wasn't given.
  
    ```
    HTTP/1.1 422 Unprocessable Entity
    Content-Length: 136
    
    {
      "message": "Validation Failed",
      "errors": [
        {
          "field": "type",
          "code": "required"
        }
      ]
    }
    ```
    
    Possible error codes are `required` and `invalid`.

### Authentication
All methods need to authenticate who you are. Before spawning browser workers and deleting a worker for example. Authentication is done using your username/password within the HTTP request. For example:

    $ curl -u "username:PASSWORD" https://api.browserstack.com/1

> A `401 Unauthorized` response is given if an unauthorized request is made.    

### HTTP Verbs
The API is kept concise and simple by making use of relevant HTTP verbs on each requests. The specifications for these are vague and their use within this API is specific but in general we follow the following rules:

  * `HEAD` - Performs the request to asses the status of a resource and expects no content response.
  * `GET` - Used to retrieve resources.
  * `POST` - Used to create new resources.
  * `PUT` - Used to update resources.
  * `DELETE` - Used to delete resources.



# API Documentation

## Getting Available Browsers
Fetches all available browsers.

    GET /browsers
  
### Output

```javascript
[
  {
    browser: 'ie',
    version: 7.0,
  },
  {
    browser: 'firefox',
    version: 2.0,
  },
  {
    browser: 'chrome',
    version: 14.0,
  } ...
]
```
  


## Create a New Browser Worker
A browser worker is simply a new browser instance. A user can have only one browser worker at a time.

    POST /worker

> This call requires authentication. A `401 Unauthorized` response is given if an unauthorized request is made.

Once a worker has been spawned you can then control this browser instance remotely.

### Parameters
A valid request must contain a `browser`, `version`, and a `url`. `timeout` is optional but defaults to 30 seconds.

#### browser
A valid browser. A list of supported browsers are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.

#### version
A valid browser version. List of supported browser versions are given using the `GET /browsers`. See the _Getting Available Browsers_ above for details.
 
#### (timeout=30)
A number in seconds before the worker is terminated. Set this to 0 to keep the worker alive indefinitely.

> IMPORTANT! If you have set the timeout to 0. Make sure you remember to terminate the worker otherwise it will continue to use up your credits.
> Irrespective of timeout parameter, a browser worker is alive for a maximum time of 30 minutes.

#### (url)
A valid url to navigate the browser to.

> Make sure the url is encoded. JavaScript: encodeURI(url), PHP: urlencode($url), 


### Response
The response will be returned when the worker has been setup and initialized. This involves loading the HTML data or navigating to the url given depending on the setup parameters. Use the id returned to perform any further communications etc.

    HTTP/1.1 200 Success
    Content-Type: application/json
    X-API-Version: 1
    
    {
      "id": "da39a3ee"
    }
    
> Note that an error with message `Not Available` is returned if all instances of that browser are busy and none is currently available.



## Terminating a worker
Use this method to terminate a worker. Useful if you set the worker up to run indefinitely or if you've received all the information needed and you want to save on credit time.

  DELETE  /worker/:id
  
The id is the id returned when you first created the worker. Once called the browser instance will be immediately terminated and will no longer be accessible.

> This call requires authentication. If the request was made unauthorized a `401 Unauthorized` response is given. Alternatively if the authorized user is not the owner of the worker or id does not exist a `403 Forbidden` response is given.
