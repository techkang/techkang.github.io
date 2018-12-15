---
layout:     post  
title:      collections 学习笔记    
subtitle:   学无止境    
date:       2018-12-15  
author:     techkang  
header-img: img/post-bg-unix-linux.jpg  
catalog: true  
tags:  
    - python    
---  

## 介绍

今天在看一份代码时，该代码调用了 collections 模块中的 defaultdict 类，该类在访问字典里没有的对象时，会自动创建一个项，瞬间觉得想见恨晚，决定学习一下 collections 模块。

## 准备阶段

在谷歌上直接搜索 collections python，直接点击了第一个结果，但看了一会，发现这是 python2 里 collections 的教程.....于是跳转到 python3 对应的教程，正式开始学习。

## ChainMap

如图该类的名字，ChainMap 的作用是把多个 map 拼接起来，这样做比多次 update() 速度快，而且可以用来模拟嵌套作用域。代码示例

    a=ChainMap({1:1,2:2},{2:3,3:4})

    In [21]: a[2] # 按顺序检索，返回第一个检索到的值
    Out[21]: 2

    In [22]: a.maps # 可以看到，原字典得以保留。实际上 ChainMap 是创建了两个引用。
    Out[22]: [{1: 1, 2: 2}, {2: 3, 3: 4}]

    In [23]: a.new_child() # A call to d.new_child() is equivalent to: ChainMap({}, *d.maps).
    Out[23]: ChainMap({}, {1: 1, 2: 2}, {2: 3, 3: 4})

    In [25]: a.new_child().parents # A reference to d.parents is equivalent to: ChainMap(*d.maps[1:])
    Out[25]: ChainMap({1: 1, 2: 2}, {2: 3, 3: 4})

    In [51]: a=ChainMap({1:1},{2:2})

    In [52]: a.update({2:3})

    In [53]: a
    Out[53]: ChainMap({1: 1, 2: 3}, {2: 2}) # 可以看到，如果调用 write 和 deletion 相关，只会在第一个 mapping 中进行操作。
    
在学习 ChainMap 时，注意到调用 new_child() 时必须加括号，而调用 parents 的时候不能加括号，查看源码可知是调用了 @property进行修饰，代码如下：

    @property
    def maps(self) -> List[Mapping[_KT, _VT]]: ...

使用 property 可以把方法变成通过属性调用，也可以方便控制赋值。这里两个 property 修饰的主要作用是控制赋值。因为在 __init__() 方法中创建了 maps，为了方便用户修改 maps，因此使用 property 去修饰。但是 parents 是不可更改的，也用 property 做控制。

通过阅读源代码，还学习到一个技巧，即使用 -> 来指定的返回值类型（经过实验，返回值类型与规定的不同也可以编译通过）。

## Counter

可以利用 Counter 类来快速方便的计数。

    In [66]: c = Counter(cats=4, dogs=8) # 这种表达方式比较方便，但有限制，即等号之前必须符合一个变量的命令方式，而输入 1=1 或是 '1'=1 都会报错。

    In [67]: c
    Out[67]: Counter({'cats': 4, 'dogs': 8})

    In [68]: c['fish'] # 查询该字典中没有的关键字不会报错
    Out[68]: 0

    In [69]: c # 查询不存在的关键字不会导致原 Counter 发生变化
    Out[69]: Counter({'cats': 4, 'dogs': 8})

    In [71]: type(Counter)==type(dict)
    Out[71]: True # Counter 是 dict 的一个子类

相对于 dict，Counter 中新实现的方法有：

    elements() # 按所计次数返回所有元素

    most_common([n])

    subtract([iterable-or-mapping]) # 直接相减，输入输出可以是负值或零

相对于 dict， Counter 中发生变化的方法有：

    fromkeys(iterable) # Counter 中没有实现该方法

推测 fromkeys 是通过 raise NotImplementedError 来实现的，然而在 PyCharm 中查看源码，发现 Counter 的实现中并没有提到 fromkeys，在 dict 中，只有 fromkeys 是用 @staticmethod 进行修饰的，然而写代码测试，子类完全可以继承父类的静态方法。创建一个 Counter 实例并调用 fromkeys，报错确实为 NotImplementedError。这时注意到，报错提到的代码文件和 PyCharm 默认打开的源文件不是同一个文件。打开抛出错误的文件，发现是 collections 的 Python 实现，之前以为 collections 的具体实现是用的 c，因此未怀疑 PyCharm 的结果，而打开该源文件，不仅有具体实现，还有注释。仔细看 PyCharm 打开的文件，发现是 .pyi 文件，检索可知 .pyi 是存根文件，类似 c 语言中的 .h 文件，但实际上不会被执行。因此，以后寻找源代码文件只能通过主动诱发错误或是使用 help 方法。

在查看源代码的过程中，发现有些函数参数列表最后一个参数是 / 。通过查询，发现该符号表示其之前的均为关键词参数，而非位置参数。例如 fun(a=1, /), 调用该函数时，如果使用 fun(a=1)， 则会报错。

另外还注意到，在 dict 中，fromkeys 是 staticmethod，而在 collections 中，变成了 classmethod。经过上网检索，发现 dict 中 fromkeys 应该也是 classmethod [https://docs.python.org/3/library/stdtypes.html?highlight=fromkeys#dict.fromkeys](https://docs.python.org/3/library/stdtypes.html?highlight=fromkeys#dict.fromkeys)。至于为什么代码中是这样，怀疑可能是一个 bug。通过 import builtins，之后利用 PyCharm 查看对应的 .pyi 文件，发现在第 659 行有注释，为 # TODO: Actually a class method (mypy/issues#328)。

Counter 还有实现了部分数学运算符。
- +：正常理解的相加
- -：计数次数相减，结果中忽略 0 和负值
- &：intersection:  min(c[x], d[x])
- |：union:  max(c[x], d[x])

## deque

双向列表，主要是可以以 O(1) 的时间复杂度在 list 开始的地方进行插入或弹出。

需要新掌握的函数有：

    extendleft(iterable)
    popleft()
    appendleft(x)

## defaultdict

主要作用是在检索未出现的元素时，赋予一个默认值。虽然该类描述最为简单，但感觉最为好用，将会简化大量计算。

## namedtuple

对于 tuple 中的每个元素，可以指定一个名称，增强可读性。在读取 csv 或是 sqlite3 的返回结果时可以考虑用该数据结构。

## OrderedDict

有序字典。需要掌握的函数有：

    popitem(last=True)
    move_to_end(key, last=True)

## UserDict， UserList, UserString

分别是对 dict, list 和 string 的包装，方便继承。因为这些类都是用 c 实现的，不能重写其默认方法。
