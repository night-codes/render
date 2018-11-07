Render is a package that provides functionality for easily rendering JSON, XML, text, binary data, and HTML templates. This package is a high-modified fork of https://github.com/unrolled/render in order to work with fasthttp, added support for gzip compression also, and a `{{ render "path/templateName"}}` func for html/template package.

## Usage
Render can be used with pretty much any web framework providing you can access the `fasthttp.RequestCtx` from your handler. The rendering functions simply wraps Go's existing functionality for marshaling and rendering data.

- HTML: Uses the [html/template](http://golang.org/pkg/html/template/) package to render HTML templates.
- JSON: Uses the [encoding/json](http://golang.org/pkg/encoding/json/) package to marshal data into a JSON-encoded response.
- XML: Uses the [encoding/xml](http://golang.org/pkg/encoding/xml/) package to marshal data into an XML-encoded response.
- Binary data: Passes the incoming data straight through to the `fasthttp.RequestCtx`.
- Text: Passes the incoming string straight through to the `fasthttp.RequestCtx`.

~~~ go
// main.go
package main

import (
    "encoding/xml"
    "github.com/valyala/fasthttp"

    "github.com/fasthttp-contrib/render"
)

type ExampleXml struct {
    XMLName xml.Name `xml:"example"`
    One     string   `xml:"one,attr"`
    Two     string   `xml:"two,attr"`
}

func main() {
    r := render.New()


	requestHandler := func(ctx *fasthttp.RequestCtx) {
  		switch string(ctx.Path()) {
  			case "/data":
     	 		 r.Data(ctx, fasthttp.StatusOK, []byte("Some binary data here."))
			case "/text":
     	 		   r.Text(ctx, fasthttp.StatusOK, "Plain text here")
  			case "/json":
     	 		   r.JSON(ctx, fasthttp.StatusOK, map[string]string{"hello": "json"})
			case "/jsonp":
     	 		 r.JSONP(ctx, fasthttp.StatusOK, "callbackName", map[string]string{"hello": "jsonp"})
  			case "/xml":
     	 		r.XML(ctx, fasthttp.StatusOK, ExampleXml{One: "hello", Two: "xml"})
			case "/html":
     	 		// Assumes you have a template in ./templates called "example.html"
        		// $ mkdir -p templates && echo "<h1>Hello {{.}}.</h1>" > templates example.html
       		    r.HTML(ctx, fasthttp.StatusOK, "example", "World")
  			default:
      	 		ctx.Error("Unsupported path", fasthttp.StatusNotFound)
  	  	}
   	}



    fasthttp.ListenAndServe(":8080", requestHandler)
}


~~~

~~~ html
<!-- templates/example.html -->
<h1>Hello {{.}}.</h1>
~~~

### Available Options
Render comes with a variety of configuration options _(Note: these are not the default option values. See the defaults below.)_:

~~~ go
// ...
r := render.New(render.Options{
    Directory: "templates", // Specify what path to load the templates from.
    Asset: func(name string) ([]byte, error) { // Load from an Asset function instead of file.
      return []byte("template content"), nil
    },
    AssetNames: func() []string { // Return a list of asset names for the Asset function
      return []string{"filename.html"}
    },
	Gzip: false, // enable it if you want render through gzip compression
    Layout: "layout", // Specify a layout template. Layouts can call {{ yield }} to render the current template or {{ partial "css" }} to render a partial from the current template.
    Extensions: []string{".html", ".tpml"}, // Specify extensions to load for templates.
    Funcs: []template.FuncMap{AppHelpers}, // Specify helper function maps for templates to access.
    Delims: render.Delims{"{[{", "}]}"}, // Sets delimiters to the specified strings.
    Charset: "UTF-8", // Sets encoding for json and html content-types. Default is "UTF-8".
    IndentJSON: true, // Output human readable JSON.
    IndentXML: true, // Output human readable XML.
    PrefixJSON: []byte(")]}',\n"), // Prefixes JSON responses with the given bytes.
    PrefixXML: []byte("<?xml version='1.0' encoding='UTF-8'?>"), // Prefixes XML responses with the given bytes.
    HTMLContentType: "application/xhtml+xml", // Output XHTML content type instead of default "text/html".
    JSContentType: "text/ecmascript", // Output ECMAScript content type instead of default "text/html".
    IsDevelopment: true, // Render will now recompile the templates on every HTML response.
    UnEscapeHTML: true, // Replace ensure '&<>' are output correctly (JSON only).
    StreamingJSON: true, // Streams the JSON response via json.Encoder.
    RequirePartials: true, // Return an error if a template is missing a partial used in a layout.
    DisableHTTPErrorRendering: true, // Disables automatic rendering of http.StatusInternalServerError when an error occurs.
})
// ...
~~~

### Default Options
These are the preset options for Render:

~~~ go
r := render.New()

// Is the same as the default configuration options:

r := render.New(render.Options{
    Directory: "templates",
    Asset: nil,
    AssetNames: nil,
	Gzip: false,
    Layout: "",
    Extensions: []string{".html"},
    Funcs: []template.FuncMap{},
    Delims: render.Delims{"{{", "}}"},
    Charset: "UTF-8",
    IndentJSON: false,
    IndentXML: false,
    PrefixJSON: []byte(""),
    PrefixXML: []byte(""),
    HTMLContentType: "text/html",
    JSContentType: "text/javascript",
    IsDevelopment: false,
    UnEscapeHTML: false,
    StreamingJSON: false,
    RequirePartials: false,
    DisableHTTPErrorRendering: false,
})
~~~


For more examples and options take a look [here](https://github.com/unrolled/render)
