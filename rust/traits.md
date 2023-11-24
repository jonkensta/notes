# Traits

## Orphan Rule

In Rust, the "orphan rule" prohibits the creation of traits or methods for types defined outside your crate. This principle ensures that the implementation of a trait is in close proximity to the trait or the type it operates on, preventing conflicts and unexpected behavior arising from multiple implementations scattered across different codebases.

To overcome this limitation, Rust provides a workaround known as the "newtype" pattern. This involves creating a newtype, essentially a wrapper around the original type, and then implementing the desired trait or method for the wrapper. This way, you can extend the functionality of types from external crates without directly modifying them.

Here's an example using the newtype pattern:

```rust
pub struct MyOption<T>(Option<T>);

// Implement the new method for the wrapper
impl<T> MyOption<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T,
    {
        match self.0 {
            Some(x) => x,
            None => f(),
        }
    }
}

fn main() {
    let my_option = MyOption(Some(42));
    let result = my_option.unwrap_or_else(|| 10);
    println!("Result: {}", result);
}
```

In this example, `MyOption` serves as a stand-in for `Option`, allowing the definition of a new method, `unwrap_or_else()`, while adhering to Rust's rules and ensuring code organization and safety.
