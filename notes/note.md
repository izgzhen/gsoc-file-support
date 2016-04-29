# Proposal Note

## Open Problems
- `File` construction
    - Currently I tried to read all bytes at the file list construction time. What if it is big?
    - What does the `stream` mean in the spec?
- Blob URL scheme: Say we send a request *downloading* the binary or make it `src` of some `<img>`, will the binary (or the image) be copied? It looks silly, but think about the response which should *look* just like a normal request over network. How should the body of response look like?

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
2. When will (or perhaps *should*) the file descriptor be created, be attached to the `Blob`, and be read? What do we put inside the `Blob`?
3. Consider some edge cases:
   1. Too many files
      1. Repeated use?
      2. Cached pool?
   2. File changes (modified, moved, buffer everything at construction?)
      1. During referencing
      2. During reading (locking mechanism?)
   3. Too large a file
      1. Buffer
      2. Partial read
      3. Cache?
      4. "streamlined programmatic ability to send data from a file to a remote server", "chunk-transferred"
4. URL
   1. It sounds like a two-sided story: On one direction, we have to *create* URLs from object; One the other direction, we have to make request and *response with the buffer object*. **But**, what does the *response* here means? If the `Blob` is local (although within same-origin — but how is the origin in the Data URL specified?), how should the response body look like (Certainly, a full copy will look stupid)? Do we have to take the requesting side into consideration?
   2. Will the Data URL be used in a remote way? Looks impossible
5. Security Issues
	1. The system-sensitive files (e.g. `/usr/bin/*`)
