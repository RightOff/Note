# 基础语法

## (_) 用法

（_）有多种用法，其中之一是作为一个特殊的标识符，通常用于表示临时或占位符的值。

+ 单个下划线（_）作为一个变量名：在交互式Python解释器中，单个下划线通常**用作一个临时变量**，表示上一个表达式的结果。例如：

  ```
  >>> x = 5
  >>> y = 10
  >>> x + y
  15
  >>> _
  15  # 单个下划线(_)表示上一个表达式的结果
  ```
+ 双下划线（__）作为前缀：双下划线作为前缀用于表示类的**私有成员**。这意味着这些成员只能在类的内部访问，无法从类的外部直接访问。

  ```
  class MyClass:
      def __init__(self):
          self.__private_var = 42

      def get_private_var(self):
          return self.__private_var

  obj = MyClass()
  print(obj.get_private_var())  # 输出 42
  print(obj.__private_var)  # 报错，无法直接访问私有成员
  ```
+ 单个下划线（_）作为名称：在约定上，单个下划线作为变量或方法名的前缀，表示这是一个**私有成员**，但是**这只是约定**，并没有实际的语法限制。例如：

  ```
  class MyClass:
      def __init__(self):
          self._private_var = 42

      def get_private_var(self):
          return self._private_var

  obj = MyClass()
  print(obj.get_private_var())  # 输出 42
  print(obj._private_var)  # 仍然可以访问私有成员，但不推荐这样做
  class MyClass:
      def __init__(self):
          self._private_var = 42

      def get_private_var(self):
          return self._private_var

  obj = MyClass()
  print(obj.get_private_var())  # 输出 42
  print(obj._private_var)  # 仍然可以访问私有成员，但不推荐这样做
  ```
+ 单个下划线（_）作为循环中的**占位符**：在迭代中，如果不需要使用循环变量的值，可以使用单个下划线作为占位符。例如：

  ```
  for _ in range(5):
      print("Hello")  # 循环5次，但不需要使用循环变量的值
  ```
+ 双下划线前缀和后缀（ **name** ）：在类中，双下划线前缀和后缀用于名称修饰（name mangling），用于避免类之间的命名冲突。例如：

  ```
  class MyClass:
      def __init__(self):
          self.__private_var = 42

      def get_private_var(self):
          return self.__private_var

  obj = MyClass()
  print(obj._MyClass__private_var)  # 使用名称修饰来访问私有成员
  ```

## placeholder()

用TensorFlow中的placeholder创建两个占位符，分别命名为"z"和"y"

```
z = tf.placeholder(tf.float32,name='z')
y = tf.placeholder(tf.float32,name='y')
```

## sigmoid_cross_entry_with_logits()

sigmoid交叉熵函数

```
cost = tf.nn.sigmoid_cross_entry_with_logits(logits=z,labels=y)	//这里的cost只是创建了一个计算结点，在之后的session.run()中才会传入并进行计算。
								//此外，这里的z,y可以只是占位符，即可以不传入实际的值
```

## Session()

在TensorFlow中，Session对象是用来计算计算图中的节点值的。

### Session().run()

`Session.run()`方法是用来执行计算图中的某些节点的，语法如下：

```
session.run(fetches, feed_dict=None, options=None, run_metadata=None)
```

其中：

* `fetches`：需要执行的节点或者张量，可以是单个节点或者张量，也可以是一个节点或张量的列表或字典。
* `feed_dict`：可选参数，用于指定输入到计算图中的占位符节点的值。
* `options`：可选参数，用于指定Session的选项。
* `run_metadata`：可选参数，用于记录运行的元数据。

## enumerate()

enumerate() 函数用于将一个可遍历的数据对象(如列表、元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在 for 循环当中。Python 2.3. 以上版本可用，2.6 添加 start 参数。语法：

```
enumerate(sequence, [start=0])		//start表示整个索引序列从start开始
```

* sequence -- 一个序列、迭代器或其他支持迭代对象。
* start -- 下标起始位置的值。

返回 enumerate(枚举) 对象。

## 库：Scikit-learn（sklearn）

```
pip install scikit-learn
```

```
from sklearn.metrics import precision_recall_fscore_support
```

这行代码导入了Scikit-learn（sklearn）库中的 `precision_recall_fscore_support`函数，该函数可以计算分类模型的精确率、召回率、F1值和支持度。精确率和召回率是分类模型中常用的性能指标，用于评估模型在不同类别上的分类准确性。F1值是精确率和召回率的加权平均值，支持度是每个类别在样本中出现的次数。

# pytorch语法

## unsqueeze()

用于在指定维度上增加一个维度

```
torch.unsqueeze(input, dim)	//其中input是输入张量，dim是要增加的维度的索引（从0开始）。
```

默认参数为1

# VGG训练

## 获取数据集

### 字典

```
self.name2label = {'empty': 0, 'occupied': 1}  # "类别名称": 编号,对自己的类别进行定义
```

这行代码定义了一个字典 `name2label`，用于将类别名称映射为对应的数字编号。其中，键值对的意义如下：

* 键是类别的名称，例如 'empty' 表示车位为空，'occupied' 表示车位已被占用。
* 值是该类别对应的数字编号，例如 0 表示空车位，1 表示已占用车位。

这个字典的目的是在读取数据集时，将每个数据样本的类别名称转换为对应的数字编号，方便模型训练时进行分类任务。

```
for name in sorted(os.listdir(os.path.join(root))):
    # 判断是否为一个目录
    if not os.path.isdir(os.path.join(root, name)):
         continue
    self.name2label[name] = self.name2label.get(name)  # 将类别名称转换为对应编号
```

这部分代码的作用是将数据集的类别名称转换为对应的编号。其中，类别名称和编号之间的映射关系是通过字典 `self.name2label` 来存储的。这个字典初始化的时候，将两个类别名称（'empty'和'occupied'）分别映射到编号0和1。

代码首先遍历了数据集中的所有子目录，然后判断子目录是否为目录。如果不是目录，则跳过本次循环。如果是目录，则将该目录的名称作为类别名称，并通过字典的 `get()` 方法来获取该类别名称对应的编号，然后将类别名称和编号之间的映射关系存储到字典 `self.name2label` 中。

这样做的目的是方便后面在处理数据集时，将类别名称转换为对应的编号。

```
model_ft = models.vgg16(pretrained=True)

# 只微调全连接层，则冻结所有卷积层的参数
for name, param in model_ft.named_parameters():
    if "classifier" not in name:
        param.requires_grad = False
```

其中，`named_parameters()` 函数返回一个迭代器，包含了模型中的所有参数，同时还包括每个参数的名称。在上面的代码中，我们遍历了模型中的所有参数，对于不在 "classifier" 中的参数，即卷积层的参数，将其 `requires_grad` 属性设置为 `False`，从而冻结卷积层的参数。而对于全连接层的参数，由于它们在 "classifier" 中，因此不会被冻结。

```
# 替换最后一层全连接层，使得输出为 num_classes
num_ftrs = model_ft.classifier[-1].in_features
model_ft.classifier[-1] = nn.Linear(num_ftrs, num_classes)
```

这段代码用于替换模型的最后一层全连接层，将其输出维度改为 `num_classes`，以适应分类任务的需要。

具体而言，这段代码首先通过 `model_ft.classifier[-1]` 访问模型的最后一层全连接层，得到其输入特征维度 `in_features`，然后创建一个新的全连接层 `nn.Linear(num_ftrs, num_classes)`，其中 `num_ftrs` 即为 `in_features`，`num_classes` 则为分类任务的类别数。

最后，将新的全连接层赋值给模型的最后一层，即 `model_ft.classifier[-1] = nn.Linear(num_ftrs, num_classes)`，完成替换操作。

### 进度条显示

```
# 迭代数据，使用 tqdm 函数包装 dataloaders[phase]
    for inputs, labels in tqdm(dataloaders[phase], desc=phase, leave=False):
        inputs = inputs.to(device)
        labels = labels.to(device)
```

在循环中，将 `dataloaders[phase]` 包装为 `tqdm` 函数后，使用 `desc` 参数来设置进度条的描述文字，使用 `leave` 参数来设置进度条是否保留在控制台中，这里设置为 `False` 表示训练完成后进度条消失。

### 指标计算

```
# 统计损失和正确率
running_loss += loss.item() * inputs.size(0)
running_corrects += torch.sum(preds == labels.data)
```

这段代码的作用是统计模型在一个batch中的损失和正确率。其中 `running_loss` 是累加器，用来累加所有batch的损失，`running_corrects` 是累加器，用来累加所有batch中的正确分类样本的数量。

**损失计算**：在每个batch中，我们通过 `loss.item() * inputs.size(0)` 来计算这个batch的损失，并将其累加到 `running_loss` 中。`inputs.size(0)` 是这个batch中样本的数量，`loss.item()` 是这个batch的损失。因此，`loss.item() * inputs.size(0)` 就是这个batch的总损失。

在深度学习中，通常使用小批量随机梯度下降（mini-batch stochastic gradient descent）来优化模型。对于一个小批量的数据，损失值的计算是通过所有样本的损失值的平均值来计算的，而不是所有样本的损失值的总和。因此，对于一个小批量的数据，损失值需要乘以这个小批量的大小，以得到整个小批量的损失值。

**正确率计算**：我们使用 `torch.sum(preds == labels.data)`，其中 `preds` 是模型对这个batch中每个样本的预测标签，`labels.data` 是这个batch中每个样本的真实标签。`preds == labels.data` 会返回一个长度为batch大小的布尔值向量，其中值为 `True` 的位置表示预测正确的位置。因此，`torch.sum(preds == labels.data)` 就是这个batch中预测正确的样本数。

最终，`running_loss` 和 `running_corrects` 的值会被用来计算整个数据集上的平均损失和正确率。

### 计算指标函数

```
def calculate_metrics(preds, labels):
    # torch.max()函数返回给定维度上输入张量最大值和对应的索引，这里指定维度为 1，也就是在每个样本的输出结果中选择最大值和对应的索引
    _, preds = torch.max(preds.data, dim=1)
    preds = preds.cpu().numpy()    # 将PyTorch张量转换为NumPy数组
    labels = labels.cpu().numpy()
    precision, recall, f1, _ = precision_recall_fscore_support(labels, preds, average='macro')
    return precision, recall, f1
```

其中 `outputs.data` 返回模型输出结果的数据张量，不包括梯度信息。注意：一开始传过来的张量preds由于带有梯度信息，因而不能直接转换为NumPy数组。因为**带有梯度信息的张量需要保持连接到计算图中**，需要使用 `detach()`方法将其与计算图分离，然后再转换为NumPy数组。可以尝试将代码中的以下行：

```
preds = preds.detach().cpu().numpy()
labels = labels.detach().cpu().numpy()
```


```
epoch_acc = epoch_acc.item()    # Tensor类型转换为Python标量值
```
