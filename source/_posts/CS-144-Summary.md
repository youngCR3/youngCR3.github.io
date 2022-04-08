---
title: CS-144 Summary
date: 2022-02-11 21:36:20
tags: CS-144
categories: Computer Network
mathjax: true 
---

# 总结

## lab0

### 功能实现

- lab0实现了位于内存中的一个byte stream，其核心是一个缓冲区buffer，writer可以将字符依次写入该缓冲区，reader可依次从缓冲区读出字符
- 使用双端队列deque模拟buffer

### 注意点

- c++ private类变量需要在定义时候初始化
- read方法的实现依赖于peek_output和pop_output，需要在pop_output中修改read_ptr
- peek时需要使用assign方法将字符从deque数据结构复制到string中

## lab1

### 功能实现

- lab1实现一个stream reassembler，它将多个byte streams中的片段按正确顺序重新组合成一个连续的字节流
- 通过map有序容器，使得`StreamReassembler`能够处理由于网络传输引起的datagrams乱序、重复等问题

### 注意点

- `StreamReassembler`的容量`capacity`，需要考虑在`ByteStream`中的字节数，以及在`_buf`中的字节数
- `StreamReassembler`的eof检测，需要考虑不写入任何字节直接到达eof的特殊情况

## lab2

### 功能实现

- 实现seqno、absolute seqno和stream index三者之间的转换
- 实现`TCPReceiver`类，从TCP的另一端接收TCP segments，将数据以及文件结尾等送入`StreamReassembler`类，并**计算ackno和window size等信息**，这些信息将通过发送给另一端的TCP segments进行传输

### 注意点

- absolute seqno和stream index之间的转换只相差1，关键在于如何实现**seqno、absolute seqno之间的转换**。将一个32位数转换为64位数，可以有很多可能的值，因此需要**通过checkpoint（取离checkpoint最接近的64位数）确定唯一的值**
- `TCPReceiver`类需要
  - 接收TCP segment，**记录ISN**用于seqno和stream index之间的转换
  - 对于每条非空TCP segment，需要计算其data首个字节对应的stream index，并调用`StreamReassembler`类的`push_substring`成员
  - 计算ackno和window size
  - 对于seqno和stream index的转换，需要考虑FIN和SYN标志的影响
  - 更新checkpoint，讲义要求将checkpoint设置为TCP最后一个reassembled的字节的stream index，程序中将其设置为上一个收到的string的首字节的stream index，也能通过
  - EOF的检测：只有当`stream_out().input_ended()`为真时，才代表EOF被reassembled

<!--more-->

#  Lab0: networking warmup

## Setting up your CS144 VM

Use a [VM image that we prepared](https://stanford.edu/class/cs144/vm_howto/vm-howto-image.html) in VirtualBox

- 安装虚拟机工具，VM VirtualBox
- 导入VM文件
- 通过powershell ssh指令，使用ssh连接到VM，`ssh cs144@localhost -p 2222`



## Networking by hand

### Fetch a Web page

- 浏览器访问http://cs144.keithw.org/hello，会看到一个句子

  ```
  Hello, CS144!
  ```

- 使用SSH连接虚拟机后，键入

  ```
  $ telnet cs144.keithw.org http
  ```

  响应如下

  ```
  Trying 104.196.238.229...
  Connected to cs144.keithw.org.
  Escape character is '^]'.
  ```

- 然后分三行依次输入，并按下回车键

  ```
  GET /hello HTTP/1.1
  Host: cs144.keithw.org
  Connection: close
  ```

  响应如下

  ```
  HTTP/1.1 200 OK
  Date: Fri, 11 Feb 2022 14:01:53 GMT
  Server: Apache
  Last-Modified: Thu, 13 Dec 2018 15:45:29 GMT
  ETag: "e-57ce93446cb64"
  Accept-Ranges: bytes
  Content-Length: 14
  Connection: close
  Content-Type: text/plain
  
  Hello, CS144!
  Connection closed by foreign host.
  ```



### Send yourself an email

- qq邮箱smtp服务授权码

  - 使用python的base64模块将其转化为base64码

  ```python
  import base64
  s1 = '974227915@qq.com'
  s2 = 'onijfuqefrubbegh'
  a1 = base64.b64encode(s1.encode('utf-8'))
  a2 = base64.b64encode(s2.encode('utf-8'))
  print(a1)
  print(a2)
  ```

  - 得到

  ```
  账户: OTc0MjI3OTE1QHFxLmNvbQ==
  授权码: b25pamZ1cWVmcnViYmVnaA==
  ```

- 使用qq邮箱发送邮件

  ```shell
  telnet smtp.qq.com 25
  HELO host
  # 登录时使用上述base64编码后的账户和授权码
  AUTH LOGIN
  OTc0MjI3OTE1QHFxLmNvbQ==
  b25pamZ1cWVmcnViYmVnaA==
  # 指定收发邮箱
  MAIL FROM: <974227915@qq.com>
  RCPT TO: <22010064@zju.edus.cn>
  DATA
  # 邮件的收发邮箱和主题
  From: 974227915@qq.com
  To: 22010064@zju.edu.cn
  Subject: Hello from cs144 lab 0
  # 正文
  Hello world.
  # 输入单个.并按下回车后发送
  .
  250 OK: queued as.
  # 输入QUIT并回车后退出
  QUIT
  221 Bye.
  ```


### Listening and connecting

- 服务器端

  ```shell
  $ netcat -v -l -p 9090
  Listening on [0.0.0.0] (family 0, port 9090)
  # After client connecting
  Connection from localhost 37504 received!
  ```

- 客户端

  ```shell
  $ telnet localhost 9090
  # After client connecting
  Trying 127.0.0.1...
  Connected to localhost.
  Escape character is '^]'.
  ```

- typing in either terminal and transfer it, then it will appears in the other terminal

![](lab0_1.png)

## Writing a network program using an OS stream socket

### Modern C++ 代码规范

- 不要使用`malloc()`和`free()`，不要使用new和delete
- 不要使用裸指针，用智能指针代替
- 不要使用C字符串，使用标准库中的`std::string`
- 不要使用C风格的类型转换，使用`static_cast`
- 尽可能通过const引用方式传递函数参数
- 尽可能定义const变量
- 避免使用全局变量，使每个变量的作用域尽可能小

### Writing webget

- 实现get_URL函数，功能为**向指定IP地址发送HTTP GET请求**，并输出所有响应

```c++
void get_URL(const string &host, const string &path) {
    Address server(host, "http");
    TCPSocket sock;
    sock.connect(server);
    sock.write("GET " + path + " HTTP/1.1\r\n", false);
    sock.write("Host: " + host + "\r\n", false);
    sock.write("Connection: close\r\n\r\n", true);
    while (!sock.eof()) {
        auto recvd = sock.read();
        cout << recvd;
    }
    sock.close();
}
```

## An in-memory reliable byte stream

### 定义类变量

- `ByteStream`容量大小`capacity`
- `ByteStream`缓存`_buf`，定义为双端队列
- 读指针和写指针（负责对读写了多少数据进行计数）`read_ptr`和`write_ptr`
- 是否已经停止写的布尔变量`end`

### `eof`函数的实现

- 根据讲义，eof的定义如下，因此只有当写入已经结束且`_buf`中的数据均被读出才到达eof，即

> When the reader has read to the end of the stream, it will reach "EOF"

```c++
bool ByteStream::eof() const { 
    // When the reader has read to the end of the stream, it will reach "EOF"
    return input_ended() && buffer_empty(); 
}
```

### 使用双端队列deque模拟buffer

- 读取即从队首读出字符并pop，写入即将字符push进队尾

- read方法的实现可借助于`peek_output`和`pop_output`

  ```c++
  //! \param[in] len bytes will be copied from the output side of the buffer
  string ByteStream::peek_output(const size_t len) const {
      size_t l = min(len, buffer_size());
      string s;
      s.assign(_buf.begin(), _buf.begin() + l);
      return s;
  }
  
  //! \param[in] len bytes will be removed from the output side of the buffer
  void ByteStream::pop_output(const size_t len) {
      size_t l = min(len, buffer_size());
      for (size_t i = 0; i < l; ++i)
          _buf.pop_front();
      read_ptr += l;
  }
  
  //! Read (i.e., copy and then pop) the next "len" bytes of the stream
  //! \param[in] len bytes will be popped and returned
  //! \returns a string
  std::string ByteStream::read(const size_t len) {
      string r = peek_output(len);
      pop_output(len);
      return r;
  }
  ```

### assign函数的使用

- 读取时，在`peek_output`中需要将队首的`l`个字符读取，可通过`assign`方法实现

  ```c++
  string ByteStream::peek_output(const size_t len) const {
      size_t l = min(len, buffer_size());
      string s;
      s.assign(_buf.begin(), _buf.begin() + l);
      return s;
  }
  ```

### 总结

- c++private类变量需要在定义时候初始化

- 使用双端队列deque模拟buffer

- read方法的实现依赖于peek_output和pop_output，需要在pop_output中修改read_ptr

- peek时需要使用assign方法将字符从deque数据结构复制到string中

### 代码

- `byte_stream.cc`

```c++
#include "byte_stream.hh"

// Dummy implementation of a flow-controlled in-memory byte stream.

// For Lab 0, please replace with a real implementation that passes the
// automated checks run by `make check_lab0`.

// You will need to add private members to the class declaration in `byte_stream.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

ByteStream::ByteStream(const size_t c) {
    capacity = c;
    write_ptr = 0;
    read_ptr = 0;
    end = false;
    _buf = deque<char>{};
}

size_t ByteStream::write(const string &data) {
    size_t c = 0;
    for (auto &ch : data) {
        if (remaining_capacity() > 0) {
            _buf.push_back(ch);
            ++write_ptr;
            ++c;
        }
    }
    return c;
}

//! \param[in] len bytes will be copied from the output side of the buffer
string ByteStream::peek_output(const size_t len) const {
    size_t l = min(len, buffer_size());
    string s;
    s.assign(_buf.begin(), _buf.begin() + l);
    return s;
}

//! \param[in] len bytes will be removed from the output side of the buffer
void ByteStream::pop_output(const size_t len) {
    size_t l = min(len, buffer_size());
    for (size_t i = 0; i < l; ++i)
        _buf.pop_front();
    read_ptr += l;
}

//! Read (i.e., copy and then pop) the next "len" bytes of the stream
//! \param[in] len bytes will be popped and returned
//! \returns a string
std::string ByteStream::read(const size_t len) {
    string r = peek_output(len);
    pop_output(len);
    return r;
}

void ByteStream::end_input() { end = true; }

bool ByteStream::input_ended() const { return end; }

size_t ByteStream::buffer_size() const { return _buf.size(); }

bool ByteStream::buffer_empty() const { return _buf.size() == 0; }

bool ByteStream::eof() const {
    // When the reader has read to the end of the stream, it will reach "EOF"
    return input_ended() && buffer_empty();
}

size_t ByteStream::bytes_written() const { return write_ptr; }

size_t ByteStream::bytes_read() const { return read_ptr; }

size_t ByteStream::remaining_capacity() const { return capacity - _buf.size(); }
```

- `byte_stream.hh`

```c++
#ifndef SPONGE_LIBSPONGE_BYTE_STREAM_HH
#define SPONGE_LIBSPONGE_BYTE_STREAM_HH

#include <deque>
#include <string>
using namespace std;

//! \brief An in-order byte stream.

//! Bytes are written on the "input" side and read from the "output"
//! side.  The byte stream is finite: the writer can end the input,
//! and then no more bytes can be written.
class ByteStream {
  private:
    // Your code here -- add private members as necessary.

    // Hint: This doesn't need to be a sophisticated data structure at
    // all, but if any of your tests are taking longer than a second,
    // that's a sign that you probably want to keep exploring
    // different approaches.
    size_t capacity = 0;
    deque<char> _buf = {};
    size_t read_ptr = 0, write_ptr = 0;
    bool end = false;

    bool _error{};  //!< Flag indicating that the stream suffered an error.

  public:
    //! Construct a stream with room for `capacity` bytes.
    ByteStream(const size_t capacity);

    //! \name "Input" interface for the writer
    //!@{

    //! Write a string of bytes into the stream. Write as many
    //! as will fit, and return how many were written.
    //! \returns the number of bytes accepted into the stream
    size_t write(const std::string &data);

    //! \returns the number of additional bytes that the stream has space for
    size_t remaining_capacity() const;

    //! Signal that the byte stream has reached its ending
    void end_input();

    //! Indicate that the stream suffered an error.
    void set_error() { _error = true; }
    //!@}

    //! \name "Output" interface for the reader
    //!@{

    //! Peek at next "len" bytes of the stream
    //! \returns a string
    std::string peek_output(const size_t len) const;

    //! Remove bytes from the buffer
    void pop_output(const size_t len);

    //! Read (i.e., copy and then pop) the next "len" bytes of the stream
    //! \returns a string
    std::string read(const size_t len);

    //! \returns `true` if the stream input has ended
    bool input_ended() const;

    //! \returns `true` if the stream has suffered an error
    bool error() const { return _error; }

    //! \returns the maximum amount that can currently be read from the stream
    size_t buffer_size() const;

    //! \returns `true` if the buffer is empty
    bool buffer_empty() const;

    //! \returns `true` if the output has reached the ending
    bool eof() const;
    //!@}

    //! \name General accounting
    //!@{

    //! Total number of bytes written
    size_t bytes_written() const;

    //! Total number of bytes popped
    size_t bytes_read() const;
    //!@}
};

#endif  // SPONGE_LIBSPONGE_BYTE_STREAM_HH
```



# Lab1: Stitching substrings into a byte stream

## 任务描述

- TCP发送方将其字节流划分为多个片段发送，使得每个片段都能存储在一个datagram中，但由于不可靠的网络传输，这些datagrams可能被乱序、丢弃、重发等，因此接收方需要将这些datagrams重组，使其恢复原来的字节流
- 实现`StreamReassembler`类，它将接受多个（1）子串，以及（2）该子串滴第一个字节在整个字节流中的下标
- 字节流中的每个字节都有自己的下标，从零开始，不断递增
- `StreamReassembler`类拥有一个`ByteStream`，当它获取字节流中的下一个字节后，便会将其写入该`ByteStream`
- **能够处理datagrams乱序、重发等情况**
- `StreamReassembler`类具有容量`capacity`，当容量满了后，不再存储到达的datagrams



## 数据结构设计

- 我们需要能够处理字符串乱序、重叠等情况，因此必须对字符按其在字节流中的下标进行排序，同时防止重复存储，由此可选用**有序容器map**存储接收到的字符
- 当我们能够将某一字节写入`ByteStream`中时就立刻写入，我们按照下标递增依次写入，假设已经写入下标为$0-x$的字节，当前我们拥有下标为$x+1$的字节，那么我们就能将其写入`ByteStream`中，因此我们维护一个变量`_expected_index`代表`ByteStream`希望我们写入的下一个字节下标



## `push_substring`方法的实现

- 首先到达的字符串`data`的第一个字符在整个字节流中的下标为`index`，假如我们使用循环变量`i`遍历字符串`data`，则`data[i]`的真实下标为`i + index`

### 不要保存下标小于`_expected_index`的字节

- 因为我们已经将该下标的字节写入了`ByteStream`中，我们无需再保存该字节，直接丢弃即可

### 容量检测

- 因为题目要求`StreamReassembler`类具有容量`capacity`，容量包含的是写入`ByteStream`且未被读出的字节数，可通过`_ouput.buffer_size()`获得，以及当前存储于`StreamReassembler`未写入`ByteStream`中的字节数，为`_buf.size()`
- 将`data`中的字符写入`_buf`后，检查是否超过了容量，若超过了则将`_buf`中最后一个元素移除（erase）

### eof检测

- 根据`ByteStream`类的要求，当writer已经到达eof时，要调用该类的`ended_input()`方法告知不会再有字节写入，因此`StreamReassembler`也需要实现eof检测
- `push_substring`接收到eof标志说明我们知道了哪个下标对应的字节是最后一个字节，当**该字节被写入`ByteStream`中**时，我们可以确定此后不会再有字节写入，因此可调用`ByteStream`的`ended_input()`方法

### eof检测的一种特殊情况

- 考虑未写入任何字节就到达eof的情况，此时我们可以直接判定eof
- 因此，我们不能将eof检测代码放在将字节写入`ByteStream`的代码中
- 正确的做法时，当写入`ByteStream`的字节是最后一个字节时，我们break。并在函数的最后进行eof检测。这样，无论是写入若干个字节后到达eof，还是未写入任何字节就到达eof，我们都能够正确地调用`ByteStream`中的`ended_input()`方法并告知eof

## 曾遇到的bug

### eof检测问题

- 上述eof检测的特殊情况，一开始时把eof检测放在了将字节写入`ByteStream`的代码中，这样若不写入任何字节直接eof的情况，程序无法正确调用`ByteStream`的`ended_input()`方法

### capacity问题

- `StreamReassembler`的容量是由`ByteStream`中的字节和存在于buf且未写入`ByteStream`的字节共同组成的

## 代码

- `stream_reassembler.cc`

```c++
#include "stream_reassembler.hh"

// Dummy implementation of a stream reassembler.

// For Lab 1, please replace with a real implementation that passes the
// automated checks run by `make check_lab1`.

// You will need to add private members to the class declaration in `stream_reassembler.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

StreamReassembler::StreamReassembler(const size_t capacity) : _output(capacity), _capacity(capacity) {}

//! \details This function accepts a substring (aka a segment) of bytes,
//! possibly out-of-order, from the logical stream, and assembles any newly
//! contiguous substrings and writes them into the output stream in order.
void StreamReassembler::push_substring(const string &data, const size_t index, const bool eof) {
    while (!_buf.empty() && _buf.begin()->first == _expected_index) {
        ++_expected_index;
        string r;
        r += _buf.begin()->second;
        _buf.erase(_buf.begin());
        _output.write(r);
        if (_eof_flag && _expected_index == _eof_index) {
            _eof = true;
            _output.end_input();
        }
    }
    if (eof) {
        _eof_index = index + data.size();
        _eof_flag = true;
    }
    for (size_t i = 0; i < data.size(); ++i) {
        if (i + index >= _expected_index) {
            _buf[i + index] = data[i]; 
            if (_buf.size() + _output.buffer_size() > _capacity) {
                auto it = _buf.end();
                it--;
                _buf.erase(it);
            }
        }
    }
}

size_t StreamReassembler::unassembled_bytes() const { return _buf.size(); }

bool StreamReassembler::empty() const { return _buf.empty(); }
```

- `stream_reassembler.hh`

```c++
#ifndef SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH
#define SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH

#include "byte_stream.hh"

#include <cstdint>
#include <string>
#include <map>
using namespace std;

//! \brief A class that assembles a series of excerpts from a byte stream (possibly out of order,
//! possibly overlapping) into an in-order byte stream.
class StreamReassembler {
  private:
    // Your code here -- add private members as necessary.

    ByteStream _output;  //!< The reassembled in-order byte stream
    size_t _capacity;    //!< The maximum number of bytes
    map<size_t, char> _buf;
    size_t _expected_index = 0;
    size_t _eof_index = 0;
    bool _eof_flag = false;
    bool _eof = false;


  public:
    //! \brief Construct a `StreamReassembler` that will store up to `capacity` bytes.
    //! \note This capacity limits both the bytes that have been reassembled,
    //! and those that have not yet been reassembled.
    StreamReassembler(const size_t capacity);

    //! \brief Receive a substring and write any newly contiguous bytes into the stream.
    //!
    //! The StreamReassembler will stay within the memory limits of the `capacity`.
    //! Bytes that would exceed the capacity are silently discarded.
    //!
    //! \param data the substring
    //! \param index indicates the index (place in sequence) of the first byte in `data`
    //! \param eof the last byte of `data` will be the last byte in the entire stream
    void push_substring(const std::string &data, const uint64_t index, const bool eof);

    //! \name Access the reassembled byte stream
    //!@{
    const ByteStream &stream_out() const { return _output; }
    ByteStream &stream_out() { return _output; }
    //!@}

    //! The number of bytes in the substrings stored but not yet reassembled
    //!
    //! \note If the byte at a particular index has been pushed more than once, it
    //! should only be counted once for the purpose of this function.
    size_t unassembled_bytes() const;

    //! \brief Is the internal state empty (other than the output stream)?
    //! \returns `true` if no substrings are waiting to be assembled
    bool empty() const;
};

#endif  // SPONGE_LIBSPONGE_STREAM_REASSEMBLER_HH
```



# Lab 2: the TCP receiver

## 任务描述

- **实现`TCPReceiver`类**，它从网络接受segments，然后将它们送`StreamReassembler`类，最终写入`ByteStream`，应用程序从`ByteStream`中读出各个字节

- 此外，`TCPReceiver`类还需要告诉发送方两个信息
  - 第一个unassembled的字节的下标，也被称为$ackno$，这是接收方需要发送方传输的第一个字节
  - 第一个unassembled的字节的下标，与第一个未接收的字节的下标之差，被称为**窗口宽度window size**

- ackno和window size描述了TCP接收方的窗口，TCP发送方只能发送下标在该范围内的字节。通过该窗口，TCP接收方能够控制数据流，**ackno通常被视作窗口的最左侧，ackno + window size通常被视作窗口的最右侧**

> TCP中
>
> - acknowledgement means 为了接收并重组更多字节，接收方所需要的下一个字节的下标；
> - flow control means接收方希望且愿意接收的下标范围；

## 任务一：64位的下标与32位的seqnos之间的转换

- `StreamReassembler`中的某个字节都有一个64位的stream index，该下标从零开始，不断递增
- 在**TCP header中**，每个字节的下标是用**32位的sequence number，也称为seqno表示的**
  - 由于32位的seqno有限，而字节数很容易就超过该范围，因此**seqno必须循环使用**，当字节下标达到$2^{32}-1$之后，下一个字节下标应为0
  - 为了安全性，TCP规定**第一个字节的seqno应该是一个随机的32位数**，称为**Initial Sequence Number，ISN**。该sequence number代表了SYN（beginning of the stream）
  - TCP传输的**开头、结束也占据了一个seqno**，因此SYN（beginning of stream）、FIN（end of stream）控制标志也被分配了一个sequence number，**SYN标志拥有的seqno即上述ISN**。需要注意的是，SYN和FIN只是控制标志，它们本身并不包含内容，因此也不是字节

- 此外，还有一种编号称为**absolute sequence number**，它是一个64位数，**总是以0开始**，并且不循环使用

### 辨析stream index、absolute sequence number和seqno

- stream index是64位数，从0开始，只对字节编码（忽略SYN和FIN控制标志）
- absolute sequence number是64位数，从0开始，包含SYN和FIN控制标志（SYN的absolute sequence number 为0）
- seqno真实存在于TCP header中，是32位数，从ISN（随机的32位数）开始，包含SYN和FIN控制标志（SYN的seqno为ISN）

### 将absolute seqno转换为seqno

- `WrappingInt32 wrap(uint64_t n, WrappinInt32 isn)`

- 分析

  - 因为n是一个64位数，而isn是一个32位数，应该将isn转为64位数进行运算，得到结果后再转回32位数，防止溢出
  - 定义`MOD=2^32`，首先n对`MOD`取模，`isn`转为64位数`isn1`，然后将`n + isn1`再次对`MOD`取模，并将结果转化回32位数

- 代码

  ```c++
  WrappingInt32 wrap(uint64_t n, WrappingInt32 isn) {
      const uint64_t mod = 0x100000000;
      n %= mod;
      uint64_t isn1 = static_cast<uint64_t>(isn.raw_value());
      uint64_t r = (n + isn1) % mod;
      return WrappingInt32{static_cast<uint32_t>(r)};
  }
  ```

  

### 将seqno转换为absolute seqno

- `uint64_t unwrap(WrappingInt32 n, WrappingInt32 isn, uint64_t checkpoint)`

- 给出seqno及isn，同时还有64位数checkpoint，将seqno转化为**最接近checkpoint的64位absolute seqno**

#### checkpoint的作用

- 因为一个32位的seqno能够转化为多个64位的absolute seqno，因此需要提供checkpoint使得结果唯一
- 在TCP中，将上一个reassembled的字节的下标作为checkpoint

#### 分析

- 首先求出seqno与isn的距离$d$（无符号数）
- 然后求checkpoint对`mod=2^32`的余数`r`，则`checkpoint-r`是一个$2^{32}$的倍数，此时一个可能的值是$v_1=checkpoint-r+d$
- 还有的可能值是$v_2=v_1-mod$，$v_3=v_1+mod$，但应注意在部分情况下，$v_2$可能不存在（小于0），$v_3$也可能不存在（超过了64位数的表示范围，溢出）
- 最后分别求$v_1$、$v_2$、$v_3$与$checkpoint$之间的差值，取最小的作为absolute seqno即可

- 代码

```c++
uint64_t unwrap(WrappingInt32 n, WrappingInt32 isn, uint64_t checkpoint) {
    const uint64_t mod = 1ul << 32;
    const uint64_t maxv = 0xffffffffffffffff;
    uint64_t d;
    if (n.raw_value() >= isn.raw_value())    
        d = static_cast<uint64_t>(n.raw_value() - isn.raw_value());
    else 
        d = mod - static_cast<uint64_t>(isn.raw_value() - n.raw_value());
    uint64_t r = checkpoint % mod;
    uint64_t v1 = checkpoint - r + d, v2, v3;
    // possible val can be v1, v1 - mod, v1 + mod
    if (v1 < mod) {
        v2 = v3 = v1 + mod;
        
    } else if (maxv - v1 < mod) {
        v2 = v3 = v1 - mod;
    } else {
        v2 = v1 + mod;
        v3 = v1 - mod;
    }
    uint64_t d1, d2, d3;
    if (v1 > checkpoint)
        d1 = v1 - checkpoint;
    else
        d1 = checkpoint - v1;
    if (v2 > checkpoint)
        d2 = v2 - checkpoint;
    else
        d2 = checkpoint - v2;
    if (v3 > checkpoint)
        d3 = v3 - checkpoint;
    else
        d3 = checkpoint - v3;
    if (d1 <= d2 && d1 <= d3)
        return v1;
    else if (d2 <= d1 && d2 <= d3)
        return v2;
    else
        return v3;
}
```



## 任务二：实现TCP Receiver

### `TCP Receiver`功能

- 接收TCP另一端传来的segments
- 使用`StreamReassembler`重组`ByteStream`
- 计算ackno和window size，这两个信息最终会通过一个outgoing segment传输回TCP的另一端

### TCP segment的结构

![](TCP segment.png)

- 本实验关注的是**seqno、payload、SYN标志和FIN标志**，它们是由sender写入，由receiver读取的

### 实现`segment_received()`成员函数

#### 功能描述

- 记录ISN，假如接收到一个设置了SYN标志的TCP Segment，需要记录ISN，因为此后的seqno与stream index之间的转换需要用到ISN
- 将数据或者eof送入`StreamReassembler`，将TCP segment的payload中的数据送入`StreamReassembler`，同时若TCP segment的header设置了FIN标志，代表已到达eof，因此同样将eof送入`StreamReassembler`

#### 分析

- 对于包含SYN flag的TCP segment，如果它带有payload，则其data的首个字节的seqno，应该是TCP header中的seqno的后一个，因为TCP header中的seqno对应的时SYN flag的seqno
- 在进行seqno和stream index之间的转换时，需要考虑SYN的影响，因此计算的`index`最后要减一

#### 代码

```c++
void TCPReceiver::segment_received(const TCPSegment &seg) {
    // set the ISN if necessary
    WrappingInt32 seqno = seg.header().seqno;
    if (seg.header().syn) {
        _isn = seqno;
        seqno = seqno + 1;
    }
    // push any data, or eof to the StreamReassembler
    if (_isn) {
        uint64_t index = unwrap(seqno, _isn.value(), _checkpoint) - 1;
        _checkpoint = index;
        _reassembler.push_substring(static_cast<const std::string>(seg.payload().str()), index, seg.header().fin);
    }
}
```

### 实现`ackno()`成员函数

#### 分析

- 我们通过`stream_out().bytes_written()`可以获取当前需要的首个字节的stream index，考虑将其转换为seqno
- 考虑SYN flag，需要将其加1
- 考虑FIN flag，也需要将其加1，但这只有在FIN flag被reassembled时才需要执行，也就是说并非收到了FIN flag为真的TCP segment就执行，而是`StreamReassembler`类将其送入`ByteStream`时才执行，这可以通过`stream_out().input_ended()`进行判断

#### 代码

```c++
optional<WrappingInt32> TCPReceiver::ackno() const { 
    if (!_isn)
        return nullopt;
    // considering syn flag
    uint64_t index = stream_out().bytes_written() + 1;
    // considering fin flag
    if (stream_out().input_ended())
        ++index;
    return wrap(index, _isn.value());
 }
```

### 实现`window_size()`成员函数

#### 分析

- `window_size`代表的是当前第一个未接收的字节编号，与当前第一个未assembled的字节编号的差
- 回忆Lab1中`StreamReassembler`类的capacity的含义，它是第一个未接收的字节编号，与第一个未被`ByteStream`的reader读出的字节编号之间的差，包含了`ByteStream`存储和`StreamReassembler`存储区中的字节
- 对比两者定义，可以发现`window_size`等于`StreamReassembler`类的capacity，减去`ByteStream`存储区的字节数，因此通过`_capacity - stream_out().buffer_size()`可以直接获得`window_size`

#### 代码

```c++
size_t TCPReceiver::window_size() const { 
    return _capacity - stream_out().buffer_size();
 }
```

## 代码

- `wrapping_integers.cc`

```c++
#include "wrapping_integers.hh"

// Dummy implementation of a 32-bit wrapping integer

// For Lab 2, please replace with a real implementation that passes the
// automated checks run by `make check_lab2`.

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

//! Transform an "absolute" 64-bit sequence number (zero-indexed) into a WrappingInt32
//! \param n The input absolute 64-bit sequence number
//! \param isn The initial sequence number
WrappingInt32 wrap(uint64_t n, WrappingInt32 isn) {
    const uint64_t mod = 1ul << 32;
    n %= mod;
    uint64_t isn1 = static_cast<uint64_t>(isn.raw_value());
    uint64_t r = (n + isn1) % mod;
    return WrappingInt32{static_cast<uint32_t>(r)};
}

//! Transform a WrappingInt32 into an "absolute" 64-bit sequence number (zero-indexed)
//! \param n The relative sequence number
//! \param isn The initial sequence number
//! \param checkpoint A recent absolute 64-bit sequence number
//! \returns the 64-bit sequence number that wraps to `n` and is closest to `checkpoint`
//!
//! \note Each of the two streams of the TCP connection has its own ISN. One stream
//! runs from the local TCPSender to the remote TCPReceiver and has one ISN,
//! and the other stream runs from the remote TCPSender to the local TCPReceiver and
//! has a different ISN.
uint64_t unwrap(WrappingInt32 n, WrappingInt32 isn, uint64_t checkpoint) {
    const uint64_t mod = 1ul << 32;
    const uint64_t maxv = 0xffffffffffffffff;
    uint64_t d;
    if (n.raw_value() >= isn.raw_value())    
        d = static_cast<uint64_t>(n.raw_value() - isn.raw_value());
    else 
        d = mod - static_cast<uint64_t>(isn.raw_value() - n.raw_value());
    uint64_t r = checkpoint % mod;
    uint64_t v1 = checkpoint - r + d, v2, v3;
    // possible val can be v1, v1 - mod, v1 + mod
    if (v1 < mod) {
        v2 = v3 = v1 + mod;
        
    } else if (maxv - v1 < mod) {
        v2 = v3 = v1 - mod;
    } else {
        v2 = v1 + mod;
        v3 = v1 - mod;
    }
    uint64_t d1, d2, d3;
    if (v1 > checkpoint)
        d1 = v1 - checkpoint;
    else
        d1 = checkpoint - v1;
    if (v2 > checkpoint)
        d2 = v2 - checkpoint;
    else
        d2 = checkpoint - v2;
    if (v3 > checkpoint)
        d3 = v3 - checkpoint;
    else
        d3 = checkpoint - v3;
    if (d1 <= d2 && d1 <= d3)
        return v1;
    else if (d2 <= d1 && d2 <= d3)
        return v2;
    else
        return v3;
}
```

- `tcp_receiver.cc`

```c++
#include "tcp_receiver.hh"

// Dummy implementation of a TCP receiver

// For Lab 2, please replace with a real implementation that passes the
// automated checks run by `make check_lab2`.

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

void TCPReceiver::segment_received(const TCPSegment &seg) {
    // set the ISN if necessary
    WrappingInt32 seqno = seg.header().seqno;
    if (seg.header().syn) {
        _isn = seqno;
        seqno = seqno + 1;
    }
    // push any data, or eof to the StreamReassembler
    if (_isn) {
        uint64_t index = unwrap(seqno, _isn.value(), _checkpoint) - 1;
        _checkpoint = index;
        _reassembler.push_substring(static_cast<const std::string>(seg.payload().str()), index, seg.header().fin);
    }
}

optional<WrappingInt32> TCPReceiver::ackno() const { 
    if (!_isn)
        return nullopt;
    // considering syn flag
    uint64_t index = stream_out().bytes_written() + 1;
    // considering fin flag
    if (stream_out().input_ended())
        ++index;
    return wrap(index, _isn.value());
 }

size_t TCPReceiver::window_size() const { 
    return _capacity - stream_out().buffer_size();
 }
```

- `tcp_receiver.hh`

```c++
#ifndef SPONGE_LIBSPONGE_TCP_RECEIVER_HH
#define SPONGE_LIBSPONGE_TCP_RECEIVER_HH

#include "byte_stream.hh"
#include "stream_reassembler.hh"
#include "tcp_segment.hh"
#include "wrapping_integers.hh"

#include <optional>

//! \brief The "receiver" part of a TCP implementation.

//! Receives and reassembles segments into a ByteStream, and computes
//! the acknowledgment number and window size to advertise back to the
//! remote TCPSender.
class TCPReceiver {
    //! Our data structure for re-assembling bytes.
    StreamReassembler _reassembler;

    //! The maximum number of bytes we'll store.
    size_t _capacity;

    std::optional<WrappingInt32> _isn = nullopt;

    uint64_t _checkpoint = 0;

  public:
    //! \brief Construct a TCP receiver
    //!
    //! \param capacity the maximum number of bytes that the receiver will
    //!                 store in its buffers at any give time.
    TCPReceiver(const size_t capacity) : _reassembler(capacity), _capacity(capacity) {}

    //! \name Accessors to provide feedback to the remote TCPSender
    //!@{

    //! \brief The ackno that should be sent to the peer
    //! \returns empty if no SYN has been received
    //!
    //! This is the beginning of the receiver's window, or in other words, the sequence number
    //! of the first byte in the stream that the receiver hasn't received.
    std::optional<WrappingInt32> ackno() const;

    //! \brief The window size that should be sent to the peer
    //!
    //! Operationally: the capacity minus the number of bytes that the
    //! TCPReceiver is holding in its byte stream (those that have been
    //! reassembled, but not consumed).
    //!
    //! Formally: the difference between (a) the sequence number of
    //! the first byte that falls after the window (and will not be
    //! accepted by the receiver) and (b) the sequence number of the
    //! beginning of the window (the ackno).
    size_t window_size() const;
    //!@}

    //! \brief number of bytes stored but not yet reassembled
    size_t unassembled_bytes() const { return _reassembler.unassembled_bytes(); }

    //! \brief handle an inbound segment
    void segment_received(const TCPSegment &seg);

    //! \name "Output" interface for the reader
    //!@{
    ByteStream &stream_out() { return _reassembler.stream_out(); }
    const ByteStream &stream_out() const { return _reassembler.stream_out(); }
    //!@}
};

#endif  // SPONGE_LIBSPONGE_TCP_RECEIVER_HH
```

