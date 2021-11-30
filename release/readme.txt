1. Set the attributes of an operator by modifying the OpInfo.json.
2. Run main.py and enter the experimental parameters like opName, strategy according to the prompt.
3. Check the result csv files to get the experimental data.

//以上是文件夹内自带的(●ˇ∀ˇ●)
//以下是我对于整个release文件夹下的代码解析(可能有些理解不到位，谅解谅解！！！)


//////////////////////////////////////////////////////////
CreateCorpus.py
//////////////////////////////////////////////////////////
函数名：createCorpus(size,shape)
输入：size语料库大小、shape张量形状
过程：调用numpy.random.randn以给定的形状创建数组，数组元素符合标准正态分布
返回：装有size个数组的队列




//////////////////////////////////////////////////////////
CreateOpJson.py
//////////////////////////////////////////////////////////
这是一个创建OpInfo.json的代码块
输入：无
过程：将确定好的所有算子相关参数写入到OpInfo.json文件中
返回：无



//////////////////////////////////////////////////////////
main.py
//////////////////////////////////////////////////////////
入口函数
输入：算子名称（必须是给定的7中之一）、语料库大小、指导策略（3选1）、运行轮数或终止数
过程：根据输入的算子名称调用readOpInfo获取算子信息，随机选择一种形状，创建输入大小的语料库，根据输入的指导策略输入运行轮数或终
      止数，并交给对应的Op_execute.py中的函数运行。
返回：无


//////////////////////////////////////////////////////////
Op_execute.py
//////////////////////////////////////////////////////////
函数名：input_withDiffDype(x,dtype,opname)
输入：x语料库、dtype数据类型（float16、32、64）、opname算子名称
过程：对语料库中的所有语料全部转化为张量，根据指定的数据类型进行转化，如果conv2d和MaxPooling2D算子则需要先进行transpose转换
返回：转化为指定数据类型的语料库



函数名：tf_opWithDiffDype(dtype, opInfo)
输入：dtype数据类型（float16、32、64）、opInfo算子信息
过程：对算子欣喜进行提取，根据不同的算子名称，调用tensorflow.keras.layers中关于各算子的部分，举例tensorflow.keras.layers.conv2d根
          据算子信息中的参数生成算子实例，在这里也可以说是卷积特征层，这种特征层类似于函数，可以接受输入并按照一定规则运算并输出。
返回：算子实例



函数名：executeOp_Random(corpus, opinfo, rounds)
输入：corpus语料库，opinfo算子信息，rounds运行轮数
过程：生成两个csv文件，diffdata.csv用于存放语料库中各数据在各种精度转化下的值，count.csv用于存放误差分析结果。在每一轮运行中，调用
          getMeanDiff计算同输入最大误差和同算子最大误差，写入csv文件。
返回：无




函数名：execute_guided(corpus, opinfo, tn, strategy)
输入：corpus语料库，opinfo算子信息，tn终止条件，strategy指导策略
过程：生成两个csv文件，diffdata.csv用于存放语料库中各数据在各种精度转化下的值，count.csv用于存放误差分析结果。直到达到终止条件前，调用
          getMeanDiff中对应的策略模式分组计算同输入最大误差和同算子最大误差，写入csv文件。
返回：无




函数名：getMeandiff(x, csv_writer, j, opname, opinfo, strategy)
输入：x语料集，csv_writer文件写入入口，j行号，opname算子名，opinfo算子信息，strategy指导策略
过程：根据算子名，分别对语料库中的输入调用input_withDiffDype转换成不同精度的张量float16、32、64，调用tf_opWithDiffDype生成对应精度的
          算子实例，在高精度算子中调用低精度张量，即低精度转高精度，计算精度转换时出现的误差，高精度转换为低精度同理。对于mean策略，调用
          numpy.mean取误差均值，而对于max策略，调用numpy.max取误差最大值。
返回：mean策略，返回每组误差均值中的最大值数组。max策略，返回每组误差最大值中的最大值数组



//////////////////////////////////////////////////////////
readOpJson.py
//////////////////////////////////////////////////////////
函数名：readOpInfo(filename,opName)
输入：filename算子信息json文件，opname算子名
过程：根据算子名称调用读取filename中对应的算子信息
返回：算子信息数组




//////////////////////////////////////////////////////////
seed.py
//////////////////////////////////////////////////////////
函数名：seed_mutation(x, corpus)
输入：x原始语料库，corpus突变后的语料库
过程：对x中的输入施加1e-4,1e-6,1e-8的扰动形成新的输入加入corpus
返回：突变后的语料库
