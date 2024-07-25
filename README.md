# **Data**-compress
An internship program for applying for a PhD in Computer Science at HUST



### 第一周进度汇报(7.10-7.16)

#### 一、主要工作

本周的主要工作可以分为阅读文献和整体项目思路规划，本周阅读了6篇课题相关的文献，可以主要分为两个方向，一个是以CompressDB、TADOC为首的利用层次结构、DAG等规则方式实现对数据的压缩，一个方向是以Lepton为首的算术编码替换了JPEG文件中的Huffman编码、利用Huffman交接词实现对图片数据的压缩。自己也学习了现在主流的压缩方法如ZSTD、LZ4等，对项目的实现方法有了初步构想。学习内容在[wushirenhk/Data-compression: An internship program for applying for a PhD in Computer Science at HUST (github.com)](https://github.com/wushirenhk/Data-compression)



1.阅读文献

CompressDB: Enabling Efficient Compressed Data Direct Processing for Various Databases

Efficient Document Analytics on Compressed Data: Method, Challenges, Algorithms, Insights

POCLib: A High-Performance Framework for Enabling Near Orthogonal Processing on Compression

TADOC: Text Analytics Directly on Compression

The Design, Implementation, and Deployment of a System to Transparently Compress Hundreds of Petabytes of Image Files for a File-Storage Service

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

​	我的个人设想是，采用类似ZSTD的方式但是解决解决压缩和解压缩时间上的性能问题，大致的思路是图片经过哈夫曼解码后再次通过ANS编码和算数编码得到压缩的文件，压缩文件经过逆过程仍能得到无损的图像。但是这个过程速度可能不高，为了解决性能的开销，我想采取多线程并行化的方法减小性能开销，其间利用Lepton中提出的Huffman交接词。整个算法的核心最后打包成库函数用于在ARM平台上进行测试，主要测试的指标有压缩比、压缩性能、压缩时间，基准平台初步规划是FUSE。

![image-20240717110541814](C:\Users\hu'kai\AppData\Roaming\Typora\typora-user-images\image-20240717110541814.png)



![image-20240717104730119](C:\Users\hu'kai\AppData\Roaming\Typora\typora-user-images\image-20240717104730119.png)



#### **五、下周计划**

​	初步实现项目的代码，尝试跑通写完的模块的代码，以及运行成功基准平台的测试样例。





### 第一周进度汇报(7.17-7.24)

#### 一、主要工作

本周的主要工作可以分为代码实现和文献阅读。本周在原有方向上继续阅读了四篇文献。按照上周的规划，选择大致的思路是图片经过哈夫曼解码后再次通过ANS编码和算数编码得到压缩的文件，压缩文件经过逆过程仍能得到无损的图像，目前完成了部分编码和压缩的工作，卡在了多线程处理的部分，大约估计完成了40%左右的内容，基准平台部分完成了FUSE文件操作系统的取文件属性、打开目录、读文件、打开文件部分，压缩和解压缩操作还未实现。这周整体感觉学习的内容还是很多，很多地方的操作都不是非常清楚，都需要自己看教程和博客，整体进度是慢于第一周的进度安排的，希望下一周能多补一些进度。

代码和学习内容在[wushirenhk/Data-compression: An internship program for applying for a PhD in Computer Science at HUST (github.com)](https://github.com/wushirenhk/Data-compression)

 

阅读文献

Context adaptive thresholding and entropy coding for very low complexity jpeg transcoding

Practical Learned Lossless JPEG Recompression with Multi-Level Cross-Channel Entropy Model in the DCT Domain

 

项目学习

Brunsli

[google/brunsli: Practical JPEG Repacker (github.com)](https://github.com/google/brunsli)

 

CMIX

https://www.byronknoll.com/cmix.html



#### 二、文献阅读

**1.Context adaptive thresholding and entropy coding for very low complexity jpeg transcoding**

一、主要工作

在本文中，我们提出了显著降低存储需求的技术，同时对质量和访问延迟的影响最小。

In this paper we propose techniques to decrease significantly the storage requirements, with minimum impact on quality and access latency.

二、实现思路

可以考虑两种类型的重新编码方法，即i)与JPEG密切相关的技术，通常提供无损性能，例如JPEG Progressive, JPEG Arithmetic或PackJPG; ii)基于最近开发的编解码器的替代编码技术，例如WebP或JPEG2000。

Two types of re-encoding approaches could be considered, namely, i) techniques closely associated to JPEG, usually providing lossless performance, e.g., JPEG Progressive, JPEG Arithmetic or PackJPG, ii) alternative coding techniques based on more recently developed codecs, such as WebP or JPEG2000

 

为了满足系统设计约束，ROMP使用从图像集合生成的大量熵编码表，其中每个表都针对特定上下文进行了优化。

In order to satisfy the system design constraints, ROMP makes use of a large set of entropy coding tables generated from a collection of images, where each of the tables is optimized for a specific context.

 

关键的区别在于，通过使用大量固定的预先计算表(这增加了编解码器的内存需求)来利用这些依赖关系，我们的复杂性明显低于竞争方法。

The key difference is that by exploiting these dependencies using a large number of fixed pre-computed tables (which increases memory requirements for the codec) we have significantly lower complexity than competing approaches.

 

L-ROMP有损压缩的部分未做阅读

 

三、实验结果

ROMP和L-ROMP，它们在Facebook的大型图像语料库上产生感知无损压缩，增益为15-28%。

ROMP and L-ROMP, that produce perceptually lossless compression with gains of 15-28% on a large corpus of images from Facebook.

 

**2.Practical Learned Lossless JPEG Recompression with Multi-Level Cross-Channel Entropy Model in the DCT Domain**

一、主要工作

我们提出了一种基于深度学习的JPEG再压缩方法，该方法在DCT域上运行，并提出了一个多级跨通道熵模型来压缩信息量最大的Y分量。

we propose a deep learning based JPEG recompression method that operates on DCT domain and propose a Multi-Level Cross-Channel Entropy Model to compress the most informative Y component.

 

考虑到存储服务的再压缩需求，对JPEG图像进行进一步压缩的方法有Lepton[21]、JPEG XL[6,7]、CMIX[1]等。

Considering the recompression needs of storage service, there exist several methods on further compression of JPEG images, e.g. Lepton [21], JPEG XL [6, 7], and CMIX [1].

 

二、实现思路

跨色相关性可以隐式(通过共享超先验)和显式(通过熵参数网络)建模。

Cross-color correlation can be modeled both implicitly (through shared hyperprior) and explicitly (through the entropy parameters network).

 

使用我们的学习分布作为概率模型的算术编码[45]所能达到的期望码长由交叉熵给出。

The expected code length arithmetic coding [45] can achieve, using our learned distribution as its probability model, is given by the cross entropy

 

三、实验结果

我们提出了一种新的多级跨通道熵模型，用于现有JPEG图像的无损再压缩，该模型在Kodak, DIV2K, CLIC上达到了最先进的性能。移动和CLIC。专业，具有合理的运行速度。我们还表明，我们用质量等级75训练的方法可以很好地推广到其他质量等级，除了像95这样的非常高的质量。据我们所知，这是第一个针对JPEG图像无损再压缩的学习方法。对于未来的工作，我们将在非常高的质量水平上探索通用性。

We propose a novel Multi-Level Cross-Channel entropy model for lossless recompression of existing JPEG images, which achieves state-of-the-art performance on Kodak, DIV2K, CLIC.mobile and CLIC.pro and has reasonable running speed. We also show that our method trained with quality level 75 can generalize well on other quality levels except very high quality like 95. To the best of our knowledge, this is the first learned method targeting lossless recompression of JPEG images. For future work, we will explore the generalizability on very high quality levels.

 

***\*Lepton\****

Horn等人提出的Lepton[21]主要关注熵模型和符号表示的优化，在对JPEG进行无损再压缩后，其存储空间减少了22%。而不是使用霍夫曼编码，Lepton使用更高效的算术编码[45]。此外，结合一元、符号和绝对值，Lepton中的表示方法优于纯一元和固定长度的二进制补码等其他编码。

Lepton [21] proposed by Horn et al, which achieves 22% storage reduction after recompressing JPEG losslessly, mainly focuses on optimization of entropy model and symbol representations. Instead of using Huffman coding, Lepton uses more efficient arithmetic coding [45]. Further more, combining unary, sign and absolute value, the representation method in Lepton outperforms other encodings like pure unary and two’s complement with fixed length.

 

***\*JPEG XL\****

JPEG XL[6,7]是一种通用的压缩方法，支持无损和有损压缩。对于现有的JPEG图像，还支持将它们无损地转码为JPEG XL。JPEG XL通过将8 × 8 DCT扩展到可变大小的DCT来实现更好的压缩比，可变大小的DCT允许块大小为8,16或32之一。此外，JPEG XL使用非对称数字系统[15]代替霍夫曼编码。JEPG XL中的量化矩阵可以局部缩放，而不是全局使用固定的量化矩阵，以更好地适应不同区域的复杂性。与JPEG中原始的DC系数预测模式相比，JPEG XL支持8种模式，将选择误差最小的模式。

JPEG XL [6, 7] is a versatile compression method supporting both lossless and lossy compression. For existing JPEG images, losslessly transcoding them to JPEG XL is also supported. JPEG XL achieves better compression ratio by extending the 8 × 8 DCT to variable-size DCT which allows block size to be one of 8, 16 or 32. Besides, JPEG XL uses Asymmetric Numeral Systems [15] in place of Huffman coding. Instead of using fixed quantization matrix globally, the quantization matrix in JEPG XL can be scaled locally to better accommodate the complexity in different areas. Compared with the primitive DC coefficient prediction mode in JPEG, JPEG XL supports eight modes and will choose the mode producing the least amount of error.

#### ***\*CMIX\****

CMIX[1]是一个通用的无损数据压缩程序，旨在以高CPU/内存使用率为代价优化压缩比。它在几个压缩基准测试中实现了最先进的结果。CMIX使用独立模型的集合来预测输入流中每个位的概率。使用上下文混合算法将模型预测组合成单个概率。

上下文混合器的输出使用一种称为二次符号估计(SSE)的算法进行细化。CMIX可以无损压缩所有数据文件，包括JPEG图像。

CMIX [1] is a general lossless data compression program aimed at optimizing compression ratio at the cost of high CPU/memory usage. It achieves state-of-the-art results on several compression benchmarks. CMIX uses an ensemble of independent models to predict the probability of each bit in the input stream. The model predictions are combined into a single probability using a context mixing algorithm.

 

The output of the context mixer is refined using an algorithm called secondary symbol estimation (SSE). CMIX can compress all data files losslessly, including JPEG images.

 

#### 三、遇到的问题

1.目前能理解顶会论文实现的代码思路，但是自己完成还是有一定难度，选择的解决方案是多阅读文献并寻找有开源项目代码的文章精度，多积累存储相关的内容和代码。

2.FUSE、MooseFS等分布式或文件用户式系统自己不熟悉，下载部署就耗费了很多时间，这个方面需要多做积累。



#### 四、进一步设想

目前是基于基于Brunsli 项目的编解码框架，大体上实现的差不多，这周更为清晰的设想是，二次压缩的方案是基于谷歌的大致思路是基于Brunsli 项目的编解码框架，在 brn 文件的编码阶段将编码过程中得到的有用信息通过哈夫曼切换词存储到 brn 文件中，在 brn 解码过程中每个线程可以通过哈夫曼切换词获取必要的编码信息，实现多线程并行编码。目前并行编码的方式我只想到这一种，但是我仍在思考是否有其他并行编码的方式，如果合适再进行添加。



#### 五、下周规划

作为基准FUSE平台完成压缩和解压缩的操作并跑通，记录压缩比、压缩性能、压缩时间，二次压缩图片算法的并行编码按照规划完成，预计能完成。