---
layout: post
title:  "Working with streams in node.js"
date:   2015-04-08 22:39:41 -0600
categories: Programming
---

Node.js is an asynchronous and event driven platform. It uses an event driven, non-blocking I/O model. If you are building something that is I/O bound, you can take advantage of streams.

## What is a stream?

A stream is essentially a unix pipe, which lets you pipe data from a readable source to a target destination. The stream is an event emitter which implements some special abstractions for handling the data. Depending on the implementation, a stream is either readable, writable, or duplex (both). Readable streams let you read from a source, while writable streams let you write to a target.

If you've used any of the popular web frameworks for node, you've probably already come in contact with streams. The HTTP server in node implements both readable and writable streams, as request and response, respectively. Additionally, if you've used node's File System module, you've likely used readable and writeable file stream methods.

Since streams are event emitters, they emit many different events which we can use at various points to handle our streams.

## Why use streams?

Because node.js is event driven and built for I/O, streams make it easy to handle these I/O tasks in a performant manner.

## Readable streams

A readable stream is an abstraction for the data that you are reading from. Data comes out of the readable stream. Readable streams emit data when they receive a 'chunk' and will emit an end event when they reach the end of the data.

A common way of reading from streams is to listen to the `data` event and attach a callback. From this callback, we can read the chunks of data supplied from the stream.

    var fs = require('fs');
	var readable = fs.createReadStream('file.txt');
	var data = '';

	readable.on('data', function(chunk) {
	    data += chunk;
	});

	readable.on('end', function() {
	    console.log(data);
	});

From this example, you can see that we create a readable stream from `fs.createReadStream` and listen to the `data` and `end` events. We receive the data in chunks.Finally, when there is no more data to read, the stream emits the end event and we are able to run the callback attached to that event emitter.

Another way of reading from a stream is by using the `.read()` method. An example taken from the api docs looks something like this:

	var readable = fs.createReadStream('file.txt');
	readable.on('readable', function() {
	  var chunk;
	  while (null !== (chunk = readable.read())) {
	    console.log('got %d bytes of data', chunk.length);
	  }
	});

In this case, `.read` returns the chunks of data until there is nothing left to read, at which point it returns `null`.

You can also `.pause` and `.resume` your readable streams, as well as check `.isPaused()`:

	readable.pause();
	readable.isPaused();

Readable streams also provide a useful way of sending your read data to a destination without having to manage the flow of data yourself. The mechanism here is called piping. You can pipe a source to a destination without having to worry about slow or fast data.

	var fs = require('fs');
	var readable = fs.createReadStream('in.txt');
	var writable = fs.createWriteStream('out.txt');

	readable.pipe(writable);

Readable streams implement the `.push(data)` method, which is akin to the data emitter: `stream.emit('data', data)`. This should be used by readable stream implementors, NOT consumers of readable streams.

## Writable streams

At this point, you should be able to guess what a writable stream is. A writable stream is an abstraction for a destination that you will write to. Some examples of writable streams would be the http request and response (on client, on server), file write streams, and stdout and stderr. Writable implements the function `.write(chunk[, encoding][, callback])` with a bool return value. The value of the return indicates whether or not writing should be throttled back. If it returns `false`, it is an indication that data had to buffer and you should discontinue writing until the emitter emits the `drain` event. If it returns `true`, it successfully wrote and you can keep writing. As stated in the api docs, the return value is strictly advisory. You won't be stopped from writing if returned `false`.

Going back to our readable stream example, let's implement a writable with what we've previously done:

	var fs = require('fs');
	var readable = fs.createReadStream('in.txt');
	var writable = fs.createWriteStream('out.txt');

	readable.setEncoding('utf8');

	readable.on('data', function(chunk) {
	    writable.write(chunk);
	});

_NOTE: By default, data is returned as a buffer object. Call .setEncoding() to return strings instead of buffer objects._

After you've finished writing to your stream, simply call `.end()` to notify the stream and it will emit the `finish` event. Often with HTTP response streams, you'll see just that.

    res.write('Hello World!');
    res.end();

or sometimes simplified to:

	response.send('Hello World!');

## Duplex & transform

Duplex streams are streams that implement both the Readable and Writable interfaces. In duplex streams, the readable and writable streams are independent of each other and have separate internal buffers. Events happen separate from each other. An example of this would be a TCP socket connection. You can think of a Duplex stream as a Readable stream that includes a Writable stream. Since Javascript doesn't have multiple prototypal inheritance, Duplex inherits from Readable prototypally and Writable parasitically. Because of this, it is up to the user to extent and implement the `_read(n)` and ` _write(chunk, encoding, callback)` methods on a duplex class.

Transform streams are duplex streams where the output is, by your implementation, factored out of the input. A read is the direct result of a write occurance. Rather than implement a `.read()` and a `.write()`, a user will implement the `.transform()` and `.flush()` methods.

	var transform = require('stream').Transform;

	transform.prototype._transform = function (data, encoding, callback) {
	  //something with data
	  callback();
	}

## Buffering

Both readable and writable streams buffer data on an internal object called _readableState.buffer or _writableState.buffer. Buffering in write streams occurs when `.write()` is called and returns false. In read streams, if the stream consumer does not call `stream.read()`, then data will sit in an internal queue on the object until the data is consumed.

The purpose of streams is to limit the buffering done to suitable levels so source and destination data speeds that don't match won't overwhelm free memory. This is one of the advantages to using `.pipe()`.

## Object streaming

Typically, streams operate on strings and buffers. Streams in object mode can push any type of javascript object.

	var transform = require('stream').Transform( { objectMode : true } );

## Real-life uses

Transform stream: [email-stripper module](https://www.npmjs.com/package/email-stripper), [read-line module](https://www.npmjs.com/package/read-line)