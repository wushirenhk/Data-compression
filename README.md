# **Data**-compress
An internship program for applying for a PhD in Computer Science at HUST



### 第一周进度汇报(7.10-7.16)

#### 一、主要工作

本周的主要工作可以分为阅读文献和整体项目思路规划，本周阅读了7篇课题相关的文献，可以主要分为两个方向，一个是以CompressDB、TADOC为首的利用层次结构、DAG等规则方式实现对数据的压缩，一个方向是以Lepton为首的



1.阅读文献

CompressDB: Enabling Efficient Compressed Data Direct Processing for Various Databases

Efficient Document Analytics on Compressed Data: Method, Challenges, Algorithms, Insights

POCLib: A High-Performance Framework for Enabling Near Orthogonal Processing on Compression

TADOC: Text Analytics Directly on Compression

The Design, Implementation, and Deployment of a System to Transparently Compress Hundreds of Petabytes of Image Files for a File-Storage Service

Practical Learned Lossless JPEG Recompression with Multi-Level Cross-Channel Entropy Model in the DCT Domain

LZ4 Compression Algorithm on FPGA



项目学习

ZSTD

[facebook/zstd: Zstandard - Fast real-time compression algorithm (github.com)](https://github.com/facebook/zstd)

LZ4

[lz4/lz4: Extremely Fast Compression algorithm (github.com)](https://github.com/lz4/lz4)

Brunsli

[google/brunsli: Practical JPEG Repacker (github.com)](https://github.com/google/brunsli)



#### 二、学习到的核心点

**1.CompressDB: Enabling Efficient Compressed Data Direct Processing for Various Databases**

**一、主要工作**

1.我们直接在压缩数据上开发高效的数据操作，例如插入、删除和更新。

We develop efficient data manipulation operations, such as insert, delete, and update, directly on compressed data.

除了之前的随机访问支持外，我们还启用了数据查询和数据操作。

Along with previous random access support, we enable both data query and data manipulation.

 

2.我们开发了CompressDB，一个集成到文件系统中的存储引擎。

We develop CompressDB, a storage engine that is integrated into file systems.

CompressDB可以在不修改数据库的情况下无缝支持各种数据库系统。

CompressDB can support various database systems seamlessly without modifying the databases.

 

3.我们将操作下推到存储系统，避免了不必要的数据在内存和磁盘之间移动，从而提高了压缩数据的处理效率。

We enable operation pushdown to storage systems, which avoids unnecessary data movement between memory and disks, thus improving processing efficiency on compressed data.

 

**二、实现思路**

由于大数据通常存储在磁盘中，因此我们的想法是基于规则压缩在存储层对压缩后的数据进行随机更新。

Since large data are usually stored in disk, our idea is to develop random update over compressed data in storage layer based on rule compression.

在元素级别，我们引入了数据洞的概念，以允许在大数据块中进行更新。

in element level, we introduce the concept of data holes to allow updates in large data blocks.

在规则层，我们开发了哈希和计数数据结构，以有效地定位规则。

In rule level, we develop hashing and counting data structures for efficiently locating rules.

在DAG级别，我们限制了DAG的深度，以将规则拆分和合并的成本保持在一个小范围内。

In DAG level, we limit the depth of the DAG to retain the cost of rule split and merge within a small range.



**2.The Design, Implementation, and Deployment of a System to Transparently Compress Hundreds of Petabytes of Image Files for a File-Storage Service**

**一、主要工作**

我们报告了我们使用另一种技术的经验:基于统计模型的特定于格式的透明文件压缩，该模型经过调优，可以在大型语料库上表现良好。

We report on our experience with a different technique: format-specific transparent file compression, based on a statistical model tuned to perform well on a large corpus

 

它用一种定制的统计模型取代了最低层的基线JPEG图像——无损霍夫曼编码——我们对该模型进行了调整，使其在存储在Dropbox中的大量JPEG图像语库中表现良好。

called Lepton, that replaces the lowest layer of baseline JPEG images— the lossless Huffman coding—with a custom statistical model that we tuned to perform well across a broad corpus of JPEG images stored in Dropbox.

 

**二、实现思路**

核心工具：Lepton作为一个独立的工具，可以对基线JPEG文件执行往返压缩和解压缩。它是开源软件，可以在多种操作系统上构建和运行。

特定设计约束：Lepton设计时考虑了分布式网络文件系统中实时压缩和解压缩的特定约束，包括跨独立块的分布、块内并行解码和流式处理。

JPEG压缩概述：JPEG图像文件由头部信息和图像数据（扫描）组成。Lepton使用现有的无损技术压缩头部，并使用算术编码压缩图像数据。

算术编码：Lepton用算术编码替换了JPEG文件中的Huffman编码，这是一种更高效的技术，可以更准确地预测系数值，从而实现更小的文件大小。

概率模型：Lepton使用一个复杂的自适应概率模型，该模型通过大量实际图像的测试开发而来。模型使用大量统计箱（bins），每个箱子跟踪特定上下文中“1”比特与“0”比特出现的概率。

上下文相关编码：Lepton的模型利用了图像中不同系数类型（如DC和AC系数）及其在块内的索引的上下文信息，以提高压缩效率。

并行解码和流式输出：Lepton能够将JPEG文件分割成独立的段，每个段由一个线程解码。这种并行化处理允许快速开始传输字节，即使整个块尚未完全解压缩。

Huffman交接词：为了支持多线程编码，Lepton在文件格式中包括了“Huffman交接词”，这允许解码器在文件中间或符号中间恢复状态。

内存分配：Lepton在处理前分配所有内存，以避免在处理输入数据时进行动态内存分配。

安全性：Lepton在读取输入数据前进入限制模式，只允许执行特定的系统调用，以增强安全性。

跨平台支持：Lepton支持Linux、MacOS、Windows、iOS、Android和Emscripten (JavaScript)平台。

部署策略：Lepton在Dropbox的生产环境中直接由后端文件服务器执行，或者在高负载时由专门的Lepton集群处理。

 

**三、实现难点**

**Round-trip transparency.**

Lepton需要确定地恢复原始文件的精确字节

Lepton needs to deterministically recover the exact bytes of the original file

 

**Distribution across independent chunks.**

Lepton必须能够解压缩JPEG文件的任何子字符串，而不需要访问其他子字符串。

Lepton must be able to decompress any substring of a JPEG file, without access to other substrings

 

**Low latency and streaming**

为了实现这一点，Lepton格式包括“霍夫曼切换词”，使解码器成为多线程的，并在请求后很快开始传输字节。

To achieve this, the Lepton format includes “Huffman handover words” that enable the decoder to be multithreaded and to start transmitting bytes soon after a request.

 

**Security**



**Memory.**

***


**四、图片压缩的常见方法**

通用熵压缩。

Generic entropy compression.

有损图像压缩。

Lossy image compression

格式感知像素精确重压缩。

Format-aware pixel-exact recompression

支持格式、保存文件的再压缩。

Format-aware, file-preserving recompression

最后一组工具可以通过往返恢复原始文件的精确字节来重新压缩JPEG文件。

A final set of tools can re-compress a JPEG file with roundtrip recovery of the exact bytes of the original file.



**3.Efficient Document Analytics on Compressed Data: Method, Challenges, Algorithms, Insights**

**POCLib: A High-Performance Framework for Enabling Near Orthogonal Processing on Compression**

**TADOC: Text Analytics Directly on Compression**

Sequitur算法：这些研究基于Sequitur算法，它是一种递归算法，可以从离散符号序列中推断出层次结构，并生成上下文无关文法（CFG），使得输出的CFG比原始数据集更紧凑。

实现思路

- **自适应遍历顺序**：根据问题和数据集的特性选择合适的遍历顺序，以提高效率。
- **数据结构设计**：设计数据结构以解决单位敏感性问题，如使用双层位图（double-layered bitmap）。
- **并行处理**：通过粗粒度并行算法和自动数据分区来克服并行处理的障碍。
- **顺序敏感性处理**：使用深度优先遍历和两级表设计来处理顺序敏感任务。



实现结果

- **CompressDirect库**：提供了一组模块，帮助开发者应用这些指南，并实现了一些常用的文档分析算法，如词频计数、排序、倒排索引等。
- **POCLib框架**：提出了一种高性能框架，支持在压缩数据上进行近正交处理。它通过一系列技术创新，如索引数据结构和算法优化，实现了对压缩数据的高效随机访问和遍历操作。





**4.熵编码/熵压缩**

​	目前主流的熵编码方式主要有哈夫曼编码和算术编码两种方式，这也是目前 JPEG 的熵编码主要方式。2014 年由 Jarek Duda 提出了非对称系统，为有限状态熵编码提供了新的编码选择。

**哈夫曼编码**

​	哈夫曼编码是速度最快的编码实现方式，也是主流 JPEG 熵编码中的一部分。哈夫曼编码的核心是根据待编码字符出现的频次生成哈夫曼树进而得到哈夫曼编码表，最后实现字符与对应的二进制编码符号间的映射。如下图 2.9 所示，哈夫曼树的建立，主要通过以下三个

​	基本环节：

​	(1)统计所有待编码字符的出现频次并根据出现频次进行升序排序生成字符频次队列。

​	(2)将字符队列前两个节点进行合并，合并后的节点的出现频次等于左右子树频次之和。

​	(3)将合并后的节点重新加入队列并重新排序，重复(2)、(3)操作直到所有字符挂载到同一棵树。



**算数编码**

​	算术编码通过一个[0,1)区间内的小数 X 实现对信源字符序列的编码，通过将每个字符按照出现的频次在[0,1)区间内划分出对应的区间，然后按照字符出现的顺序逐次乘上对应区间所占的比例不断的缩小区间，最后得到一个包含所有区间信息的浮点数，在解码时通过该浮点数所在的区间不断进行逆向的除法操作即可实现解码。算术编码的优点是他在理论上可以无限接近于熵编码的最优结果(相对于哈夫曼编码可以减少 22%左右的存储空间)，但由于算术编码需要的解码时间相对哈夫曼编码更长以及算术编码本身的专利限制，目前绝大多数的 JPEG 编码在熵编码环节使用的都是哈夫曼编码。



#### 三、产生的疑问

​	1.项目中的技术诉求是：基于ARM平台进行原型验证，备份软件已压缩数据的压缩比相比基线算法提升50%，二次压缩性能400MB/s/core。但是这里压缩文件的类型是否可以自己去确定，比如说我想完成单独针对图片的二次无损压缩？

​	2.目前一些主流的二次压缩方法如ZSTD和LZ4，它们都宣称可以实现较高的压缩比以及较好的压缩性能，其实是满足现有的技术诉求的。但是ZSTD是进行完整解压后重删压缩，时间开销大，所以这个项目可以要么1.利用类似ZSTD的方法，但需要解决压缩和解压缩时间上的性能问题，或是2.压缩数据重建，基于推测压缩函数及参数，再参数推测不准的情况下，仍能还原原始压缩数据，我的理解正确吗？

![image-20240717104419761](C:\Users\hu'kai\AppData\Roaming\Typora\typora-user-images\image-20240717104419761.png)

![image-20240717104108178](C:\Users\hu'kai\AppData\Roaming\Typora\typora-user-images\image-20240717104108178.png)



#### 四、初步设想

​	我的个人设想是，采用类似ZSTD的方式但是解决解决压缩和解压缩时间上的性能问题，大致的思路是图片经过哈夫曼解码后再次通过ANS编码和算数编码得到压缩的文件，压缩文件经过逆过程仍能得到无损的图像。但是这个过程速度可能不高，为了解决性能的开销，我想采取多线程并行的方法减小性能开销。整个算法的核心最后打包成库函数用于在ARM平台上进行测试，主要测试的指标有压缩比、压缩性能、压缩时间，基准平台初步规划是FUSE。

![image-20240717110541814](C:\Users\hu'kai\AppData\Roaming\Typora\typora-user-images\image-20240717110541814.png)



![image-20240717104730119](C:\Users\hu'kai\AppData\Roaming\Typora\typora-user-images\image-20240717104730119.png)



#### **五、下周计划**

​	初步实现项目的代码，尝试跑通代码，以及运行成功基准平台的测试样例。