# ActionController
Controller inherits from `ApplicationController` which itself inherits from `ActionController::Base`.

The routings determine which controller and action to run, then Rails **create an instance** of that controller and runs the public method **with the same name as action**.


## Parameters
Data is from query string or POST body, they are all stored in the `params` which acts like a hash.

To store an array, append `[]` to the key name.

    GET /clients?ids[]=1&ids[]=2&ids[]=3

    params[:ids] => ["1", "2", "3"]

To store an hash, include key names

    POST /clients?client[name]=Acme&client[phone]=12345

    params[:client][:name] => "Acme"

JSON/XML parameters are automatically parsed and stored in `params` too.

It also stores params defined in routings.

To setup default params, create `default_url_options` method within controllers, it will affect actions within its scope.

## Session
Specify the session storage with `ActionDispatch::Session`.

Configuration resides in `config/initializers/session_store.rb`

Session data could be accessed through `session`. 

Add through adding data to `session[:key]`, deleting by set them to `nil`.

`flash` is a special part of session which is cleared with each request. So tha data will be avaiable for **both current and next requests**.

Use `flash.keep` to carry over flash messages.

Use `flash.now` to make the flash messages life cycle to current request.

## Cookies
Access through the `cookies` method.

Delete by `cookies.delete(:key)`.

## Filters
They are hooks shared among normal controller actions.

`before_filter` runs any actions, they might intervene a request directly.

`after_filter` has access to the response data but cannot intervene the actions.

`around_filter` is used to run associated actions by *yielding*, such as Rack middleware.

All filters could be skipped through `skip_<type>_filter`

## Request and Response Object
`request` contains an instance of request `AbstractRequest`, available property includes:

    host
    domain
    format
    method
    <method>?
    headers
    port
    protocol
    query_string
    remote_ip
    url
    path_parameters
    query_parameters
    request_parameters

`response` returns a response object, available property includes:

    body
    status
    location
    content_type
    charset
    headers

## Streaming
`send_data` and `send_file` will stream the contents of the file to the end user.

`:disposition` is used to address the whether the file is downloadable.

`:buffer_size` to adjust the block size loaded to memory every time.

`:type` to specify the MIME type.

Use `Mime::Type.register "application/pdf", :pdf` to create RESTful Download.

## Rescue
Use `rescure_from <ErrorType>, :with => <lambda>` to handle exceptions. 
