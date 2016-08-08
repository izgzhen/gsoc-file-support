Final weeks
====

I will batch the left two weeks into this single schedule.

The following is copied from week 13's report, with updates.

## [updated] TODOs before final evaluation

Priority of completion before final: [H] - High, [M] - Medium, [L] - Low, [x] - finished.

+ Performance tuning
    + [x] Reduce overhead by caching etc
    + [x] Stress-proof: Don't break things just because we (try to) upload a 1GB file **(NOTE: slow though)**
+ Correctness evaluation
    + [x] Abstract out current implementation and propose some basic invariants/properties to check
    + [x] Also, due to the complexity of current design, a clear description of relationship with spec would be good to handle the future spec changes
+ Testing
    + [x] Make a checklist of existing tests and try to solve them
    + [x] Add more tests, esp. covering some intricate points of spec
+ Final evaluation
    + [M] Going through the in-tree doc and make sure that the it is complete & consistent
    + [L] Produce a list of related TODOs for future reference
+ (Nice-to-have-but-not-critical) Functional requirements
    + [L] ArrayBuffer/ArrayBufferView support
    + [L] Blob URL fragment identifier implementation
    + [L] `URL.createFor` implementation

Basically the major heavy work is done by the time of writing this line.

I will get the "final evaluation" part done, and starting something else depending on the situation.
