Analysis
====

In this analysis, we will focus on the reference relations and how to map implementation to specification. Thus, a lot of unrelated operations/data-structures are omitted, and `slice_pos` attribute of a reference will be implicit. Also, the resource will be an unified concept and it will always be available (as long as you have reference to it).

## Core Abstraction Operations

1. **construct blob $b$ with $Res$**: constructing a *readable* Blob-reference (or *bref*) $b$ to *resource* $Res$ in *state*
1. **close blob $b$**: make bref $b$ *unreadable*
3. **slice blob $b$ to get $b'$**: Create a new bref $b'$ referencing the same resource as of $b$ with same readability as of $b$
4. **create url $u$ pointing to $b$**: Create a *valid* URL-reference (*uref*) to $b$
5. **revoke url $u$**: Make $u$ *invalid*
6. **read $b$**: Access resource through bref $b$. If $b$ is *unreadable*, then this operation will fail
7. **load $u$**: Access resource through $u$. If $u$ is invalid, or $u$ is a valid reference to an unreadable $b$, then this operation will fail

![](abs.jpeg)

## Core Implementation

### `filemanager_thread`

`FileManager`: start looping, dispatch messages to spawned worker threads, the messages:

- `SelectFile(s)`
- `ReadFile`
- `PromoteMemory`
- `AddSlicedURLEntry`
- `RevokeBlobURL`
- `DecRef`
- `IncRef`
- `ActivateBlobURL`

### `resource_thread`
`CoreResourceManager` has a `filemanager_chan`, and it will dispatch load request to handler `load_blob` from `blob_loader::factory` when called on `"blob"` scheme.

### `blob`
+ `enum BlobImpl`
    - `File(id)`
    - `Memory(bytes)`
    - `Sliced(JS<Blob>)`
+ `Blob`
    + DOM interfaces
    + Internal methods
        + `get_bytes`
        + `get_blob_url_id`
        + `promote`
        + `create_sliced_url_id`
        + `clean_up_file_resource`

### `htmlinputelement`
Select files and create `File` DOM object(s) accessible from script.

### `dom/url.rs`
+ `URL::createObjectURL`
+ `URL::revokeObjectURL`

![](impl.jpeg)

## Mappings
1. bref $b$: `Blob` + `FileStoreEntry`
2. uref $u$: `Url`
3. state $S$: `FileManagerStore` + heap
4. construct blob:
    - `Blob::new_from_file`
    - `Blob::new_from_bytes`
5. close blob: `Blob::Close`
6. slice blob: `Blob::new_sliced`
7. create uref $u$: `URL::createObjectURL`
8. revoke uref $u$: `URL::revokeObjectURL`
8. read: `FileReader::read`
9. load: `blob_loader::load_blob`
10. readability: `blob.closed`
11. validity: *good* `id` and `entry.is_valid_url`
12. resource: `Vec<u8>` + `FileMetaData`

## Informal Reasoning
#### Theorem #1: After `b.Close()`, $b$ is *unreadable*, and uref $u$ points to $b$ is *invalid* uref.

Proof: If `b.closed == true`, then it *must* been closed in the past, and because we don't have a method to revert this operation, thus `b.closed` should stay `true` once set. Then apply lemma #1.1. QED.

#### Lemma #1.1: If `b.closed == true`, then $b$ is *unreadable*, and uref $u$ points to $b$ is *invalid* uref.
 
Proof: Since `FR::read(b)` will check `b.closed`, thus first point is proved; And if exists $u \rightarrow b$, then $u$ must be established with `URL::createObjectURL(b)` (TODO), thus `b` must be a file-based one, i.e. `b.blob_impl = BlobImpl::File(id)`, and `id` is part of $u$; Now, consider `id` is *good* (*bad* case is trivial), then `id` is mapped to a `FileStoreEntry`, which has a validity bit, and this bit will be unset by `close` if it exists. Also, this bit can't be reversed to valid since the only way to do this is by `ActivateBlobURL`, which is only called at the time of URL creation. And when `load(u)` request is handled, we check against this bit, thus $u$ uref will be invalid. QED.

#### Theorem #2: After `b.Slice()` returns $b'$, $b'$ will be a valid reference to what $b$ references at that time, and will stay so until it is closed, if $b$ is valid at that time.

Proof. Case analysis on the then-valid $b$:

1. if $b$ refers to a resource through fileID `id`, then since we `inc_ref_id(id)`, thus the resource `id` points to maintains a correct number of references, thus resource will live as long as $b'$. And as long as we have the `JS` GC-pointer in `Sliced`, parent `id` is accessible, thus the resource is accessible.
2. If $b$ refers to a `BlobImpl::Memory`, then as long as we have the `JS` GC-pointer in `Sliced`, the in-memory buffer will be intact.
3. If $b$ refers to a `BlobImpl::Sliced`, first inductively, since we will not construct a `Sliced` with `Sliced` in it, thus any `Sliced(b')` should have `b'` as only memory or file. In file-grandpa case, we will do `inc_ref_id` again, thus accessibility is intact; and trivial in the memory-based one.

#### Theorem #3: After `createObjectURL(b)` returns $u$, $u$ uref will be able to access the resource indirectly unless it is revoked or the `b` bref is closed


Proof. Case analysis on `b`:

1. `b` is file-based with `id`, then it will be activated, and since FSE with set validity will not be removed, plus it is not yet revoked or `b` closed, entry will always be valid.
2. `b` is mem-based, then during the `get_blob_url_id` it will get promoted thus nothing can effect it except revocation and close.
3. `b` is `sliced(b')`, then by fat tree lemma, `b'` will either be file-based or mem-based, mem-based case is trivial; and for file-based, because we will do an `inc_ref_id`, thus maintaining a correct number of references to resource through `id`

The point is that after `slice` happened and at that time real parent $b$ (which can only be either file-based or mem-based) is *readable*, then afterwards whatever happens $b$ (`close` or `revoke`) will not affect $b'$. For `close` this is apparent, since we create a new `Blob` structure; and for `revoke`, there are two possibility:

1. $b$ is file-based at the time of `slice`: In this case, whenever we `createObjectURL` for $b'$, we will `create_sliced_url_id`, which will create a reference on the fm side and `inc_ref` on it to ensure a correct lifetime (XXX: necessary?), then since this entry has its own `is_valid_url` bit, and we only check validity of the outmost entry in `try_read_file`, thus it is correct.
2. mem-based $b$ gets promoted after `slice`
    * it is promoted after we `createObjectURL` on $b'$:
        + In this case, it means that while relation $b' \rightarrow b$ holds, the implementation of $b$ changed (because someone `createObjectURL` on it as well). Then, we revoke it. But $b$ is not the only source of referencing to the resource! As far as I can see, the problem is that, the referencing relationship on the `script` side is explicit (`JS<T>` managed, i.e. GC'd), while the referencing relationship on the `net` side is implicit (manually maintained ref-count), and in this case, basically we should have a way of cast from explicit one to implicit one. (XXX: so there might be a bug in current implementation). So a potential solution is let `dec_ref` only happens at the time of `Drop`, while `revoke` just the validity at the time of `revokeObjectURL`.
    + It is promoted indirectly just at the time we `createObjectURL` on $b'$:
        + In this case, we will create references on fm side, which should be correct


#### Theorem #4: `createObjectURL` will return a valid $u$ point to the $b$ (though the readability of $b$ is always true) until revocation

Proof: Case analysis on $b$,

1. mem-based $b$: we will promote $b$ with validity as `true`, and except for using `RevokeBlobURL`, we can't revert this validity plus this validity guarantee the lifetime of the entry (i.e. `id` is at least as *good* as $u$), this case is proved.
2. file-based $b$: In this case, we will try `ActivateBlobURL` (this needs us to prove first the `id` is *good*, i.e., $\exists b, own(b, id) \Rightarrow good_S(id)$)
3. sliced $b$ of $b'$: Case analysis on $b'$
    - file-based $b'$: We will `create_sliced_url_id`, which ensure the reference validity on fm side
    - mem-based $b$: This can be proved by 2

#### Theorem #5: Calling `revokeObjectURL` on $u$, if $\exists b, u \rightarrow b$, then this reference will be invalidated afterwards

Proof: In our implementation, $u \rightarrow b$ means `u.id` is a *good* id, which is mapped to a FSE with validity bit set.

Thus, by calling `dec_ref` with `unset_url_validity = true`, then `validity` will be set to `false`. Now by lifting the validity bit to the abstract reference edge, we close the proof.

## More Formal Reasoning
### Starting from user's perspective...

Define $S \vDash \text{loadable}(u, r)$ as $$S \vDash \texttt{load}(u) \{ res.\text{isResponseOf}(res, r) \}$$.

Define $S \vDash \text{readable}(b, r)$ as $$ \exists l.S(l) \mapsto b\wedge S \vDash \texttt{read}(b) \{ bytes.\text{isBytesOf}(bytes, r)\}$$


Then, constructions:

Select file:

$$\{ S \vDash \text{isValidPath}(p, r) \} \texttt{new_from_selected}(p) \{ b.\exists fresh(l). S [l \mapsto b] \vDash \text{readable}(b, r)\}$$

Construct from bytes:

$$ \{ \text{isBytesOf}(bytes, r) \} \texttt{new_from_bytes}(bytes) \{ b.\exists fresh(l). S [l \mapsto b] \vDash\text{readable}(b, r) \}$$

Close: $$\exists l.\{ S(l) \mapsto b\}\texttt{close}(b) \{ S[l \mapsto b'] \vDash \forall r, \neg readable(b', r)\}$$

Slice: $$\{ S \vDash \text{readable}(b, r)\} \texttt{slice}(b) \{ b'. \exists fresh(l). S [l \mapsto b'] \vDash \text{readable}(b', r)\}$$

Create URL: $$\{ S \vDash readable(b, r)\}\texttt{createURL}(b) \{ u.S \vDash loadable(u, r) \}$$

Revoke URL: $$\texttt{revokeURL}(u) \{ \forall r, \neg S \vDash loadable(u, r) \}$$

### Some key invariants
Define $NoLeak(S)$ as $\forall id.(\exists e. S \vDash id \leadsto e) \iff (\exists o.Owns(o, id))$.

Here, the $o$ might be sliced `FileStoreEntry`, `BlobImpl::File` blob, or user (URL). Apparently, the first two are countable, so we account them in `refs` field of `e` (`FileStoreEntry`), and the last one is uncountable, so we reflect this in `is_valid_url` field.

Thus, we have: $$\forall e, id. (S \vDash id \leadsto e) \rightarrow (e.refs = \#b.FileBased(b, id) + \#e'.(\exists id', S\vDash id' \leadsto e' \wedge Sliced(e', id)))$$

and $$S \vDash Owns(o, id) = (\exists b. S \vDash BlobOwner(b, o) \wedge FileBased(b, id))\vee (\exists e.EntryOwner(e, o) \wedge S \vDash Sliced(e, id)) \vee (\exists u.URLOwner(u, o) \wedge (\exists r.S \vDash loadable(u, r)))$$

TODO: Now it should be relatively easy to further verify against the implementation.


## Misc

### Spec says
What is a blob:

> A Blob object refers to a byte sequence, and has a size attribute which is the total number of bytes in the byte sequence, and a type attribute, which is an ASCII-encoded string in lower case representing the media type of the byte sequence.

Close blob:

> A Blob is said to be closed if its `close()` method has been called

Slice blob:

> The `slice()` method returns a new Blob object with bytes ranging from the optional start parameter up to but not including the optional end parameter, and with a type attribute that is the value of the optional contentType parameter.

> S has a readability state equal to that of O’s readability state.

> Note: The readability state of the context object is retained by the Blob object returned by the `slice()` call; this has implications on whether the returned Blob is actually usable for read operations or as a Blob URL.


load:

> Note: The algorithm assumes that invoking methods have checked for readability state. A Blob in the CLOSED state must not have a read operation called on it.

> Responses that do not succeed with a 200 OK act as if a network error has occurred. Network errors are used when:
> The Blob URL does not have an entry in the Blob URL Store.
> The Blob has been closed; this also results in the Blob URL not having an entry in the Blob URL Store.

create uref:

> If called with a Blob argument that has a readability state of CLOSED, user agents must return the output of the Unicode Serialization of a Blob URL.

> Add an entry to the Blob URL Store for url and blob.


> If the url refers to a Blob that has a readability state of CLOSED OR if the value provided for the url argument is not a Blob URL, OR if the value provided for the url argument does not have an entry in the Blob URL Store, this method call does nothing. User agents may display a message on the error console.

> A global object which exposes `URL.createObjectURL()` or `URL.createFor()` must maintain a Blob URL Store which is a list of Blob URLs created by the `URL.createObjectURL()` method or the `URL.createFor()` method, and the Blob resource that each refers to.

> When this specification says to add an entry to the Blob URL Store for a Blob URL and a Blob input, the user-agent must add the Blob URL and a reference to the Blob it refers to to the Blob URL Store.

revoke uref:

> If the context object has an entry in the Blob URL Store, remove the entry that corresponds to the context object.

>  Revocation of a Blob URL decouples the Blob URL from the resource it refers to, and if it is dereferenced after it is revoked, user agents must act as if a network error has occurred

> When this specification says to remove an entry from the Blob URL Store for a given Blob URL or for a given Blob, user agents must remove the Blob URL and the Blob it refers to from the Blob URL Store. Subsequent attempts to dereference this URL must result in a network error.
