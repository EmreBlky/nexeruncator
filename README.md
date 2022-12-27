<div align="center"><img src="https://user-images.githubusercontent.com/10853207/209234630-b29fbaaa-536b-4899-8eda-3a42a2d73023.png" width=300></div>

<p align="center"></p>

<div align="center">
<img src="https://img.shields.io/badge/license-GPLv3-green">
<img src="https://img.shields.io/badge/Go-v1.19-00ADD8">
</div>

# neXeruncator

nexeruncator; extracts or inserts javascript source file from or to nexe-compiled binaries.

* The word `nexeruncator` is derived from `nexe` and `aberuncate`. Nothing special!

## Notes

From a reverse engineer's point of view, the nexe-compiled application consists of 4 main parts:

* NodeJS part, which contains NodeJS runtime and big in size, ~55mb
* Initializer javascript code.
* User's javacript code.
* 32 byte footer part.

Initializer javascript part contains name of user's javascript source filename. Let's say we have a javascript file "awesome.js". That means "awesome.js" string occurs two times in initializer part.

This is how the nexe-compiled binary ends:

```txt
Offset(h) 00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F

035E3820        3C 6E 65 78 65 7E 7E 73 65 6E 74 69 6E 65    <nexe~~sentine
035E3830  6C 3E 00 00 00 00 00 FE CE 40 00 00 00 00 00 00  l>.....þÎ@......
035E3840  43 40                                            C@
```

The above sniplet is 32 bytes. The first 16 byte is the string literal `<nexe~~sentinel>`. The second 16 byte is actually comprise of 2 8 byte parts. And they are in little endian. Let's check [Nexe][gh-nexe]'s sources: `nexe/src
/
compiler.ts`

```ts
<snip>
    const code = this.code(),
        codeSize = Buffer.byteLength(code),
        lengths = Buffer.from(Array(16))

    lengths.writeDoubleLE(codeSize, 0)
    lengths.writeDoubleLE(this.bundle.size, 8)
    return new (MultiStream as any)([
        binary,
        toStream(code),
        this.bundle.toStream(),
        toStream(Buffer.concat([Buffer.from('<nexe~~sentinel>'), lengths])),
    ])
  }
}
```

The first 8 byte is `codeSize` which is what I called jsInit. It is a kind og initializer javascript code just before user's javascript code and finished with double semi-coloumns. And the second 8 byte is size of user's appended javascript source file.

## Resources

* [Github - Nexe][gh-nexe]

## License

This project is under GPLv3 license.

[gh-nexe]: https://github.com/nexe/nexe/
