# Feed Tables

Feed tables is a parser for Google Spreadsheets data that is in a form of a talbe.
Such spreadsheet has field names in the first row and field values in the remaining
rows of the table.

## How to Install

    npm install feed-tables

## Spreadsheet data

You must have a Google spreadsheet ready. For example, a spreadsheet at
... contains data about people such as a name, street, city. 

## How to use (Node.js)

First, you need to load the data from the spreadsheet. In Node.js, you can use
http and url libraries and the following `receive` function:

```js
var http = require('http');
var url = require('url');

var receive = function(dataUrl, dataReady) {
    var urlp = url.parse(dataUrl);
    var service = http.createClient(urlp.port, urlp.hostname);
    var req = service.request('GET', urlp.pathname + urlp.search,
        {'host': urlp.hostname});

    var data = null, status = null, finished = false;
    
    req.on('response', function (res) {
        var body = "";
        status = res.statusCode;
        
        res.on('data', function (chunk) {
            body += chunk;
        });
        
        res.on('end', function () {
            data = JSON.parse(body);
            if (!finished) {
                finished = true;
                dataReady(data);
            }
        });
    });

    req.end();
    
    setTimeout(function() {
        if (!finished) {
            finished = true;
            dataReady(null);
        }
    }, 2000);
};
```

You can use either `CellsFeed` or `ListFeed` parser to parse the data. This depends on the format
of the spreadsheet you want to use. Cells feeds are larger as every spreadsheet cell data 
is in a separated atom feed entry. The `CellsFeed` parser expects that there are names of fields 
in the first row of the spreadsheet. 

The following code shows how to use the `CellsFeed` parser.

```js
var ft = require('feed-tables');

receive("http://spreadsheets.google.com/...",
 function(data) {
        if (data) {
            var table = new ft.CellsFeed(data);
            
            // you can know access the data
            for (var r = 0; r < table.length; r++) {
                var row = table.getRow(i);
                conlose.log("name: " + row.name + ", street: " + row.street, " city: " + row.city + "\n");
            }
        } else
            console.log("Timeout while fetching the Google Spreadsheet data.");
    });
```

TODO `ListFeed` parser



## License 

(The MIT License)

Copyright (c) 2011 Tomas Vitvar &lt;tomas@vitvar.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
