
## File Input as form data set

> Otherwise, if the field element is an input element whose type attribute is in the File Upload state, then for each file selected in the input element, append an entry to the form data set with the name as the name, the file (consisting of the name, the type, and the body) as the value, and type as the type. If there are no selected files, then append an entry to the form data set with the name as the name, the empty string as the value, and application/octet-stream as the type.

However, the `value` in `FormDatum` is a `String`, which means that we need to decide a way to copose the name, type and body of file.

