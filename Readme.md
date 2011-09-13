# Debugging memory leaks in node.js

This is a quick tutorial for debugging memory leaks in node.js.

## Step 1: Create some leaking code

In order to debug a memory leak, we first need to create one. This is as easy
as adding a bunch of objects to an array every 1s inside a file called `leak.js`
(also included in this repo).

``` javascript
require('v8-profiler');

// It is important to use named constructors (like the one below), otherwise
// the heap snapshots will not produce useful outputs for you.
function LeakingClass() {
}

var leaks = [];
setInterval(function() {
  for (var i = 0; i < 100; i++) {
    leaks.push(new LeakingClass);
  }

  console.error('Leaks: %d', leaks.length);
}, 1000);
```

## Step 2: Install the debugging tools

First of all, you need to install the `v8-profiler` module that we included
in our test file. Without that, you will not be able to take heap snapshots
in the debugger.

```
mkdir node_modules
npm install v8-profiler
```

Once you have that installed, you need to install the node inspector utility
by doing:

```
npm install -g node-inspector
```

## Step 3: Debug the memory leak

With all the above in place, you can now debug your memory leak by running the
`leak.js` file in debug mode:

```
node --debug leak.js
```

**Note:** You can also send the SIGUSR1 signal to an existing node process to
enable the debug mode as you need it.

Now that your app is running (and leaking memory), start the node inspector in
a new shell/tab to debug the memory leak:

```
node-inspector
```

In order to use the inspector, use Google Chrome to open up the url of the
debugger which should be: [http://0.0.0.0:8080/debug?port=5858][http://0.0.0.0:8080/debug?port=5858].

You should see a typical Chrome Developer Tools window:

* Select "Profiles" from the tabs on top
* Hit the "Enable Profiling" button
* Now click the eye icon on the bottom left to take a heap snapshot
* Wait a few seconds
* Click the eye icon again to take another heapsnapshot
* Click the "Count +/-" column on top to sort by it.

You should now see something like this:

![Screenshot]()

In our case this lets you see that there have been 2200 new LeakingClass
objects created since the last snapshot, and that there are 29000 of them
total in the heap at this point.

That's it, you have learned the basics of tracking down memory leaks in
node.js.