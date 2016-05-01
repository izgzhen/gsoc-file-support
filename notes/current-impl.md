# Investigation on current implementation

## FileReader behaviour

```javascript
var file = e.files[0];
var reader = new FileReader;
reader.onload = function (event) { console.log("complete"); };
reader.readAsBinaryString(file);
```

Here, I selected a 600MB `.iso` file in input element `e`.

I tested this on Chrome, Safari, and Firefox.

First thing to note is that, they don't read until I issued the `reader.readAsXXX` command.

Second, Safari's console is down after throwing a `DOM Exception` error; Firefox throws a `buffer overflow` exception but still working; Chrome is best, it read the whole 600 MB file within about 4 seconds on my laptop.

One thing to note is that, the Chrome will throw a *busy reading* error if you issue the same command closely after the first time.



