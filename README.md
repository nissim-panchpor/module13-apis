# Module 11: Accessing Web APIs

You've used Python to load data from external files (either text files or locally-saved `.csv` files), but it is also possible to programmatically download data directly from web sites on the internet. This allows scripts to always work with the latest data available, performing analysis on data that may be changing rapidly (such as from social networks or other live events). Web services may make their data easily accessible to computer programs like Python scripts by offering an **Application Programming Interface (API)**. A web service's API specifies _where_ and _how_ particular data may be accessed, and many web services follow a particular style known as _Representational State Transfer (REST)_. This module will cover how to access and work with data from these _RESTful APIs_.


<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Contents**

- [Resources](#resources)
- [Web APIs](#web-apis)
  - [RESTful Requests](#restful-requests)
    - [URIs](#uris)
      - [Query Parameters](#query-parameters)
      - [Access Tokens and API Keys](#access-tokens-and-api-keys)
    - [HTTP Verbs](#http-verbs)
- [Accessing Web APIs](#accessing-web-apis)
- [JSON Data](#json-data)
  - [Parsing JSON](#parsing-json)
  - [Flattening Data](#flattening-data)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Resources
- [URIs (Wikipedia)](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)
- [HTTP Protocol Tutorial](https://code.tutsplus.com/tutorials/http-the-protocol-every-web-developer-must-know-part-1--net-31177)
- [Programmable Web](http://www.programmableweb.com/) (list of web APIs; may be out of date)
- [RESTful Architecture](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) (original specification; not for beginners)
- [JSON View Extension](https://chrome.google.com/webstore/detail/jsonview/chklaanhfefbnpoihckbnefhakgolnmc?hl=en)
- [Networked Programs (Downey)](https://books.trinket.io/pfe/12-network.html); [Using Web Services (Downey)](https://books.trinket.io/pfe/13-web.html)
- [`requests` documentation](http://docs.python-requests.org/en/master/) ([quickstart](http://docs.python-requests.org/en/master/user/quickstart/))

## Web APIs
An **interface** is the point at which two different systems meet and _communicate_: exchanging informations and instructions. An **Application Programming Interface (API)** thus represents a way of communicating with a computer application by writing a computer program (a set of formal instructions understandable by a machine). APIs commonly take the form of **functions** that can be called to give instructions to programs&mdash;the set of functions provided by a module like `math` or `turtle` make up the API for that module.
While most APIs provide an interface for utilizing _functionality_, other APIs provide an interface for accessing _data_. One of the most common sources of these data apis are **web services**: websites that offer an interface for accessing their data.

- (Technically the interface is just the function **signature** which says how you _use_ the function: what name it has, what arguments it takes, and what value it returns. The actual library is an _implementation_ of this interface).

With web services, the interface (the set of "functions" we can call to access the data) takes the form of **HTTP Requests**&mdash;that is, a _request_ for data sent following the _**H**yper**T**ext **T**ransfer **P**rotocol_. This is the same protocol (way of communicating) used by your browser to view a web page! An HTTP Request represents a message that your computer sends to a web server (another computer on the Internet which "serves", or provides, information). That server, upon receiving the request, will determine what data to include in the **response** it sends _back_ to the requesting computer. With a web browser the response data takes the form of HTML files that the browser can _render_ as web pages; with data APIs the response data will be structured data that we can convert into structures such as lists or dictionaries.

In short, loading data from an Web API involves sending an **HTTP Request** to a server for a particular piece of data, and then receiving and parsing the **response** to that request.

### RESTful Requests
There are two parts to a request sent to an API: the name of the **resource** (data) that you wish to access, and a **verb** indicating what you want to do with that resource. In many ways, the _verb_ is the function you want to call on the API, and the _resource_ is an argument to that function.

#### URIs
Which **resource** you want to access is specified with a **Uniform Resource Indicator (URI)**. A URI is a generalization of a URL (Uniform Resource Locator)&mdash;what we commonly think of as "web addresses". URIs act a lot like the _address_ on a postal letter sent within a large organization such as a university: you indicate the business address as well as the department and the person, and will get a different response (and different data) from Alice in Accounting than from Sally in Sales.

- Note that the URI is known as the **identifier** for the resource, while the **resource** is the actual _data_ that you want to access.

Like postal letter addresses, URIs have a very specific format used to direct the request to the right resource.

![URI Schema](img/uri-schema.png)

Not all parts of the format are required (e.g., you don't need a `port`, `query`, or `fragment`). Important parts of the format include:

- `scheme` (`protocol`): the "language" that the computer will use to communicate the request to this resource. With web services this is normally `https` (**s**ecure HTTP)
- `domain`: the address of the web server to request information from
- `path`: which resource on that web server you wish to access. This may be the name of a file with an extension if you're trying to access a particular file, but with web services it often just looks like a folder!
- `query`: extra **parameters** (arguments) about what resource to access.

The `domain` and `path` usually specify the resource. For example, `www.domain.com/users` might be an _identifier_ for a resource which is a list of users. Note that we can also have "subresources" by adding extra pieces to the path: `www.domain.com/users/joel` might refer to the specific "joel" user in that list.

With an API, the domain and path are often viewed as being broken up into two parts:

- The **Base URI** is the domain and part of the path that is included on _all_ resources. It acts as the "root" for any particular resource. For example, the [Spotify API](https://developer.spotify.com/web-api/endpoint-reference/) has a base URI of `https://api.spotify.com`, while the [UNHCR API](http://data.unhcr.org/wiki/index.php/API_Documentation.html) has a base URI of `http://data.unhcr.org/api/`

- An **Endpoint**, or which resource on that domain you want to access. Each API will have _many_ different endpoints.

  For example, Spotify includes endpoints such as:

  - `/v1/tracks/{id}` to refer to a track with a specific `id` (the `{}` indicate a "variable", in that you can put any id in there not the string `"id"`)
  - `/v1/artists/:id/albums` to refer to a specific artist's albumbs (the `:` is another way to indicate a variable)
  - `/v1/browse/new-releases` to refer to a list of new releases

  The UNHCR includes enpoints such as:

  - `countries/show/:id` to refer to region information about a specific country
  - `stats/persons_of_concern` to refer to statistics about people seeking asylum

Thus we equivalently talk about accessing a particular **resource** and sending a request to a particular **endpoint**.

##### Query Parameters
Often in order to access only partial sets of data from a resource (e.g., to only get some users) we also include a set of **query parameters**. These are like extra arguments that are given to the request function. Query parameters are listed after a question mark **`?`** in the URI, and are formed as key-value pairs (similar to in a _dictionary_). The key (_parameter name_) is listed first, followed by an equal sign **`=`**, followed by the value (_parameter value_). Note that we can't include any spaces in URIs, and don't include quotes around strings! We can include multiple query parameters by putting an ampersand **`&`** between each key-value pair:

```
?firstParam=firstValue&secondParam=secondValue&thirdParam=thirdValue
```

Exactly what parameter names you need to include (and what are legal values to assign to that name) depends on the particular web service. Common examples include having parameters named `q` or `query` for searching, with a value being whatever term you want to search for: in [`https://www.google.com/search?q=programming`](https://www.google.com/search?q=programming), the resource `/search` takes a query parameter `q` with the term you want to search for!

##### Access Tokens and API Keys
Many web services require you to register with them in order to send them requests. This allows them to limit access to the data, as well as to keep track of who is asking for what data (usually so that if someone starts "spamming" the service, they can be blocked).

To facilitate this tracking, many services provide **Access Tokens** (also called **API Keys**)&mdash;these are unique strings of letters and numbers that uniquely identify a particular developer (like a secret password that only works for you). Web services will require you to include your _access token_ as a query parameter in the request; the exact name of the parameter varies, but it often looks like `access_token` or `api_key`. When exploring a web service, keep an eye out for whether they require such tokens.

_Access tokens_ act a lot like passwords, you will want to keep them secret and not share them with others. This means that you **should not include them in your committed files**, so that the passwords don't get pushed to GitHub and shared with the world. The best way to do this in Python is to create a separate script file in your repo (e.g., `apikeys.py`) which includes exactly one line: assigning the key to a variable:

```python
## in `apikeys.py`
my_api_key = "123456789abcdefg"
```

You can then include this file in the **`.gitignore`** list in your repo; that will keep it from even possibly being committed with your code!

In order to access this variable in your "main" script, you can **`import`** this file as a module. The module is the name of the file, and you can access specific variables from it to include them in the "global" scope. Note that importing a file as a module will execute each line of code in that module (that isn't in a "main" block):

```python
## in `myScript.R`

from apikeys import my_api_key

print(my_api_key)  # key is now available!
```

- Note that this assumes the `apikeys.py` file is inside the same folder as the script being run. See [the documentation](https://docs.python.org/3/tutorial/modules.html#the-module-search-path) for details on how to handle other options.

Anyone else who runs the script will simply need to provide a `my_api_key` variable to access the API using their key, keeping everyone's accounts separate and private!


**Additionally** Watch out for APIs that mention using [OAuth](https://en.wikipedia.org/wiki/OAuth) when explaining API keys. OAuth is a system for performing **authentification**&mdash;that is, letting you or a user log into a website from your application (like what a "Log in with Facebook" button does). OAuth systems require more than one access key, and these keys ___must___ be kept secret. See the [`requests_oauthlib`](https://github.com/requests/requests-oauthlib) library for details.


#### HTTP Verbs
When we send a request to a particular resource, we need to indicate what we want to _do_ with that resource! This is done by specifying an **HTTP Verb** in the request. The HTTP protocol supports the following verbs:

- `GET`	Return a representation of the current state of the resource
- `POST`	Add a new subresource (e.g., insert a record)
- `PUT`	Update the resource to have a new state
- `PATCH`	Update a portion of the resource's state
- `DELETE`	Remove the resource
- `OPTIONS`	Return the set of methods that can be performed on the resource

By far the most common verb is `GET`, which is used to "get" (download) data from a web service.

We combine the verb and the endpoint to indicate what we want to do to a particular resource. Thus we can say:

```
GET /v1/search?type=artist&q=bowie
```

in order to GET data from the `/v1/search` resource where `type` is `artist` and `q` is `bowie`&mdash;that is, to download the results of a search for artists named "bowie".

Overall, this structure of treating all data on the web as a **resource** which we can interact with via **HTTP Requests** is refered to as the **REST Architecture** (REST stands for _REpresentational State Transfer_). This is a standard way of structuring computer applications that allows them to be interacted with in the same way as everyday websites. Thus a web service that enables data access through named resources and responds to HTTP requests is known as a **RESTful** service, with a _RESTful API_.


## Accessing Web APIs
So to access a Web API, you just need to send an HTTP Request to a particular URI! You can easily do this with the browser: simple navigate to a particular address (base URI + endpoint), and that will cause the browser to send a GET request and display the resulting data in the browser. For example, you can send a request to search Spotify for artists named "bowie" with:

```
https://api.spotify.com/v1/search?type=artist&q=bowie
```

(Note that the data you'll get back is structued in JSON format. See below for details).

In Python we can most easily send GET requests using the [`requests`](http://docs.python-requests.org/en/master/) module. This is a **third-party** library (not built into Python!), but is included by default with Anaconda and so can just be imported:

```python
import requests
```

This library provides a number of functions that reflect HTTP verbs. For example, the **`get()`** function will send an HTTP GET Request to the specified URI:

```python
response = requests.get("https://api.spotify.com/v1/search?type=artist&q=bowie")  # get bowie releases
```

While it is possible to include _query parameters_ in the URI, `requests` also allows you to include them as a _dictionary_, making it easy to set and change variables (instead of needing to do complex String concatenation):

```python
query_params = {'type':"artist", 'q':"bowie"}
response = requests.get("https://api.spotify.com/v1/search", params = query_params)
```

- _Protip:_ You can acess the `response.url` property to see the URI where a request was sent to. This is useful for debugging: print out the URI that you constructed to paste it into your browser!

If you try printing out the `response` variable, you'll just see a simple output: `<Response [200]>`. The `200` is a an [HTTP Status Code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes): an integer giving details about how the request went (`200` means "OK" the request was answered successefully. Other common codes are `404 Not Found` if the resource isn't there, or `403 Forbidden` if you don't have permission to access the resource, such as if your API key isn't specified correctly).

HTTP Status Codes are part of the **response header**. Each **response** has two parts: the **header**, and the **body**. You can think of the response as a envelope: the _header_ contains meta-data like the address and postage date, while the _body_ contains the actual contents of the letter (the data).

- You can view all of the headers in the `response.headers` property.

Since you're almost always interested in working with the _body_, you will need to extract that data from the response (e.g., open up the envelope and pull out the letter). You can get this content as, for example, the `text` property:

```python
# extract content from response, as a text string
body = response.text
```

## JSON Data
If you print out this text, you'll see what looks like an oddly-formatted dictionary. This is because most APIs will return data in **JavaScript Object Notation (JSON)** format. Like `.csv`, this is a format for writing down structured data&mdash;but while `.csv` files organize data into rows and columns, JSON allows you to organize elements into _sequences_ and _key-value pairs_&mdash;similar to Python lists and dictionaries!

JSON format looks a lot like the literal format for Python _lists_ and _dictionaries_. For example, consider the below JSON notation for Programmer Ada:


```json
{
  "first_name": "Ada",
  "job": "Programmer",
  "salary": 78000,
  "in_union": true,
  "pets": ["rover", "fluffy", "mittens"],
  "favorites": {
    "music": "jazz",
    "food": "pizza",
    "numbers": [12, 42]
  }
}
```

JSON uses curly braces **`{}`** to define sets of key-value pairs (called _objects_, not "dictionaries"), and square brackets **`[]`** to define ordered sequences (called _arrays_, not "lists"). Like in Python, keys and values are separated with colons (**`:`**), and elements are separated by commas (**`,`**). As in Python, key-value pairs are often written on separate lines for readability, but this isn't required.

Note that unlike in Python dictionaries, keys need to be character strings (so in quotes), while values can either be character strings, numbers, booleans (written in all lower-case as `true` and `false`), arrays (list), or other objects (dictionaries).

- Thus JSON can be seen as a way of writing normal Python dictionaries (or lists of dictionaries), just with a few restrictions on key and value type.

JSON response data provided by the `requests` library can automatically be converted into the Python lists and dictionaries you are used to by using the **`.json()`** function:

```python
query_params = {'type':"artist", 'q':"bowie"}
response = requests.get("https://api.spotify.com/v1/search", params = query_params)
data = response.json()
type(data)  # <class 'dict'>
            # may also be <class 'list'> for some resources
```

Note that it is also possible to use Python to convert from (and to!) a "JSON String" into standard data types like _list_ or _dictionaries_ using the **`json`** module:

```python
import json

#convert to dictionary or list
data = json.loads(json_string)

#convert to json
my_data = {'a':1, 'b':2}
my_json = json.dumps(my_data)
```

- Note that these method names end in **`s`**: they are the "load **s**tring" and "dump **s**tring" methods (`load` and `dump` are for files!)

In practice, web APIs often return highly nested JSON objects (lots of dictionaries in dictionaries); you may need to go a "few levels deep" in order to access the data you really care about. Use the `keys()` dictionary method (or your web browser!) to inspect the returned data structure and find the values you are interested in. Note that it can be useful to "flatten" the data into a simpler structure for further access.

- For example: the above Spotify search results are actually found in the `artists.items` element in the result (that is, the `'items'` key inside the `'artists'` dictionary inside the response). So you might extract that list to be able to work with the data that you care about:

  ```python
  query_params = {'type':"artist", 'q':"bowie"}
  response = requests.get("https://api.spotify.com/v1/search", params = query_params)
  data = response.json()
  result_items = data['artists']['items']  # "flatten" into new variable
  print( len(result_items) )  # 20, number of results returned
  ```
