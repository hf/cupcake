Cupcake
=======

Cupcake is a simple binary message format, and an implementation in Rust. 

## Motivation

There are a lot of ways to serialize a message. You can concatenate bytes. You
can use JSON (no!). You can use protocol buffers; or the latest fad flat
buffers. You can use MessagePack, BSON, CBOR... The list goes on...

However, all of these formats have their uses, mostly in business-level code.
They become difficult to use as a container for more generic messages. They
usually have some form of "parsing" or non-trivial computational overhead in
accessing parts of the message. Not all are good with binary data (JSON!).

Cupcake aims to solve these issues and be a container format of sorts. You
should put your JSON in a Cupcake, but a Cupcake is not a JSON!

## Spec

Yes. In a README. That simple.

### Version 1

#### Definitions

 - byte: 8 bits, smallest addressable value
 - magic: cupcake magic bytes, `0xF9 0xC9`
 - version: cupcake version, a single byte, fixed value `0x01`
 - tag: user defined tag value, a single byte
 - slice: a string of bytes, up to 255 bytes long
 - slices: a group of slices, up to 255 slices per message
 - slice-size: a single byte, describing the size of a slice
 - extension: a string of bytes, up to 4294967296 (2^32) bytes long
 - extension-size: a string of 4 bytes, describing the size of the extension

#### Layout

A Cupcake message is laid out like so:

```
0:                            [0xF9] 
1:                            [0xC9] 
2:                            [0x01] 
3:                            [tag] 
4:                            [slices] 
5:                            [extension-size-3] 
6:                            [extension-size-2]
7:                            [extension-size-1]
8:                            [extension-size-0]
9:                            { [slice-size:0] ... [slice-size:255] }
9 + slices:                   { slice:0 } ... { slice:255 }
9 + slices + sum(slice-size): { extension }
```

Where `[name]` corresponds to a single byte; `{ name }` corresponds to a
sequence of bytes.

#### Validation

A Cupcake message (`msg` of length `n`) is valid if and only if: 

 1. `n >= 9`
 2. `0xF9 == msg[0]`
 3. `0xC9 == msg[1]`
 4. `0x01 == msg[2]`
 5. `let total_slice_size = sum(msg[9 + i]) where i in (0 upto msg[4])`
 6. `let extension_size = (msg[5] << (8*3)) | (msg[6] << (8*2)) | (msg[7] << (8*1)) | (msg[8] << (8*0))`
 7. `n == 9 + total_slice_size + extension_size`

#### Access

A Cupcake message (`msg` of length `n`) can be accessed only if valid, with the
following formulas:

**Access the i-th slice**

Assumes that `i < slices`.

```
slice_i_offset := 9 + msg[4] + (sum(msg[9 + j]) where j in (0 upto i))
slice_i_length := msg[9 + i]
```

**Access the extension data**

```
extension_offset := 9 + msg[4] + (sum(msg[9 + i]) where i in (0 upto msg[4]))
extension_length := (msg[5] << (8*3)) | (msg[6] << (8*2)) | (msg[7] << (8*1)) | (msg[8] << (8*0))
```

## License

This distribution is Copyright &copy; 2017 Stojan Dimitrovski. It is licensed
under the MIT license. You can find the full text under `LICENSE`.


