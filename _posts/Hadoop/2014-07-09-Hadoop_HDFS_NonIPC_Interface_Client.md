---
layout: post
title: Hadoop 流式接口 
category: Hadoop
tags: 
keywords: 
description: 
---


## 简介
客户端通过ClientProtocol的API对HDFS文件/目录元数据进行远程调用操作，这些命令发送到名字节点执行。

文件数据的读写采取流式访问数据的方法，而不通过IPC调用。HDFS没有采取IPC调用实现文件的读写，是因为IPC实现文件读写，效率达不到系统要求，而Hadoop需要支撑超大文件。HDFS基于TCP的流式数据访问接口，有利于数据的批量处理，提高数据的吞吐量。从中可以看到，技术的选择是根据业务需求而定。

HDFS基于TCP的流式基于除了读写数据之外，还有数据块替换、数据块拷贝和数据块校验信息读等接口。

## 流式接口

数据节点的流式接口操作码定义在org.apache.hadoop.hdfs.protocol.DataTransferProtocol类中。

	public interface DataTransferProtocol {
	  // Processed at datanode stream-handler
	  public static final byte OP_WRITE_BLOCK = (byte) 80;
	  public static final byte OP_READ_BLOCK = (byte) 81;
	  /**
	   * @deprecated As of version 15, OP_READ_METADATA is no longer supported
	   */
	  @Deprecated public static final byte OP_READ_METADATA = (byte) 82;
	  public static final byte OP_REPLACE_BLOCK = (byte) 83;
	  public static final byte OP_COPY_BLOCK = (byte) 84;
	  public static final byte OP_BLOCK_CHECKSUM = (byte) 85;
	  ...
	}

OP_***定义了具体的操作码，根据这些常量的名字我们基本上就可以得到此操作码的具体应用。

### 读数据
在HDFS中，读数据就是从数据节点中的某个数据块读取一段字节流到客户端或者其他数据节点，它对应的操作码为81。

#### 请求报文

数据节点读请求报文如下：

     +-------------------------------------------------------------------------+
     | TRANSFER_VERSION  |    81     |      blockId     |    generationStamp   |
     +-------------------------------------------------------------------------+
     |   startOffset     |  length   |      clientName  |    token             |
     +-------------------------------------------------------------------------+

* TRANSFER_VERSION(接口版本号)和读操作码(81)确定双方的数据通信格式
* 数据块ID(blockId)和版本号(generationStamp)确定目标数据快
* 偏移量(startOffset)和长度(length)表示要读取此数据块的数据范围

#### 应答报文

数据节点读请求应答头报文如下：

     +----------------------------------------------------------------------------------------+
     | OP_STATUS_SUCCESS  |    CHECKSUM_CRC32     |      bytesPerCheckSum     |    [offset]   |
     +----------------------------------------------------------------------------------------+

* 第一个字段表示返回码，OP_STATUS_SUCCESS表示成功
* CHECKSUM_CRC32表示为CRC的校验类型
* bytesPerCheckSum：校验块长度，默认为512字节

应答头报文后接着为多个应答数据包报文，数据包报文包含了实际的校验数据和文件数据。
此数据包格式既应用于读数据也应用于写数据。

     +-----------------------------------------------------------+
     | packagelen  |    offset     |      seqno     |    tail    |
     +-----------------------------------------------------------+
     |   len       |  checksum ... |       data...  |   
     +----------------------------------------------+

当读取的数据量较大时，需发送多个数据包。

* packagelen：包长度，通过此字段，在TCP流上可以分帧。
* offset：数据在数据块的偏移量。
* seqno：顺序号
* tail：是否为读操作响应的最后一个数据包 
* len：数据长度
* checksum：校验数据
* data：文件数据

从数据节点上读取数据时，将使用基于TCP流的三种格式的数据包，分别为度请求包、读应答头包和数据包等。
读请求的应答需要处理数据及其校验信息的对应关系。

### 写数据

写/追加数据首先需要建立数据流管道，采用数据流管道的目的是：降低单个节点的数据流带宽，将带宽分布到各个数据节点上。

             Data Package               Data Package               Data Package
	Client <--------------> Datanode1 <--------------> Datanode2 <--------------> Datanode3 
	              ACK                       ACK                        ACK

假设客户端写数据的副本数为3，则写数据的流程为：

* Client将数据发送到第一个数据节点
* 第一个数据节点保存数据，同时将数据发送到第二个节点
* 第二个节点保存数据，同时将数据发送到第三个节点
* 第三个节点接收数据，向第二个节点发送确认包
* 第二个节点接收第三个节点的确认包，同时携带自己的确认包，发送给第一个节点
* 同理，第一个节点组合后面两个节点的确认包和自己的确认包发送给Client

相对于客户端同时向多个节点写数据的方式，数据流管道能够有效的对数据进行分流。

数据节点写请求报文：

     +------------------------------------------------------------------------------+
     | TRANSFER_VERSION   |    80         |      blockId     |    generationStamp   |
     +------------------------------------------------------------------------------+
     |   pipelineSize     |  isRecovery   |      client      |    hasSrcDataNode    |
     +------------------------------------------------------------------------------+
     |   srcDatanode...   |   numTargets  |  targets...      |      tocken          |   
     +------------------------------------------------------------------------------+  
     |  dataChecksum      |
     +--------------------+  

* 前4个字段的含义与读请求报文一致，写请求的操作码为80，读请求的操作码为81.
* pipeline：数据流管道的大小，即此管道的数据节点数目
* isRecovery：表示此写操作是不是错误恢复过程的一部分
* 若数据源是某个客户端，client为客户端名字
* 若客户端为空，则hasSrcDataNode为1
* srcDatanode为数据源节点信息
* numTargets：数据目标节点个数，当数据节点为管道中的最后一个节点时，numTargets=0
* targets：目标数据节点信息
* dataChecksum：数据校验方式，包括校验类型和校验块大小。

数据节点请求应答报文：

     +------------------------------------------------------------------------------+
     | OP_STATUS_SUCCESS   |   firstBasLink...                                      |
     +------------------------------------------------------------------------------+

* OP_STATUS_SUCCESS：成功返回码，若为OP_STATUS_ERROR，firstBasLink提供管道流中第一个出错的数据节点地址信息

写请求发送后，请求放将发送一个或多个写数据包，包含写数据的内容。其中包的格式与读数据包的格式一致。

客户端通过数据流管道发送数据，管道上的数据节点接收数据并写入磁盘后，需要给上流节点发送确认包，这些确认包从最后一个节点出发，逆流而上，直到数据源。数据源既可以是客户端也可以是数据节点。

数据流管道是HDFS针对海量数据写的优化。

在HDFS中，除了数据读写操作接口外，还有数据块复制，数据块替换和读数据校验信息等基于TCP流的接口，在此，将不一一介绍。