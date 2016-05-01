% File Support Project Design Document (Draft)
% Zhen Zhang
% \today


## Architecture
### Abstract
The major components are implemented in script thread and file manager thread. File manager abstracts out the file resource, providing a message-based, whole-file, buffered reading service. The script thread integrates different kinds of backends (memory and file) into a sliced, synchronous (but lazy) interface. `FileReader` and `FileReaderSync` are the sole APIs that can read actual data in JavaScript.

The `FileManagerThread` will be spawned at the time of constellation initialization and attached to the `Window` DOM object.

### `File` and `Blob` DOM object

The `File` objects are used in two cases: First, it can be constructed in JavaScript through standard constructor API; second and internally it will only be constructed at the time of processing the click event of `<input type="file">` element.

The `click` event processing: First, a `SelectFile(s)` message is sent to the `FileManagerThread` and blocked until a list of `(FileName, FileID, LastModifiedTime)` are returned. Then, the triple is used to construct the `File` DOM objects, and the objects compose into a `FileList` together.

NOTE: `FileName` is only the name part, the complete path is not returned.

The JavaScript constructor of `File` has a `FileName` argument as well, however, it is mostly a "nickname" and the whole purpose of this constructor is to paste several sources of data together and call it a "file". It doesn't has a `FileID` and it doesn't need to because it is actually memory-backed!

`FileReader(Sync)` and other users (XHR and form submission) of `File` and `Blob` will call a synchronous getter interface called `get_bytes`, and a sliced `&[u8]` will be returned. It will block a while if it is the first time to read, but if not, the buffer in `FileManagerThread` will can directly accessed very fast.


### `FileManagerThread`
The `FileManagerThread` provides `SelectFile(s)`, `ReadFile` message interfaces.

XXX: Close file?

`SelectFile` will first spawn the dialog, prompt user to select one file, get the file path back, create one file handler, and finally return the triple of stripped `FileName`, an unique `FileID` mapped to the actual file handler, and the `lastModified` time. The file will not be read now, but its `lastModified` time will be recorded.

The `ReadFile` message holds the `FileID`, and the `FileManagerThread` will first check if the `FileID` is legal, and if so, will check if the accompanied buffer is filled. If filled, return the `Arc<Vec<u8>>` instantly, or else trigger the reading. In reading, first the time stamp will be checked and if the file is modified, an exception will be thrown, the file handler will be closed, and the `FileID` will be invalidated. If not modified, a file system lock will be put on the file to own it exclusively, then the *whole content* will be read into the buffer and an immutable `Arc<Vec<u8>>`  will be returned to the requester.

### `FileReader` DOM Object

File reader can be either sync or async. In the async case, some event handlers are provided by users. It also needs to handle different read mode (`readAsBinaryString`, `readAsText` and `readAsArrayBuffer`). This part's implementation is mostly complete.

### Blob URL Scheme

## Dependencies
1. `ArrayBuffer` and `ArrayBufferView` (Servo)
2. `libnfd` (Library)

