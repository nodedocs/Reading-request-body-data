# Reading Request Body Data

Reading the data from a POST request (i.e. a form submission) can be a little bit of a pitfall in Node, so we're going to go through an example of how to do it properly.  

The first step, obviously, is to listen for incoming data - the trick is to wait for the data to finish, so that you can process all the form data without losing anything. 

## Setting up the Server

Here is a quick script that shows you how to do exactly that:

    var http = require('http');
    var postHTML = 
      '<html><head><title>Post Example</title></head>' +
      '<body>' +
      '<form method="post">' +
      'Input 1: <input name="input1"><br>' +
      'Input 2: <input name="input2"><br>' +
      '<input type="submit">' +
      '</form>' +
      '</body></html>';

    http.createServer(function (req, res) {
      var body = "";

      req.setEncoding('uft8');

      req.on('data', function (chunk) {
        body += chunk;
      });
      req.on('end', function () {
        console.log('POSTed: ' + body);
        res.writeHead(200);
        res.end(postHTML);
      });
    }).listen(8080);

The variable `postHTML` is a static string containing the HTML for two input boxes and a submit box - this HTML is provided so that you can `POST` example data.

> This is NOT the right way to serve static HTML - please see [How to Serve Static Files](link) for a more proper example.

With the HTML out of the way, we create a server to listen for requests. It is important to note, when listening for POST data, that the `req` object is also a readable stream for the request body data.  This object will, therefore, emit a `data` event whenever a 'chunk' of incoming data is received; when there is no more incoming data, the `end` event is emitted. So, in our case, we listen for `data` events. Once all the data is received, we log the data to the console and send the response.

> Something important to note is that the event listeners are being added immediately after the request object is received. If you don't immediately set them, then there is a possibility of missing some of the events. If, for example, an event listener was attached from inside a callback, then the `data` and `end` events might be fired in the meantime with no listeners attached!

One important bit of this is where we set the request encoding. We are expecting UTF-8-encoded data, so we set the encoding to that. If we didn't set the encoding, the "data" event handler function would be getting buffers instead of strings. Since UTF-8 can be multi-byte, we could eventually chop-off some characters at the wrong place. By setting the encoding to UTF-8, Node automatically does the right thing and only emits data that is UTF-8 valid.

## Testing the Server

You can save this script to `server.js` and run it with

    $ node server.js`

Once you run it you will notice that occassionally you will see lines with no data, e.g. `POSTed: `. This happens because regular `GET` requests go through the same codepath. In a more 'real-world' application, it would be proper practice to check the type of request and handle the different request types differently.