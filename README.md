
# 2022人工智能安全课程作业，内容是使用SVHN数据集进行识别练习

# 使用训练/测试数据集
SVHM（Street View House Number），这是一个基于实拍图片制作而成的数字识别数据集。其风格与MNIST数据集相似，每张图像中是裁剪后获得的一个数字，并且是数字0-9相关的十分类，但整个数据集支持识别、检测、无监督三种任务。同时，虽然是实拍数据集，但SVHN识别集的图像被处理得很小（尺寸为32x32，通道为3），样本量也在10万左右，可以在CPU上实现迭代，是非常适合用来走完整流程的数据集。
（存在问题：虽然图片中数字都基本在正中央，但是数据集的分辨率很低，识别起来有一定的难度）

# 网络架构
分别基于两个很经典的架构，Vgg和残差神经网络(ResNet),利用他们比较浅但是学习能力又比较强的特点来构筑自己的架构。由于这两个原始架构是在ImageNet数据集上构建的，图像的尺寸、输入、输出都发生了变化，需要自己重新调整。自己构建的两个架构取名为MyResNet和MyVgg。

# 提前停止
在本神经网络中，规定当连续n次迭代中损失函数的减小值都低于阈值tol时，将学习率进行衰减，并且设置“连续n次”中的“n”这个超参数为5。同时，损失函数的减小值并不是在这一轮迭代和上一轮迭代中进行比较，我们需要让本轮迭代的损失与历史迭代最小损失比较，如果历史最小损失 - 本轮迭代的损失 > tol，我们才认可损失函数减小了。这种设置对于不稳定的架构不太友好，如果我们发现模型不稳定，则可以设置较小的阈值。

# 训练/测试/监控/保存权重/绘图
在函数fit_test中，整合了这些所有流程，方便调用。其中相关参数说明如下：      
    net: 实例化后的网络  
    batchdata：使用Dataloader分割后的训练数据  
    testdata：使用Dataloader分割后的测试数据  
    criterion：所使用的损失函数  
    opt：所使用的优化算法   
    epochs：一共要使用完整数据集epochs次  
    tol：提前停止时测试集上loss下降的阈值，连续5次loss下降不超过tol就会触发提前停止  
    modelname：现在正在运行的模型名称，用于保存权重时作为文件名  
    PATH：将权重文件保存在path目录下  

# 模型选择和结果评估
当所有准备工作都完成后，我们开始进入模型选择的阶段。在这里我们让两个不同的架构（MyResNet和MyVgg）分别运行5次，每次迭代3个epochs，以此来观察两个架构的稳定性及潜力。以下是在我自己电脑的GPU上分别的运行情况（前一个是MyResNe，后一个是MyVgg）： 
![image](https://user-images.githubusercontent.com/46295395/201079394-16e51127-d891-4a6a-9e4d-fa79de260d15.png)  
![image](https://user-images.githubusercontent.com/46295395/201079838-105fc207-d8ae-4679-9c1d-8f13aa72d949.png)  
从评估结果来看，虽然SVHN数据集尺寸小、数据量也不多，但要达到其benchmark上所展示的结果，还是需要一些技巧和较长的训练时间。而在现有架构上，残差网络和VGG都有各自的问题。  
——偏差：首先来评估模型在准确率上的表现。从前3个epochs的结果来看，残差网络的起点明显比VGG高很多，经过一个epoch，残差网络在训练集上基本能够达到45%以上的准确率，但VGG最高只能达到32%，大部分时候都在25%徘徊。从损失上看，VGG首次迭代时损失总是奇高无比，残差网络在训练集上的损失相对稳定。3个epochs后，几乎每轮迭代中，残差网络在训练集上能够达到的水平一路走高，基本会超出VGG大约10%以上，但测试集上的表现两者相差不多，都在75%~85%之间徘徊。现在的结果说明MyVgg类表现出来的学习能力不足，残差网络表现出来的学习能力较强，但是泛化能力上两者不分伯仲。  
——方差：残差网络在训练集上的表现基本稳定，不仅每次训练时都一路走高，并且3个epochs后得到的结果比较相似，但在测试集上就是疯狂跳舞，5次训练中有4次都出现了测试集表现“先高再低”的情况，并且都在第三个epoch处就触发了第一次"提前停止"的阈值，这可能意味着模型在测试集上的表现高度不稳定。VGG同样在训练集上一路走高，但3个epochs之后得到的准确率上下存在8%-10%的波动，这一点在测试集上也表现了出来。从结果来看，两个架构的都不太稳定，但VGG比残差网络更不稳定。
——过拟合与欠拟合：模型是否过拟合需要在大量迭代之后才能观察出来，因为训练前期训练集与测试集的损失都很高，即便训练集的损失比测试集的损失低一些，也不能说明模型过拟合，因此通常在3个epochs下是看不出什么趋势的。
——效率：从运行结果中时间的记录来看，残差网络与VGG在GPU上的计算速度基本相当，两者差异不大。

# 模型调优/参数调整



