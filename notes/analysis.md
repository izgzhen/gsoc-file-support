Analysis
====

## Abstraction 
### `net`, `net_traits` side

#### `net/filemanager_thread`

1. `open_dialog`: filter list, default path === OS & User ==> `Result<Path(s)>`
2. Exposed interfaces: `FileManagerThreadFactory`, `UIProvider`, `TFDProvider`
3. `FileManager`: start looping, dispatch message to spawned worker threads
    - `SelectFile(s)`
    - `ReadFile`
    - `PromoteMemory`
    - `AddSlicedURLEntry`
    - `RevokeBlobURL`
    - `DecRef`
    - `IncRef`
    - `ActivateBlobURL`

#### `net_traits/filemanager_thread`
+ `FileOrigin`
+ `RelativePos`
+ `SelectedFileId`
+ `SelectedFile`
+ `FilterPattern`
+ `FileManagerThreadMsg`

#### `net/resource_thread`
+ `CoreResourceManager` has a `filemanager_chan`
    + Dispatch on `"blob"` scheme: `blob_loader::factory`
    + Which in term calls `load_blob`

### `script` side

#### `dom/blob`
+ `FileBlob` (with cached meta-info/bytes)
+ `BlobImpl` (`File`, `Memory`, `Sliced`)
+ `Blob`
    + DOM interfaces
    + Internal methods
        + `get_bytes`
        + `get_blob_url_id`
        + `promote`
        + `create_sliced_url_id`
        + `clean_up_file_resource`

#### `dom/file`
Some constructors, not very important

#### `dom/htmlinputelement`
Select files and create `File` DOM object(s) accessible from script


#### `dom/url.rs`
+ `createObjectURL`
+ `revokeObjectURL`

## Properties
1. Refcount --> correctness of lifetime & accessibility
2. Slice --> effects on the refcount, and is slicing itself right

