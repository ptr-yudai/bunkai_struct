# bunkai struct

All-in-one library to parse binary structure written for [ptrlib](https://github.com/ptr-yudai/ptrlib).

## Usage

Like this:

```python
data  = b'\xde\xad\xbe\xef' # magic
data += b'fizzbuzz\0'       # name
data += b'\x72\x01'         # version
data += b'\x11\xf0' + b'\x33\xf1' # children
data += b'\xff\xff\xff\xff\xff\xff\xff\xff' + b'\x01' # v1_data
my_struct = 'MyStruct' <= Struct(
    'magic' <= u32be,
    'name' <= VariableArray(lambda c,_: c!=0, s8),
    'version' <= BitStruct(
        'major' <= BitInt(4),
        'minor' <= BitInt(4),
        'is_beta' <= BitInt(1),
    ),
    'children' <= Array(
        2,
        Struct(
            'x' <= Enum(u8, CHILD_X1=0x11, CHILD_X2=0x22, CHILD_X3=0x33),
            'y' <= Enum(u8, ['CHILD_Y1', 'CHILD_Y2'], startsfrom=0xf0),
        )
    ),
    'data' <= Union(
        'v1_data' <= Struct(
            'value' <= s32,
            'flag'  <= u8,
        ),
        'v2_data' <= Struct(
            'value' <= s64,
            'flag'  <= u8,
        ),
    )
)

print(my_struct.parse(data))
```

The example code above will output the following data:

```
{
  'magic': 3735928559,
  'name': [102, 105, 122, 122, 98, 117, 122, 122],
  'version': {'major': 2, 'minor': 7, 'is_beta': 1},
  'children': [
    {'x': 'CHILD_X1', 'y': 'CHILD_Y1'},
    {'x': 'CHILD_X3', 'y': 'CHILD_Y2'}
  ],
  'data': {
    'v1_data': {'value': -1, 'flag': 255},
    'v2_data': {'value': -1, 'flag': 1}
  }
}
```
