- 浅拷贝：就是拷贝后，没有进行真正的复制，而是复制的对象和原对象都指向同一个地址
- 深拷贝：是真正的复制了一份，复制的对象是指向了新的地址

对象的copy是浅拷贝，mutablecopy是深拷贝

容器包含对象的拷贝，无论是copy还是mutablecopy都是浅拷贝，要想实现对象的深拷贝，必须自己提供拷贝方法

### NSSring

- 对于非容器不可变对象的copy为浅拷贝，mutableCopy为深拷贝
- 浅拷贝获得的地址和原来的地址是一致的，返回的对象为不可变对象
- 深拷贝返回新的内存地址，返回对象为可变类型

### NSMutablecopy

- 对于非容器可变对象copy为深拷贝
- mutablecopy也为深拷贝
- 并且copy和mutablecopy返回对象都为可变对象

### NSArray

- copy是浅拷贝
- mutablecopy确实返回了一个新的容器，为不可变类型
- 容器里的元素仍然是浅拷贝

### NSMutableString

- copy确实返回了一个新容器，可变类型，增加值会崩溃
- mutablecopy返回一个新容器
- 容器内的元素仍然是浅拷贝