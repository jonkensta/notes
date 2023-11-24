## References and Borrowing

In Rust, the borrow checker enforces a rule to prevent data races and ensure memory safety. This rule states that there can be at most one mutable reference to a value at a time.

It's crucial to understand that ownership and borrowing are distinct concepts in Rust:

1. Ownership:
   - When you own a value, you have full control over it, including the ability to mutate it.
   - Ownership is exclusive; there can only be one owner of a value at a time.
2.  Borrowing:
   - Borrowing allows temporary access to a value without transferring ownership.
   - When a value is mutably borrowed, only the holder of the mutable reference can modify the value, and the owner is temporarily restricted from modifying or reading it until the mutable reference goes out of scope.

So, even though the owner could conceptually be considered a reference, Rust's borrow checker ensures that the owner cannot mutate or read the value while it is mutably borrowed. This design prevents potential data races and guarantees memory safety by disallowing simultaneous mutable references
