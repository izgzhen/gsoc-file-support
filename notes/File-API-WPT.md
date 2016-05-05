File API WPT Investigation
-----

Current failure cases:

1. `blob`
    + Construction parameter `BlobParts`
    + Slicing doesn't work
    + Content type doesn't work
    + Blob URL is not supported
2. `file`
    + Construction: More type in `fileBits`, and other construction methods such as file name, illegal types, lastModified, etc.
    + FileReader in Worker: Failed
    + Attributes: `length`, `lastModified`
3. `filelist`
    + Check `instanceof`
    + Check if method of
    + Check if the item method returns null when no file selected
    + Check if length is fileList's attribute
    + Check if the fileList length is 0 when no file selected
4. Read data
    + Determining encoding
    + `InvalidStateErrorException` for event handler of `readAsArrayBuffer` of `FileReader`
    + `readAsArrayBuffer`
5. Blob URL
    + Create objects
    + Create via XHR
    + Revoke
6. `FileReader` States
    + `EMPTY`, `LOADING`, `DONE` etc.
7. `FileReaderSync`

