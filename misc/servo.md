Notes on Servo
====

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

---

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
The DOM tree of Servo has versioned nodes and COW. So write can modify the DOM in parallel with readers.

*The interface to the DOM is not currently type-safe.*

### Display list
The list is a sequence of high-level drawing commands created by the layout task.

The list is deeply immutable.



