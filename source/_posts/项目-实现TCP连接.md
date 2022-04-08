---
title: 项目-实现TCP连接
date: 2022-03-01 14:03:58
tags: 
    - Computer Network
    - 面试
    - 项目
mathjax: true
categories: 项目
---

**CS-144项目要点总结**

<!--more-->

# TCP连接

## `ByteStream`类

### 功能设计

- **固定大小的缓存区**：本质上是一个具有固定容量的缓存区，读进程可以从中读各个字节，写进程可以往里写入各个字节。当缓存区空时读进程阻塞，缓存区满时写进程阻塞
- **读者与写者**
  - 对**TCP接收方**来说，读进程即上层的应用程序，写进程负责将TCP发送方传来的TCP报文段的有效载荷放入缓存区
  - 对**TCP发送方**来说，写进程即上层的应用程序，读进程即TCP sender，负责将字节读出放入TCP报文段，并发送出去

### 成员设计

- `ByteStream`类的数据成员分析
  - `capacity`指示缓存区容量
  - `buf`存储当前缓存区的字节，可用队列实现
  
- `ByteStream`类的成员函数分析

  - `write(const string&)`，将某个字符串的每个字符写入缓存区
  - `read(size_t)`，从缓存区读出若干个字节
  - `peek_output(size_t)`，返回缓存区最开始的若干个字节
  - `pop_output(size_t)`，将缓存区的最开始的若干个字节弹出（应该先调用`peek_output`读出再调用`pop_output`弹出）
  - `remaining_capacity()`，当前缓存区的剩余容量
  - `buffer_size()`，当前缓存区存储的字节数
  - `buffer_empty()`，当前缓存区是否为空
  - `input_ended()`，读进程调用该方法获取字节流是否已经到达文件结尾
  - `end_input()`，写进程调用该方法告知读进程已经没有更多的字节写入

  - `bytes_read()`，已经读出的字节数
  - `bytes_write()`，已经写入的字节数

### 关键问题分析

- **读数据**时缓存区为**空**，或**写数据**时缓存区**已满**，应该怎么做？

  - 读数据时缓存区为空，则终止此次读操作，只返回已读的字节
  - 写数据时缓存区已满，则终止此次写操作，丢弃剩下的字节，并返回已写入的字节数

- 如何判定EOF？

  - 这个EOF可以认为是**为应用层设置的标志**，因此只有当应用层读到所有数据时，才判定为EOF，因此需要**同时考虑读进程和写进程**

  - 当写进程写入最后一个字节时，它会调用`end_input()`告知已经到达文件结尾
  - 我们还需考虑读进程，当读进程在写进程到达文件结尾后，它还需要将缓存区的所有字节读出，此时才认为到达了EOF

### 数据结构与算法设计

- 考虑按序读写，读进程总是从读出最早写入的字节，写进程总是在已写入的字节后面写入下一个字节，这是一个FIFO的模型，因此可用**队列**实现
- 特别地，由于接口设计要求`peek_output(size_t)`要求拷贝缓存区最前面的若干个字节，而`std::queue`的迭代器不支持算术运算，因此使用`std::deque`实现，它能够在常数时间内完成**队首、队尾的插入和删除**操作，同时它的**迭代器支持算术运算**

### 类定义

```c++
class ByteStream {
  private:
    // Your code here -- add private members as necessary.

    // Hint: This doesn't need to be a sophisticated data structure at
    // all, but if any of your tests are taking longer than a second,
    // that's a sign that you probably want to keep exploring
    // different approaches.
    size_t capacity = 0;
    deque<char> buf;
    size_t read_cnt = 0, write_cnt = 0;
    bool end = false;
    bool _error{};  //!< Flag indicating that the stream suffered an error.
  public:
    // omitted
};
```

### 成员实现

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

### 总结

- 使用双端队列`std::deque`模拟buffer

- `read`方法的实现依赖于`peek_output`和`pop_output`

- `peek`时需要使用`assign`方法将字符从`std::deque`拷贝到`std::string`中



## `StreamReassembler`类

### 功能设计

- 提取TCP报文段的有效载荷，将其重组为有序的字节流
- TCP发送方将字节流放入多个TCP报文段中，最终封装进IP数据报中传输，但由于网络层提供不可靠服务，因此IP数据报可能乱序、丢失或重复，为此TCP接收方必须将报文段重组为有序的连续字节流
- `StreamReassembler`类接收一串字节，首字节的序号，并在合适的时候将字节送入一个`ByteStream`对象（因此`StreamReassembler`作为该`ByteStream`的写进程，上层的应用程序作为该`ByteStream`的读进程）

### 成员设计

- 数据成员

  - `capacity`表示`StreamReassembler`缓存区的容量

  - `output`是所拥有的`ByteStream`对象
  - `expected_index`表示下一个要写入`ByteStream`对象的字节的序号
  - `eof_index`标记文件结尾对应的序号，当`expected_index`达到`eof_index`时，调用`ByteStream`的`end_input()`告知已经到达了文件结尾

- 成员函数

  - `push_string(const string&, const uint64, const bool)`，将字符串写入`ByteStream`对象，**序号**为字符串**第一个字节**在字节流中的序号，EOF标志为真代表到达**文件结尾**
  - `stream_out()`返回`StreamReassembler`拥有的`ByteStream`对象，该接口用于更上层的数据结构访问底层的`ByteStream`
  - `unassembled_bytes()`返回`StreamReassembler`接收到但未送入`ByteStream`的字节数，实际上就是`StreamReassembler`类缓存区的字节数
  - `empty()`返回缓存区是否为空

### 关键问题分析

- `StreamReassembler`类的容量`capacity`的含义是什么？

  - 此处我们将`ByteStream`类的缓存区也视为`StreamReassembler`缓存区的容量，因为`StreamReassembler` “has-A” `ByteStream`，所以计算当前缓存区大小时需要同时考虑`ByteStream`和`StreamReassembler`对象的缓存区

  - 故剩余容量$rc$为
    $$
    rc=c-ByteStreamBufferSize-StreamReassemblerBufferSize
    $$

  - `remain_capacity = capaicity-ByteStream.bufSize-StreamReassembler.bufSize`

  - 超出该容量的字节将被直接丢弃。特别地，这里采取的措施是根据当前被上层应用程序读取的最后一个字节的序号`lastRead`计算出我们能**存储的最大字节序号**`lastRead+capacity`，而对于超出该序号的字节我们将直接**丢弃（即使此时缓存区未满）**

  - 上一点所描述的策略，其实是由于TCP的**流量控制**策略决定的，因为它要求在双方传输TCP报文段时，接收方在**接收窗口字段**提供窗口大小，而该窗口大小是由接收方的缓存区减去当前已写入而未读取的字节数求得的，发送方将根据该窗口大小`rwnd`调整自己的发送速率，发送方只会允许最多`rwnd`个已发送而未确认的字节。当接收窗口为零时，发送方发出去的数据就无法被接收（实际上，为了维持正常通信，发送方仍然会发送带有一个字节数据的TCP报文段，其目的主要是为了获取接收方的接收窗口信息更新，这里涉及TCP连接的机制）

- 如何处理收到的字符串**重复、乱序**等的问题？

  - `StreamReassembler`类在一个**有序结构**里面存储字节，如果使用`std::map`则解决了重复和乱序的问题
  - 对于字节**丢失**的问题，由于TCP提供**可靠数据传输**，超时重传机制保证了数据最终交付，因此`StreamReassembler`认为所有字节都会到达

### 数据结构与算法设计

- `StreamReassembler`类采用`std::map`，分析可见上述关键问题分析第二点

### 类定义

```c++
//! \brief A class that assembles a series of excerpts from a byte stream (possibly out of order,
//! possibly overlapping) into an in-order byte stream.
class StreamReassembler {
  private:
    // Your code here -- add private members as necessary.

    ByteStream _output;  //!< The reassembled in-order byte stream
    size_t _capacity;    //!< The maximum number of bytes
    map<size_t, char> _buf;
    size_t _expected_index;
    size_t _eof_index;
    bool _eof_flag;
  public:
    // ...
};
```

### 成员实现

```c++
#include "stream_reassembler.hh"

// Dummy implementation of a stream reassembler.

// For Lab 1, please replace with a real implementation that passes the
// automated checks run by `make check_lab1`.

// You will need to add private members to the class declaration in `stream_reassembler.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

StreamReassembler::StreamReassembler(const size_t capacity) : _output(capacity), _capacity(capacity), _buf({}), _expected_index(0), _eof_index(0), _eof_flag(false) {}

//! \details This function accepts a substring (aka a segment) of bytes,
//! possibly out-of-order, from the logical stream, and assembles any newly
//! contiguous substrings and writes them into the output stream in order.
void StreamReassembler::push_substring(const string &data, const size_t index, const bool eof) {
    if (eof) {
        _eof_index = index + data.size();
        _eof_flag = true;
    }
    // push bytes into StreamReassembler buffer
    for (size_t i = 0; i < data.size(); ++i) {
        if (i + index >= _expected_index && i + index < _expected_index + _capacity - _output.buffer_size()) {
            if (_buf.count(index + i) == 0) {
                _buf[index + i] = data[i];
            }

        }        
    }
    // push bytes into ByteStream buffer, if the expected byte exists
    string s;
    while (!_buf.empty() && _buf.begin()->first == _expected_index) {
        s += _buf.begin()->second;
        ++_expected_index;
        _buf.erase(_buf.begin());
        if (_eof_flag && _expected_index == _eof_index)
            break;
    }
    _output.write(s);
    if (_eof_flag && _eof_index == _expected_index) {
        _output.end_input();
    }
}

size_t StreamReassembler::unassembled_bytes() const { return _buf.size(); }

bool StreamReassembler::empty() const { return _buf.empty(); }
```

### 总结

- `StreamReassembler`存储的字节序号$i$必须满足
  $$
  expectedIndex \leq i \lt expectedIndex + capacity - ByteStreamBufferSize
  $$

- EOF检测问题

  - 当`push_string`的`eof`标志位置位时，我们必须保存对应的`eof_index`，其含义是当`expected_index==eof_index`时到达了文件结尾
  - 假如我们向`ByteStream`写入一个字节后`expected_index`到达了`eof_index`，代表我们已到达了文件结尾，不会再写入任何其他字节，此时调用`ByteStream`对象的`end_input()`方法告知文件结尾
  - 特别地，如果我们不写入任何字节就到达了文件结尾，则我们的`eof_index`是0，与`expected_index`初始值是0。这意味着，我们不能把EOF检测的代码段放在将字节写入缓存区的循环中，否则上述情况将无法检测到EOF（没有任何字节要写入缓存区）
  - 同样地，由于上述问题，`eof_index`的初始值难以设定，除非将其类型设为有符号数并把初始值设为-1。此外，我们可以额外设置一个`eof_flag`，初始置为false，当到达文件结尾时置位。进行EOF检测时，必须同时满足`eof_flag&&expected_index==eof_index`

## `TCP Receiver`类

### 工具函数：字节编号之间的转换

- 对于字节的**编号**存在**三种类型**，分别是**字节流编号stream index**、TCP报文段中的**序号seqno**、**绝对序号absolute seqno**，TCP接收方需要编写**工具函数**，实现三者之间的相互**转换**

  - `StreamReassembler`中的某个字节都有一个**64位的stream index**，该下标**从零开始**，不断递增
  - 在TCP报文段中，每个字节的下标是用**32位的seqno**表示的。由于32位能表示的范围较小，因此**seqno循环使用**。为了安全性，seqno随机指定**初始序号ISN**，后面的报文段在该ISN的基础上循环递增。TCP传输的**SYN报文段**和**FIN报文段**也**占用一个seqno**
  - 绝对序号，64位的absolute seqno。从0开始，不循环使用。TCP传输的**SYN报文段**和**FIN报文段**也**占用一个absolute seqno**；与之相对的，**stream index**只对有效字节编号

- absolute seqno与stream index之间的转换，只考虑有效字节
  $$
  absolute\ seqno-1 = stream \ index
  $$

- absolute seqno与seqno之间的转换

  

### 功能设计

- 实现TCP的接收方，从接收的TCP报文段提取数据，并转换为字节流输入`StreamReassembler`中
- 此外，TCP接收方还需要告诉发送方两个信息，以实现**流量控制**
  - **最小的未重组的字节的序号**，也叫做**确认号**`ackno`，它代表当前接收方希望收到的第一个字节
  - **接收窗口**，这是第一个未重组的字节序号到第一个未接收的字节序号之间的大小

### 成员设计

- **数据成员**
  - `capacity`，`TCPReceiver`的容量，本质上就是`StreamReassembler`的容量
  - `isn`，存储由发送方随机指定的TCP连接的初始序号
  - `eof`，当接收到发送方的FIN报文段时，意味着到达文件结尾，不会有更多数据到来
  - `reassembler`，接收方所拥有的底层`StreamReassembler`对象
  - `checkpoint`，用于进行absolute seqno到seqno转换的特殊成员，这种转换会**将seqno转换到离checkpoint最近的absolute seqno**，保证了转换的结果唯一，总是**将当前重组的最后一个字节的absolute seqno作为最新的checkpoint**

- **成员函数**
  - `segment_received(const TCPSegment&)`，**接收**一个**TCP报文段**，需要将其中的数据送入`StreamReassembler`
  - `ackno()`，计算并返回确认号
  - `window_size()`，计算并返回接收窗口
  - `unassembled_bytes()`，计算并返回已接受当未确认的字节数
  - `stream_out()`，获取`TCPReceiver`所拥有的（`StreamReassembler`所拥有的）`ByteStream`的接口

### 关键问题分析

- 实现`TCPReceiver`并不涉及复杂的算法，但是细节较多。关键在于seqno与absolute seqno与stream index之间的**转换关系**，需要注意的点是它们**是否考虑SYN标志位、FIN标志位**以及它们之间的影响
- `segment_received`
  - 若接收到**SYN报文段**，需要**记录ISN**
  - 将TCP报文段数据的首个字节的序号seqno转换为stream index（先将其转换为**absolute seqno**，然后考虑SYN标志位**再减1**）
  - 调用`push_substring`将该字符串送入`StreamReassembler`对象
- `ackno`
  - 首先获取该字节的`stream index`，即`StreamReassembler`中的`expected_index`，但由于它是private成员，我们无法直接获取。可以通过`ByteStream`中的`bytes_written`获得，该值就是期望写入的下一个字节的stream index
  - 将该值转换为absolute seqno时，考虑SYN报文段，因此加1
  - 特别地，我们还需要考虑FIN报文段，而FIN报文段被确认的标志就是`StreamReassembler`检测到文件结尾，并调用`ByteStream`的`end_input`方法，因此我们可以通过`ByteStream`的`input_ended()`判断FIN报文段是否被确认，如确认则再加1，从而获得absolute seqno
  - 最后将其转换为seqno并返回
- `window_size`
  - 根据TCP流量控制的相关定义，该值等于容量将去`ByteStream`缓存区中的字节数，它代表着TCP发送方允许发送但未确认的最多字节数，也代表着`StreamReassembler`缓存区中允许的最多字节数

### 数据结构与算法设计

- `TCPReceiver`类没有使用特殊数据结构，依赖于`StreamReassembler`类实现字节流的重组

### 类定义

```c++
class TCPReceiver {
    //! Our data structure for re-assembling bytes.
    StreamReassembler _reassembler;

    //! The maximum number of bytes we'll store.
    size_t _capacity;

    std::optional<WrappingInt32> _isn = nullopt;

    uint64_t _checkpoint = 0;
};
```

### 成员实现

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

size_t TCPReceiver::window_size() const { return _capacity - stream_out().buffer_size(); }
```

### 总结

- `ackno`的特殊情况，当我们确认服务端FIN报文段时，仍然需要返回一个ack（第四次挥手），而此时的`seqno`需要**同时考虑`SYN`和`FIN`**，因此我们把**下一个字节的`stream index`加二才得到了对应的`absolute seqno`**



## `TCP Sender`类

 ### 功能设计

- TCP发送方，从`ByteStream`对象读取上层应用程序传输的**字节**，封装成**TCP报文段**并**发送**
- 只要`ByteStream`中有上层应用程序写入的字节，且接收窗口非空，则发送TCP报文段
- 缓存**已发送未确认**的TCP报文段
- 如果TCP报文段一定时间内没有得到确认（**超时**），则**重传**

### 成员设计

- 略

### 关键问题分析

- **重传计时器**的实现
  - 其重传超时间隔`RTO`是变量，而重传超时间隔初始值`RTO_initial`是常量
  - 当TCP发送端发送一个TCP报文段前（非重传），或收到ACK，会把`RTO`**重设**为`RTO_initial`
  - 当重传计时器**超时**，则`RTO`**加倍**，并重传当缓存中的第一个未确认TCP报文段
  - 每当**发送**一个TCP报文段时，如果计时器未启动，则**启动计时器**
  - 当接收到一个ACK后，如果输出缓冲中没有报文段，即所有发送的报文段都确认了，则**停止计时器**
- 根据`ByteStream`和**接收窗口**的情况决定**发送报文段**
  - 如果当前窗口还有空位（`window_size-bytes_in_flight()>0`），则**发送TCP数据报**，有三种情况
    - **SYN报文段**：如果`next_seqno`为0说明还未发送任何报文段，此时应该先发送SYN报文段）
    - **普通报文段**：尽可能多（不超出`TCPConfig::MAX_PAYLOAD_SIZE`，理论上当TCP首部、IP数据报首部不包含选项字段时均为**20字节**，故该值为**1460字节**，因为以太网帧的数据字段上限为**1500字节**）地从`ByteStream`中读取数据并发送
    - **FIN报文段**：如果`ByteStream`处于`input_ended()`状态并且缓存中的字节已全部读出则标志着文件结尾，此时可以发送FIN报文段
  - 将数据放入报文段后，调用`TCPSegment`类的`length_in_sequence_space()`方法获取TCP数据报的字节数，并更新`next_seqno`，然后将该数据报**发送**并**放入输出缓存**
  - 如果计时器未启动则**启动计时器**
- **收到ACK**时的行为
  - 将`ackno`转换为absolute seqno，如果是**新的**`ackno`则需要
  - 更新发送方保存的`ackno`
  - 重设重传计时器的**`RTO`为初始值**
  - 从输出缓存中将**已确认的报文段弹出**
  - 检查缓存区是否还有未确认报文段，若有则**重启计时器**，否则**停止计时器**
- 如果收到ACK中的接**收窗口是0，如何处理？**
  - 如果接收窗口为0，从理论上分析我们不应该发送任何TCP报文段
  - 倘若我们不发送任何TCP报文段且当前所有发送的报文段都已确认，则此后我们可能无法收到任何接收方的报文，这样即使接收方的应用程序从`ByteStream`读取数据使得接收窗口大小重新变为非0，我们也无法得知该信息
  - 根据上述分析，若**接收窗口为0**，我们会发送一个**带有1字节数据的TCP报文段**。该TCP报文段的主要作用从返回的报文中**获取窗口信息**

### 数据结构与算法设计

- 不涉及任何数据结构，因为大部分的工作已经在`ByteStream`中完成
- 需要设计重传计时器`RetransmissionTimer`类，主要涉及时间的更新，以及对`RTO`的更新

### 类定义

```c++
class RetransmissionTimer {
  private:
    unsigned int RTO_;
    const unsigned int RTO_initial_;
    size_t cur_;
    bool running_ = false;
    // check if expire happened
    bool getAlarm_();

  public:
    RetransmissionTimer(const unsigned int RTO);
    // reset the timer
    void reset();
    // restart the timer
    void restart();
    // start at a certain time
    void start();
    // stop the timer
    void stop();
    // update current time, check if expire happened
    bool update(size_t time);
    // reset RTO
    void resetRTO();
    // double RTO
    void doubleRTO();
};

class TCPSender {
  private:
    //! our initial sequence number, the number for our SYN.
    WrappingInt32 isn_;
    //! outbound queue of segments that the TCPSender wants sent
    std::queue<TCPSegment> segments_out_{};
    //! retransmission timer for the connection
    unsigned int initial_retransmission_timeout_;
    //! outgoing stream of bytes that have not yet been sent
    ByteStream stream_;
    //! the (absolute) sequence number for the next byte to be sent
    uint64_t next_seqno_{0};
    //! the outstanding data structure
    std::queue<TCPSegment> outstanding_{};
    size_t window_size_ = 1;
    RetransmissionTimer timer_;
    unsigned int retransmissions_cnt_ = 0;
    uint64_t ackno_ = 0;
    void reset_consecutive_retransmissions_();
    uint64_t bytes_cnt_ = 0;
    bool fin_ = false;

  public:
	// ...
};
```

### 成员实现

```c++
#include "tcp_sender.hh"
#include "tcp_config.hh"
#include <iostream>
#include <random>

// Dummy implementation of a TCP sender

// For Lab 3, please replace with a real implementation that passes the
// automated checks run by `make check_lab3`.

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

//! \param[in] capacity the capacity of the outgoing byte stream
//! \param[in] retx_timeout the initial amount of time to wait before retransmitting the oldest outstanding segment
//! \param[in] fixed_isn the Initial Sequence Number to use, if set (otherwise uses a random ISN)
TCPSender::TCPSender(const size_t capacity, const uint16_t retx_timeout, const std::optional<WrappingInt32> fixed_isn)
    : isn_(fixed_isn.value_or(WrappingInt32{random_device()()}))
    , initial_retransmission_timeout_{retx_timeout}
    , stream_(capacity)
    , timer_(retx_timeout) {}

uint64_t TCPSender::bytes_in_flight() const { return next_seqno_ - ackno_; }

void TCPSender::fill_window() {
    // make each individual TCPSegment as big as possible
    size_t size;;
    while (!fin_ && (size = (window_size_? window_size_: 1) - bytes_in_flight())) {
        TCPSegment seg = TCPSegment();
        // SYN segment doesn't carry data
        if (next_seqno_ == 0) {
            seg.header().syn = true;
        } else {
            string payload = stream_.read(min(size, TCPConfig::MAX_PAYLOAD_SIZE));
            seg.payload() = Buffer(std::move(payload));
            size -= seg.payload().size();
            // FIN segment doesn't carry data
            if (size && payload.size() == 0 && stream_.buffer_empty() && stream_.input_ended()) {
                seg.header().fin = true;
                fin_ = true;
            }
        } 
        if (seg.length_in_sequence_space()) {
            seg.header().seqno = wrap(next_seqno_, isn_);
            next_seqno_ += seg.length_in_sequence_space();
            outstanding_.push(seg);
            segments_out_.push(seg);
            bytes_cnt_ += seg.length_in_sequence_space();
            timer_.start();
        } else
            break;
    }
}

//! \param ackno The remote receiver's ackno (acknowledgment number)
//! \param window_size The remote receiver's advertised window size
void TCPSender::ack_received(const WrappingInt32 ackno, const uint16_t window_size) {
    uint64_t seqno = unwrap(ackno, isn_, next_seqno_);
    // impossible ackno is ignored
    if (seqno > next_seqno_)
        return;
    window_size_ = window_size;
    if (seqno > ackno_) {
        timer_.resetRTO();
        timer_.restart();
        reset_consecutive_retransmissions_();
        ackno_ = seqno;
    }
    if (outstanding_.empty())
        timer_.stop();
    else {
        // lookup outstanding and pop the acknowledged segments
        while (!outstanding_.empty() && seqno >= unwrap(outstanding_.front().header().seqno, isn_, next_seqno_) +
                                                     outstanding_.front().length_in_sequence_space()) {
            outstanding_.pop();
        }
    }
}

//! \param[in] ms_since_last_tick the number of milliseconds since the last call to this method
void TCPSender::tick(const size_t ms_since_last_tick) {
    // retransmit
    if (timer_.update(ms_since_last_tick) && !outstanding_.empty()) {
        TCPSegment seg = outstanding_.front();
        segments_out_.push(seg);
        if (window_size_ > 0) {
            retransmissions_cnt_++;
            timer_.doubleRTO();
        }
        // reset and start with a doubled RTO
        timer_.restart();
    }
}

unsigned int TCPSender::consecutive_retransmissions() const { return retransmissions_cnt_; }

void TCPSender::reset_consecutive_retransmissions_() { retransmissions_cnt_ = 0; }

void TCPSender::send_empty_segment() {
    TCPSegment seg = TCPSegment();
    // Dont update _next_seqno
    seg.header().seqno = wrap(next_seqno_, isn_);
    segments_out_.push(seg);
}


//! \param[in] RTO the initial value of RTO
RetransmissionTimer::RetransmissionTimer(const unsigned int RTO) : RTO_{RTO}, RTO_initial_{RTO}, cur_{0} {}

void RetransmissionTimer::reset() {
    running_ = false;
    cur_ = 0;
}

void RetransmissionTimer::restart() {
    running_ = true;
    cur_ = 0;
}

// RetransmissionTimer can start at a certain time
void RetransmissionTimer::start() {
    if (!running_) {
        running_ = true;
        cur_ = 0;
    }
}

bool RetransmissionTimer::update(size_t time) {
    cur_ += time;
    return getAlarm_();
}

// resset RTO
void RetransmissionTimer::resetRTO() { RTO_ = RTO_initial_; }

// double RTO
void RetransmissionTimer::doubleRTO() { RTO_ *= 2; }

// check if expire happend
bool RetransmissionTimer::getAlarm_() {
    return running_ && cur_ >= RTO_;
}

// stop the timer
void RetransmissionTimer::stop() { running_ = false; }
```

### 总结

- **发送新的TCP报文段的行为**
- **收到ACK时的行为**
- **对于重传计时器的管理**
- **当接收窗口为0时的行为**

## `TCP Connection`类

### 功能设计

- 将已实现的`TCP Receiver`类和`TCP Sender`类组合称为`TCP Connection`类，使其具备TCP的基本功能，并将其用于与实际因特网进行通信
- 当`TCP Connection`类**接收TCP报文段**时，它会
  - 如果RST标志位置位，则将输入流和输出流都设置为错误状态，并终止连接
  - 否则，将报文段传递给`TCP Receiver`处理
  - 如果`ACK`标志位置位，将`ackno`和`window_size`告诉`TCP Sender`
  - 如果报文段占据了至少一个seqno（即并非空报文段），则`TCP Connection`至少应该返回一条报文段，以反映自己ackno和window size的变化情况
  - 可能收到一种特殊的报文段：**keep-alive报文段**，此时无论它是否携带有效数据，都应该发送一条空报文段以告知本节点仍处于活跃状态
- 当`TCP Connection`类**发送报文段**时，它会
  - 在发送前通过`TCP Receiver`获取ackno和接收窗口信息，在报文段设置相应字段，并将`ACK`标志位置位
- 操作系统将定时调用`TCP Connection`类的`tick`方法以告知当前的时间
  - 调用`TCP Sender`的`tick`方法以告知其当前时间
  - 如果**连续重传次数**超过了`TCPConfig::MAX_RETX_ATTEMPTS`，则中止连接并发送**reset报文段**给对等方（RST标志位置位的空报文段）
  - **end the connection cleanly** if necessary

### 成员设计

- `connect()`，发送SYN报文段，通过三次握手建立TCP连接
- `write(const string&)`，由上层的应用程序调用，将数据写入并通过TCP连接发送
- `remaining_outbound_capacity()`，当前可以写入的字节数量
- `end_input_stream()`，终止输出`ByteStream`（但仍然可以接收数据）
- `inbound_stream()`，接收到的数据写入`ByteStream`对象的接口
- `bytes_in_flight()`，已发送未确认的字节数
- `unassembled_bytes()`，收到但是未重组的字节数，等于`StreamReassembler`缓存区的字节数
- `time_since_last_segment_received()`，从收到上个TCP报文段后经过的毫秒数
- `state()`，返回`TCPSender`、`TCPReceiver`和`TCPConnection`对象

### 关键问题分析

- **收到或发送RST报文段**后进行什么操作？
  - 将inbound和outbound的`ByteStream`对象设置为错误状态
  - 活跃标志`active`置为false，**直接中止当前TCP连接**
- 什么时候主动**发送RST报文段**？
  - 当连续**重传次数超过了上限值**时
  - 当`TCPConnection`的**析构函数**被调用，而此时还处于`active`状态时
- 什么时候**发送空报文段**？
  - 当TCP必须发送一条报文段，而又没有数据要发送时，则发送空报文段
  - 必须**收到一条包含数据的报文段**，则必须**告诉对方ackno和window size**信息

- 什么时候**终止TCP连接**？如何终止TCP连接？

  - 根据`TCP Sender`类中的分析，当上层应用程序以到达文件结尾，且缓存区的所有数据都已发送并确认，则发送FIN报文段请求终止TCP连接

  - TCP连接断开时要执行四次挥手，具体地

    - A向B发送FIN报文段
    - B向A发送对FIN报文段的确认
    - B（将输出缓存的所有数据发送并确认后）向A发送FIN报文段
    - A向B发送对FIN报文段的确认

  - 执行完上述过程后，对于B

    - 如果它收到了A的确认，则可以立刻断开TCP连接
    - 如果没收到，则超时重传FIN报文段

  - 根据上述分析，可知A无法直接中止连接，因为它必须确定B收到了自己对FIN报文段的确认，A应该如何确定呢？

    - **等待10倍`RTO`时间**，如果没收到B重传的FIN报文段，则认为B已经收到了自己的ACK，中止TCP连接

    > 实践中我们等待**2倍的MSL (Maximum Segment Life)**，是任何报文在网络上存在的最长时间，因为B收到自己的ACK最多经历1MSL，而B重传FIN报文段到达A最多再经历1MSL，因此最长等待时间2MSL。如果仍然未等到，则说明B已经收到ACK并关闭连接了。

    - 若在此过程中**收到了B重传的FIN报文段**，则再次**发送ACK**，并**重新开始计时**

  - 上述四次挥手后**A等待的时间称为linger time**

  - 相反，B收到了A的ACK后无需等待就直接关闭连接，这是因为A是主动发送FIN报文段的一方，因此当B收到它对自己FIN报文段的ACK后就可以**肯定A已经完成了所有数据的传输工作**，因此它直接关闭TCP连接

### 数据结构与算法分析

- 不涉及数据结构与算法

### 类定义

```c++
#ifndef SPONGE_LIBSPONGE_TCP_FACTORED_HH
#define SPONGE_LIBSPONGE_TCP_FACTORED_HH

#include "tcp_config.hh"
#include "tcp_receiver.hh"
#include "tcp_sender.hh"
#include "tcp_state.hh"

//! \brief A complete endpoint of a TCP connection
class TCPConnection {
  private:
    TCPConfig cfg_;
    TCPReceiver receiver_;
    TCPSender sender_;

    //! outbound queue of segments that the TCPConnection wants sent
    std::queue<TCPSegment> segments_out_{};

    //! Should the TCPConnection stay active (and keep ACKing)
    //! for 10 * _cfg.rt_timeout milliseconds after both streams have ended,
    //! in case the remote TCPConnection doesn't know we've received its whole stream?
    bool linger_after_streams_finish_{true};

    //! \brief read Segments from _sender, push them onto _segments_out, to send them over TCP
    void send_(const bool rst, const bool force);

    TCPSegment readSegment_();

    size_t now_ = 0;
    size_t last_recv_ = 0;
    bool active_ = true;

    void checkLinger_();

    bool syn_ = false;

  public:
    //! \name "Input" interface for the writer
    //!@{

    //! \brief Initiate a connection by sending a SYN segment
    void connect();

    //! \brief Write data to the outbound byte stream, and send it over TCP if possible
    //! \returns the number of bytes from `data` that were actually written.
    size_t write(const std::string &data);

    //! \returns the number of `bytes` that can be written right now.
    size_t remaining_outbound_capacity() const;

    //! \brief Shut down the outbound byte stream (still allows reading incoming data)
    void end_input_stream();
    //!@}

    //! \name "Output" interface for the reader
    //!@{

    //! \brief The inbound byte stream received from the peer
    ByteStream &inbound_stream() { return receiver_.stream_out(); }
    //!@}

    //! \name Accessors used for testing

    //!@{
    //! \brief number of bytes sent and not yet acknowledged, counting SYN/FIN each as one byte
    size_t bytes_in_flight() const;
    //! \brief number of bytes not yet reassembled
    size_t unassembled_bytes() const;
    //! \brief Number of milliseconds since the last segment was received
    size_t time_since_last_segment_received() const;
    //!< \brief summarize the state of the sender, receiver, and the connection
    TCPState state() const { return {sender_, receiver_, active(), linger_after_streams_finish_}; };
    //!@}

    //! \name Methods for the owner or operating system to call
    //!@{

    //! Called when a new segment has been received from the network
    void segment_received(const TCPSegment &seg);

    //! Called periodically when time elapses
    void tick(const size_t ms_since_last_tick);

    //! \brief TCPSegments that the TCPConnection has enqueued for transmission.
    //! \note The owner or operating system will dequeue these and
    //! put each one into the payload of a lower-layer datagram (usually Internet datagrams (IP),
    //! but could also be user datagrams (UDP) or any other kind).
    std::queue<TCPSegment> &segments_out() { return segments_out_; }

    //! \brief Is the connection still alive in any way?
    //! \returns `true` if either stream is still running or if the TCPConnection is lingering
    //! after both streams have finished (e.g. to ACK retransmissions from the peer)
    bool active() const;
    //!@}

    //! Construct a new connection from a configuration
    explicit TCPConnection(const TCPConfig &cfg)
        : cfg_{cfg}, receiver_(cfg.recv_capacity), sender_(cfg_.send_capacity, cfg_.rt_timeout, cfg_.fixed_isn) {}

    //! \name construction and destruction
    //! moving is allowed; copying is disallowed; default construction not possible

    //!@{
    ~TCPConnection();  //!< destructor sends a RST if the connection is still open
    TCPConnection() = delete;
    TCPConnection(TCPConnection &&other) = default;
    TCPConnection &operator=(TCPConnection &&other) = default;
    TCPConnection(const TCPConnection &other) = delete;
    TCPConnection &operator=(const TCPConnection &other) = delete;
    //!@}
};

#endif  // SPONGE_LIBSPONGE_TCP_FACTORED_HH

```

### 成员实现

```c++
#include "tcp_connection.hh"

#include <iostream>

// Dummy implementation of a TCP connection

// For Lab 4, please replace with a real implementation that passes the
// automated checks run by `make check`.

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

size_t TCPConnection::remaining_outbound_capacity() const { return sender_.stream_in().remaining_capacity(); }

size_t TCPConnection::bytes_in_flight() const { return sender_.bytes_in_flight(); }

size_t TCPConnection::unassembled_bytes() const { return receiver_.unassembled_bytes(); }

size_t TCPConnection::time_since_last_segment_received() const { return now_ - last_recv_; }

void TCPConnection::segment_received(const TCPSegment &seg) {
    last_recv_ = now_;
    // if the rst (reset) flag is set, sets both the inbound and outbound streams to the error state and kills the
    // connection permanently.
    if (seg.header().rst) {
        sender_.stream_in().set_error();
        receiver_.stream_out().set_error();
        active_ = false;
    } else {
        receiver_.segment_received(seg);
        if (seg.header().syn) {
            syn_ = true;
        }
        // if the ack flag is set, tells the TCPSender about the fields it cares about on incoming segments: ackno and
        // window size.
        if (seg.header().ack) {
            sender_.ack_received(seg.header().ackno, seg.header().win);
        }
        // if the incoming segment occupied any sequence numbers, the TCPConnection makes sure
        // that at least one segment is sent in reply, to react an update in the ackno and window size.
        if (seg.length_in_sequence_space() > 0)
            send_(false, true);
        else if (receiver_.ackno().has_value() && seg.header().seqno == receiver_.ackno().value() - 1)
            // respond to keep-alive segment
            send_(false, true);
        else
            send_(false, false);
    }
}

bool TCPConnection::active() const { return active_; }

size_t TCPConnection::write(const string &data) {
    size_t res = sender_.stream_in().write(data);
    send_(false, false);
    return res;
}

TCPSegment TCPConnection::readSegment_() {
    // Set Ackno and Window size field of outgoing TCP segments
    TCPSegment seg = sender_.segments_out().front();
    if (receiver_.ackno().has_value()) {
        seg.header().ack = true;
        seg.header().ackno = receiver_.ackno().value();
    }
    seg.header().win = receiver_.window_size();
    sender_.segments_out().pop();
    return seg;
}

//! \param[in] rst if set, TCPSender must send an empty segment with rst set.
//! \param[in] force if set, TCPSender must at least send one segment.
void TCPConnection::send_(const bool rst, const bool force) {
    if (rst) {
        TCPSegment seg;
        seg.header().rst = true;
        sender_.stream_in().set_error();
        receiver_.stream_out().set_error();
        active_ = false;
        segments_out_.push(seg);
        return;
    }

    bool force_ = force;
    if (syn_) {
        checkLinger_();
        sender_.fill_window();
        while (!sender_.segments_out().empty()) {
            force_ = false;
            TCPSegment seg = readSegment_();
            segments_out_.push(seg);
        }
    } else
        force_ = false;
    if (force_) {
        sender_.send_empty_segment();
        TCPSegment seg = readSegment_();
        segments_out_.push(seg);
    }
}

void TCPConnection::checkLinger_() {
    if (inbound_stream().input_ended() && !(sender_.stream_in().input_ended())) {
        linger_after_streams_finish_ = false;
    }
}

//! \param[in] ms_since_last_tick number of milliseconds since the last call to this method
void TCPConnection::tick(const size_t ms_since_last_tick) {
    sender_.tick(ms_since_last_tick);
    now_ += ms_since_last_tick;
    // Abort the connection, and send a reset segment to the peer (an empty segment with the rst flag set)
    // if the number of consecutive retransmissions is more than an upper limit TCPConfig::MAX RETX ATTEMPTS.
    if (sender_.consecutive_retransmissions() > TCPConfig::MAX_RETX_ATTEMPTS)
        send_(true, true);
    else
        send_(false, false);
    // end the connection cleanly if necessary
    // If the inbound stream ends before the TCPConnection has reached EOF on its outbound stream, this variable needs
    // to be set to false.
    checkLinger_();

    if (inbound_stream().input_ended() && sender_.stream_in().input_ended() && sender_.bytes_in_flight() == 0) {
        if (linger_after_streams_finish_) {
            if (now_ - last_recv_ >= cfg_.rt_timeout * 10) {
                active_ = false;
            }
        } else
            active_ = false;
    }
}

void TCPConnection::end_input_stream() {
    sender_.stream_in().end_input();
    send_(false, false);
}

void TCPConnection::connect() {
    syn_ = true;
    send_(false, true);
}

TCPConnection::~TCPConnection() {
    try {
        if (active()) {
            cerr << "Warning: Unclean shutdown of TCPConnection\n";

            // Your code here: need to send a RST segment to the peer
            send_(true, true);
        }
    } catch (const exception &e) {
        std::cerr << "Exception destructing TCP FSM: " << e.what() << std::endl;
    }
}
```

### 总结

- 如何正确的关闭TCP连接？四次挥手后为何还要等待？
- RST报文段的作用是什么？需要进行什么操作？

## 测试结果

![](1.png)

## 遇到的bug（总结遇到的bug和如何解决，面试准备）

- `checkpoint`应该设置为重组的最后一个字节的absolute seqno

# 网络层、链路层

## 路由器接口

### 功能设计

- **转发数据报**
- **IP地址**到**链路层地址**的转换，若交换机表中存在则直接索引，若不存在则需通过**ARP查询**获取

### 成员设计

- `send_datagram(const InternetDatagram&, const Address&)`，将该数据报生成以太网帧并发送到目的地址
  - 如果该目的地址的链路层**MAC地址已知**，则可以立即**成帧并发送**
  - 如果其MAC地址**未知**，则运行**ARP协议**获取
  - 将数据报存储在**输出缓存队列中等待**，直到ARP响应报文到来，获取对应的MAC地址，再从输出队列中取出数据报发送
- `recv_frame(const EthernetFrame&)`，接收以太网的链路层帧，向上传递给对应的网络层协议
  - 接收以太网帧，对比MAC地址，如果不是**自己的MAC地址**或**广播地址**，则丢弃该帧
  - 否则，从以太网帧提取数据报
    - 如果是**IPv4数据报**，则提取并返回
    - 如果是**ARP响应报文**，则提取并存储该映射关系，同时调用`send_datagram`发送该地址的缓存队列中的IP数据报
    - 如果是**ARP查询报文**且**目的地址是自己的IP地址**，则返回带映射关系的ARP响应报文
- `tick(const size_t)`，时间控制
  - 控制时间
  - 交换机表中的表项**只能存储30s**，过期即丢弃
  - **5秒内**不要向同一个IP地址发送相同的ARP查询报文

### 关键问题分析

- 放在队列中的数据报什么时候传输？
  - 当数据报的目的IP地址在交换机表中没有对应表项时，必须**暂存队列**中，等待ARP响应报文获取该地址
  - 当交换机表更新时，即可以超看更新的IP地址是否有需要发送的数据报，有则将队列中的数据报都发送
  - 交换机表更新有两种可能
    - **收到ARP响应报文**，对应我们之前发送的某个ARP查询报文
    - **收到其他链路层帧**，提取其MAC地址和IP地址，并把该映射关系放入交换机表（交换机表的自学习特性）
- 数据结构的选用
  - 本题涉及**交换机表的存储**，以及**历史ARP查询报文的存储**（防止短时间内对同一IP地址重复查询），都可以用**哈希表**实现，详见“数据结构与算法分析”

### 数据结构与算法分析

- **交换机表**以何种数据结构存储表项？

  交换机表的作用是将**IP地址映**射到**MAC地址**，而且要存储保存的时间，一旦**过期**（实验中设置为30s）则该表项无效，需要（通过**ARP协议**）重新获取，采用`unordered_map<uint32_t, pair<EthernetAddress, size_t>>`实现

- **五秒内**不能向**同一个IP地址重复发送ARP查询报文**，因此需要用`unordered_map<uint32_t, size_t>`存储各个IP地址及其上一条ARP查询报文的时间，每次发送前先查找，如果上次查询的时间不超过5s则应该继续等待

### 类定义

```c++
class NetworkInterface {
  private:
    //! Ethernet (known as hardware, network-access-layer, or link-layer) address of the interface
    EthernetAddress _ethernet_address;

    //! IP (known as internet-layer or network-layer) address of the interface
    Address _ip_address;

    //! outbound queue of Ethernet frames that the NetworkInterface wants sent
    std::queue<EthernetFrame> _frames_out{};

    std::unordered_map<uint32_t, std::pair<EthernetAddress, size_t>> f_;
    std::unordered_map<uint32_t, size_t> request_;

    std::unordered_map<uint32_t, std::queue<InternetDatagram>> q;
    size_t curr;
  public:
    // ...
};
```

### 成员实现

```c++
#include "network_interface.hh"
#include "arp_message.hh"
#include "ethernet_frame.hh"
#include <iostream>

// Dummy implementation of a network interface
// Translates from {IP datagram, next hop address} to link-layer frame, and from link-layer frame to IP datagram

// For Lab 5, please replace with a real implementation that passes the
// automated checks run by `make check_lab5`.

// You will need to add private members to the class declaration in `network_interface.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

using namespace std;

//! \param[in] ethernet_address Ethernet (what ARP calls "hardware") address of the interface
//! \param[in] ip_address IP (what ARP calls "protocol") address of the interface
NetworkInterface::NetworkInterface(const EthernetAddress &ethernet_address, const Address &ip_address)
    : _ethernet_address(ethernet_address), _ip_address(ip_address), f_({}), request_({}), q({}), curr(0) {
    cerr << "DEBUG: Network interface has Ethernet address " << to_string(_ethernet_address) << " and IP address "
         << ip_address.ip() << "\n";
}

//! \param[in] dgram the IPv4 datagram to be sent
//! \param[in] next_hop the IP address of the interface to send it to (typically a router or default gateway, but may also be another host if directly connected to the same network as the destination)
//! (Note: the Address type can be converted to a uint32_t (raw 32-bit IP address) with the Address::ipv4_numeric() method.)
void NetworkInterface::send_datagram(const InternetDatagram &dgram, const Address &next_hop) {
    // convert IP address of next hop to raw 32-bit representation (used in ARP header)
    const uint32_t next_hop_ip = next_hop.ipv4_numeric();
    if (f_.count(next_hop_ip)) {
        if (f_[next_hop_ip].second + 30000 < curr)
            f_.erase(next_hop_ip);
        else {
            EthernetFrame frame;
            frame.payload() = dgram.serialize();
            frame.header().src = _ethernet_address;
            frame.header().dst = f_[next_hop_ip].first;
            frame.header().type = EthernetHeader::TYPE_IPv4;
            _frames_out.push(frame);
            return;
        }
    } 
    q[next_hop_ip].push(dgram);
    if (request_.count(next_hop_ip) && request_[next_hop_ip] + 5000 >= curr)
        return;
    EthernetFrame frame;
    ARPMessage msg;
    msg.opcode = ARPMessage::OPCODE_REQUEST;
    msg.sender_ethernet_address = _ethernet_address;
    msg.sender_ip_address = _ip_address.ipv4_numeric();
    msg.target_ip_address = next_hop_ip;
    frame.payload() = msg.serialize();
    frame.header().src = _ethernet_address;
    frame.header().dst = ETHERNET_BROADCAST;
    frame.header().type = EthernetHeader::TYPE_ARP;
    _frames_out.push(frame);
    request_[next_hop_ip] = curr;
}

//! \param[in] frame the incoming Ethernet frame
optional<InternetDatagram> NetworkInterface::recv_frame(const EthernetFrame &frame) {
    if (frame.header().dst == ETHERNET_BROADCAST || frame.header().dst == _ethernet_address) {
        if (frame.header().type == EthernetHeader::TYPE_ARP) {
            ARPMessage msg;
            if (!(msg.parse(frame.payload()) == ParseResult::NoError))
                return nullopt;
            if (msg.target_ip_address != _ip_address.ipv4_numeric())
                return nullopt;
            f_[msg.sender_ip_address] = std::make_pair(msg.sender_ethernet_address, curr);
            // send an ARP reply Message
            if (msg.opcode == ARPMessage::OPCODE_REQUEST) {
                EthernetFrame reply_frame;
                ARPMessage reply;
                reply.opcode = ARPMessage::OPCODE_REPLY;
                reply.sender_ethernet_address = _ethernet_address;
                reply.sender_ip_address = _ip_address.ipv4_numeric();
                reply.target_ethernet_address = msg.sender_ethernet_address;
                reply.target_ip_address = msg.sender_ip_address;
                reply_frame.payload() = reply.serialize();
                reply_frame.header().src = _ethernet_address;
                reply_frame.header().dst = msg.sender_ethernet_address;
                reply_frame.header().type = EthernetHeader::TYPE_ARP;
                _frames_out.push(reply_frame);
            }
            // send datagram in the output buffer if possible
            Address addr = Address::from_ipv4_numeric(msg.sender_ip_address);
            while (!q[msg.sender_ip_address].empty()) {
                send_datagram(q[msg.sender_ip_address].front(), addr);
                q[msg.sender_ip_address].pop();
            }
        } else if (frame.header().type == EthernetHeader::TYPE_IPv4) {
            InternetDatagram dgram;
            if (dgram.parse(frame.payload()) == ParseResult::NoError)
                return dgram;
            else
                return nullopt;
        }
    }
    return nullopt;
}

//! \param[in] ms_since_last_tick the number of milliseconds since the last call to this method
void NetworkInterface::tick(const size_t ms_since_last_tick) { 
    curr += ms_since_last_tick; 
}
```

### 总结

- 本实验重点是对**网络层数据平面、链路层**的实现，重点在于对IP地址、链路层帧、链路层帧表头、IP数据报、ARP报文等类的**接口的正确使用**
- 实现了**交换机表**的**自学习、过期**等机制
- 实现了**ARP协议**中发送ARP查询报文、回复ARP响应报文等机制，当发送ARP查询报文时使用的是**MAC广播地址**`FF-FF-FF-FF-FF-FF`
- 协议层次特点及其带来好处，本实验同时涉及了**网络层、链路层中的操作**，由于网络层数据报完全作为链路层帧的数据发送，因此**从链路层帧提取数据报**，以及**将数据报封装成链路层帧**的操作都非常方便，通过`parse`、`serialize`等成员即可完成

# 网络层-控制平面

## 路由器

### 功能设计

- 维护路由器的**转发表**
- 当数据报到来时，进行**基于目的地转发**
- 维护**数据报的TTL值**并据此判断其是否应该被丢弃

### 成员设计

- `add_route(route_prefix, prefix_length, next_hop, interface_num)`，**新增一个转发表项**，IP地址前缀为`route_prefix`，前缀长度为`prefix_length`，通往该地址的路径上下一个路由器的地址是`next_hop`，对应的路由器输出端口编号是`interface_num`
- `route_one_datagram(InternetDatagram&)`，将一个**数据报**进行**转发**
  - 若找到了**匹配的表项**，则从中选择前缀长度最长的一个（**最长前缀匹配规则**），并根据该表项的`next_hop`和`interface`信息进行转发
  - 若**没有**找到对应的表项，则忽略该数据报

### 关键问题分析

- 进行前缀匹配时，假如该表项的地址为`addr1`，目的地址为`addr2`，而该表项的**前缀长度**为`len`，则**`add1`和`add2`的前`len`都一致时，才认为它们匹配**
- **转发表**应该使用什么数据结构实现？
  - 如果直接用**数组存储**，则对于每次查找，都要将目的地址与每个表项进行对比，才能完成最长前缀匹配，因此时间复杂度是$O(N)$的，其中$N$为查找时转发表的表项数量
  - 借助**前缀树Trie**，可以使每次查找的时间降为$O(logN)$

### 数据结构与算法分析

- **前缀树**实现$O(logN)$的最长前缀匹配规则

### 类定义

```c++
class Trie {
public:
    int interface_num;
    std::optional<Address> next_hop{};
    std::shared_ptr<Trie> left{};
    std::shared_ptr<Trie> right{};

    Trie(int i = 0, std::optional<Address> hop=std::nullopt): interface_num(i), next_hop(hop) {}

    void createLeftChild(int i = -1, std::optional<Address> hop = std::nullopt) {
        left = std::make_shared<Trie>(i, hop);
    } 
    
    void createRightChild(int i = -1, std::optional<Address> hop = std::nullopt) {
        right = std::make_shared<Trie>(i, hop);
    }
};

class Router {
    //! The router's collection of network interfaces
    std::vector<AsyncNetworkInterface> _interfaces{};

    //! Send a single datagram from the appropriate outbound interface to the next hop,
    //! as specified by the route with the longest prefix_length that matches the
    //! datagram's destination address.
    void route_one_datagram(InternetDatagram &dgram);
    std::shared_ptr<Trie> route_table{std::make_shared<Trie>(-1)};
};
```

### 成员实现

```c++
#include "router.hh"

#include <iostream>

using namespace std;

// Dummy implementation of an IP router

// Given an incoming Internet datagram, the router decides
// (1) which interface to send it out on, and
// (2) what next hop address to send it to.

// For Lab 6, please replace with a real implementation that passes the
// automated checks run by `make check_lab6`.

// You will need to add private members to the class declaration in `router.hh`

template <typename... Targs>
void DUMMY_CODE(Targs &&... /* unused */) {}

//! \param[in] route_prefix The "up-to-32-bit" IPv4 address prefix to match the datagram's destination address against
//! \param[in] prefix_length For this route to be applicable, how many high-order (most-significant) bits of the route_prefix will need to match the corresponding bits of the datagram's destination address?
//! \param[in] next_hop The IP address of the next hop. Will be empty if the network is directly attached to the router (in which case, the next hop address should be the datagram's final destination).
//! \param[in] interface_num The index of the interface to send the datagram out on.
void Router::add_route(const uint32_t route_prefix,
                       const uint8_t prefix_length,
                       const optional<Address> next_hop,
                       const size_t interface_num) {
    cerr << "DEBUG: adding route " << Address::from_ipv4_numeric(route_prefix).ip() << "/" << int(prefix_length)
         << " => " << (next_hop.has_value() ? next_hop->ip() : "(direct)") << " on interface " << interface_num << "\n";

    // Your code here.
    // route_table[route_prefix] = make_tuple(prefix_length, next_hop, interface_num);
    auto curr = route_table;
    for (size_t i = 0; i < prefix_length; ++i) {
        if (route_prefix & (1 << (31 - i))) {
            if (!curr->right) 
                curr->createRightChild();
            curr = curr->right;
        } else {
            if (!curr->left)
                curr->createLeftChild();
            curr = curr->left;
        }
    }
    curr->next_hop = next_hop;
    curr->interface_num = interface_num;
}

//! \param[in] dgram The datagram to be routed
void Router::route_one_datagram(InternetDatagram &dgram) {
    // Your code here.
    if (dgram.header().ttl <= 1)
        return;
    // Trie
    uint32_t addr = dgram.header().dst;
    auto curr = route_table;
    int interface = -1;
    optional<Address> hop = nullopt;
    if (curr->interface_num >= 0) {
        interface = curr->interface_num;
        hop = curr->next_hop;
    }
    for (size_t i = 0; i < 32; ++i) {
        if (addr & (1 << (31 - i))) {
            if (curr->right)
                curr = curr->right;
            else 
                break;
        } else {
            if (curr->left)
                curr = curr->left;
            else
                break;
        }
        if (curr->interface_num >= 0) {
            interface = curr->interface_num;
            hop = curr->next_hop;
        }
    }
    if (interface == -1)
        return;
    --dgram.header().ttl;
    Address ad = hop.has_value()? hop.value(): Address::from_ipv4_numeric(addr);
    _interfaces[interface].send_datagram(dgram, ad);


    /* brute force matching
    uint32_t addr = dgram.header().dst;
    size_t maxlen = 0;
    uint32_t prefix = 0;
    for (auto& it: route_table) {
        bool flag = true;
        for (size_t i = 0; i < std::get<0>(it.second); ++i) {
            if ((addr & (1 << (31 - i))) != (it.first & (1 << (31 - i)))) {
                flag = false;
                break;
            }
        }
        if (flag && std::get<0>(it.second) > maxlen) {
            maxlen = std::get<0>(it.second);
            prefix = it.first;
        }
    }
    Address ad = std::get<1>(route_table[prefix]).has_value()? std::get<1>(route_table[prefix]).value(): Address::from_ipv4_numeric(addr);
    --dgram.header().ttl;
    _interfaces[std::get<2>(route_table[prefix])].send_datagram(dgram, ad);
    */
    
}

void Router::route() {
    // Go through all the interfaces, and route every incoming datagram to its proper outgoing interface.
    for (auto &interface : _interfaces) {
        auto &queue = interface.datagrams_out();
        while (not queue.empty()) {
            route_one_datagram(queue.front());
            queue.pop();
        }
    }
}
```

### 总结

- 通过**前缀树**代替数组存储转发表，**优化了**最长前缀匹配的时间复杂度，从$O(N)$将为$O(logN)$
- 实现前缀树时，由于需要**动态分配内存**，因此采用**智能指针**`shared_ptr`管理每个树节点的分配
- 查找前缀树时，受一般前缀树的思路影响，导致了bug，具体来说
  - 以字典树为例，字典树将**单词存储**起来，**给定**一个**前缀**，看词典中是否有匹配的单词
  - 这里我们的前缀树**存储**转发表中的网址**前缀**，**给定**一个**目的地址**，查找最长匹配前缀。因此查找时，必须到达前缀的末尾才认为匹配成功
  - 我们通过节点的`next_hop`和`interface_num`值辨别是否到达了前缀的结尾，如果一个节点并非对应前缀结尾，则其`interface_num`的值为-1
  - 查找过程中，我们不断更新`hop`和`interface`，分别代表当前最长匹配前缀对应节点的`next_hop`和`interface_num`，最终查找结束我们就得知，应该**将数据报转发到编号为`interface`的输出端口**，且下一台**路由器的地址为`next_hop`**

# CS-144 lab最终测试结果

- 至此，CS-144实验的各个模块已完成，测试结果如下！成功撒花！

![](2.png)
