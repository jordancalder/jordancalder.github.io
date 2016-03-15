---
layout: post
title:  "Child processes in node.js"
date:   2015-04-21 18:38:36 -0600
categories: Programming
---

## What are child processes

Child processes are external processes, which are children of the process that launched them. Because Node is optimized for effecient I/O, some tasks which might be CPU intensive, will block the event loop. Processes that block the event loop can slow down your application for all users and create a bad experience. Child processes help free up those processes to externally handle those CPU bogging tasks and free up your loop to operate on non-blocking tasks.

## Creating child processes

There are two ways to create child processes in Node: `require('child_process').spawn()` and `require('child_process').fork()`.

### child_process.spawn(command[, args][, options])

Spawn is designed to run system commands accepting the first parameter as the system command and the second as an array of command arguments. It doesn't execute code beyond this, although you can attach listeners to it to allow your parent process to interact with it.

	var ls = require('child_process').spawn('ls', ['-lh', '/']);

	ls.stdout.on('data', function (data) {
	  console.log(data);
	});

### .fork()

Fork is a special instance of spawn, which launch a new instance of the V8 engine. Essentially you can create multiple workers to run on the same Node core.

## Usage

ChildProcess is an event emitter with three streams associated with them: `stdout`, `stdin`, and `stderr`. These streams may be shared with the parent io streams.

### Events
*	error - fired when process cannot spawn, close, or receive a message from a parent process.
*	exit - Emitted after child process ends.
*	close - Emitted when all stdio streams from child processes have ended. Distinct from `exit` since multiple child processes may share io streams.
*	disconnect - Emitted after calling the `.disconnect()` method in parent or child.
*	message - Responde to `.send(message, [sendHandle])` method.

## Examples

### Fork - run script

	var p = require('child_process').fork(__dirname + '/child.js');

	p.on('message', function(data) {
	  console.log('parent process received message:', data);
	});

	p.send('hey');

`/child.js`

    process.on('message', function(data) {
	  console.log('child process received message:', data);
	});

	process.send('hello');

*Note: .send() is synchronous and should not be used for sending large amounts of data. Pipe data instead.*