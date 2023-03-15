# 多模态大模型机器人应用-行业调研

## 技术背景

### 研究总结

2017年，Google提出了**Transformer**模型，彻底颠覆了**自然语言处理**的传统做法，在该领域大获成功。2021年，Google再次发布**Vision Transformer**工作，将Transformer首次应用于**计算机视觉**领域，撼动了卷积神经网络的统治地位。Vision Transformer将图像和自然语言“**大一统**”，使用结构类似的Transformer进行处理，这种**统一的数据特征抽取**方式让行业看到了**多模态模型**的未来。 2019年Google的自然语言Transformer工作BERT，以及2022年Meta的计算机视觉Transformer工作MAE，使用类似的方式，对原始自然语言/图像数据进行部分遮盖，通过“完形填空”任务，即预测被遮盖部分的内容，对**Transformer模型进行自监督训练**，这种自监督训练模式不再需要数据标注的工作，从而很大程度增加了可用数据量，使得模型效果提升，训练难度降低。自监督训练方式也成为了Transformer模型的训练范式。随着2021年OpenAI的工作CLIP发布，使用自然语言-图片配对的数据集，利用自然语言作为监督信号，训练视觉模型以对齐自然语言和图像。这个工作真正结合了视觉和自然语言的特征，使得图片分类预测不再是有限的集合。从CLIP开始正式开启了**自然语言-视觉的多模态大模型研究**。2022年开始，陆续有工作关注**多模态大模型在机器人领域的应用**，现有工作在机器人**规划、控制、导航**等主要任务上都进行了探索。各领域分别选择了几项重要工作在后文简述，使用流程图的形式总结每个模型中的信息流动（模型的输入和输出）：

![Untitled](./pics/Untitled.png)
<img src="pics/Untitled.png" width=45% >

黄色，蓝色，绿色分别代表模型的输入数据，使用的模型，以及输出数据

**PaLM-E（规划）**

2023年3月6日，在GPT4.0发布前一周，Google发布的多模态大模型。PaLM-E使用多模态句子作为输入，文字和图片分别使用Google最新的大模型PaLM和ViT-22B进行编码，同时整合场景内物体、状态等信息，编码后全部放入PaLM进行监督训练，监督信号是一段文本。PaLM-E工作使用了10%的机器人任务数据，90%的文本-视觉任务数据进行训练，其认为在多种任务上训练可以取得更好的迁移效果，使得模型在处理真实机器人任务的同时依然可以很好的解决文本-视觉任务。PaLM-E模型在处理机器人任务时，获取到用户使用自然语言定义的一个长期任务目标，根据当前机器人的状态和感知信息，生成逐步的低级文本指令，交由下游控制模块执行，根据每一步的执行结果及场景的变化，机器人会重新对下一步进行规划。注意，此处的下游控制模块，依然是一个多模态大模型的方法。

![Untitled](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled%201.png)

**Control Transformer（控制）**

2023年1月24日，Microsoft发布了Control Transformer，将大模型常用的自监督训练方式以及预训练-微调的训练部署方式延续到了控制任务上。预训练阶段，通过两个短期特征指标（预测下一时刻的观测/正运动学，预测上一时刻的动作/逆运动学）以及一个长期指标（随机遮盖一些观测-动作序列，进行预测）来学习观测-动作的特征。Control Transformer最终输出一个控制序列的向量，表征对输入序列中每个观测和动作的理解。在执行控制任务时，将Control Transformer输出的向量作为输入传递给一个额外的神经网络，并使用该网络来预测下一个动作，不同任务需要进行微调。该工作并没有在真实场景下进行测试，在MuJoCo仿真测试中，观测数据包括机器人的位置、速度、关节角度等信息，动作信息是机器人执行的动作（例如向前走、旋转）。

![Untitled](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled%202.png)

**Robotic Transformer（控制）**

2022年12月13日Google发布Robotic Transformer-1，框架十分简洁，将图像与文本指令抽取特征，再放入Transformer直接训练，对EverydayRobots公司机器人的机械臂状态和移动底盘状态直接进行学习。PaLM-E正是使用了这个方法作为控制模块。

![Untitled](pics/Untitled%203.png)

**LM-Nav（导航）**

2022年7月26日Google发布的第一篇应用大模型进行机器人导航的工作LM-Nav。首先使用Vision-Navigation Model（VNM，具体为ViNG方法）创建拓扑地图，由人控制机器人进行环境探索，将图片作为拓扑地图的节点，节点之间的边用Traversability函数赋予权重，该函数通过学习对从一节点导航至另一节点的时间步数，给出节点间的边长权重。完成拓扑地图构建后，导航分为两个模块：规划，利用构建好的拓扑地图进行图搜索，给出从当前位置到目标点所需要经过的路标点；控制，基于两个节点的图片，以及机器人里程计输出的两个节点间的相对位姿训练ViNG方法的控制器，从而对当前机器人位置和任一拓扑地图节点间的位姿进行预测，最后利用机器人的里程计和PD控制器完成控制。

在导航执行过程中，用户输入的完整语言导航指令被LLM（GPT-3模型）转化为一系列文本形式的landmarks，文本landmarks与拓扑图中的节点图片进行匹配，生成从机器人当前位置到每个landmark的导航路径，VNM模型输出导航路径上下一个节点到机器人当前位置的相对位姿，控制机器人前往下个节点。

从语言模型角度来看，这篇文章是多模态大模型在机器人领域应用的突破，是**LLM第一次被用在机器人导航**这一关键应用上。从机器人领域来看，本工作的工程显得格外“**复古**”，传感器使用单个RGB相机，单线激光雷达，轮式里程计，都像是十年前的机器人配置。同时，除了RGB图像外，IMU，单线激光雷达，GPS等数据均未参与多模态模型训练。使用两个拓扑图上两个RGB节点的图像直接学习相对位姿，似乎也不是一个准确高效的方法。多模态大模型想要真正在机器人领域落地，必须仔细考虑与先进传感器，尤其是**三维激光雷达的适配**。

![Untitled](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled%204.png)

**ChatGPT for Robotics（应用）**

![Untitled](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled%205.png)

2023年2月20日Microsoft发布的一篇工程性质的文章，主要提供了使用ChatGPT完成机器人任务代码开发的思路以及Prompt Engineering，用户提供高层函数库（链接真实的机器人API）以及对任务目标的描述（Prompt），ChatGPT按照给定的高层函数库生成对应代码，从而链接机器人API，并进行部署。该文章给出了使用语言大模型进行机器人开发的思路。

### 研究展望

1. 从目前研究结果来看，不论是**模型参数量增大**还是**训练数据量增大**，都会带来进一步的效果提升，目前还未看到参数量与数据量增长对模型效果提升的瓶颈。这意味着在未来关于AI通用大模型的研究会持续进展，参数量与数据量会持续增长直至达到瓶颈。增大的参数量与数据量会进一步导致大公司在**基础大模型上的垄断**地位，中小公司及独立的高校课题组由于**算力资源不足**，**数据量不足**，将越来越**难以入局基础大模型**。大公司研究院独立发文，或高校出人，大公司出算力合作发文，将成为基础大模型研究的常态。
2. **大模型在边缘的部署方式**将得到进一步研究。大模型在完成一些对响应速度要求较高的任务、需要在无网络环境或网络不稳定环境工作时，无法采用云计算+网络传输计算结果的方式。而千亿级参数的前向推理对于边缘计算平台来说挑战很大。一种可能的方式是利用训练好的大模型，在特定任务上进行fine-tuning，训练出对**特定任务有效的轻量化模型**，从而可以在边缘，算力有限的情况下进行部署。
3. 多模态数据通过各自不同的模型进行处理，转化为token形式，以预训练的语言大模型作为接口融合 **[特征抽取模型-预训练融合模型]** 的趋势已经明晰。**预训练融合模型如何有效进行自监督训练**（使用多模态句子作为输入）可能成为研究重点之一。Transformer的自监督训练方法在自然语言处理（BERT方法）与计算机视觉（MAE方法）领域都取得了很大成功，对句子挖空进行预测或对图片遮盖一部分进行预测，自监督的训练方式使得数据收集变得简单，不需要额外进行人工标注，这使得数据量增大，模型有效性提升。自监督训练方式在多模态大模型上依然存在该优势，目前只有SMART方法在训练Control Transformer时部分使用了自监督的方式，对观测和执行的序列数据进行随机遮盖，学习被遮盖的部分。
4. 对于不同模态数据的**特征抽取模型**而言：激光雷达可以获取准确的三维点云测量数据，已成为机器人领域最不可或缺的传感器之一。如果想要将多模态大模型真正应用于机器人领域，尤其是处理机器人感知相关的任务，将**激光雷达的三维点云数据引入多模态大模型框架**处理是必由之路。从2020年开始陆续有研究使用Transformer处理点云，但大多数并未针对激光雷达点云的**稀疏特性**进行操作。目前还**没有研究将多模态大模型与激光雷达结合**，传统的**语义研究**可能会被大模型完全取代。除激光雷达外，多模态大模型对**IMU**，**GNSS**等其他传感器的适配也是必要的。

## 应用前景

![Untitled](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled%206.png)

多模态大模型，核心仍然是语言大模型。多模态数据由Transformer技术编码，以语言模型为接口统一处理，分别与用户及机器人终端进行交互。**[自然语言指令-多模态大模型-终端执行机构] 新人机交互范式**已然形成，**Language User Interface (LUI)** 的概念在沉寂多年后有再次浮出水面的趋势。

用户使用语言进行交互的延迟，大模型在边缘前向推理速度较慢，云端推理结果传递到终端的网络延迟，这些特性决定了多模态大模型只使用于**无实时性要求**、**低交互频率**的任务。在机器人相关领域，多模态大模型的潜在应用场景（物流配送、流水线作业、服务机器人、导盲机器人、自动驾驶）分析如下：

### 自动驾驶

![Cruise Self-driving Car](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled.webp)

Cruise Self-driving Car

**应用前景：**

对于自动驾驶应用场景，在规范的城市道路行车，一般而言用户只需要设置目的地即可，**并无较多人机交互需求**，多模态大模型在自动驾驶任务上的应用最重要的可能并非语言交互。对于**感知**中最重要的**定位**任务，其本质上来说就是不同**时间-空间数据的对齐**，现有工作已经很好的解决了图-文对齐的问题，只要针对**激光雷达**的Transformer方法有了进展，激光雷达-相机-IMU-GNSS等多模态数据通过大模型预测汽车位姿就指日可待了。对于**控制**任务，汽车在标准**化结构化场景**，以规范的方式运行，其actions相比于在非标准化场景运行的移动机器人或机械臂更加容易学习和预测。对于**规划及语言交互**，多模态大模型的潜在应用场景可能是非标准城市道路（越野场景等），非具体目的地指向的导航任务。如“向前走100米，经过大树后向右前方行驶”。

**限制因素：**

自动驾驶技术目前较为独立，多传感器融合的感知技术路线已经明确。Transformer技术在先进传感器数据，尤其是**在三维激光雷达点云上的有效性**仍然有待探讨。对于感知控制等**实时性要求极高**的任务，**多模态大模型在边缘平台的推理速度**仍是重要限制因素。

### 导盲机器人

![BostonDynamics Spot ARM](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled%207.png)

BostonDynamics Spot ARM

**应用前景：**

将导盲机器人独立于服务机器人之外分析，是因为多模态大模型的技术成熟将为盲人群体带来颠覆性的改变。盲人群体只能依赖语言进行交互，而多模态大模型带来的**语言-视觉-执行**机器人系统，将给盲人的生活带来极大便利。设想将多模态大模型部署在BostonDynamics Spot ARM上，他将可以感知周围的环境，用语言进行描述，例如“前面有一条马路”，“右侧是南方科技大学”，“我们还要向前走5米才能到商场的电梯”；他将可以准确的接收语言指令，使用机械臂完成“按下电梯6楼的按钮”，“拿起货架上的矿泉水”等对应的任务。

**限制因素：**

由于涉及人身安全问题，多模态大模型预测准确度，在**边缘的推理速度**；机器狗的感知能力，导航避障任务**准确度**，**续航**问题；**交互有效性**，机器人选择信息告知/自主执行 等都会成为这一应用真正落地的重要限制因素。

### 服务机器人

![Temi Robot](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled.jpeg)

Temi Robot

**应用前景：**

服务机器人的能力将得到前所未有的提升，以基于自然语言的人机交互为核心，具备多模态数据的处理能力。服务机器人将能够理解用户的语言指令，结合多模态数据感知，匹配语言指令做出规划和执行。除了传统的清洁、物体识别抓取等功能外，陪伴也将是一项重要能力，通过语音、图片的输入，对用户当前情感状态进行预测，并匹配情感状态使用不同输出模型。

**限制因素：**

交互有效性，智能化家用机器人带来的用户数据隐私问题。

### 物流配送

![菜鸟 ”小蛮驴“无人配送车](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled%201.jpeg)

菜鸟 ”小蛮驴“无人配送车

![美团 配送无人机](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled%202.jpeg)

美团 配送无人机

**应用前景：**

使用多模态模型完成OCR任务，获得目的地信息，从而进行分拣及相关路径规划。同时，在配送过程中，机器人获取的图像也将与目的地信息进行匹配，**准确定位**到小区、楼栋、单元，甚至到楼层和住户。机器人的感知、自身定位、路径规划导航等任务，与自动驾驶中的任务类似，同样可以使用多模态大模型完成。

**限制因素：**

多模态大模型在边缘平台的推理速度；配送成本的控制

### 流水线作业

![ABB IRB2400 Robot](%E5%A4%9A%E6%A8%A1%E6%80%81%E5%A4%A7%E6%A8%A1%E5%9E%8B%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%BA%94%E7%94%A8-%E8%A1%8C%E4%B8%9A%E8%B0%83%E7%A0%94%204a94df77c1c44979a0a046116481193d/Untitled%203.jpeg)

ABB IRB2400 Robot

**应用前景：**

根据目前研究成果来看，我们有理由相信流水线作业的机械臂会是多模态大模型最先落地的应用场景。利用多模态大模型对特定任务中的物体识别、机械臂控制进行学习，从而使得机械臂可以很好的完成分拣、装配等常规流水线任务。

**限制因素：**

多模态大模型**在边缘的推理速度**。

## 行业概况

当前多模态大模型仍处于基础模型研究的阶段，主要由**OpenAI (Microsoft), Google**引领，以月为单位进行**快速模型迭代**。国内的大模型研究以华为-诺亚多模态大模型，华为-循环智能-盘古大模型，澜舟科技-孟子大模型，北京智源研究院-悟道大模型等为主，可能具备一定的**中文数据优势**，但在算力、人才、研究基础上**落后**。国内外的中小企业由于算力与数据等资源不足，**没有条件入局基础模型研究**。目前多模态大模型在机器人领域尚未产生真正落地的产品，初创公司应该找准**细分市场**，随着多模态大模型的成熟，将其应用于**下游机器人任务**，重点关注**多模态大模型对各种机器人传感器数据的适配**、**多模态大模型的轻量化以在边缘计算平台部署**、**使用自然语言的人机交互方式**等。

在机器人领域，以下公司值得关注：

### 达闼科技

成立于2015年，智能机器人领域的独角兽头部企业。该公司在机器人技术上已经有较长时间的积淀，以云端大脑、海睿OS云端大脑操作系统为核心，有**人形服务机器人**、**功能型机器人**两种产品品类。该公司非常重视机器人与人工智能的结合，认为机器人需要从“功能机”向“智能机”升级，以**云计算**为核心，重点关注服务机器人，开展**多模态机器人大模型**研发。

> “2023年2月24日，**吉宏股份**与**达闼机器人**签订战略合作协议，双方拟合作共建人工智能及云端机器人研究院及合资公司，在**多模态云端大脑及操作系统**、**多模态机器人通用大模型（RobotGPT)**、双足人形机器人行业应用研发等方面展开合作，并逐步探索未来可落地实施的其他产业合作项目。”
> 

> “达闼科技在2022年7月获批 **云端机器人国家新一代人工智能开放创新平台**，该平台基于达闼自研的云端机器人的操作系统海睿OS建设。达闼科技成为国内第一家围绕人工智能和云端机器人建设新一代人工智能开放创新平台的“国家队”企业。”
> 

**历史融资状况**（信息来源 36氪创投平台）：

| 融资轮次 | 融资时间 | 融资金额 | 投资方 |
| --- | --- | --- | --- |
| B+轮 | 2021.04 | 10亿人民币 | 国盛集团 |
| B轮 | 2019.03 | 3亿美元 | 软银愿景基金，上海市人民政府，金地集团，博将资本 |
| A+轮 | 2017.11 | / | 中科招商 |
| A轮 | 2017.02 | 1亿美元 | 启明星辰，凯旋创投，软银愿景基金，中科乐创，华登国际，中关村发展集团，融诚科技，博将资本，富士康，深创投 |
| 天使轮 | 2016.05 | 3000万美元 | 软银中国资本，华登国际，富士康 |

### Everyday Robots

Everyday Robots从Google[X]孵化，成为Alphabet的子公司，然而在2023年1月，整个项目被Google裁掉，技术与人员被并入Google Research现有的项目中。Everyday Robots认为机器人需要从被预先编程好逐步走向基于数据和学习，从而智能化。他们**参与了Google多模态大模型相关的研究**，Robotic Transformer（2021年），PaLM-SayCan（2022年），PaLM-E（2023年，研发人员及机器人参与），在机器人技术与大模型结合上有较深的技术积累。然而，就是这样一家Google的独立子公司，因为**无法盈利被撤裁**，值得反思。

多模态大模型和机器人技术的结合看似已经有了很不错的成果，未来可期，但想要真正落地还需要一段时间的研发，一方面需要**通用多模态大模型**的进一步提升，另一方面需要**针对机器人相关传感器做好大模型的适配**，与前沿机器人技术结合。同时，在多模态大模型机器人应用方向上创业的公司需要找准细分领域，明确用户需求，解决好特定问题，避免在“通用性”上过度纠结，明确”**盈利**“而非”研究“为目标，使用基础大模型针对特定机器人任务进行优化即可。使用自然语言作为人机交互媒介，要避免机器人**反馈过多无效信息**，注重提升**交互体验**。
Google作为在多模态大模型-机器人领域里走在最前面的公司，其内部是否会再次孵化子公司，或有相关人员重新创业，值得关注。

除此之外，并无更多将多模态大模型应用于机器人领域的公司。总体上来说这一领域仍处于**技术储备阶段**，需要观望，但非常值得期待。

机器人领域应用外，还有一些大模型公司也值得关注，此处不再展开介绍：Boson.ai，Mu Li与其导师Alex Smola从Amazon离职，投身大模型创业，CLIPr，Twelve Labs，Lightricks，Jasper，Stability.ai，Hugging Face；小冰科技在2022年11月进行了A+轮11亿人民币的融资；澜舟科技在2023年3月进行了Pre-A+轮数亿人民币融资，进行基础大模型训练；衔远科技于2023年3月完成数亿元人民币的天使轮融资，致力于链接消费者与商品。

[相关研究](https://www.notion.so/b1a8b4c5134b4bc6b9b8c22a9030a2cf)
