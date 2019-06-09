---
layout: post
title:  "Explore Flatbuffer Data Layout"
date:   2019-06-09 +0800
categories: coding programming protocol rpc flatbuffer
---

I write a simple demo to figure out the data layout of flatbuffer as the [official doc](https://google.github.io/flatbuffers/flatbuffers_guide_tutorial.html) do not make enough sense to me.

Image we have some boxes to store things. Each box have a ‘name’ to identify the owner, a ‘weight’ and a ‘goods’ fields to track all the things in it. Each good have a ‘category’ associate with it. Then we can write the following FlatBuffer scheme to represent the box and goods.

```flatbuffer
FlatBuffer Scheme
namespace glove.flatbuffer.example;

enum Category:byte { Clothes = 0, Instruments = 1, Foods }

struct Good {
  category:Category = Foods;
}

table Box {
  name:string;
  weight:int;
  goods:[Good];
}

root_type Box;
```

Now, a person named ‘wzy’ puts his ‘honey’ and ‘clothes’ into a box, using the python way.

```python
import flatbuffers
from glove.flatbuffer.example import Category, Good, Box


builder = flatbuffers.Builder(9527)

Box.BoxStartGoodsVector(builder, 2)
honey = Good.CreateGood(builder, Category.Category.Foods)
shirt = Good.CreateGood(builder, Category.Category.Clothes)
goods = builder.EndVector(2)

name = builder.CreateString("wzy")

Box.BoxStart(builder)
Box.BoxAddName(builder, name)
Box.BoxAddWeight(builder, 80)
Box.BoxAddGoods(builder, goods)
box = Box.BoxEnd(builder)

builder.Finish(box)
pickeled = builder.Output()

fout = open("example.bin", "wb")
fout.write(pickeled)
fout.close()
```

According to the code, Wzy’s actions are recorded into a binary file -- example.bin. This binary file can be converted into json format easily by FlatBuffer’s compiler.

Shell command to convert a binary format data to json

```shell
flatc --raw-binary -t glove.fbs -- example.bin
```

Here is the converted json format.

```json
{
  name: "wzy",
  weight: 80,
  goods: [
    {
      category: "Clothes"
    },
    {
      category: "Foods"
    }
  ]
}
```

Note that the key parts are not quoted in key-value pairs of converted json, so it's not standard json format.

Let’s analyze the binary data layout.

Hexdump of data in binary format

```hex
00000000  10 00 00 00 00 00 0a 00  10 00 0c 00 08 00 04 00
00000010  0a 00 00 00 14 00 00 00  50 00 00 00 04 00 00 00
00000020  03 00 00 00 77 7a 79 00  02 00 00 00 00 02 00 00
```

The beginning is a dword offset to root table, which located at 0x10.

The first dword of root table is an offset to vtable. To get the address of vtable, we subtract the address of root table with the offset of vtable and get 0x06 (0x10 - 0x0a = 0x06).

For vtable located at 0x06, the first word is the size of vtable, which is 0x0a, the next word is the size of data stored inline in box table, and the following 3 words are offset to the ‘name’, ‘weight’ and ‘goods’ fields in box.
To get ‘name’ field, we add address of root table with its offset and get 0x1c (0x10 + 0x0c = 0x1c). Now we arrive at 0x1c, and the data located at 0x1c is a dword offset 0x04 to the string as its a string/vector is stored outside of the table. Then, we can find the string by move 0x04 bytes from where we are, which lead us to address 0x20. The first dword is the length of the string, which is 0x03. The following 4 bytes is the content of the string plus a ‘\0’ terminator.

To get ‘weight’ filed, we add 0x10 with 0x08. As the weight field is a scalar, it is stored inline in the root table, the weight value is 0x80 as address 0x18.

To get the ‘goods’ vector, we add 0x10 with 0x04, the value stored there is an offset to the actual ‘goods’ vector. To find the ‘goods’ vector, we add the address of the offset with the value stored there (0x14 + 0x14 = 0x28) and arrived at address 0x28. The first dword is the size of the vector, which is 0x02, and the following 2 bytes are the enum value stored, which are 0x00 and 0x02.

Other parts in the binary file are paddings.

Maybe you have a hard time reading the text blob above, so I made a table to explain the layout.

|                                definition | address | content     |                    |
|------------------------------------------:|:-------:|-------------|--------------------|
|                      offset to root table |   0x00  | 10 00 00 00 | 0x10               |
|                                   padding |   0x04  | 00 00       |                    |
|                                **vtable** |         |             |                    |
|                            size of vtable |   0x06  | 0a 00       | 10                 |
|           inline data size of ‘box’ table |   0x08  | 10 00       | 16                 |
|   offset of ‘name’ start from ‘box’ table |   0x0a  | 0c 00       | 0x10 + 0x0c = 0x1c |
| offset of ‘weight’ start from ‘box’ table |   0x0c  | 08 00       | 0x10 + 0x08 = 0x18 |
|  offset of ‘goods’ start from ‘box’ table |   0x0e  | 04 00       | 0x10 + 0x04 = 0x14 |
|                **box table / root table** |         |             |                    |
|                          offset of vtable |   0x10  | 0a 00 00 00 | 0x10 - 0x0a = 0x06 |
|         offset of ‘goods’ start from here |   0x14  | 14 00 00 00 | 0x14 + 0x14 = 0x28 |
|                   inline data of ‘weight’ |   0x18  | 50 00 00 00 | 80                 |
|          offset of ‘name’ start from here |   0x1c  | 04 00 00 00 | 0x1c + 0x04 = 0x20 |
|                         **‘name’ string** |         |             |                    |
|                                    length |   0x20  | 03 00 00 00 |                    |
|                                   content |   0x24  | 77 7a 79 00 | “wzy\0”            |
|                        **‘goods’ vector** |         |             |                    |
|                                    length |   0x28  | 02 00 00 00 |                    |
|                        enum value Clothes |   0x2b  | 00          |                    |
|                          enum value Foods |   0x2e  | 02          |                    |
|                                   padding |   0x2e  | 00 00       |                    |
