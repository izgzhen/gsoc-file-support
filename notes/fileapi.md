# File and Blob

## Blob

[Spec](https://w3c.github.io/FileAPI/#dfn-Blob)

[Implementation](https://github.com/servo/servo/blob/master/components/script/dom/blob.rs)

TODOs:

+ Other `blobParts` types (`ArrayBuffer`, `ArrayBufferView`)
+ Needs implementation of the URL scheme

## File

[Spec](https://w3c.github.io/FileAPI/#file)

TODOs:

* Constructor: `fileBits,` `FilePropertyBag`
* Attribute: `lastModified`

## Read and FileReader

[Spec of Read](https://w3c.github.io/FileAPI/#reading-data-section)

[Spec of FileReader](https://w3c.github.io/FileAPI/#APIASynch)


TODOs:

* `void readAsArrayBuffer(Blob blob);`
* `readonly attribute FileReaderResult? result;`


