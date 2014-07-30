﻿#io包

----------

##简介

##概览

#变量
```go
var EOF = errors.New("EOF")
```
EOF是当没有数据输入时返回的读入错误。函数应该通过返回EOF来给出一个信号，优雅地指示输入结束。如果EOF意外的发生在一个有结构的数据流中，会产生ErrUnexpectedEOF或者其他能给出更加详细信息的错误，这些错误都是相似的。

```go
var ErrClosedPipe = errors.New("io: read/write on closed pipe")
```
ErrClosedPipe是用来指明操作一个已经关闭的管道时产生的错误。
```go
var ErrNoProgress = errors.New("multiple Read calls return no data or error")
```
当多次调用Read，没有数据返回或者读取错误，多个io.Reader的客户端会返回ErrNoProgress。ErrNoProgress经常作为一个损坏的io.Reader的标记。
```go
var ErrShortBuffer = errors.New("short buffer")
```
ErrShortBuffer说明读取需要的buffer比提供的更长。
```go
var ErrShortWrite = errors.New("short write")
```
ErrShortWrite意味着一次写入得到的bytes比需要的少，但是无法返回一个显示的错误。
```go
var ErrUnexpectedEOF = errors.New("unexpected EOF")
```
ErrUnexpectedEOF意味着在读取一个固定大小的块或者数据结构时遇到了EOF。

###func Copy
```go
func Copy(dst Writer, src Reader) (written int64, err error)
```
从`src`到`dst`复制数据，直到遇到EOF或者产生错误为止。返回已经复制的数据和在复制过程中遇到的第一个错误，如果错误存在的话。

一次成功的复制返回err==nil，而不是err==EOF。因为复制定义为从`src`读取直到EOF，它不会把EOF当作错误对待。

如果`src`实现了WriterTo接口，复制就通过调用src.WriteTo(dst)实现。否则，如果`dst`实现了ReaderFrom接口，复制通过调用dst.ReadFrom(src)实现。

>可能的异常： io.ErrShortWrite:写入数据不等于读取数据。

###func CopyN
```go
func CopyN(dst Writer, src Reader, n int64) (written int64, err error)
```
从`src`向`dst`拷贝`n`字节数据直到遇到错误。返回已经考不的字节数和拷贝过程中最近的一次错误。返回时只有err==nil，写入的字节数才为`n`。

如果`dst`实现了ReaderFrom接口，拷贝就通过它实现。

###func ReadAtLeast
```go
func ReadAtLeast(r Reader, buf []byte, min int) (n int, err error)
```
从`r`向`buf`读取数据直到已经读取了至少`min`字节的数据。返回已经拷贝的数据的字节数和错误（如果读取的字节数不足）。只有没有任何数据读取时error才会是EOF。如果EOF在读取数据少于`min`字节时发生，则返回ErrUnexpectedEOF。如果min比buf的长，返回ErrShortBuffer错误。只有err==nil，在返回时n>=min才成立。

###func ReadFull
```go
func ReadFull(r Reader, buf []byte) (n int, err error)
```
从`r`到`buf`精确地读取len(buf)字节的数据。返回已经拷贝的字节数和错误（如果读取的字节不足）。只有没有数据读取时才会返回EOF。如果在读取部分数据时发生了EOF错误，则返回ErrUnexpectedEOF。在返回时，只有err==nil，n==len(buf)才成立。

###func WriteString
```go
func WriteString(w Writer, s string) (n int, err error)
```
将字符串`s`的数据写入`w`，`w`接收一个字节数组。如果`w`已经实现了WriteString方法，将会直接被调用。

###type ByteReader interface
```go
type ByteReader interface {
    ReadByte() (c byte, err error)
}
```
ByteReader是封装了ReadByte方法的接口。ReadByte 从输入读取并返回下一个字节。如果没有可用的字节，将产生错误`err`。

###type ByteScanner interface
```go
type ByteScanner interface {
    ByteReader
    UnreadByte() error
}
```
ByteScanner是一个接口，它将UnreadByte 方法和基本的ReadByte组合。UnreadByte产生下一次的ReadByte调用，来返回和上一次ReadByte调用相同数目的字节。连续调用两次UnreadByte，中间如果没有调用ReadByte，将会产生错误。

###type ByteWriter interface
```go
type ByteWriter interface {
    WriteByte(c byte) error
}
```
ByteWriter是封装了WriteByte方法的接口。

###type Closer interface
```go
type Closer interface {
    Close() error
}
```
Closer是接口，封装了基本的Close方法。在第一次调用Close后的行为是为定义的。在具体的实现中可能会记录自己的行为。

###type LimitedReader struct
```go
type LimitedReader struct {
    R Reader // underlying reader
    N int64  // max bytes remaining
}
```
LimitedReader 从R读取数据担心限制了返回的数据为N字节。每次调用读取都会更新N来反映新的剩余的数量。

###func (*LimitedReader) Read
```go
func (l *LimitedReader) Read(p []byte) (n int, err error)
```

###type PipeReader struct
```go
type PipeReader struct {
    // contains filtered or unexported fields
}
```
PipeReader是一个只读的单工管道。

###func Pipe
```go
func Pipe() (*PipeReader, *PipeWriter)
```
创建了在内存的一个同步的管道。它可以用来连接io.Reader和io.Writer。一端的读取和另一端的写入匹配，在两者之间直接复制数据，没有内部的缓冲。互相调用读取和写入（带有Close也可以）是安全的。Close会在即将完成的I/O完成后结束。并行地调用读取，并行地调用写入都是安全的：独立的调用都是顺序封闭的。

###func (*PipeReader) Close
```go
func (r *PipeReader) Close() error
```
关闭reader，随后向单工写入的管道写入数据将会导致ErrClosedPipe错误。

###func (*PipeReader) CloseWithError
```go
func (r *PipeReader) CloseWithError(err error) error
```
关闭reader，随后向单工写入的管道写入数据将会返回err错误。

###func (*PipeReader) Read
```go
func (r *PipeReader) Read(data []byte) (n int, err error)
```
实现了标准Read接口。从管道读取数据，直到一次写入到达或者写入端被关闭才会产生阻塞。如果写入端被关闭时产生了`err`错误，错误将会返回，否则错误是EOF。

###type PipeWriter struct
```go
type PipeWriter struct {
    // contains filtered or unexported fields
}
```

###func (*PipeWriter) Close
```go
func (w *PipeWriter) Close() error
```

###func (*PipeWriter) CloseWithError
```go
func (w *PipeWriter) CloseWithError(err error) error
```

###func (*PipeWriter) Write
```go
func (w *PipeWriter) Write(data []byte) (n int, err error)
```

###type ReadCloser interface
```go
type ReadCloser interface {
    Reader
    Closer
}
```

###type ReadSeeker interface
```go
type ReadSeeker interface {
    Reader
    Seeker
}
```

###type ReadWriteCloser interface
```go
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

###type ReadWriteSeeker interface
```go
type ReadWriteSeeker interface {
    Reader
    Writer
    Seeker
}
```

###type ReadWriter interface
```go
type ReadWriter interface {
    Reader
    Writer
}
```

###type Reader interface
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

###func LimitReader
```go
func LimitReader(r Reader, n int64) Reader
```

###func MultiReader
```go
func MultiReader(readers ...Reader) Reader
```

###func TeeReader
```go
func TeeReader(r Reader, w Writer) Reader
```

###type ReaderAt interface
```go
type ReaderAt interface {
    ReadAt(p []byte, off int64) (n int, err error)
}
```

###type ReaderFrom interface
```go
type ReaderFrom interface {
    ReadFrom(r Reader) (n int64, err error)
}
```

###type RuneReader interface
```go
type RuneReader interface {
    ReadRune() (r rune, size int, err error)
}
```

###type RuneScanner interface
```go
type RuneScanner interface {
    RuneReader
    UnreadRune() error
}
```

###type SectionReader struct
```go
type SectionReader struct {
    // contains filtered or unexported fields
}
```

###func NewSectionReader
```go
func NewSectionReader(r ReaderAt, off int64, n int64) *SectionReader
```

###func (*SectionReader) Read
```go
func (s *SectionReader) Read(p []byte) (n int, err error)
```

###func (*SectionReader) ReadAt
```go
func (s *SectionReader) ReadAt(p []byte, off int64) (n int, err error)
```

###func (*SectionReader) Seek
```go
func (s *SectionReader) Seek(offset int64, whence int) (int64, error)
```

###func (*SectionReader) Size
```go
func (s *SectionReader) Size() int64
```

###type Seeker interface
```go
type Seeker interface {
    Seek(offset int64, whence int) (int64, error)
}
```

###type WriteCloser interface
```go
type WriteCloser interface {
    Writer
    Closer
}
```

###type WriteSeeker interface
```go
type WriteSeeker interface {
    Writer
    Seeker
}
```

###type Writer interface
```go
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

###func MultiWriter
```go
func MultiWriter(writers ...Writer) Writer
```

###type WriterAt interface
```go
type WriterAt interface {
    WriteAt(p []byte, off int64) (n int, err error)
}
```

###type WriterTo interface
```go
type WriterTo interface {
    WriteTo(w Writer) (n int64, err error)
}
```
