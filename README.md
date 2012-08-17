#   Good Job

A simple way to write complex asynchronous code.

Good Job is a Node.js module that lets you glue interdependent asynchronous calls (_Tasks_) into complex _Jobs_. These _Jobs_ can then be executed at once, converted to functions, extended and combined.

Good Job goes the extra mile by providing functions and options for error handling, _Task_ logging and _Job_ timeouts. Good Job makes it easier to find bugs, recover from unexpected situations and notice code that needs refactoring.

Although not tested, Good Job should run on any JavaScript platform that provides a synchronous `require(module_name)` function.

<!-- Inspired by [async.auto()](https://github.com/caolan/async#auto) and [Make](http://en.wikipedia.org/wiki/Make_%28software%29). -->

##  Quick Example

```javascript
/// It makes sense to use a very short name for good-job variable.
/// "X" is used as a convention in this documentation.
var X =         require("good-job");

X.run({
    file_name:  "/etc/hosts",

    contents:   X.call("fs#readFile", X.get("file_name"), "base64")
                .onError(console.error),
    
    output:     X.callSync(console.log, X.get("contents")),
});
```

##  Download

Install using Node Package Manager (npm):

    $ npm install good-job

Download from GitHub:

    $ git clone git://github.com/emilis/good-job.git

Or get the zipped version [here](https://github.com/emilis/good-job/zipball/master).

##  Documentation

All of the functions exported by good-job (except [options()](#X.options)) create a new _Job_, _Task_ or _Request_ and call their own methods. You can then use [method chaining](http://en.wikipedia.org/wiki/Method_chaining) to extend them:

```javascript
/// Create a new Task, specify its callback and error handler:
X.wait("check_permission")
    .call(fs.readFile, X.get("file_name"))
    .onError(console.error, "Failed to read the file.");
    
/// Create a new Request that will transform its result:
X.get("file_contents")
    .apply(function(contents){
        return contents.toLowerCase();
    });
```

### Exports and methods:

<table>
    <thead><tr>
            <th colspan="2">good-job exports</th>
            <th><a href="#Job">Job</a></th>
            <th><a href="#Task">Task</a></th>
            <th><a href="#Request">Request</a></th>
    </tr></thead>
    <tbody>
        <tr><td>job, extend</td>
            <td>&rarr;</td>
            <td><a href="#Job.extend">Job.extend</a></td>
            <td colspan="2"></td></tr>
        <tr><td>tasks</td>
            <td>&rarr;</td>
            <td><a href="#Job.addTasks">Job.addTasks</a></td>
            <td colspan="2"></td></tr>
        <tr><td>set</td>
            <td>&rarr;</td>
            <td><a href="#Job.set">Job.set</a></td>
            <td colspan="2"></td></tr>
        <tr><td>createFunction</td>
            <td>&rarr;</td>
            <td><a href="#Job.compile">Job.compile</a></td>
            <td colspan="2"></td></tr>
        <tr><td>run</td>
            <td>&rarr;</td>
            <td><a href="#Job.run">Job.run</a></td>
            <td colspan="2"></td></tr>
        <tr><td>wait</td>
            <td>&rarr;</td>
            <td></td>
            <td><a href="#Task.wait">Task.wait</a></td>
            <td></td></tr>
        <tr><td>call</td>
            <td>&rarr;</td>
            <td></td>
            <td><a href="#Task.call">Task.call</a></td>
            <td></td></tr>
        <tr><td>callSync</td>
            <td>&rarr;</td>
            <td></td>
            <td><a href="#Task.callSync">Task.callSync</a></td>
            <td></td></tr>
        <tr><td>subTasks</td>
            <td>&rarr;</td>
            <td></td>
            <td><a href="#Task.subTasks">Task.subTasks</a></td>
            <td></td></tr>
        <tr><td>get</td>
            <td>&rarr;</td>
            <td colspan="2"></td>
            <td><a href="#Request.get">Request.get</a></td></tr>
        <tr><td>getAll</td>
            <td>&rarr;</td>
            <td colspan="2"></td>
            <td><a href="#Request.getAll">Request.getAll</a></td></tr>
        <tr>
            <td colspan="2" valign="top">
                <a href="#X.options">options</a>
            </td><td valign="top">
                <a href="#Job.clone">Job.clone</a><br>
                <a href="#Job.setCallback">Job.setCallback</a><br>
                <a href="#Job.setErrback">Job.setErrback</a><br>
                <a href="#Job.options">Job.options</a><br>
                <a href="#Job.log">Job.log</a><br>
                <a href="#Job.logDone">Job.logDone</a><br>
            </td><td valign="top">
                <a href="#Task.onError">Task.onError</a><br>
                <a href="#Task.exitOnError">Task.exitOnError</a><br>
                <a href="#Task.compile">Task.compile</a><br>
            </td><td valign="top">
                <a href="#Request.map">Request.map</a></br>
                <a href="#Request.apply">Request.apply</a><br>
                <a href="#Request.getValue">Request.getValue</a><br>
                <a href="#Request.toTask">Request.toTask</a><br>
                <a href="#Request.getDependencies">Request.getDependencies</a><br>
        </td></tr>
    </tbody>
</table>


<a name="X"></a>
##  good-job

<a name="X.options"></a>
### options(obj)

Utility function: wraps a given object as an _Options_ object so that it won't be mixed up with a task list by a Job.

__Arguments__

* obj       -   Key-value options for a Job.


<a name="Job"></a>
##  Job

A _Job_ manages asynchronous tasks: tracks their dependencies, executes them when possible, logs their errors. A _Job_ is a dictionary with task names for keys and tasks (or their results) for values.

<a name="Job.extend"></a>
### Job.extend(...)<br><sup>extend(...)</sup>

Adds more tasks for the _Job_, sets callback or error handler and options. Returns the _Job_.

__Arguments__

Any number of the following:

* Object    -   Specifies a task list. Keys for task names, values for _Tasks_ (see [Job.addTasks](#Job.addTasks)).
* _Options_ -   Sets options for the _Job_. You can create _Options_ with [X.options](#X.options).
* Function  -   Sets callback for the _Job_. If the callback is already set, sets the error handler (see [Job.setCallback](#Job.setCallback), [Job.setErrback](#Job.setErrback)).

__Example__

```javascript
X.extend(
    X.options({
        id:     "MyJob",
        log:    true,
    }),
    {
        task1:  ...,
        task2:  ...,
    },
    function(err, result) {
        /// This will be called when the job finishes successfully.
        /// This would also be called if the job had no error handler.
    },
    function(err, result) {
        /// This will be called when a task emits an error.
    });
```

<a name="Job.addTasks"></a>
### Job.addTasks(task\_list)<br><sup>tasks(task\_list)</sup>

Adds more tasks for the _Job_. Returns the _Job_.

__Arguments__

* task\_list -   An Object with keys for task names and values for _Tasks_. _Requests_ get converted to _Tasks_. Other values are marked as finished tasks for the _Job_.

__Example__

```javascript
myJob.addTasks({
    file_name:  "/etc/hosts",

    contents:   X.call(fs.readFile, X.get("file_name"), "ascii"),

    print:      X.call(console.log, X.get("contents")),
});
```


<a name="Job.set"></a>
### Job.set(name, value)<br><sup>set(name, value)</sup>

Add task or set value for the _Job_. Returns the _Job_.

__Arguments__

* name      -   String. Name of the _Task_ or value.
* value     -   _Task_, _Request_ (will be [converted to Task](#Request.toTask)) or any other value (will be marked as a finished task).

__Example__

```javascript
var myJob = X.job();

myJob.set("file_name",  "/etc/hosts");
myJob.set("contents",   X.call("fs#readFile", X.get("file_name"), "ascii"));
myJob.set("print",      X.call(console.log, X.get("contents"))); 

myJob.run();
```


<a name="Job.compile"></a>
### Job.compile(arg\_names, ...)<br><sup>createFunction(arg\_names, ...)</sup>

Returns an asynchronous function that runs the _Job_ with the given arguments used as finished task results.

__Arguments__

* arg\_names -   An Array specifying how function arguments will be named inside the _Job_. E.g. `["file_name", "mode"]`.
* Further arguments [extend](#Job.extend) the _Job_ before creating the function.

__Example__

Create a function that joins contents of two files:

```javascript
/// You could use "job.compile(["f1_name","f2_name])" on an existing job.
var cat2files = X.createFunction(
    ["f1_name", "f2_name"],
    {
        f1_contents:    X.call(fs.readFile, X.get("f1_name"), "utf8"),
        f2_contents:    X.call(fs.readFile, X.get("f2_name"), "utf8"),
        cat:            X.get(["f1_contents","f2_contents"])
                        .apply(function(c1, c2){
                            return [c1, c2].join("\n");
                        }),
    });

/// We can now use the function. Note that it is asynchronous:
cat2files("/etc/hosts", "/etc/passwd", console.log);
```

<a name="Job.run"></a>
### Job.run(...)<br><sup>run(...)</sup>

Executes the _Job_.

__Arguments__

* If there are any, the _Job_ is [extended](#Job.extend) before running.

<a name="Job.clone"></a>
### Job.clone()

Returns a clone of the _Job_ object. Needed when executing the _Job_ more than once, because _Jobs_ save their execution state. See also [Job.compile](#Job.compile).

<a name="Job.setCallback"></a>
### Job.setCallback(cb)

Sets a function to call when the _Job_ finishes. If the _Job_ has an error handler, this function will only be called on success. Returns the _Job_.

_Note that if a task uses [Task.exitOnError](#Task.exitOnError), this Job callback won't be called._

__Arguments__

* cb    -   `function(err, result) { ... }`

<a name="Job.setErrback"></a>
### Job.setErrback(cb)

Sets a function to call when the _Job_ finishes with error - a _Task_ returns an error or a timeout occurs. Returns the _Job_.

_Note that if a task uses [Task.exitOnError](#Task.exitOnError), this Job error handler won't be called._

* cb    -   `function(err, result) { ... }`

<a name="Job.options"></a>
### Job.options(obj)

Overwrites the current _Job_ options. Returns the _Job_.

__Supported Options__

* id    -   ID for the job that is used for logging.
* log   -   `true` to turn on logging.
* timeout - Number of miliseconds after which the _Job_ is stopped if not finished.

__Example__

```javascript
X.run(
    X.options({
        id:         module.id,
        log:        true,
        timeout:    2000, // 2 seconds
    }),
    {
        t1: X.callSync(console.log, new Date()),

        t2: X.call(function(cb){
                /// This task will cause the Job to timeout:
                setTimeout(cb, 3000, null, "Long task result");
            }),
        
        t3: X.callSync(console.log, X.get("t2")), // This task will not run
    });
```

<a name="Job.log"></a>
### Job.log(msg\_type, ...)

Prints a log message for the _Job_.

__Arguments__

* msg\_type, - One of: INFO, EXEC, VALUE, FUNC, DONE, LATE, WARN, ERROR, TIMEOUT, DEBUG.
* Further arguments get printed as they are.

<a name="Job.logDone"></a>
### Job.logDone(name, err, result)

Prints a log message for a _Task_ or _Job_ that is done.

__Arguments__

* name  -   Job or Task name.
* err, result   -   Arguments passed to Task or Job callback (only their types are printed).

<a name="Task"></a>
##  Task

A _Task_ is a wrapper around a function that exposes its dependencies for the [Job](#Job). It handles error results, catches exceptions.

<a name="Task.wait"></a>
### Task.wait(...)<br><sup>wait(...)</sup>

Explicitely adds dependencies for the _Task_. Returns the _Task_.

__Arguments__

* Names (Strings) of other tasks in the _Job_.

__Example__

```javascript
X.wait("t1", "t2", "t3")...
```

<a name="Task.call"></a>
### Task.call(fn, ...)<br><sup>call(fn, ...)</sup>

Wraps an asynchronous function. Returns the _Task_.

__Arguments__

* fn    -   A function or a [function URI](#function-uris). Note that functions will lose their context (`this` value) unless [bound](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/bind). The function has to accept callback as its last argument.
* Further values will be passed as arguments to the function when it is called. _Request_ values will be added to the _Task_ dependencies and resolved when the function is called.

__Example__

```javascript
/// Note that this only creates the Task and does not execute it:
X.call("fs#readFile", "/etc/hosts", "utf8");
```

<a name="Task.callSync"></a>
### Task.callSync(fn, ...)<br><sup>callSync(fn, ...)</sup>

Wraps a synchronous function. Returns the _Task_.

__Arguments__

* fn    -   A function or a [function URI](#function-uris). Note that functions will lose their context (`this` value) unless [bound](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/bind). The function has to return its result.
* Further values will be passed as arguments to the function when it is called. _Request_ values will be added to the _Task_ dependencies and resolved when the function is called.

__Example__

```javascript
/// Note that this only creates the Task and does not execute it:
X.callSync("fs#readFileSync", "/etc/hosts", "utf8");
```

<a name="Task.subTasks"></a>
### Task.subTasks(result\_map, tasks)<br><sup>subTasks(result\_map, tasks)</sup>

Adds a _Job_ as a _Task_. Returns the _Task_.

__Arguments__

* result\_map    -   An Array of dependency names or an Object with keys for parent _Job_ task names and values for child _Job_ task names.
* tasks         -   A task list. See [Job.addTasks](#Job.addTasks).

__Example__

```javascript
X.run({
    file_name:  "/etc/hosts",
    file:       X.subTasks({"file_name":"name"},
                {
                    contents:   X.call("fs#readFile", X.get("name"), "utf8"),
                    stat:       X.call("fs#stat", X.get("name")),
                }),
    print:      X.callSync(console.log, X.get("file.name"), X.get("file.stat.ctime")),
});
```

<a name="Task.onError"></a>
### Task.onError(fn, ...)

Sets an error handler for the _Task_. This error handler is executed when the _Task_ returns an error or throws an exception. Returns the _Task_.

_Job_ callback/error handler will be called after this function is called.

__Arguments__

* fn    -   A function or a [function URI](#function-uris). The function has to accept `err,result` as its last arguments.
* Further values will be passed as arguments to the function when it is called. _Request_ values will be added to the _Task_ dependencies and resolved when the function is called.

__Example__

```javascript
X.run({
    t1: X.call("fs#readFile", "does-not-exist", "ascii")
        .onError(console.error, "The file did not exist!"),
    },
    function(err, result) {
        console.log("Job result", err, result);
    });
```

<a name="Task.exitOnError"></a>
### Task.exitOnError(fn, ...)

Sets an error handler for the _Task_. This error handler is executed when the _Task_ returns an error or throws an exception. Returns the _Task_.

_Job_ callback/error handler *will not* be called if this function is called.

__Arguments__

* fn    -   A function or a [function URI](#function-uris). The function has to accept `err,result` as its last arguments.
* Further values will be passed as arguments to the function when it is called. _Request_ values will be added to the _Task_ dependencies and resolved when the function is called.

__Example__

```javascript
X.run({
    t1: X.call("fs#readFile", "does-not-exist", "ascii")
        .exitOnError(console.error, "The file did not exist!"),
    },
    function(err, result) {
        /// This will not be called if t1 fails.
        console.log("Job result", err, result);
    });
```

<a name="Task.compile"></a>
### Task.compile()

Creates and returns a function from the _Task_. The function accepts callback as its first argument and an object holding requested dependencies as its second argument.

__Example__

```javascript
var getFileContents = X.call("fs#readFile", X.get("file_name"), "utf8").compile();

getFileContents(console.log, {file_name:"/etc/hosts"});
```

<a name="Request"></a>
##  Request

A _Request_ specifies a _Task_ dependency on a result from some other _Task_ and provides ways to transform this result before using. _Requests_ can also be used as _Tasks_ in a _Job_.

<a name="Request.get"></a>
### Request.get(query)<br><sup>get(query)</sup>

Specifies what value to extract from _Job_ results (when it will be needed). Returns the _Request_.

__Arguments__

* query -   A String with the name of a _Task_ or a String with the name of an expected property of a _Task_ (e.g. "t1.prop1.prop2"), or an Array of such strings.

__Example__

```javascript
X.get("env.SHELL").getValue(process);
```

<a name="Request.getAll"></a>
### Request.getAll()<br><sup>getAll()</sup>

Used as a placeholder for all _Job_ results. Returns the _Request_.

__Example__

```javascript
X.getAll().getValue({a:42});
```

<a name="Request.map"></a>
### Request.map(fn)

Sets a map function to use when getting value from a result. Returns the _Request_.

__Arguments__

* fn    -   A function that maps over a result _Array_ from [query](#Request.get).

__Example__

```javascript
/// This should print: [1, 4, 9 ] [1, 2, 3 ]
X.run({
    input:  [1,4,9],
    roots:  X.get("input").map(Math.sqrt),
    print:  X.callSync(console.log, X.get("input"), X.get("roots")),
});
```

<a name="Request.apply"></a>
### Request.apply(fn)

Sets a filter function to use when getting value from results. Returns the _Request_.

__Arguments__

* fn    -   A function that takes results from [queries](#Request.get) as arguments.

__Example__

```javascript
/// These should print the commands to list your home directory:

X.get("HOME")
    .apply(function(home){
        return "ls " + home;
    })
    .getValue(process.env);

X.get(["SHELL","HOME"])
    .apply(function(shell, home){
        return shell+' -c "ls '+home+'"';
    })
    .getValue(process.env);

X.getAll()
    .apply(function(env){
        return "ls " + env.HOME;
    })
    .getValue(process.env);
```

<a name="Request.getValue"></a>
### Request.getValue(results)

Resolves this _Request_ for the given _Job_ results and returns the requested value.

__Arguments__

* results   -   An Object with keys for _Task_ names and values for their results.

__Example__

```javascript
X.get("a").getValue({a:42});
```

<a name="Request.toTask"></a>
### Request.toTask()

Converts the _Request_ to a _Task_ that just returns the requested value when possible. Returns the _Task_.

__Example__

```javascript
/// See also Task.compile().
var returnAProperty = X.get("a").toTask().compile();
returnAProperty(console.log, {a:42});
```

<a name="Request.getDependencies"></a>
### Request.getDependencies()

Returns an Array of names of  _Tasks_ that this _Request_ needs to finish before getting the value.

__Example__

```javascript
X.get("a.b.c").getDependencies();
```

## Other

<a name="function-uris"></a>
### Function URIs

Function URIs are Strings representing a function that can be loaded from some external module.

URIs consist of a _path_ and a _fragment_ separated by "#". `"path#fragment"` is used as `require("path")["fragment"]` when compiling a _Task_.

[Task.call](#Task.call), [Task.callSync](#Task.callSync), [Task.onError](#Task.onError), [Task.exitOnError](#Task.exitOnError) accept URIs in place of functions.

__Example__

```javascript
/// These are equivalent:

var fs =    require("fs");
X.call(fs.readFile, "/etc/hosts", "utf8");

X.call("fs#readFile", "/etc/hosts", "utf8");
```

##  About Good Job

### Copyright and License

Copyright 2012 UAB PriceOn <http://www.priceon.lt/>.

This is free software, and you are welcome to redistribute it under certain conditions; see LICENSE.txt for details.

### Author

Emilis Dambauskas <emilis@priceon.lt>, <http://emilis.github.com/>.
