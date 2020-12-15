### Tuple

- 元组把多个值合并成单一的复合型的值
- 元组内的值可以是任何类型，而且可以不必是同一类型

```swift
let error = (1, "没有权限")
print(error.0)
print(error.1)
```

### 元素命名

- 元组中的每一个元素可以指定对应的元素名称
- 如果没有指定名称的元素也可以使用下标的方式来引用

```swift
let error = (errorCode: 1, errorMessage: "没有权限")
print(error.errorCode)
print(error.errorMessage)
```

### Tuple修改

- 用var定义的元组就是可变元组，let定义的元组就是不可变元组
- 不管是可变元组还是不可变元组，元组在创建后就不能增加和删除元素
- 可以对可变元组的元素进行修改，但是不能改变其类型
- any类型可以改为任何类型

### Tuple分解

- 以将一个元组的内容分解成单独的常量或变量
- 如果只需要使用其中一部分数据，不需要的数据可以用下划线（_）代替

```swift
let error = (1, "没有权限")
let (errorCode, errorMessage) = error
print(errorCode)
print(errorMessage)
```



