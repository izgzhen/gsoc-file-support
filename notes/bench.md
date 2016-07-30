Simple benchmarking
===

## Code
```js
<input type="file" id="file-input">

<button onclick="test();">Test</button>

<script type="text/javascript">
    function test() {
        var reader = new FileReader;
        var f = document.getElementById("file-input").files[0];

        var now = new Date;

        reader.onload = function (evt) {
            console.log((new Date - now) / 1000);
        }

        reader.readAsText(f);
    }
</script>

```

The first column below represents the size of selected file. The second column is output of `console.log`, i.e. seconds used to read an on-disk file.

Tested on a Macbook Pro, git hash `de06525`.

## Reference (WebKit)

- 1.7 MB -- 0.009
- 5.2 MB -- 0.033
- 114.7 MB -- 0.203
- 1 GB -- 3.144

### Servo

- 1.7 MB -- 1.523
- 5.2 MB -- 4.162
- 114.7 MB 95.501
- 1 GB -- 90.161

