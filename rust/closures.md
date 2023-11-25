# Closures

## Capturing References or Moving Ownership

Closures can captures from their environment in three ways:

1. Borrowing mutably
2. Borrowing immutably
3. Taking ownership

If you want to force the closure to take ownership of the values it uses in the environment even though the body of the closure doesn’t strictly need ownership, you can use the `move` keyword before the parameter list.

## Moving Captured Values Out of Closures

Once values are captured or moved into closures,
the closure body defines what happens to those values later when the closure is called.
A closure body can do one of the following:

-   move a captured value out of the closure by returning it
-   mutate the captured value
-   neither move nor mutate the value, or
-   capture nothing from the environment to begin with.

## `Fn` Traits

The way that a closure body moves or mutates the values it has captured from the environment affects which traits it implements.
Closures will automatically implement the following traits depending on how the closure's body handles the values:

-   `FnOnce` applies to closures that can only be called once beacuse they move a captured value.
-   `FnMut` applies to closures that do not move values out of its body but may mutate a captured values.
-   `Fn` applies to closures that do not move or mutate captured values.

Below is an example of a function that only implements `FnOnce` because it moves a value out of its environment.

```rust
fn main() {
    let mut my_vec = vec![];
    let value = String::from("function called");

    let can_be_called_once = || {
        my_vec.push(value);
    };

    can_be_called_once();
    can_be_called_once(); // error[E0382]: use of moved value: `can_be_called_once`
}
```
