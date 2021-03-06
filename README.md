lz4
====

This repository contains binding for lz4 compression library (https://github.com/Cyan4973/lz4).

LZ4 is a very fast lossless compression algorithm, providing compression speed at 400 MB/s per core, with near-linear scalability for multi-threaded applications. It also features an extremely fast decoder, with speed in multiple GB/s per core, typically reaching RAM speed limits on multi-core systems.

[![Build Status](https://travis-ci.org/bozaro/lz4-rs.svg?branch=master)](https://travis-ci.org/bozaro/lz4-rs)
[![Crates.io](https://img.shields.io/crates/v/lz4.svg)](https://crates.io/crates/lz4)

Rustdoc: https://bozaro.github.io/lz4-rs/lz4/

## Usage

Put this in your `Cargo.toml`:

```toml
[dependencies]
lz4 = "*"
```

And this in your crate root:

```rust
extern crate lz4;
```

Sample code for compression/decompression:
```rust
extern crate lz4;

use std::iter::FromIterator;
use std::env;
use std::fs::File;
use std::io::Result;
use std::io::Read;
use std::io::Write;
use std::path::Path;

fn main()
{
	println!("LZ4 version: {}", lz4::version());
	let suffix = ".lz4";
	for arg in Vec::from_iter(env::args())[1..].iter()
	{
		if arg.ends_with(suffix)
		{
			decompress(&Path::new(arg), &Path::new(&arg[0..arg.len()-suffix.len()])).unwrap();
		}
		else
		{
			compress(&Path::new(arg), &Path::new(&(arg.to_string() + suffix))).unwrap();
		}
	}
}

fn compress(src: &Path, dst: &Path) -> Result<()>
{
	println!("Compressing: {:?} -> {:?}", src, dst);
	let mut fi = try!(File::open(src));
	let mut fo = try!(lz4::EncoderParams::new().build(try!(File::create(dst))));
	try!(copy(&mut fi, &mut fo));
	match fo.finish() {
		(_, result) => result
	}
}

fn decompress(src: &Path, dst: &Path) -> Result<()>
{
	println!("Decompressing: {:?} -> {:?}", src, dst);
	let mut fi = try!(lz4::Decoder::new(try!(File::open(src))));
	let mut fo = try!(File::create(dst));
	copy(&mut fi, &mut fo)
}

fn copy(src: &mut Read, dst: &mut Write) -> Result<()>
{
	let mut buffer: [u8; 1024] = [0; 1024];
	loop
	{
		let len = try! (src.read(&mut buffer));
		if len == 0
		{
			break;
		}
		try!(dst.write_all(&buffer[0..len]));
	}
	Ok(())
}
```
