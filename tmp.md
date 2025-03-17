# mmap 内存映射原理

## 传统 I/O 操作流程
```mermaid
sequenceDiagram
    participant App as 用户应用程序
    participant UB as 用户缓冲区
    participant Kernel as 内核
    participant KB as 内核缓冲区
    participant Disk as 磁盘

    Note over App,Disk: 传统 I/O (read/write)
    App->>Kernel: 1. read() 系统调用
    Note right of App: 用户态切换到内核态
    Kernel->>Disk: 2. DMA 拷贝
    Disk-->>KB: 3. 数据从磁盘拷贝到内核缓冲区
    KB->>UB: 4. CPU 拷贝
    Note right of App: 内核态切换回用户态
    Kernel-->>App: 5. read() 调用返回
    App->>Kernel: 6. write() 系统调用
    Note right of App: 用户态切换到内核态
    UB->>KB: 7. CPU 拷贝
    KB-->>Disk: 8. DMA 拷贝
    Note right of App: 内核态切换回用户态
    Kernel-->>App: 9. write() 调用返回
    
    Note over App,Disk: 总计：4次上下文切换，4次数据拷贝（2次CPU拷贝，2次DMA拷贝）
```

## mmap 内存映射原理
```mermaid
sequenceDiagram
    participant App as 用户应用程序
    participant UB as 用户缓冲区
    participant Kernel as 内核
    participant KB as 内核缓冲区
    participant Disk as 磁盘

    Note over App,Disk: mmap + write 方式
    App->>Kernel: 1. mmap() 系统调用
    Note right of App: 用户态切换到内核态
    Kernel-->>App: 2. 建立内核缓冲区到用户缓冲区的映射
    Note right of App: 内核态切换回用户态
    Kernel->>Disk: 3. DMA 拷贝
    Disk-->>KB: 4. 数据从磁盘拷贝到内核缓冲区
    Note over KB,UB: 5. 内核缓冲区与用户缓冲区共享，无需CPU拷贝
    App->>Kernel: 6. write() 系统调用
    Note right of App: 用户态切换到内核态
    UB->>KB: 7. CPU 拷贝
    KB-->>Disk: 8. DMA 拷贝
    Note right of App: 内核态切换回用户态
    Kernel-->>App: 9. write() 调用返回
    
    Note over App,Disk: 总计：4次上下文切换，3次数据拷贝（1次CPU拷贝，2次DMA拷贝）
```

## mmap 工作原理图
```mermaid
graph TD
    subgraph "用户空间"
        A[用户进程] --> B[用户虚拟内存地址]
    end
    
    subgraph "内核空间"
        C[内核缓冲区] --> D[物理内存页]
    end
    
    subgraph "存储设备"
        E[磁盘文件]
    end
    
    B <-.映射关系.-> C
    E <--> |DMA拷贝| D
    
    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style C fill:#bfb,stroke:#333,stroke-width:2px
    style D fill:#fbb,stroke:#333,stroke-width:2px
    style E fill:#bbb,stroke:#333,stroke-width:2px
```

## mmap 的优势

1. **减少数据拷贝次数**：通过内存映射，减少了内核缓冲区到用户缓冲区的一次 CPU 拷贝

2. **减少内存使用**：直接映射文件到用户空间，避免了额外的缓冲区开销

3. **提高 I/O 性能**：特别是对大文件的读写操作，性能提升明显

4. **支持共享内存**：多个进程可以通过 mmap 共享同一块物理内存

## mmap 的应用场景

1. 大文件处理
2. 进程间通信
3. 数据库系统
4. 高性能 I/O 框架（如 Netty、Kafka 等） 



## Channel




```mermaid

classDiagram
    class Channel {
        <<interface>>
        +ChannelPipeline pipeline()
        +ChannelConfig config()
        +boolean isOpen()
        +boolean isRegistered()
        +boolean isActive()
        +ChannelMetadata metadata()
        +SocketAddress localAddress()
        +SocketAddress remoteAddress()
        +ChannelFuture closeFuture()
        +boolean isWritable()
        +Channel read()
        +Channel flush()
    }
    
    class AbstractChannel {
        <<abstract>>
        -ChannelPipeline pipeline
        -EventLoop eventLoop
        -ChannelFuture closeFuture
    }
    
    class NioSocketChannel {
        -java.nio.channels.SocketChannel ch
    }
    
    class NioServerSocketChannel {
        -java.nio.channels.ServerSocketChannel ch
    }
    
    Channel <|-- AbstractChannel
    AbstractChannel <|-- NioSocketChannel
    AbstractChannel <|-- NioServerSocketChannel
    
    
    
```


```mermaid
graph TD
    A[客户端] <--> B[Channel]
    B <--> C[ChannelPipeline]
    C <--> D[ChannelHandler]
    B <--> E[EventLoop]
    E <--> F[EventLoopGroup]
    F <--> G[线程池]
    
    style B fill:#f96,stroke:#333,stroke-width:2px
```