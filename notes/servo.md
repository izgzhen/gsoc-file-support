Notes on Servo
===

## Overview

* Task-based architecture: Major components in the system should be factored into actors with isolated heaps, with clear boundaries for failure and recovery.
* Concurrent, tiled and layered rendering.

Engine is splited into:

1. Content
2. Layout
3. Renderer
4. Compositor

It also manages

* Image cache
* Resource loading
* Network

## Constellation

Each constellation instance manages a pipeline of tasks that:

1. Accepts input
2. Runs JavaScript against the DOM (Script: create and own the DOM and execute the JavaScript)
3. Performs layout (Layout: Take a snapshot of the DOM, calculates styles, and constructs the flow tree)
4. Builds display lists
5. Renders display lists to tiles (Renderer: Renders visible portions to one or more tiles, possibly in parallel)
6. Composites the final image to a surface (Compositor: Composites the tiles and send to the screen; also, it manages the UI response)

The two complex data structures:

1. DOM
2. Display list

> Figuring out an efficient and type-safe way to represent, share, and/or transmit these two structures is one of the major challenges for the project.

### DOM
[DOM doc](https://doc.servo.org/script/dom/index.html)

The DOM tree of Servo has versioned nodes and COW. So write can modify the DOM in parallel with readers.

*The interface to the DOM is not currently type-safe.*

### Display list
The list is a sequence of high-level drawing commands created by the layout task.

The list is deeply immutable.


## API

### Commonly seen wrappers
1. [JS](https://doc.servo.org/script/dom/bindings/js/struct.JS.html): A traced reference to a DOM object. It is very dangerous; if garbage collection happens with a JS<T> on the stack, the JS<T> can point to freed memory. This should only be used as a member field in other DOM objects.
2. [Root](https://doc.servo.org/script/dom/bindings/js/struct.Root.html): A rooted reference to a DOM object. The JS value is pinned for the duration of this object's lifetime; roots are additive, so this object's destruction will not invalidate other roots for the same JS value. Roots cannot outlive the associated RootCollection object. It resides on the stack.
3. [DOMRefCell](https://doc.servo.org/script/dom/bindings/cell/struct.DOMRefCell.html): A mutable field in the DOM. Enhanced version of common `RefCell`
4. [Trusted](https://doc.servo.org/script/dom/bindings/refcounted/struct.Trusted.html): A safe wrapper around a raw pointer to a DOM object that can be shared among threads for use in asynchronous operations. The underlying DOM object is guaranteed to live at least as long as the last outstanding Trusted<T> instance.

[Doc on smart pointer managed by JS runtime](http://doc.servo.org/script/dom/bindings/js/)


### Traits:
1. [JSTraceable](https://doc.servo.org/script/dom/bindings/trace/trait.JSTraceable.html): A trait to allow tracing (only) DOM objects.
2. [Reflectable](https://doc.servo.org/script/dom/bindings/reflector/trait.Reflectable.html): A trait to provide access to the [Reflector](https://doc.servo.org/script/dom/bindings/reflector/struct.Reflector.html) for a DOM object.
