# 性能主题

本专题针对平台二次开发、python模型开发过程中的性能优化部分做出通用性的建议。


## 模型开发
在应用场景中，模型的性能主要取决于模型推理速度，这由模型结构设计、GPU推理性能、模型量化等因素综合决定。然而，在开发过程中，依旧有诸多因素会影响性能，例如，由于推理与训练不同，可能会从文件直接获取数据，而非如训练时一样，通过封装好的`DataLoader`来读取数据。

:::info 数据加载类 `DataLoader`
TODO
:::

此外，许多模型并非简单地接受数据，有时会需要加上各类的预处理操作。尽管预处理操作的底层设计通常是高效的，但由于 Python 语言本身性能较弱以及开发者经验不足，无法编写出高效率的 Python 代码，导致程序存在性能问题。

本节聚焦 Python 语言特性，对于开发过程中的常见操作给出通用性的建议。一般来说，性能高的Python代码有如下特点：

1. **短**。通常越短的代码越快，例如简单的迭代操作可以用列表推导式完成。尽管这并非绝对，但是对于一般场景，几乎没有例外，除非你有扎实的编程基础，否则你可以认为这句话是对的。更加简单的来说，那就是代码的 **for 循环代码块** 越少，性能越好。
2. **向量化运算**。对于处理数据的场景，例如相加两个长度为3的向量 `a = np.array([1,2,3])` 和 `b = np.array([2,4,6])`，通常可以`a + b`直接相加，而非使用循环对每一个元素相加。其内涵就是向量化运算。在本质上依旧是 **for 循环** 越少，性能越好。
3. **并行**。对于多核心的CPU，并行处理可以显著加快速度。第 2 条的向量化本质就是并行。


### 数据操作部分

本节介绍一些高频的数据操作，包括numpy、torch使用时的典型操作。这些操作在日常使用时经常用到，请熟练使用。

#### 数据升维度 <Badge type="danger" text="重要" />

**1. 场景一：** 有一张(3, 512, 512)的图像，需要放入神经网络推理，但是模型接受的输入尺寸是`( b, c, h, w )`，也就是缺少了一个batch size维度。请处理这张图像，使之符合(1, 3, 512, 512)的输入尺寸。

**2. 场景二：**  有两张相同尺寸的RGB图像被读取为 `img1` 和 `img2`，请将它们组合成一个`(2, c, h, w)` 的输入。

**3. 场景二：**  有`n`张相同尺寸的RGB图像文件路径保存在一个list里，请将它们组合成`(n, c, h, w)` 的输入。
