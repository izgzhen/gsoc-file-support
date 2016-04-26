# Proposal Note

## Open Problems
- What kind of enhancement do we have to do in `htmlinputelement.rs`. In my idea,
    * Line 96: `// TODO: selected files for file input`
    * Line 620: `atom!("image") | atom!("file") => return None, // Unimplemented`
    * Line 375: `ValueMode::Filename => { // TODO: return C:\fakepath\<first of selected files> when a file is selected`
- Up to front-end interaction, there is something like *queue the task* and *algorithm that runs in the task*, I don't really have an idea of how this works (you should checkout the code!)
- Big file size. In my idea, there are following points
    - The file might be modified during the process, so will `std::io` lock it someway?
    - If we are just using `FileReader` to do local processing, we might not need to read that much bytes. Can we control that? Is it going to give performance improvements?
    - If we are going to send it to network, why the traditional form-based solution is not optimal? If we can *chunk* it and send in a streamed way, is it related our side (or XHR side?) But the form control still have to do thing *the form way* right?
- About the URL scheme: Say we send a request *downloading* the binary or make it `src` of some `<img>`, will the binary (or the image) be copied? It looks silly, but think about the response which should *look* just like a normal request over network. How should the body of response look like?
- Alternative [KISS-UI](https://github.com/KISS-UI/kiss-ui)

## Solved Problems
- About FileList API, It is said to be *at risk* to be replaced by Array. Do we have to take this into consideration?
    + We still need to implement it, since the `files` attribute for form inputs is quite useful and the spec hasn't yet changed to make it an array. So no, not really, though if you'd like to not implement this and spend more time working on something else that's totally fine.
- When is a file on disk read into the memory and used to construct a Blob object.
    + It seems like the spec doesn't require UA to follow any exact steps, and only some behavioral equivalence should be fine
    + This should be discussed further by my idea of implementation maybe.
- Drag & Drop
    + We don't have to do this.
- What is the idea behind using `GenerationId`?
    + It takes part in [planned navigation](https://html.spec.whatwg.org/multipage/#planned-navigation).


## Points
1. `FormData` can be viewed as a heterogeneous dictionary. It is not very important.
2. [MIME-Type](https://html.spec.whatwg.org/multipage/infrastructure.html#mime-type)
    + What are the related media types to this project?
    + When they should be taken care of? Submission? `type` attribute? `accept` check?
3. [File upload state](https://html.spec.whatwg.org/multipage/forms.html#file-upload-state-(type=file))
    + Selected files, each as (name, type, body) tuple
1. What is the difference between `Blob` and `File`?
   + `File` is a wrapper over `Blob` when used in the DOM. `File` has additional properties such as `lastModifiedDate`, `type`, `name`; while the `Blob` is interface to the underlying data storage (whether it would be in-memory or on-disk) (but `Blob` also has a `type` tag)
2. When will (or perhaps *should*) the file descriptor be created, be attached to the `Blob`, and be read?
   1. **Created**: The `click` event of a `<input type="file">` element will trigger a dialog, in which the user selects (multiple) files and filepath(s) will be used to create file descriptors
   2. **Attached:** The created file descriptors will be read (see [std::io API](https://doc.rust-lang.org/std/io/trait.Read.html)) through a buffer (how large, owned by who?). Currently, the data buffer in `Blob` is an `Arc<Vec<u8>>`, while the buffer used by `io` is `&mut [u8]`. But due to laziness, we might make `Blob` a `enum` of two types of data source: a file descriptor, or an in-memory `Arc<Vec<u8>>` (which might be constructed from `DOMString` or similar).
   3. **Used**: It will be used by `FileReader` API or the form submission logic (which is not very clear to me now). At that time, we will need to get the required range of data from the file. We need to set up a buffer, and read the file (to what extent? Do we have to be economic?)
3. Consider some edge cases:
   1. Too many files
      1. Repeated use?
      2. Cached pool?
   2. File changes (modified, moved)
      1. During referencing
      2. During reading (locking mechanism?)
   3. Too large a file
      1. Buffer
      2. Partial read
      3. Cache?
      4. "streamlined programmatic ability to send data from a file to a remote server", "chunk-transfered"
   4. **It looks like not a lot of edge/error/exception** cases are specified in the spec, but I should take care!
4. URL
   1. It sounds like a two-sided story: On one direction, we have to *create* URLs from object; One the other direction, we have to make request and *response with the buffer object*. **But**, what does the *response* here means? If the `Blob` is local (although within same-origin — but how is the origin in the Data URL specified?), how should the response body look like (Certainly, a full copy will look stupid)? Do we have to take the requesting side into consideration?
   2. Will the Data URL be used in a remote way? Looks impossible
5. Security Issues
	1. The system-sensitive files (e.g. `/usr/bin/*`)
