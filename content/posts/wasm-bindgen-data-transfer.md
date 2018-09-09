---
title: "Wasm-Bindgen"
date: 2018-09-09T12:33:01+02:00
draft: true
---

In recent projects of mine, I've been using WebAssembly quite a bit. WebAssembly (Wasm) is "a new binary instruction format for a stack based virtual machine" that let's you use languages besides JavaScript to run code on a web page. In my opinon, the most promising of these languages, due to its lack of a need for a runtime and great tooling is Rust. The best way to use Rust with WebAssembly is through Wasm-Bindgen. Wasm-Bindgen makes it easy to write Rust code that compiles down to web assembly that is easily interopable.

In this post, we'll be examining how Wasm-Bindgen creates a bridge between Rust and JavaScript. We'll take a look at major Rust types and how those types are transformed and made consumable from Rust.

## The Problem

Before we can look at how Wasm-Bindgen works, we need to understand the base interoperability between Wasm and JavaScript. Wasm has an extremely limited interoperability story. Wasm only understands 32 and 64-bit floating point and integer numbers. Beyond this, Wasm is capable of calling JavaScript functions (as long as the expect just 32 and 64 bit floats and integers) and it can expose it's own functions to javascript (again only excepting those 4 types of arguments). Naturally this makes working with Wasm pretty difficult.

This is where Wasm-Bindgen comes in. Instead of directly writing Rust and Javascript that deals with this very limited interopability directly, the library/tool let's you write Rust and Javascript that deals with a much more rich set of types. Wasm-Bindgen then generates the glue code that JavaScript and Wasm actually understand.

Now that we have this background we can look at the code that one might right in Rust and Javascript and then the generated code that makes this possible. I won't take the time to explain exactly how Wasm-Bindgen is used - the code examples here should be clear enough on their own for our purposes. To learn how to actually create a project using Wasm-Bindgen, you can take a look at their [wondeful documentation](https://rustwasm.github.io/wasm-bindgen/).

## Numbers

The easiest example of how Wasm-Bindgen enables interoperability is through numbers. Since Wasm supports integers and floats (32- and 64-bit), Wasm-Bindgen normally has do to very little. Despite numbers being relatively easy, they still go through similar machinary as other types and thus are good first examples.

Take for example the following function:

```rust
#[wasm_bindgen]
pub fn test(a: u16, b: u8) ->  u16 {
  a + (b as u16)
}
```

Using the awesome tool [`cargo-expand`](https://github.com/dtolnay/cargo-expand) we can expand the `wasm_bindgen` macro and see what code is generated for us (remember to use the `cargo-expand` tool with the `wasm32-unknown-unknown` target or else you won't see the right thing!):

```rust
// ...imports above
pub fn test(a: u16, b: u8) -> u16 {
  a + (b as u16)
}
#[export_name = "test"]
#[allow(non_snake_case)]
#[cfg(all(target_arch = "wasm32", not(target_os = "emscripten")))]
pub extern "C" fn __wasm_bindgen_generated_test(
  arg0: <u16 as ::wasm_bindgen::convert::FromWasmAbi>::Abi,
  arg1: <u8 as ::wasm_bindgen::convert::FromWasmAbi>::Abi,
  ) -> <u16 as ::wasm_bindgen::convert::IntoWasmAbi>::Abi {
  ::wasm_bindgen::__rt::link_mem_intrinsics();
  let _ret = {
    let mut __stack = unsafe { ::wasm_bindgen::convert::GlobalStack::new() };
    let arg0 = unsafe { <u16 as ::wasm_bindgen::convert::FromWasmAbi>::from_abi(arg0, &mut __stack) };
    let arg1 = unsafe { <u8 as ::wasm_bindgen::convert::FromWasmAbi>::from_abi(arg1, &mut __stack) };
    test(arg0, arg1)
  };
  <u16 as ::wasm_bindgen::convert::IntoWasmAbi>::into_abi(_ret, &mut unsafe {
    ::wasm_bindgen::convert::GlobalStack::new()
  })
}

// Next comes the wbindgen_describe function ...
```

Wow, a lot to unpact there. Ok, let's start at the top. We first have our function completely unchanged. That's good to know.

Next comes another function with a lot of attributes above it. The first attribute `export_name = "test"` is very interesting. This function will be exported with the name "test", the same name as our function we wrote. That means that when someone calls the "test" function on our WebAssembly module, they'll actually be calling this generated function. Of course, within the generated function is the function, we wrote, so our code will eventually get called. But what's the generated code doing?

We're going to skip over the `link_mem_intrinsics`. This is an interesting function that you can read about [here](https://github.com/rustwasm/wasm-bindgen/blob/630ac1c16942fadd758be7fa711dcd0b26bce2a2/src/lib.rs#L781-L829).

First, notice that our arguments are now `<u16 as ::wasm_bindgen::convert::FromWasmAbi>::Abi` and `<u8 as ::wasm_bindgen::convert::FromWasmAbi>::Abi`. This means this function will be accepting a some value of type `Abi` that's defined on a trait `FromWasmAbi` that's defined for both u16 and u8. `FromWasmAbi` is the trait that defines how values are converted from the types that WebAssembly understands to the richer types Rust is capable of dealing with.

We can see that the arguments are converted using the `from_abi` function to turn the values into the right Rust code. So what does `from_abi` look like for `u16`?

```rust
impl FromWasmAbi for u16 {
  type Abi = u32;

  #[inline]
  unsafe fn from_abi(js: u32, _extra: &mut Stack) -> u16 { js as u16 }
}
```

Interesting - we now know that the argument we're getting to the exported function is actually a u32, which makes sense. A `u32` is a 32-bit integer, one of the 4 types WebAssembly is capable of understanding. All the `from_abi` function does is take the `u32` and convert it to a `u16`. Pretty straight forward. This means that if the calling code calls our `test` function with a number larger than what can fit in a 16 bit integer, the top 16 bits will just be dropped.

The story for our `u8` argument is exactly the same with the `from_abi` function just turning the `u32` into a `u8` instead of a `u16`.

But what's with the weird `GlobalStack` that's passed to all these functions? As we see it's not needed at all for our purposes (none of the `from_abi` functions are using it!), but it will play a role when we have functions that take more complicated types. Stay tuned!

Lastly, the opposite of `from_abi` - `into_abi` is called with the result of the call to our original `test` function. As you might guess the `IntoWasmAbi` trait is the opposite of the `FromWasmAbi` trait. It takes Rust types and describes how to turn them into one of the 4 types that WebAssembly actually understands. Let's take a look at the implementation of this trait for `u16`:

```rust
impl IntoWasmAbi for u16 {
  type Abi = u32;

  #[inline]
  fn into_abi(self, _extra: &mut Stack) -> u32 { self as u32 }
}
```

As we'd expect the function just takes our `u16` and turns it into a `u32` that WebAssembly will understand.

And that's the story from Rust and WebAssembly for numbers. The function will be compiled and can be called from JavaScript as is.

What happens if we pass the wrong type to our WebAssembly module (like say a string or an object)? Well, in reality when we compile a WebAssembly module we're not given direct access to the functions defined in that module. Instead wrappers are created automatically that coerce the arguments to the proper type if possible. These corecions follow the same rules that corecions in normal JavaScript do (e.g., undefined is coerced to 0).

In the above example, if you feed a negative number to the function and have compiled in debug mode, the module will panic, because the negative number is interpreted as a positive number and the additon causes an overflow. Compiling in release mode causes Rust to not panic when a `+` addition leads to an overflow so everything works fine.

## Strings

Ok numbers are fairly straight forward. What about strings? First, we'll take a look at owned Strings with the following example:

```rust
#[wasm_bindgen]
pub fn test(mut a: String) -> String {
  a.push_str(" :)");
  a
}
```

And expanding this out, we get:

```rust
pub fn test(mut a: String) -> String {
  a.push_str(" :)");
  a
}
#[export_name = "test"]
#[allow(non_snake_case)]
#[cfg(all(target_arch = "wasm32", not(target_os = "emscripten")))]
pub extern "C" fn __wasm_bindgen_generated_test(
  arg0: <String as ::wasm_bindgen::convert::FromWasmAbi>::Abi,
  ) -> <String as ::wasm_bindgen::convert::IntoWasmAbi>::Abi {
  ::wasm_bindgen::__rt::link_mem_intrinsics();
  let _ret = {
    let mut __stack = unsafe { ::wasm_bindgen::convert::GlobalStack::new() };
    let arg0 = unsafe { <String as ::wasm_bindgen::convert::FromWasmAbi>::from_abi(arg0, &mut __stack) };
    test(arg0)
  };
  <String as ::wasm_bindgen::convert::IntoWasmAbi>::into_abi(_ret, &mut unsafe {
    ::wasm_bindgen::convert::GlobalStack::new()
  })
}
```

As you can tell the example of the String is exactly the same as the example we saw before (albeit this time with one fewer argument). Let's see if the `from_abi` implemenation for String is different from `u16` and `u8`.
