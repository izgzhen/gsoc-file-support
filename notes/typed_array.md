## Mail-list discussion

### [Rooting APIs and idiomatic Rust](https://groups.google.com/d/topic/mozilla.dev.servo/tqqwgAmhp8o/discussion)

jdm: Specifically, my problems stem from this - given that creating a typed array is fallible, as is wrapping an existing typed array reflector, we decided to make the constructors for the typed array APIs return `Result` values. Unfortunately this interacts poorly with `AutoGCRooter`, since it stores pointers to Rust objects that end up moving when unwrapping the `Result`.

terrence: I think we've already solved this for you from the C++ side, although it will probably take a bit of new rust wrapping to make work. The weirdness with the `AutoFooRooters` is that they conflate traceability with root creation/destruction. We're currently working to move everything over to a world where things can be traceable -- derives from `JS::Traceable` in C++ or implements the Traceable trait in rust -- as a separate concept from `JS::Rooted`. The upshot is twofold: first you can have `Rooted<MyTraceable>`; secondly, (and more relevantly for your particular problem) is that construction is decisively split from the initialization of the `Rooted`. In C++ terms this looks like: 

The old way: 

    class Foo : public CustomAutoRooter { 
      virtual void trace(JSTracer*) {} 
    }; 
    Foo foo(cx, ...args...); 

The new way: 

    class Foo : public JS::Traceable { 
      static void trace(Foo* self, JSTracer* trc) {} 
    }; 
    Rooted<Foo> foo(cx, Foo(...args...));  // Note the separate Foo and Rooted constructors.

In Mozilla's C++ dialect, these two are roughly equivalent because fallible init goes though a manual `|bool init(...args...)|` function after rooting (and the tracer needs to deal with partial init). In the rust world that I envision, you'd be able to fallibly construct something like (please excuse me if my rust ideoms are old /and/ shaky): 

    let foo: Rooted<Foo> = match Foo::new(..args..) { 
      Err(err) => handleErr!(...), 
      Ok(foo) => Rooted<Foo>(foo) 
    };

jdm: This looks like a plausible solution to my problem. Nevertheless, it looks like if `Foo::new` returns a value which contains GC pointers then  it's a potential hazard, right? 

## [SM: GC_Rooting_Guide](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/GC_Rooting_Guide)


There are some situations when using `JS::Rooted<T>` is not possible, or is undesirable for performance reasons.  To cover these cases, there are various `AutoRooter` classes that can be used.

If your case is not covered by one of these, it is possible to write your own by deriving from `JS::CustomAutoRooter` and overriding the virtual `trace()` method.  The implementation should trace all the GC things contained in the object by calling the `JS_CallTracer()` functions.

## Servo Impl

- [servo/components/script/dom/bindings/conversions.rs](https://github.com/servo/servo/blob/master/components/script/dom/bindings/conversions.rs#L499)
- [servo/components/script/dom/crypto.rs](https://github.com/servo/servo/blob/master/components/script/dom/crypto.rs#L8)
- [servo/components/script/dom/webglrenderingcontext.rs](https://github.com/servo/servo/blob/master/components/script/dom/webglrenderingcontext.rs#L10-L11)

## Servo PR/Issues

- [Assertion failure: `this->is<js::TypedArrayObject>()` with bufferData #6601](https://github.com/servo/servo/issues/6601), [Related PR](https://github.com/servo/servo/pull/9065/)
- [Add a high-level API for JS typed arrays #6779](https://github.com/servo/servo/pull/6779)
- [webgl: Make a general way to get data from a JS array buffer view #8970](https://github.com/servo/servo/pull/8970)
- [DataView type in WebIDL](https://github.com/servo/servo/issues/9530)

## Related
+ [Implement DOMMatrix. #8509](https://github.com/servo/servo/issues/8509)
+ [Port SpiderMonkey interface object support from Gecko #5014](https://github.com/servo/servo/issues/5014)


