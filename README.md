# Comparing ways to concatenate strings in Rust 1.61 nightly (1.58 stable)
## Intro

There are many ways to turn a `&str` into a `String` in Rust and therefore many ways to concatenate two `&str`s.

Here I benchmark several different ways to concatenate the strings `"2014-11-28"`, `"T"` and `"12:00:09Z"` into `"2014-11-28T12:00:09Z"`.

Thanks to all the comments on and discussion on [reddit](https://www.reddit.com/r/rust/comments/48fmta/seven_ways_to_concatenate_strings_in_rust_the/) where I posted these originally only 7 benchmarks. Some go into the details of what is going on in the background of these operations.


## How to run?

* benchmarks: `cargo +nightly bench`
* tests: `cargo +nightly test --benches`


## Results (on my machine)

```bash
$ cargo +nightly bench

running 36 tests
test array_concat_test ... ignored
test array_join_long_test ... ignored
test array_join_test ... ignored
test collect_from_array_to_string_test ... ignored
test collect_from_vec_to_string_test ... ignored
test format_macro_implicit_args_test ... ignored
test format_macro_test ... ignored
test from_bytes_test ... ignored
test mut_string_push_str_test ... ignored
test mut_string_push_string_test ... ignored
test mut_string_with_capacity_push_str_char_test ... ignored
test mut_string_with_capacity_push_str_test ... ignored
test mut_string_with_too_little_capacity_push_str_test ... ignored
test mut_string_with_too_much_capacity_push_str_test ... ignored
test string_from_all_test ... ignored
test string_from_plus_op_test ... ignored
test to_owned_plus_op_test ... ignored
test to_string_plus_op_test ... ignored
test array_concat                                 ... bench:          30 ns/iter (+/- 2)
test array_join                                   ... bench:          22 ns/iter (+/- 0)
test array_join_long                              ... bench:          24 ns/iter (+/- 1)
test collect_from_array_to_string                 ... bench:          32 ns/iter (+/- 2)
test collect_from_vec_to_string                   ... bench:          35 ns/iter (+/- 24)
test format_macro                                 ... bench:          67 ns/iter (+/- 1)
test format_macro_implicit_args                   ... bench:          67 ns/iter (+/- 1)
test from_bytes                                   ... bench:           0 ns/iter (+/- 0)
test mut_string_push_str                          ... bench:          24 ns/iter (+/- 0)
test mut_string_push_string                       ... bench:          69 ns/iter (+/- 1)
test mut_string_with_capacity_push_str            ... bench:          10 ns/iter (+/- 0)
test mut_string_with_capacity_push_str_char       ... bench:          10 ns/iter (+/- 0)
test mut_string_with_too_little_capacity_push_str ... bench:          39 ns/iter (+/- 0)
test mut_string_with_too_much_capacity_push_str   ... bench:          11 ns/iter (+/- 0)
test string_from_all                              ... bench:          43 ns/iter (+/- 0)
test string_from_plus_op                          ... bench:          34 ns/iter (+/- 26)
test to_owned_plus_op                             ... bench:          29 ns/iter (+/- 4)
test to_string_plus_op                            ... bench:          27 ns/iter (+/- 0)

test result: ok. 0 passed; 0 failed; 18 ignored; 18 measured; 0 filtered out; finished in 31.34s
```

#### The same results rearranged fastest to slowest

```
0 ns/iter (+/- 0)       from_bytes
10 ns/iter (+/- 0)      mut_string_with_capacity_push_str
10 ns/iter (+/- 0)      mut_string_with_capacity_push_str_char
11 ns/iter (+/- 0)      mut_string_with_too_much_capacity_push_str
22 ns/iter (+/- 0)      array_join
24 ns/iter (+/- 0)      mut_string_push_str
24 ns/iter (+/- 1)      array_join_long
27 ns/iter (+/- 0)      to_string_plus_op
29 ns/iter (+/- 4)      to_owned_plus_op
30 ns/iter (+/- 2)      array_concat
32 ns/iter (+/- 2)      collect_from_array_to_string
34 ns/iter (+/- 26)     string_from_plus_op
35 ns/iter (+/- 24)     collect_from_vec_to_string
39 ns/iter (+/- 0)      mut_string_with_too_little_capacity_push_str
43 ns/iter (+/- 0)      string_from_all
67 ns/iter (+/- 1)      format_macro
67 ns/iter (+/- 1)      format_macro_implicit_args
69 ns/iter (+/- 1)      mut_string_push_string
```

## Examples explained


### `array_concat()`
```rust
let datetime = &[DATE, T, TIME].concat();
```


### `array_join()`
```rust
let datetime = &[DATE, TIME].join(T);
```


### `array_join_long()`
```rust
let datetime = &[DATE, T, TIME].join("");
```


### `collect_from_array_to_string()`
```rust
let list = [DATE, T, TIME];
let datetime: String = list.iter().map(|x| *x).collect();
```

### `collect_from_vec_to_string()`
```rust
let list = vec![DATE, T, TIME];
let datetime: String = list.iter().map(|x| *x).collect();
```

### `format_macro()`

```rust
let datetime = &format!("{}{}{}", DATE, T, TIME);
```

### `format_macro_implicit_args()`

```rust
let datetime = &format!("{DATE}{T}{TIME}");
```

### `from_bytes()` ⚠️ don't actually do this

```rust
use std::ffi::OsStr;
use std::os::unix::ffi::OsStrExt;
use std::slice;

let bytes = unsafe { slice::from_raw_parts(DATE.as_ptr(), 20) };

let datetime = OsStr::from_bytes(bytes);
```

### `mut_string_push_str()`

```rust
let mut datetime = String::new();
datetime.push_str(DATE);
datetime.push_str(T);
datetime.push_str(TIME);
```

### `mut_string_push_string()`

```rust
let mut datetime = Vec::<String>::new();
datetime.push(String::from(DATE));
datetime.push(String::from(T));
datetime.push(String::from(TIME));
let datetime = datetime.join("");
```

### `mut_string_with_capacity_push_str()`

```rust
let mut datetime = String::with_capacity(20);
datetime.push_str(DATE);
datetime.push_str(T);
datetime.push_str(TIME);
```

### `mut_string_with_capacity_push_str_char()`

```rust
let mut datetime = String::with_capacity(20);
datetime.push_str(DATE);
datetime.push('T');
datetime.push_str(TIME);
```

### `mut_string_with_too_little_capacity_push_str()`

```rust
let mut datetime = String::with_capacity(2);
datetime.push_str(DATE);
datetime.push_str(T);
datetime.push_str(TIME);
```

### `mut_string_with_too_much_capacity_push_str()`

```rust
let mut datetime = String::with_capacity(200);
datetime.push_str(DATE);
datetime.push_str(T);
datetime.push_str(TIME);
```

### `string_from_all()`

```rust
let datetime = &(String::from(DATE) + &String::from(T) + &String::from(TIME));
```

### `string_from_plus_op()`

```rust
let datetime = &(String::from(DATE) + T + TIME);
```

### `to_owned_plus_op()`

```rust
let datetime = &(DATE.to_owned() + T + TIME);
```

### `to_string_plus_op()`

```rust
let datetime = &(DATE.to_string() + T + TIME);
```

