# bunkai struct

All-in-one library to parse binary structure written for [ptrlib](https://github.com/ptr-yudai/ptrlib).

## Usage

Like this:

```python
data  = b'\xde\xad\xbe\xef' # magic
data += b'fizzbuzz\0'       # name
data += b'\x01\x27'             # version
data += b'\x11\xf0' + b'\x33\xf1' # children
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
    )
)

print(my_struct.parse(data))

# --> {'magic': 3735928559, 'name': [102, 105, 122, 122, 98, 117, 122, 122], 'version': {'major': 1, 'minor': 0, 'is_beta': 1}, 'children': [{'x': 'CHILD_X1', 'y': 'CHILD_Y1'}, {'x': 'CHILD_X3', 'y': 'CHILD_Y2'}]}
```
