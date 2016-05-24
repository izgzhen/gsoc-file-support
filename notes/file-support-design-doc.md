% File Support Project Design Document (Draft)
% Zhen Zhang
% \today


## Architecture
### Abstract
The major components are implemented in script thread and file manager thread. File manager abstracts out the file resource, providing a message-based reading service. The script thread integrates different kinds of backends (memory and file) into a sliced, synchronous (but lazy) interface. `FileReader` and `FileReaderSync` are the sole APIs through which we can read actual data in JavaScript.

The `FileManagerThread` will be spawned at the time of constellation initialization and is part of the `ResourceThreads`, which can be accessed through `global`.

### `File` and `Blob` DOM object

The `File` objects are used in two cases: First, it can be constructed in JavaScript through standard constructor API. In this case, it is mostly a recombination of existing data slices and requires the user to specify meta info. As a result, although it is a `File`, the `Blob` backing it will be of `BlobImpl::Memory` type in fact.

Second and internally, it will be constructed at the time of processing the activation event of `<input type="file">` element. First, a `SelectFile(s)` message is sent to the `FileManagerThread` and blocked synchronously until a list of `(FileName, FileID, LastModifiedTime, TypeString)` are returned. Then, the meta info is used to construct the `File` DOM objects, and the objects compose into a `FileList` together.

We will lazy-load the resource when we actually need it, so the `File` itself can be constructed rather fast. Only by reading it -- either we need to encode its content into the form submission or requested by `FileReader`, will the actual I/O happen. Besides, we can chunk the IPC transmission of file content, and cache the content in `Blob` if it is plausible.

**NOTE:** `FileName` is only the name part, the complete path is not returned. Also, this will be a `PathBuf` which doesn't necessarily have to be UTF-8 compliant until further processing.

`FileReader(Sync)` and other users (XHR and form submission) of `File` and `Blob` will call a synchronous getter interface called `get_bytes`, and a sliced `&[u8]` will be returned. Whether `FileReader` might be blocked is decided by how we transmit the file content and whether we cache it.


### `FileManagerThread`
The `FileManagerThread` provides `SelectFile(s)`, `ReadFile`, `DeleteFileID` message interfaces.

XXX: Close file?

`SelectFile` will first spawn the dialog, prompt user to select one file, get the file path back, create a `uuid` and map the ID to the file path, infer the MIME type, and finally return the stripped `FileName`, the `FileID`, the `lastModified` time, and the `TypeString`. The file will not be read now, but its `lastModified` time will be recorded. And we might check the modified time for consistency across multiple reads.

The `ReadFile` message holds the `FileID`, and the `FileManagerThread` will first check if the `FileID` is mapped, and if so, it will start the reading. In reading, first the time stamp will be checked and if the file is modified, an exception will be thrown, and the `FileID` will be invalidated. If normal, the file will be read and sent in a chunked fashion.

### `FileReader` DOM Object

File reader can be either sync or async. In the async case, some event handlers are provided by users. It also needs to handle different read mode (`readAsBinaryString`, `readAsText` and `readAsArrayBuffer`). This part's implementation is mostly complete, but we need to be careful with the chunked transmission part.

### Blob URL Scheme
TODO

## Dependencies
1. [ ] `ArrayBuffer` and `ArrayBufferView` (Servo)
2. [ ] `libnfd` (Library)
3. [x] `sequence` to `ForOf` iterator (instead of `Vec<T>`) (Servo)
