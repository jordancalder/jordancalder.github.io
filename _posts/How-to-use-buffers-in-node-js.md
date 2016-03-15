title: "How to use buffers in node.js"
date: 2015-04-07 13:55:07
categories:
- Programming
tags:
- Javascript
- Node.js
---

## What are buffers?

A buffer is a chunk of memory that is allocated outside of the V8 heap. It is used to deal with raw binary data and can be instantiated from node's Buffer class. Since many MIME-type files sent and received through browsers, servers and web clients are formatted as an octet-stream/byte-stream (binary data), we need a way to handle this data, since Javascript doesn't play nicely with binary data on its own.

## Why use buffers?

Most of the time, in the browser, you'll be dealing with unicode encoded strings. Javascript handles that sort of stuff great on its own. However, when dealing with binary streams (filesystem, TCP), Javascript doesn't do so well. You need a way to work with binary data in a fast and efficient way.

## Examples

### Creating Buffers

Node.js' API [docs](https://nodejs.org/api/buffer.html) show that you can create buffers in one of the following ways:

    new Buffer(size)
    new Buffer(array)
    new Buffer(buffer)
    new Buffer(str[, encoding])

So, to create an uninitialized buffer of a 8-byte size, you would write:

    var buffer = new Buffer(8)

To allocate data to the buffer, pass in an array of octets as a parameter:

	var buffer = new Buffer([3, 4, 0, 5])

An example of passing in a string with an encoding might look like this:

    var buffer = new Buffer("Hello World!", "utf-8")

The Buffer class accepts `ascii`, `ucs2`, `binary`, and `base64` as valid encodings.

### Writing to Buffers

The Buffer class has a `.write()` method and accepts a max of 4 parameters.

    .write(string[, offset][, length][, encoding])

The first param is the string that will be written to the buffer. The second is an offset, or the index to begin writing the string. Length is the number of bytes to write. The final parameter supplied is the encoding to use on the string.

    var buffer = new Buffer(16)
    buffer.write('Hello World', 'utf-8')

This will return `11` (the number of bytes written to the buffer). We could then add more to the buffer by writing:

    buffer.write('!', 11, 'utf-8')

returning `1`.

If the buffer was not initialized with enough space to fit the entire string, it will write a partial string to the buffer.

### Reading from Buffers

There are various methods in the Buffer class for reading from a buffer. By far the quickest and easiest (and sometimes dirtiest) way is by using the .toString() method.

    .toString([encoding][, start][, end])

Given our first buffer example, we could print out our buffer like so:

    buffer.toString('utf-8')

which would return `'Hello World!0\u0001\u0001\u0000'`. Since we didn't use all of our buffer, we get a little more that our written string. We actually get the unused bytes attached. We could use the start and end parameters to extract just the string that we want:

    buffer.toString('utf-8', 0, 12)

Another option we have is using the StringDecoder class. This class operates like the toString method, but has additional support for utf-8 encoded strings.

    var StringDecoder = require('string_decoder').StringDecoder
	var decoder = new StringDecoder('utf8')

	var buffer = new Buffer('Hello World!', 'utf-8')
	decoder.write(buffer)

This will print simply `Hello World!`. If we want to get anything that was left over in the buffer, we can use `.end()`. This will return any trailing bytes in the buffer.

### More in the Buffer Class

#### .isBuffer(obj)

Returns true if obj is buffer.

#### .isEncoding(encoding)

Returns true if buffer matches the specified encoding.

#### .byteLength(string[, encoding])

Returns the byte length of a string. Not the same as `String.prototype.length` since the number of characters does not always equal bytes.

#### .length

Simply the size of the buffer in bytes. Not the size of the contents of the buffer.

#### .copy(targetBuffer[, targetStart][, sourceStart][, sourceEnd])

Copies a region of the source buffer to a region in the target buffer.

#### .slice([start][, end])

Returns a new buffer that references the same memory allocated from the first. Offset and cropped from `start` to `end`.

#### .toJSON()

Returns a json representation of the buffer object.

    var buffer = new Buffer('Hello World')
	var json = JSON.stringify(buffer)

Returns `{"type":"Buffer","data":[72,101,108,108,111,32,87,111,114,108,100]}`.

#### .concat(list[, totalLength])

Returns a buffer which is a concatenated result from the supplied list parameter.

#### .compare(buf1, buf2) or .compare(otherBuffer)

Useful for sorting. Returns a number which indicates if one comes before the other in sort order.

	var arr = [Buffer('1234'), Buffer('0123')]
	arr.sort(Buffer.compare)

#### .equals(otherBuffer)

Returns bool. True if buffers bytes match.