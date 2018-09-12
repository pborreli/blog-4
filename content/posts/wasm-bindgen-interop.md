---
title: "Rust and JavaScript Interop"
date: 2018-09-12T09:33:01+02:00
---

In recent projects of mine, I've been using [WebAssembly](https://webassembly.org/) quite a bit. WebAssembly (Wasm) is "a new binary instruction format for a stack based virtual machine" that lets you use languages besides JavaScript to run code on a web page - usually either for performance reasons or to run code you'd like to share across different platforms. In my opinion, the most promising of these languages, due to its lack of a need for a runtime and great tooling is [Rust](https://www.rust-lang.org/).

The best way to use Rust with WebAssembly is through [Wasm-Bindgen](https://github.com/rustwasm/wasm-bindgen). Wasm-Bindgen makes it easy to write Rust code that compiles down to Web Assembly that is easily interoperable with JavaScript. It is both a library for generating the boilerplate Rust for functions that JavaScript can call as well as a cli tool for generating the boilerplate JavaScript that can easily interop with WebAssembly.

In this post, we'll be examining how Wasm-Bindgen creates a bridge between Rust and JavaScript. We'll take a look at numbers and strings and how those types are transformed and made consumable from Rust (as compiled WebAssembly) by JavaScript. This post will take a look at internals to Wasm-Bindgen. What's written below is true for the current version of Wasm-Bindgen at the time of this writing (`0.2.21`), but the details might change in the future.

## The Problem

Before we can look at how Wasm-Bindgen works, we need to understand the base interoperability between Wasm and JavaScript. Wasm has an extremely limited interoperability story. Wasm code only understands 32 and 64-bit floating point and integer numbers. Beyond this, Wasm is capable of calling JavaScript functions (as long as they expect just 32 and 64 bit floats and integers), and it can expose its own functions to JavaScript (again only accepting those 4 types of arguments). Naturally this makes working with Wasm pretty difficult. How can we write functions that take richer types like strings, arrays, objects and classes?

This is where Wasm-Bindgen comes in. Instead of directly writing Rust and JavaScript that deals with this very limited interoperability directly, the library/tool let's you write Rust and JavaScript that deals with a much more rich set of types. Wasm-Bindgen then generates the glue code that boils the JavaScript and Wasm interop to only use the 4 allows types.

Now that we have this background we can look at the code that one might write in Rust and JavaScript and then the generated code that makes this possible. I won't take the time to explain exactly how Wasm-Bindgen is used - the code examples here should be clear enough on their own for our purposes. To learn how to actually create a project using Wasm-Bindgen, you can take a look at their [wonderful documentation](https://rustwasm.github.io/wasm-bindgen/).

## Numbers

The easiest example of how Wasm-Bindgen enables interoperability is through numbers. Since Wasm supports integers and floats (32- and 64-bit), Wasm-Bindgen normally has to do to very little. Despite numbers being relatively easy, they still go through similar machinery as other types and thus make for a good first example.

Take for example the following function:

```rust
#[wasm_bindgen]
pub fn add(a: u16, b: u8) ->  u16 {
  a + (b as u16)
}
```

Here we have a pretty simple rust function named `add` that has been annotated with the `wasm_bindgen` attribute. From JavaScript we'll be able to call this function like so: `wasm.add(1, 2)`. In order to be able to do this though, Wasm-Bindgen will generate both Rust and JavaScript code for us.

Using the awesome tool [`cargo-expand`](https://github.com/dtolnay/cargo-expand) we can expand the `wasm_bindgen` macro and see what Rust code is generated for us (remember to use the `cargo-expand` tool with the `wasm32-unknown-unknown` target or else you won't see the right thing!):

```rust
// ... imports above

pub fn add(a: u16, b: u8) -> u16 {
  a + (b as u16)
}

#[export_name = "add"]
#[allow(non_snake_case)]
#[cfg(all(target_arch = "wasm32", not(target_os = "emscripten")))]
pub extern "C" fn __wasm_bindgen_generated_add(
  arg0: <u16 as ::wasm_bindgen::convert::FromWasmAbi>::Abi,
  arg1: <u8 as ::wasm_bindgen::convert::FromWasmAbi>::Abi,
) -> <u16 as ::wasm_bindgen::convert::IntoWasmAbi>::Abi {
  ::wasm_bindgen::__rt::link_mem_intrinsics();
  let _ret = {
    let mut __stack = unsafe { ::wasm_bindgen::convert::GlobalStack::new() };
    let arg0 = unsafe { <u16 as ::wasm_bindgen::convert::FromWasmAbi>::from_abi(arg0, &mut __stack) };
    let arg1 = unsafe { <u8 as ::wasm_bindgen::convert::FromWasmAbi>::from_abi(arg1, &mut __stack) };
    add(arg0, arg1)
  };
  <u16 as ::wasm_bindgen::convert::IntoWasmAbi>::into_abi(_ret, &mut unsafe {
    ::wasm_bindgen::convert::GlobalStack::new()
  })
}

// Next comes the wbindgen_describe function ...
```

Wow, a lot to unpack there. Ok, let's start at the top. We first have our `add` function completely unchanged. That's good to know.

Next comes another function named `__wasm_bindgen_generated_add` with a lot of attributes above it. The first attribute `export_name = "add"` is very interesting. This function will be exported with the name `add` (i.e., the same name as the function we wrote ourselves) and not with the name `__wasm_bindgen_generated_add`. That means that when someone calls the `add` function on our WebAssembly module, they'll actually be calling this generated function. Of course, within the generated function is the original function `add` that we wrote, so our code will eventually get called. But what's the generated code doing?

We're going to skip over the `link_mem_intrinsics`. This is an interesting function that you can read about [here](https://github.com/rustwasm/wasm-bindgen/blob/630ac1c16942fadd758be7fa711dcd0b26bce2a2/src/lib.rs#L781-L829).

First, notice that our arguments are now `<u16 as ::wasm_bindgen::convert::FromWasmAbi>::Abi` and `<u8 as ::wasm_bindgen::convert::FromWasmAbi>::Abi`. This means this function will accept values of type `Abi` which is an associated type defined on a trait name `FromWasmAbi`. (If you're unfamiliar with associated types you can read about them [here](https://doc.rust-lang.org/book/second-edition/ch19-03-advanced-traits.html#specifying-placeholder-types-in-trait-definitions-with-associated-types)) `FromWasmAbi` is the trait that defines how values are converted from the types that WebAssembly understands (namely 32- and 64-bit integers and floats) to the richer types Rust is capable of dealing with. The associated type `Abi` is the type of value we'll be getting passed from JavaScript (i.e. one of the four types WebAssembly understanding). As we'll see in second when we take a look at the FromWasmAbi implementation for `u8` and `u16` the `Abi` associated is in fact `u32` (one of the four types WebAssembly understands).

On the fourth and fifth lines of function we can see that the arguments are converted using the `from_abi` function that `FromWasmAbi` defines. Again, `from_abi` will take a value of some type that WebAssembly can understand and turn it into richer types that our Rust code can work with. So what does `from_abi` look like for `u16`?

```rust
impl FromWasmAbi for u16 {
  type Abi = u32;

  #[inline]
  unsafe fn from_abi(js: u32, _extra: &mut Stack) -> u16 { js as u16 }
}
```

Interesting - we now know that the argument we're getting to the exported function is actually a u32. All the `from_abi` function does is take the `u32` and convert it to a `u16`. Pretty straight forward. This means that if the calling code calls our `add` function with a number larger than what can fit in a 16 bit integer, the top 16 bits will just be dropped.

The story for our `u8` argument is exactly the same with the `from_abi` function just turning the `u32` into a `u8` instead of a `u16`.

But what's with the weird `GlobalStack` that's passed to all these functions? As we see it's not needed at all for our purposes (none of the `from_abi` functions are using it!). It is used for richer types than numbers and strings though.

Lastly, the opposite of `from_abi` - `into_abi` is called with the result of the call to our original `add` function. As you might guess the `IntoWasmAbi` trait is the opposite of the `FromWasmAbi` trait. It takes Rust types and describes how to turn them into one of the 4 types that WebAssembly actually understands. Let's take a look at the implementation of this trait for `u16`:

```rust
impl IntoWasmAbi for u16 {
  type Abi = u32;

  #[inline]
  fn into_abi(self, _extra: &mut Stack) -> u32 { self as u32 }
}
```

As we'd expect the function just takes our `u16` and turns it into a `u32` that WebAssembly will understand.

And that's the story from Rust and WebAssembly for numbers. The function will be compiled and can be called from JavaScript as is.

In fact, the generated JavaScript that the wasm-bindgen cli tool produces is just a simple wrapper around a direct call to our WebAssembly module:

```javascript
export function add(arg0, arg1) {
  return wasm.add(arg0, arg1)
}
```

What happens if we pass the wrong type to our WebAssembly module (like say a string or an object)? Well, in reality when we compile a WebAssembly module we're not given direct access to the functions defined in that module. Instead the browser runtime provides wrappers that coerce the arguments to the proper type if possible. These coercions follow the same rules that coercions in normal JavaScript do (e.g., undefined is coerced to 0).

In the above example, if you feed a negative number to the function and have compiled in debug mode, the module will panic, because the negative number is interpreted as a positive number and the addition causes an overflow (which panics in debug Rust). Compiling the Rust code in release mode causes Rust to not panic when a `+` addition leads to an overflow so everything works as expected.

## Strings

Ok numbers are fairly straight forward. What about strings? First, we'll be taking a look at owned Strings with the following example:

```rust
#[wasm_bindgen]
pub fn make_smile(mut a: String) -> String {
  a.push_str(" :)");
  a
}
```

And expanding this out, we get:

```rust
pub fn make_smile(mut a: String) -> String {
  a.push_str(" :)");
  a
}

#[export_name = "make_smile"]
#[allow(non_snake_case)]
#[cfg(all(target_arch = "wasm32", not(target_os = "emscripten")))]
pub extern "C" fn __wasm_bindgen_generated_make_smile(
  arg0: <String as ::wasm_bindgen::convert::FromWasmAbi>::Abi,
  ) -> <String as ::wasm_bindgen::convert::IntoWasmAbi>::Abi {
  ::wasm_bindgen::__rt::link_mem_intrinsics();
  let _ret = {
    let mut __stack = unsafe { ::wasm_bindgen::convert::GlobalStack::new() };
    let arg0 = unsafe {
      <String as ::wasm_bindgen::convert::FromWasmAbi>::from_abi(arg0, &mut __stack)
    };
    make_smile(arg0)
  };
  <String as ::wasm_bindgen::convert::IntoWasmAbi>::into_abi(_ret, &mut unsafe {
    ::wasm_bindgen::convert::GlobalStack::new()
  })
}
```

As you can tell the example of the String is very similar to the example we saw before (albeit this time with one fewer argument). Let's see how the `from_abi` implementation for String is different from `u16` and `u8`.

```rust
impl FromWasmAbi for String {
  type Abi = <Vec<u8> as FromWasmAbi>::Abi;

  #[inline]
  unsafe fn from_abi(js: Self::Abi, extra: &mut Stack) -> Self {
    String::from_utf8_unchecked(<Vec<u8>>::from_abi(js, extra))
  }
}
```

This is interesting. We see here that we're calling the Rust standard library `String::from_utf8_unchecked` method which turns a buffer of bytes into a String without checking that it is actually valid utf-8. This means if we manage to pass a buffer of bytes to this function that's not actually valid utf-8, weird things will happen. We'll see in a little bit how the generated JavaScript protects against this.

The argument to the `String::from_utf8_unchecked` function is a `Vec<u8>` that comes from calling `from_abi` for `Vec<u8>`. The implementation is based on the `FromWasmAbi` for `Vec<T>` which itself is based on the implementation of `FromWasmAbi` for `Box<[T]>`. In the implementation of `FromWasmAbi` for `Box<[u8]>` we can finally see the concrete type of `Abi` is `WasmSlice`. A `WasmSlice` is simply a struct that contains `len` field (i.e., the length of our slice) of type `u32` and a `ptr` field (i.e., a pointer to the beginning of the slice) of type `u32`. Let's first have a look at both the `FromWasmAbi` implementations:

```rust
impl<T> FromWasmAbi for Vec<T> where Box<[T]>: FromWasmAbi<Abi = WasmSlice> {
  type Abi = <Box<[T]> as FromWasmAbi>::Abi;

  unsafe fn from_abi(js: Self::Abi, extra: &mut Stack) -> Self {
    <Box<[T]>>::from_abi(js, extra).into()
  }
}

impl FromWasmAbi for Box<[u8]> {
  type Abi = WasmSlice;

  #[inline]
  unsafe fn from_abi(js: WasmSlice, extra: &mut Stack) -> Self {
    let ptr = <*mut u8>::from_abi(js.ptr, extra);
    let len = js.len as usize;
    Vec::from_raw_parts(ptr, len, len).into_boxed_slice()
  }
}

```

So we can see the implementation for `Vec<T>` just relies on the implementation for `Box<[T]>` and the fact that the Rust standard library allows us to turn a `Box<[u8]>` into a `Vec<u8>`. The implementation of `FromWasmAbi` for `Box<[u8]>` knows how to get a pointer and a length from the `WasmSlice` it gets passed and then turn those values into a `Vec` using the standard libraries function `Vec::from_raw_parts`. It then turns that `Vec` into a `Box<[u8]>`.

And if we keep going down the rabbit hole, we can look into the implementation of `from_abi` for `*mut u8`, but it's not very interesting since it just converts the type from `u32` to a `*mut` pointer.

So now we know that our top level exported function takes a `WasmSlice` which gets converted through `Box<[u8]>` to `Vec<u8>` and finally to `String` which our original function can work on.

Finally, `<String as ::wasm_bindgen::convert::IntoWasmAbi>::into_abi` gets called. Again, this is just the opposite of the `from_abi` functions we've been looking at. Let's have a look:

```rust
impl IntoWasmAbi for String {
  type Abi = <Vec<u8> as IntoWasmAbi>::Abi;

  #[inline]
  fn into_abi(self, extra: &mut Stack) -> Self::Abi {
    self.into_bytes().into_abi(extra)
  }
}
```

Here we see our String gets converted into bytes (i.e., a `Vec<u8>`) and then the `into_abi` implementation for `Vec<u8>` is called:

```rust
impl<T> IntoWasmAbi for Vec<T> where Box<[T]>: IntoWasmAbi<Abi = WasmSlice> {
  type Abi = <Box<[T]> as IntoWasmAbi>::Abi;

  fn into_abi(self, extra: &mut Stack) -> Self::Abi {
    self.into_boxed_slice().into_abi(extra)
  }
}

impl IntoWasmAbi for Box<[u8]> {
  type Abi = WasmSlice;

  #[inline]
  fn into_abi(self, extra: &mut Stack) -> WasmSlice {
    let ptr = self.as_ptr();
    let len = self.len();
    mem::forget(self);
    WasmSlice { ptr: ptr.into_abi(extra), len: len as u32 }
  }
}
```

We can see here how the implementation of `IntoWasmAbi` mirrors `FromWasmAbi`. One interesting thing of now is the use of `mem::forget` in the `into_abi` implementation for `Box<[u8]>` which tells Rust to not drop the `Box<[u8]>`. Without this line Rust would automatically deallocate the `Box<[u8]>` and `ptr` would be a dangling pointer.

Now we have a pretty good understanding of what's happening inside the Rust code that's been compiled to WebAssembly. But to fully understanding what's going on, we'll need to look at the generated code on the JS side:

```javascript
export function make_smile(arg0) {
  const [ptr0, len0] = passStringToWasm(arg0);
  const retptr = globalArgumentPtr();
  wasm.make_smile(retptr, ptr0, len0);
  const mem = getUint32Memory();
  const rustptr = mem[retptr / 4];
  const rustlen = mem[retptr / 4 + 1];
  const realRet = getStringFromWasm(rustptr, rustlen).slice();
  wasm.__wbindgen_free(rustptr, rustlen * 1);
  return realRet;
}
```

Ok, so when we call the `make_smile` function on the JavaScript side, we first take the argument to our function (i.e., a JavaScript String) and pass it to a function call `passStringToWasm`. This function is responsible for converting our String to utf-8 (since JavaScript strings are not normally utf-8, but Rust expects utf-8 strings), allocating space in the Wasm heap, and putting the string there. Let's take a look:

```javascript
function passStringToWasm(arg) {
  const buf = cachedEncoder.encode(arg);
  const ptr = wasm.__wbindgen_malloc(buf.length);
  getUint8Memory().set(buf, ptr);
  return [ptr, buf.length];
}
```

Here we see `cachedEncoder` (simply a [utf-8 encoder](https://developer.mozilla.org/en-US/docs/Web/API/TextEncoder) that is cached for reuse) changing the string to utf-8. Next we call a function on our wasm module that Wasm-Bindgen has generated for us `__wbindgen_malloc` that simply allocates memory on the WebAssembly heap. Next we call `getUint8Memory` which simply gives us a view of the WebAssembly heap as a [`Uint8Array`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) and `set` which sets the utf-8 buffer we've created at the point in memory we just allocated. Finally we return the pointer to that memory and the length of the buffer.

After, passing the String to Wasm, we call the `globalArgumentPtr` function to get a pointer to somewhere in the Wasm heap. This is a special place in the heap decided by Wasm-Bindgen where we can put function return values that we need to return by pointer. Why we need to return a value by pointer is something we'll discuss below. `globalArgumentPtr` uses another Wasm-Bindgen generated function like `__wbindgen_malloc` named `__wbindgen_global_argument_ptr` to accomplish this.

When we finally get to calling `make_smile` on our WebAssembly module we call the function with three arguments `retptr`, `ptr0` and `len0` and we don't get a return value. But wait a minute - our exported function in the Rust code has only one argument and a return value... Wouldn't we expect the call signature of our exported function from Rust to match the call signature that we see when we use that function from JavaScript?

The reason for this is the weirdness of what Rust and LLVM decided the "ABI" of function calls for the Wasm target would be. There are two rules that combine together to produce the interesting call signature we see in the JavaScript:
* Complex arguments (i.e. arguments that are combinations of the 4 basic types Wasm supports) are "splatted" (meaning passed as separate arguments)
* Values that are "too big" are returned by pointer that is passed as the first argument

The first rule means that our `WasmSlice` which as we've seen is a struct composed of two `u32`s is broken into those two values and those two values are passed as separate arguments. The second rule means that instead of our function returning a `WasmSlice` it instead puts that `WasmSlice` as the location specified by the first argument. Why a struct of two `u32`s is considered too big when Wasm supports 64-bit numbers is a topic for another time...

Once this is done, the function then turns the pointer and the length inside the `WasmSlice` into a string using `getStringFromWasm`:

```javascript
function getStringFromWasm(ptr, len) {
  return cachedEncoder.decode(getUint8Memory().subarray(ptr, ptr + len));
}
```

Here, we're again using the `cachedEncoder` this time to decode the memory stored between `ptr` and `ptr + len` as a utf-8 string.

We can then return this to the calling JavaScript code. But before we do this, we must first free the memory where our `WasmSlice` return value was since we won't be using it anymore.

And that's it! We've successfully take a JavaScript String, converted it to utf-8, moved it onto the Wasm Heap, and passed a pointer and length to Wasm. Inside of Wasm we've converted that pointer and length to a Rust String, called our original `make_smile` function and then returned back a `WasmSlice` pointer and length. Finally on the JavaScript side we reformed a JavaScript string from the `WasmSlice` located in the Wasm heap and finally freed that `WasmSlice`.

Wow that's a lot of work!

## Conclusion

As you can see, there's quite a bit of machinery happening to generate Rust and JavaScript code for facilitating interop between JavaScript and WebAssembly. Hopefully this post gave you a basic understanding of how this code works. If you'd like to learn more about how JavaScript and WebAssembly can be made to easily talk to one another, let me know on [Twitter](https://twitter.com/itchyankles)!
