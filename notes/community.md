Community
----

> Everything about living a decent life in Servo community :P
>
> *Community* here is actually more about effective corporation, I think


1. `r? @SomeOne`  in a comment and the reviewer gets changed
2. `./mach update-cargo -p uuid --precise 0.2.2` to resolve dependency problem
3. If there is duplicated code in `Constructor` of parent and child, don't nest `Constructor` but instead write another helper function
4. Write low-level raw bindings in a `-sys` crate
5. When someone `ping` you, a `pong` looks more professional than a `?`
6. Every time you have to `unwrap` something, ask yourself why
7. Change `FAIL` to `PASS` in test `.ini` to get more error info out
8. Test `.ini`s might contain out-dated specs
9. Try to note down points of discussion on IRC
10. With major changes to the script interface, go through the WPT tests again
11. Avoid adding another layer of indirection (from the viewpoint of impl) if it is not necessary
12. Changing inner interfaces might break the unit tests, use `./mach test-unit`
13. `rebase` and `checkout` might invalidate the built packages -- cache two or more repos locally might avoid extra cost (of time)
14. You can have multiple tracks -- Slow ones and tight ones, progressing in parallel
15. Learn to review by yourself *effectively* first
16. State clearly (and succinctly) your design and concern in commit message
