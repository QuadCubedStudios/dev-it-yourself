# Data Types Used Across Projects
For many projects in this repository, you will be dealing with a few standard data types. You can find their descriptions here:

## Endianness
Values in multi-byte formats have two ways of being stored in memory or on disk. This is dependent on the addresses of each byte as follows (Assume N to be the address of the value):

1. Little-endian: The Least Significant Bit (LSB) is stored in the lowest order byte. This is the most common order you will find in modern systems:

    | N      | N + 1  | N + 2  | N + 3  |
    | ------ | ------ | ------ | ------ |
    | byte 0 | byte 1 | byte 2 | byte 3 |

2. Big-endian: The Most Significant Bit (MSB) is stored in the lowest order byte. This can be found in network protocols or on certain older architectures:

    | N      | N + 1  | N + 2  | N + 3  |
    | ------ | ------ | ------ | ------ |
    | byte 3 | byte 2 | byte 1 | byte 0 |

For most cases you will likely be working in Little Endian, and projects will specify if you need to work with Big Endian values.

## Unsigned Integer
| Name  | Description |
| ----- | ----------- |
| `u8`  | Unsigned 8-bit (1-byte) integer  |
| `u16` | Unsigned 16-bit (2-byte) integer |
| `u32` | Unsigned 32-bit (4-byte) integer |
| `u64` | Unsigned 64-bit (8-byte) integer |

## Signed Integer
| Name  | Description |
| ----- | ----------- |
| `i8`  | Signed 8-bit (1-byte) integer  |
| `i16` | Signed 16-bit (2-byte) integer |
| `i32` | Signed 32-bit (4-byte) integer |
| `i64` | Signed 64-bit (8-byte) integer |

## IEEE Floating Point
| Name  | Description |
| ----- | ----------- |
| `f32` | Single-precision 32-bit (4-byte) floating point |
| `f64` | Double-precision 64-bit (8-byte) floating point |
| `f80` | Extended-precision 80-bit (10-byte) floating point |

## Textual
| Name  | Description |
| ----- | ----------- |
| `char` | ASCII Character |
| `utf8` | Unicode UTF-8 string |

## Arrays
Any type can be suffixed with `[]` to specify an array of that type. A number can be specified between the brackets to signify an array size, or left empty for an unbounded array.

```c
// Bounded
u8[1024] byte_buffer;
// Unbounded
u8[] data_segment;
```

Some array types are common:
| Name  | Description |
| ----- | ----------- |
| `ascii` | ASCII String *without* null termination, equivalent to `char[]` |
| `ascii_n` | ASCII String *with* null termination, equivalent to `char[]` with a subsequent null byte |

## Composite Structures
Most types will end up composing other data types into one, these will be represented as follows:
```c
StructureName {
    type_1 field_name_1;
    type_2 field_name_2;
    // ...
}
```

Unless mentioned otherwise, the fields of a structure are stored contiguously.