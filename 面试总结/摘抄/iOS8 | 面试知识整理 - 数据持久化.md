#### 1. iOS中数据持久化方案有哪些？

- `NSUserDefault` 简单数据快速读写
- `Property list` (属性列表)文件存储
- `Archiver` (归档)
- `SQLite` 本地数据库
- `CoreData`

#### 2. 什么是序列化和反序列化，用来做什么

- 序列化- 把对象转化为字节序列的过程
- 反序列- 化把直接序列恢复成对象
- 作用- 把对象写到文件或者数据库中，并且读取出来

#### 3. OC中实现复杂对象的存储

- 遵循NSCoding协议，实现复杂对象的存储，实现该协议后可以对其进行打包或者解包，转化为NSDate

#### 4. SQLite 数据存储是怎么用?

- 添加SQLite动态库:
- 导入主头文件:#import <sqlite3.h>
- 利用C语言函数创建\打开数据库，编写SQL语句

#### 5. CoreData是什么?

- CoreData是iOS5之后才出现的一个框架，本质上是对SQLite的一个封装，它提供了对象-关系映射(ORM)的功能，即能够将OC对象转化成数据，保存在SQLite数据库文件中，也能够将保存在数据库中的数据还原成OC对象，通过CoreData管理应用程序的数据模型，可以极大程度减少需要编写的代码数量！

#### 6. 简单描述下客户端的缓存机制?

1. 缓存可以分为:内存数据缓存、数据库缓存、文件缓存
2. 每次想获取数据的时候
3. 先检测内存中有无缓存
4. 再检测本地有无缓存(数据库\文件)
5. 最终发送网络请求
6. 将服务器返回的网络数据进行缓存(内存、数据库、文件)， 以便下次读取

#### 7. 什么是NSManagedObject模型？

NSManagedObjcet是NSObject的子类，Core Date的重要组成部分。是一个通用类，实现了Core Date模型层所需的基本功能，用户可以通过NSManagedObjcet建立自己的数据模型。

#### 8. 说一说你对SQLite的认识

- SQLite是目前主流的嵌入式关系型数据库，其最主要的特点就是轻量级、跨平台，当前很多嵌入式操作系统都将其作为数据库首选。

#### 9. 说一说你对FMDB的认识

- FMDB是一个处理数据存储的第三方框架，框架是对sqlite的封装，整个框架非常轻量级但又不失灵活性，而且更加面向对象。
- 我们知道直接使用libsqlite3进行数据库操作其实是线程不安全的，如果遇到多个线程同时操作一个表的时候可能会发生意想不到的结果。为了解决这个问题建议在多线程中使用FMDatabaseQueue对象，相比FMDatabase而言，它是线程安全的。
- 将事务放到FMDB中去说并不是因为只有FMDB才支持事务，而是因为FMDB将其封装成了几个方法来调用，不用自己写对应的sql而已。其实在在使用libsqlite3操作数据库时也是原生支持事务的（因为这里的事务是基于数据库的，FMDB还是使用的SQLite数据库），只要在执行sql语句前加上“begin transaction;”执行完之后执行“commit transaction;”或者“rollback transaction;”进行提交或回滚即可。另外在Core Data中大家也可以发现，所有的增、删、改操作之后必须调用上下文的保存方法，其实本身就提供了事务的支持，只要不调用保存方法，之前所有的操作是不会提交的。在FMDB中FMDatabase有beginTransaction、commit、rollback三个方法进行开启事务、提交事务和回滚事务。

#### 10. 什么是沙盒机制?

- 每个iOS程序都有一个独立的文件系统（存储空间），而且只能在对应的文件系统中进行操作，此区域被称为沙盒。应用必须待在自己的沙盒里，其他应用不能访问该沙盒。

#### 11. 沙盒目录结构是怎样的？

沙盒结构

- Documents：常用目录，iCloud备份目录，存放数据,这里不能存缓存文件,否则上架不被通过
- Library
    - Caches：存放体积大又不需要备份的数据,SDWebImage缓存路径就是这个
    - Preference：设置目录，iCloud会备份设置信息
- tmp：存放临时文件，不会被备份，而且这个文件下的数据有可能随时被清除的可能

#### 12. 使用 NSUserDefaults 时，如何处理布尔的默认值？(比如返回 NO，不知道是真的 NO 还是没有设置过)

- 如果使用 `- (void)setBool:(BOOL)value forKey:(NSString *)defaultName;`方法,来进行存储,就可以获取到正确的 bool 值
- 如果使用 `- (void)setObject:(nullable id)value forKey:(NSString *)defaultName;`,需要在获取到值后在转为 bool类型

#### 13. 代码题目分析,打印结果是什么?



```objectivec
NSUserDefaults *userdefault = [NSUserDefaults standardUserDefaults];
BOOL flag = NO;
[userdefault setObject:@(flag) forKey:@"flag"];

if ([userdefault objectForKey:@"flag"]) {
    BOOL eq = [userdefault objectForKey:@"flag"];
    if (eq) {
        NSLog(@"a");
    }else{
        NSLog(@"b");
    }
}else{
    BOOL eq = [userdefault objectForKey:@"flag"];
    if (eq) {
        NSLog(@"c");
    }else{
        NSLog(@"d");
    }
}
```

打印结果 a
分析: 包装成 oc 对象,OC对象有值,转 bool 都是 yes

#### 11. 如果后期需要增加数据库中的字段怎么实现，如果不使用CoreData呢？

- 编写SQL语句来操作原来表中的字段
- 增加表字段：ALTER TABLE 表名 ADD COLUMN 字段名 字段类型;
- 删除表字段：ALTER TABLE 表名 DROP COLUMN 字段名;
- 修改表字段：ALTER TABLE 表名 RENAME COLUMN 旧字段名 TO 新字段名;

#### 14.FMDB使用 线程与事务

1. FMDatabaseQueue 使用该类保证线程安全,串行队列
2. 事物是一个并发控制的基本单元，所谓的事务，它是一个操作序列，这些操作要么都执行，要么都不执行，它是一个不可分割的工作单位。

#### 15.xml 和 json 区别

- XML的优点
    格式统一，符合标准； 容易与其他系统进行远程交互，数据共享比较方便。
- XML的缺点:
    XML文件庞大，文件格式复杂，传输占带宽；服务器端和客户端都需要花费大量代码来解析XML，导致服务器端和客户端代码变得异常复杂且不易维护；客户端不同浏览器之间解析XML的方式不一致，需要重复编写很多代码；服务器端和客户端解析XML花费较多的资源和时间。
- JSON的优点：
    数据格式比较简单，易于读写，格式都是压缩的，占用带宽小；易于解析，客户端JavaScript可以简单的通过eval()进行JSON数据的读取；支持多种语言，包括ActionScript, C, C#, ColdFusion, Java, JavaScript, Perl, PHP, Python, Ruby等服务器端语言，便于服务器端的解析；
- JSON的缺点:
    没有XML格式这么推广的深入人心和喜用广泛，没有XML那么通用性；JSON格式目前在Web Service中推广还属于初级阶段。

#### 17.什么是事务?

- 作为单个逻辑工作单元执行的一系列操作，而这些逻辑工作单元需要具有原子性，一致性，隔离性和持久性
- 是并发控制的基本单元。所谓的事务，它是一个操作序列，这些操作要么都执行，要么都不执行，它是一个不可分割的工作单元。例如，银行转账工作：从一个账号扣款并使另一个账号增款，这两个操作要么都执行，要么都不执行。所以，应该把它们看成一个事务。
- 事务是一种机制，用于维护数据库的完整性

#### 18. 熟悉常用SQL语句



```jsx
create database name
drop database name
alter table name add column col type
select * from table1 where col=value
select count as totalcount from table1
select sum(field1) as sumvalue from table1
'insert into table1 (field1,field2) values(value1,value2) '
delete from table1 where something
update table1 set field1=value1 where field1 like ’%value1%'
```

参考：[http://www.cnblogs.com/acpe/p/4970765.html](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.cnblogs.com%2Facpe%2Fp%2F4970765.html)

#### 19.当数据库中的某项数据未 null 时候,通过FMDB获取的数据为

```
[NSNull null]
```



作者：Leon_520
链接：https://www.jianshu.com/p/c8a39b53179e
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。