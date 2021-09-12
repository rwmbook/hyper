


# **hyper** : Interactive Hypermedia Shell

_Exploring an interactive REPL/shell for interacting with HTTP-based hypermedia services_

## Summary
The **hyper** utility is a simple command-line style shell/REPL for interacting with an online services/APIs. While a fully-functional HTTP client, **hyper** is especially good at dealing with hypermedia services including [Collection+JSON](http://amundsen.com/media-types/collection/), [SIREN](https://github.com/kevinswiber/siren), and [HAL](https://datatracker.ietf.org/doc/html/draft-kelly-json-hal-08). There are plans to add support for [PRAG+JSON](https://mamund.github.io/prag-json/), [MASH+JSON](https://mamund.github.io/mash-json), and possibly [UBER](http://uberhypermedia.com/) in the future. 

Along with HTTP- and mediatype-aware commands, **hyper** also supports some convience functionality like SHELL commands, configuration file management, and a LIFO stack to handle local memory variabes. 

Importantly, **hyper** is not just a shell/REPL, it is a hypermedia DSL. It encourages users to `think' in hypermedia. Rather than writing complex HTTP queries that look like this (an example that works fine in **hyper**):

```
ACTIVATE http://localhost:8181/task/
  WITH-METHOD PUT
  WITH-BODY title=testing&tags=hyper&completeFlag=false 
  WITH-ENCODING application/x-www-form-urlencoded
  WITH-HEADERS {"if-none-match":"*"}
```

The **hyper** shell can also use mediatype-aware convience commands to locate, parse, fill, and execute inline hypermedia controls. This results in a much more readable **hyper** exeperience:

```
STACK PUSH {
  "title":"testing",
  "tags":"hyper",
  "completeFlag":"false"
}

ACTIVATE http://locahost:8181/home/
ACTIVATE WITH-FORM taskFormAdd WITH-STACK 
```

In both cases, the same work is completed. In the first example, a human can read all the docs and examples and craft a successful HTTP PUT request. This works until the server changes a parameter (e.g. moves from PUT to POST).

In the second example, the **hyper** engine loads available data (it could have been from disk using `STACK LOAD task-record.txt`) and uses identified hypermedia controls (in this case the `taskFormAdd` control) to complete the request. This will continue to work even if HTTP details are changed  (like changing PUT to POST)-- as long as the hypermedia form `taskFormAdd` is included in the response.

## Motivation
The idea for this shell comes from other REPL-style interactive CLIs like `node` and command-line tools like `curl`. You can start a stateful client session by typing `hyper` at the command line. Then you can make an HTTP request (`ACTIVATE`) and manipulate the responses. You can also write hyper commands in a file and pipe this file into hyper for a scripted experience: (`hyper < scripts/sample.txt > scripts/sample.log`).

**Hyper** is "mediatype-aware" -- that is, it recognizes well-known media types and offers convience methods for dealing with them. For example after loading a SIREN response, you can use the following commands:

```
# SIREN example
GOTO http://rwcbook10.herokuapp.com
SIREN LINKS
SIREN ENTITIES
SIREN ACTIONS

GOTO WITH-REL taskFormListByUser WITH-QUERY {"assignedUser" : "alice"}
```

That last command 1) uses the `href` associated with the SIREN action element identified by the `rel:taskFormListByUser`,  2) supplies a querystring argument and 3) makes the HTTP request.

Another way to use **hyper** is to load the data stack with some name/value pairs and then use a named form within the response to execute an action. Like this:

```
# read list 
GOTO http://rwcbook10.herokuapp.com

# add data to the stack and execute the write operation
STACK PUSH {"title":"just another one","tags":"with-test","completeFlag":"false"}
GOTO WITH-FORM taskFormAdd WITH-STACK 

# check the write results using the same stack data
GOTO WITH-FORM taskFormListByTag WITH-STACK
SIREN PATH $..*[?(@property==='tags'&&@.match(/with-test/i))]^
```

<p style="background-color:pink;color:red;"><b>NOTE</b>: Spaces are significant in HYPER commands. The above example shows spaces are properly handed within embeded quotes (<code>STACK PUSH {"title":"just another one","tags":"with-test","completeFlag":"false"}</code>). However, there must be no spaces <i>between</i> elements in the JSON segments.</p>

Note that the client will use whatever URL, HTTP method, and body encoding the server indicates. Also, notice that the client will automatically match up any form fields on the stack to fill in the form. Even when the server changes details (new URL, different method, etc.), the client will be able to handle the write operation without changes.


You can also use JSONPath to query responses:

```
SIREN PATH $.entities.*[?(@property==='id'&&@.match(/rmqzgqfq3d/i))]^.[id,title,href,type]
```
You can also use **hyper** to program a modificaiton of existing records:

```
REQUEST WITH-URL http://rwcbook10.herokuapp.com WITH-ACCEPT application/vnd.siren+json
REQUEST WITH-PATH $.entities[0].href WITH-ACCEPT application/vnd.siren+json
STACK PUSH WITH-PATH $.properties
STACK SET {"tags":"fishing,skiing,hiking"}
REQUEST WITH-FORM taskFormEdit WITH-STACK WITH-ACCEPT application/vnd.siren+json
EXIT

```

In the above example, the **hyper** :

 * Calls the root resource of the serivce asking for a SIREN formatted response
 * Locates the first item in the collection, pulls its HREF value and calls that record
 * Pushes the item properties of the record onto the local stack
 * Updates the `tags` value on the stack to reflect the change
 * Uses the `taskFormEdit` form, fills it with values from the stack and makes the request
 * Once all is done, the script exits
 
Similar options exist for HAL, CollectionJSON, JSON+FORMS, and other formats. These various format types are defined using external plug-ins that can be created and just dropped into the `/plugins/` folder to be loaded at runtime.

## Examples
See the [scripts](scripts/) folder for lots of working examples.

## Futures
Some notes on future enhancements

 * [Conditionals](conditionals.md)

## Feature tracking
This is a work in progress and totally unstable/unreliable. Here the current workplan and status for this project:

 - [x] : Initial CLI loop
 - [x] : support for piped scripts (in and out)
 - [x] : support for **#** - comment lines
 - [x] : support for **VERSION** - returns version info
 - [x] : support for **EXIT|STOP** - halt and exit with 0
 - [ ] : support for **EXIT-ERR** - halt and exit with 1 
 - [x] : support for **EXIT-IF** - halt and exit with 1 if simple condition is met
 - [x] : support for .. INVALID-URL _<url|#$>_ : returns TRUE if the string is NOT a valid URL
 - [x] : support for .. STACK-EMPTY : returns TRUE if there is nothing on the internal stack
 - [x] : support for **CLEAR** - clears the console
 - [x] : support for **SHELL** simple SHELL (bash/dos) support
 - [x] : support for .. LS|DIR _[folder/path]_
 - [x] : support for **PLUGINS** returns list of loaded external plug-ins
 - [x] : support for **CONFIG** (READ) returns NVP of saved config data
 - [x] : support for .. FILE|LOAD _[filename]_ loads config file (defaults to "hyper.config")
 - [x] : support for .. SAVE|WRITE _[filename]_ loads config file (defaults to "hyper.config")
 - [x] : support for .. SET _<{n:v,...}>_ shared config file write
 - [x] : support for .. CLEAR removes all settings
 - [x] : support for .. RESET resets to default settings
 - [x] : support for .. REMOVE _<string>_ removes the named item
 - [x] : support for **STACK**  JSON object LIFO stack
 - [x] : support for .. CLEAR|FLUSH clears all the items from the stack
 - [x] : support for .. PEEK displays the JSON object at the stop of the stack
 - [x] : support for .. PUSH _<{n:v,...}>_ adds a new JSON object to the stack
 - [x] : support for .. PUSH WITH-REPONSE adds a new item on the stack from the top of the _response_ stack
 - [x] : support for .. PUSH WITH-PATH _<path-string|$#>_ adds a new item on the stack which is the result of the JSONPATH
 - [x] : support for .. EXPAND-ARRAY expands the array on the top of the stack into n-items on the stack.
 - [x] : support for .. POP removes the top item from the stack
 - [x] : support for .. LEN|LENGTH returns depth of the stack
 - [x] : support for .. SET _<{"n":"v",...}>_ update the JSON object on the top of the stack
 - [x] : support for .. LOAD|FILE _[filename]_  reads a single JSON object from disk onto the stack (defaults to hyper.stack)
 - [x] : support for .. SAVE|WRITE _[filename]_ writes the top item on the stack to disk (defaults to hyper.stack)
 - [x] : support for .. DUMP _[filename]_ writes the full stack to disk (defaults to hyper.dump)
 - [x] : support for .. FILL _[filename]_ replaces the current stack with contents in disk file (defaults to hyper.dump)
 - [x] : support for **OAUTH** OAuth 2.0 support
 - [x] : support for .. LOAD _[filename]_ loads OAuth config file (defaults to "oauth.env")
 - [x] : support for .. SAVE _[filename]_ loads OAuth config file (defaults to "oauth.env")
 - [x] : support for .. DEFINE _<string>_ _<{n:v,...}>_ Creates an entry in the OAuth configuration
 - [x] : support for .. UPDATE _<string>_ _<{n:v,...}>_ Modifies settings in an existing OAuth configuration
 - [x] : support for .. GENERATE|GEN _<string>_ Generates an access token using the configuration data
 - [x] : support for .. REMOVE _<string>_ removes the named item
 - [x] : support for **ACTIVATE**|CALL|GOTO|GO - makes an HTTP request
 - [x] : support for .. WITH-URL _<url|$#>_ - uses URL to make the request
 - [x] : support for .. WITH-REL _<string|$#>_ - uses HREF value on the associated in-doc element 
 - [x] : support for .. WITH-ID _<string|$#>_ - uses HREF value on the associated in-doc element
 - [x] : support for .. WITH-NAME _<string|$#>_ - uses HREF value on the associated in-doc element
 - [x] : support for .. WITH-PATH _<json-path-string|$#>_ - uses value from JSONPath result as the URL
 - [x] : support for .. WITH-ACCEPT _string|$#>_ - sets the accept header directly
 - [x] : support for .. WITH-HEADERS _<{n:v,...}|$#>_ - request headers
 - [x] : support for .. WITH-OAUTH _<string|$#>_ - Sets the `authorization` header using the named OAUTH token 
 - [x] : support for .. WITH-BASIC _<string|$#>_ - Sets the `authorization` header using the named BASIC config 
 - [x] : support for .. WITH-QUERY _<{n:v,...}|$#>_ - query string args as JSON nvps
 - [x] : support for .. WITH-BODY _<name=value&...|$#>_ - for POST/PUT/PATCH (defaults to app/form-urlencoded)
 - [x] : support for .. WITH-METHOD _<string}|$#>_ - to set HTTP method (defaults to GET)
 - [x] : support for .. WITH-ENCODING _<media-type|$#>_ - to set custom encoding for POST/PUT/PATCH
 - [x] : support for .. WITH-FORMAT - sets `accept` header w/ config value
 - [x] : support for .. WITH-PROFILE - sets `link` profile header w/ config value
 - [x] : support for .. WITH-FORM _<name}|$#>_ - uses the metadata of the named form (URL, METHOD, ENCODING, FIELDS) to construct an HTTP request (SIREN-ONLY)
 - [x] : support for .. WITH-STACK - uses the top level STACK item as a set of vars for other operations (e.g. to fill in forms, supply querystring values, headers, etc.
 - [x] : support for .. WITH-DATA _<name=value&...|$#>_ - for use fill in forms with ad-hoc data
 - [x] : support for **DISPLAY**|SHOW (PEEK) - show saved reponse (from top of the LIFO stack)
 - [x] : support for .. ALL - returns the complete interaction (request, response metadata, response body)
 - [x] : support for .. METADATA|META - returns the response metadata (URL, status, & headers)
 - [x] : support for .. REQUEST - returns request info (URL, method, querystring, body, headers)
 - [x] : support for .. URL - returns actual URL of the response
 - [x] : support for .. STATUS - returns HTTP status of the response
 - [x] : support for .. CONTENT-TYPE - returns HTTP content-type of the response
 - [x] : support for .. HEADERS - returns the complete HTTP header collection of the response
 - [x] : support for .. POP remove response from top of the stack
 - [x] : support for .. LENGTH - returns length of saved stack
 - [x] : support for .. PATH _<JSONPath|$#>_ returns results of a JSONPath query from top-of-stack response
 - [x] : support for .. XPATH _<XMLPath|$#>_ returns results of an XPath query from top-of-stack response
 - [x] : support for **CJ** returns a strong-typed version of response from top of the stack (`vnd.collection+json`)
 - [x] : support for .. METADATA returns metadata array from a collection+JSON response
 - [x] : support for .. LINKS returns links array from a collection+JSON response
 - [x] : support for .. ITEMS returns items array from a collection+JSON response
 - [x] : support for .. QUERIES returns queries array from a collection+JSON response
 - [x] : support for .. TEMPLATE returns template collection from a collection+JSON response
 - [x] : support for .. ERROR|ERRORS returns error object from a collection+JSON response
 - [x] : support for .. RELATED returns the related object from a collection+JSON response
 - [x] : support for .. ID|NAME|REL|TAG _<string|$#>_ returns a single node
 - [ ] : support for .. IDS|RELS|NAMES|TAGS returns a simple list 
 - [x] : support for .. PATH _<JSONPath|$#>_ returns results of a JSONPath query from a collection+JSON response
 - [x] : support for **HAL** returns a strong-typed version of response from top of the stack (`vnd.hal+json`)
 - [x] : support for .. LINKS returns links array from a HAL response
 - [x] : support for .. EMBEDDED returns items array from a HAL response
 - [x] : support for .. ID|REL|KEY|NAME|TAG _<string|$#>_ returns a single node
 - [ ] : support for .. IDS|RELS|KEYSTAGS returns a simple list 
 - [x] : support for .. PATH _<JSONPath|$#>_ returns results of a JSONPath query from a HAL response
 - [x] : support for **SIREN** returns a strong-typed version of response from top of the stack (`vnd.siren+json`)
 - [x] : support for .. LINKS returns links array from a SIREN response
 - [x] : support for .. ACTIONS|FORMS returns actions array from a SIREN response
 - [x] : support for .. ENTITIES returns entities array from a SIREN response
 - [x] : support for .. PROPERTIES returns properties array from a SIREN response
 - [x] : support for .. TAG|CLASS _<string|$#>_ returns nodes associated with the CLASS value
 - [x] : support for .. ID|ENTITY _<string|$#>_ returns an entity associated with the ID
 - [x] : support for .. REL|LINK _<string|$#>_ returns a link associated with the REL
 - [x] : support for .. NAME|FORM|ACTION _<string|$#>_ returns an action associated with the NAME
 - [ ] : support for .. IDS|RELS|NAMES|FORMS|TAGS|CLASSES returns a simple list 
 - [x] : support for .. PATH _<JSONPath|$#>_ returns results of a JSONPath query from a SIREN response
 - [x] : support for **WSTL** returns a strong-typed version of response from top of the stack (`vnd.wstl+json`)
 - [x] : support for .. TITLE returns title string from a WSTL response
 - [x] : support for .. ACTIONS returns actions array from a WSTL response
 - [x] : support for .. DATA returns entities array from a WSTL response
 - [x] : support for .. RELATED returns related object from a WSTL response
 - [x] : support for .. CONTENT returns content object from a WSTL response
 - [x] : support for .. ID|REL|NAME|FORM|TAG|TARGET _<string|$#>_ returns a single node
 - [ ] : support for .. IDS|RELS|NAMES|FORMS|TAGS|TARGETS returns a simple list
 - [x] : support for .. PATH _<JSONPath|$#>_ returns results of a JSONPath query from a WSTL response
 - [x] : support for **FJ** returns a strong-typed version of JSON+FORMS response from top of the stack (`forms+json`)
 - [x] : support for .. METADATA returns metadata array from a response
 - [x] : support for .. LINKS returns links array from a response
 - [x] : support for .. ITEMS returns items array from a response
 - [x] : support for .. ID _<string|$#>_ returns an element (metadata, link, item) associated with the ID
 - [x] : support for .. TAG _<string|$#>_ returns matching nodes
 - [x] : support for .. REL _<string|$#>_ returns a link associated with the REL
 - [x] : support for .. NAME _<string|$#>_ returns an element (metadata, link, property) associated with the NAME
 - [x] : support for .. IDS|NAMES|RELS|FORMS|TAGS returns a simple list 
 - [x] : support for .. PATH _<JSONPath|$#>_ returns results of a JSONPath query from a response
 - [ ] : support for **MASH** returns a strong-typed version of response from top of the stack (`vnd.mash+json`)
 - [ ] : support for .. METADATA returns metadata array from a response
 - [ ] : support for .. LINKS returns links array from a response
 - [ ] : support for .. ITEMS returns items array from a response
 - [ ] : support for .. TAG _<string|$#>_ returns matching nodes
 - [ ] : support for .. ID _<string|$#>_ returns an element (metadata, link, item) associated with the ID
 - [ ] : support for .. REL _<string|$#>_ returns a link associated with the REL
 - [ ] : support for .. NAME _<string|$#>_ returns an element (metadata, link, property) associated with the NAME
 - [ ] : support for .. IDS|NAMES|RELS|FORMS|TAGS returns a simple list 
 - [ ] : support for .. PATH _<JSONPath|$#>_ returns results of a JSONPath query from a SIREN response
 - [ ] : support for **PRAG** returns a strong-typed version of response from top of the stack (`vnd.prag+json`)
 - [ ] : support for .. METADATA returns metadata array from a PRAG response
 - [ ] : support for .. LINKS returns links array from a PRAG response
 - [ ] : support for .. ITEMS returns items array from a PRAG response
 - [ ] : support for .. ID _<string|$#>_ returns an element (metadata, link, item) associated with the ID
 - [ ] : support for .. TAG _<string|$#>_ returns matching nodes
 - [ ] : support for .. REL _<string|$#>_ returns a link associated with the REL
 - [ ] : support for .. NAME _<string|$#>_ returns an element (metadata, link, property) associated with the NAME
 - [ ] : support for .. IDS|NAMES|RELS|FORMS|TAGS returns a simple list 
 - [ ] : support for .. PATH _<JSONPath|$#>_ returns results of a JSONPath query from a SIREN response
 
## TODO Items
Here's a list of things I think need to be done before #HyperLang (as I like to call it) is "complete":

 - [ ] : improved support for VERBOSE setting, possibly levels. See noting (0), see status (1), see errors (2)
 - [ ] : support for URITemplates - required for HAL (and other formats?)
 - [ ] : support for HAL-FORMS - could greatly enhance HAL support
 - [ ] : support for IF-ERROR - error checking (`4xx`, `5xx`)
 - [ ] : support for JUMP _{label}_ - jump to defined label in the script (might be forward-only jumping)
 - [ ] : **Loops** : Do we really want to implement loops? If yes, then it MUST be a single line element. Like list comprehensions in Python. For example `WHILE STACK NOT EMPTY ACTIVATE WITH-FORM taskAddForm WITH-STACK POP`. 
 - [ ] : **Branching** : Would like to avoid branching but might consider `JUMP EXIT` or `JUMP :label|line` (much harder)
 
## Dependencies
These modules are used in the hyper app.

 * https://github.com/ForbesLindesay/sync-request
 * https://www.npmjs.com/package/jsonpath-plus
 * https://www.npmjs.com/package/stack-lifo
 * https://www.npmjs.com/package/html2json
 * https://www.npmjs.com/package/glob
 * https://www.npmjs.com/package/valid-url
 * https://www.npmjs.com/package/xmldom
 * https://www.npmjs.com/package/xpath

## Plug-ins Support
You an author your own **hyper** plug-in and place it in the `/plugins/` folder of the project. It will be automatically loaded at runtime. See [Plug-In Authoring](plugin-authoring.md) for details.
 
## Source Code
You'll find the source code for this utility in the [src](src/) folder. 

**WARNING**: This is all proof-of-concept code and it's pretty messy. I spend time exploring new features much more than I do properly grooming the source code. If you're offended by this kind of behavior, don't look behind the curtain on this one -- I'll only disappoint you[grin]. -- @mamund

