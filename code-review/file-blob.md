# File and Blob

Review tasks:

- [x] `components/script/dom/webidls/Blob.webidl`
- [x] `components/script/dom/blob.rs`
- [x] `components/script/dom/webidls/File.webidl`
- [x] `components/script/dom/file.rs`
- [x] `components/script/dom/webidls/FileReader.webidl`
- [x] `components/script/dom/filereader.rs`

## Blob

### [Spec](https://w3c.github.io/FileAPI/#dfn-Blob)

Difference: `Constructor(sequence<(ArrayBuffer or ArrayBufferView or Blob or DOMString)> blobParts)` versus `Constructor(DOMString blobParts)`
### [Implementation](https://github.com/servo/servo/blob/master/components/script/dom/blob.rs)

1. The `DataSlice` looks like a rather general construct, why it is implemented here?
   + The relative positioning scheme is implemented here
   + The `size` is delegated to this structure
2. Why is the `Reflector` useful? When will it be used
   + Only a `Reflector::new()` appears in code now
3. We need to accept other `blobParts` types, and in a sequence way (?)
4. Closing: Needs implementation of the URL scheme

## File

### [Spec](https://w3c.github.io/FileAPI/#file)

Differences

* Constructor: `[Constructor(sequence<(Blob or DOMString or ArrayBufferView or ArrayBuffer)> fileBits, [EnsureUTF16] DOMString fileName, optional FilePropertyBag options), Exposed=Window,Worker]`
* `readonly attribute long long lastModified;`
* `FilePropertyBag`

### Implementation

The implementation is not very complete. But the complete one won't be too long. According to the spec, it seems like that `File` interface is just a wrapper over read data (`Blob`, `DOMString` etc.)

It won't be too hard to implement.

## FileList

It is *at risk* to be replaced by a general `Array` finally.

## Read and FileReader

[Spec of Read](https://w3c.github.io/FileAPI/#reading-data-section)

[Spec of FileReader](https://w3c.github.io/FileAPI/#APIASynch)

### Spec diff

* `Exposed = Worker`
* `void readAsArrayBuffer(Blob blob);`
* `readonly attribute FileReaderResult? result;`

### Implementation
Quite a bit … Now I am impressed with a task queue, with different methods  (Text, ArrayBuffer, DataURL), and two modes (sync/async).

Besides, I begin to understand the picture — we should call FFI to load the binary of file from FS, and fill it into some buffer. Then, we deal with the format etc.
