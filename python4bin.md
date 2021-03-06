
# 文本和字节序列
- 人类使用文本，计算机使用字节序列.
- Python 3 明确区分了人类可读的文本字符串和原始的字节序列.
- “字符串”是个相当简单的概念：一个字符串是一个字符序列.
    - 在 2015 年，“字符”的最佳定义是 Unicode 字符.


- 把字符的标识(码位)转换成字节序列的过程是编码；把字节序列转换成码位的过程是解码.
    - 把字节序列想成计算机理解的晦涩难懂的密码，把Unicode 字符串想成人类可读的文本
    - 那么把字节序列变成字符串就是解码(decode)
    - 把字符串变成字节序列就是编码(encode)


```python
s = 'café'
len(s) # 'café' 字符串有4个Unicode字符.
```




    4




```python
b = s.encode('utf8') # 使用UTF-8把str对象编码成bytes对象.
b # bytes 字面量以b开头
```




    b'caf\xc3\xa9'




```python
len(b) # 字节序列b有5个字节（在UTF-8中，“é”的码位编码成两个字节）.
```




    5




```python
b.decode('utf8') # 使用UTF-8把bytes对象解码成str对象
```




    'café'



* bytes对象的各个元素是介于 0~255（含）之间的整数.


```python
cafe = bytes('café', encoding='utf_8') # bytes 对象可以从 str 对象使用给定的编码构建.
cafe
```




    b'caf\xc3\xa9'




```python
cafe[0] # 各个元素是 range(256) 内的整数.
```




    99




```python
cafe[:1] # bytes 对象的切片还是 bytes 对象，即使是只有一个字节的切片.
```




    b'c'



* 虽然二进制序列其实是整数序列，但是它们的字面量表示法表明其中有 ASCII 文本. 因此，各个字节的值可能会使用下面三种不同方式显示.                      
    * 可打印的 ASCII 范围内的字节（从空格到 ~），使用 ASCII 字符本身.
        * 标准ASCII码的码长是一个字节,共8位.
    * 制表符、换行符、回车符和 \ 对应的字节，使用转义序列 \t、\n、\r 和 \\.
    * 其他字节的值，使用十六进制转义序列（例如，\x00 是空字节）.
    * 因此，在示例 4-2 中，我们看到的是 b'caf\xc3\xa9'：前 3 个字节 b'caf' 在可打印的 ASCII 范围内，后两个字节则不然.
    
* *struct* 模块提供了一些函数，把打包的字节序列转换成不同类型字段组成的元组，还有一些函数用于执行反向转换，把元组转换成打包的字节序列.

* 需要在多台设备中或多种场合下运行的代码，一定不能依赖默认编码。打开文件时始终应该明确传入 encoding= 参数，因为 不同的设备使用的默认编码可能不同.



# struct模块
* 计算机基础知识
    * bit 0 or 1
    * 1Byte=8bit 
* 把一个32位无符号整数变成字节， 也就是4个长度的bytes


```python
n = 10240099
b1 = (n & 0xff000000) >> 24 
b2 = (n & 0xff0000) >> 16
b3 = (n & 0xff00) >> 8
b4 = n & 0xff
bs = bytes([b1, b2, b3, b4])
print("b1: {} | b2: {} | b3: {} | b4: {} | chr(b1): {}| chr(b2): {} | chr(b3): {} | chr(b4): {}".
      format(b1, b2, b3, b4, hex(b1), hex(b2), chr(b3), chr(b4)))
bs
```

    b1: 0 | b2: 156 | b3: 64 | b4: 99 | chr(b1): 0x0| chr(b2): 0x9c | chr(b3): @ | chr(b4): c





    b'\x00\x9c@c'



* Remark
    * 与运算：n & 0xff000000 获取整型n最高字节,其余字节置0.
    * 移位：>>24 向右位移24位,由0xff000000 变为 0x000000ff, 其他也是类似.
    * b1, b2, b3, b4的值为0， 156， 64， 99
    * 在字面量表示法中b1, b2为十六进制下的0x00, 0x9c, b3, b4转成ascii码中的@和c


* struct的pack函数把任意数据类型变成bytes


```python
import struct
struct.pack('>I', 10240099)
```




    b'\x00\x9c@c'



* pack的第一个参数是处理指令，'>I'的意思是：
    * ｀>｀表示字节顺序是大端（big-endian），｀I｀表示4字节无符号整数.
    * 后面的参数个数要和处理指令一致.
* unpack把bytes变成相应的数据类型


```python
# 根据>IH的说明，后面的bytes依次变为I：4字节无符号整数和H：2字节无符号整数
struct.unpack('>IH', b'\xf0\xf0\xf0\xf0\x80\x80')
(4042322160, 32896)
```




    (4042322160, 32896)



* 与命名元组结合


```python
from collections import namedtuple
record = b'raymond   \x32\x12\x08\x01\x08'
Student = namedtuple('Student', 'name serialnum school gradelevel')
Student._make(struct.unpack('<10sHHb', record))
```




    Student(name=b'raymond   ', serialnum=4658, school=264, gradelevel=8)



# 一个典型例子
我们来讨论一个典型的读写二进制文件的例子，每一帧包括：帧头，实体，帧尾，不妨假定
```
HEAD＝0x17AB17CD
TAIL＝0x19BA23DC
struct entity {
name char[20]  20B
serialnum int  4B
school int     4B
gradelevel int 4B
}
```
- HEAD TAIL采用大端
- ENTITY采用小端

# 生成模拟数据

这里用到来Faker这个第三方来生成人的名字，需要用pip安装

$ pip install Faker



```python
import random
from faker import Faker
fake = Faker()
record_num = 2000000
record_list = [(fake.name()[:20].encode('utf-8'), 
                random.randint(1000, 9999), 
                random.randint(1, 10), 
                random.randint(1, 6))
               for _ in range(record_num)]
record_list[:10]
```




    [(b'Heather Harmon', 6520, 10, 2),
     (b'Sarah Campbell', 2835, 8, 4),
     (b'Rhonda Taylor', 7188, 5, 1),
     (b'Tammy Clarke', 8381, 6, 4),
     (b'Eric Adams', 2715, 3, 2),
     (b'Jeremy Sanchez', 9978, 10, 5),
     (b'Anthony Thompson', 3493, 8, 6),
     (b'Ronald Fox Jr.', 5894, 2, 6),
     (b'Douglas Munoz', 4828, 2, 6),
     (b'Jerry Simmons', 3174, 3, 4)]



# 写入二进制文件

将数据写入data.bin中


```python
import struct
import random
HEAD = (0x17AB17CD).to_bytes(4, byteorder='big')
TAIL = (0x19BA23DC).to_bytes(4, byteorder='big')
HEAD_LEN = len(HEAD)
TAIL_LEN = len(TAIL)
ENTITY_LEN = 32

ENTITY_FORMAT = struct.Struct('<20siii')

data_list = []
for record in record_list: 
    entity_bytes = ENTITY_FORMAT.pack(*record)
    frame_bytes = b''.join([HEAD, entity_bytes, TAIL])
    rand_num = random.random()
    if rand_num > 0.5:
        # add noise some time
        data_list.append(b'noise')
        
    data_list.append(frame_bytes)
    
with open("data.bat", "wb") as f:
    f.write(b''.join(data_list))
```

# 读取数据
从二进制文件data.bat中读取并解析数据


```python
def do_with_record(*record):
    pass

def read(file_path):
    with open(file_path, 'rb') as f:
        f.seek(0, 2)
        num_bytes = f.tell()

        i = 0
        record_list = []
        while i < num_bytes:
            f.seek(i)
            head_bytes = f.read(HEAD_LEN)

            if head_bytes == HEAD:
                j = i + HEAD_LEN + ENTITY_LEN
                f.seek(j)
                tail_bytes = f.read(TAIL_LEN)
                if tail_bytes == TAIL:
                    f.seek(i + HEAD_LEN)
                    entity = f.read(ENTITY_LEN)
                    name, serialnum, school, gradelevel = ENTITY_FORMAT.unpack(entity)
                    do_with_record(name, serialnum, school, gradelevel)
                    i = j + TAIL_LEN
                else:
                    i += 1
            else:
                i += 1
```


```python
%timeit read('data.bat')
```

    6.8 s ± 244 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


# 多进程解析
思路是：按照cpu核心的数量，把文件平分，启动相应的子进程解析对应的文件块，并记录下该文件块中第一个同步头和最后一个同步尾的位置，最后主进程按照每个子进程记录的位置合并被切割点隔开的字节序列并解析。

多进程中，每个进程都是独立的，各自持有一份数据，无法共享。有四种用于进程数据共享的方法
- queues
- Array
- Manager.dict
- pipe

在这种场景下，看一下Manager.dict的用法


```python
from multiprocessing import Process, Manager

# 每个子进程执行的函数
# 参数中，传递了一个用于多进程之间数据共享的特殊字典
def func(i, d):
    d[i] = i + 100
    print(d.values())

# 在主进程中创建特殊字典
m = Manager()
d = m.dict()

for i in range(5):
    # 让子进程去修改主进程的特殊字典
    p = Process(target=func, args=(i, d))
    p.start()
p.join()
print(d)
```

    [100]
    [100, 101]
    [100, 101, 103]
    [100, 101, 103, 102]
    [100, 101, 103, 102, 104]
    {0: 100, 1: 101, 3: 103, 2: 102, 4: 104}



```python

import struct
from multiprocessing import Process, Manager, cpu_count

def read_basic(file_path, start, end):
    i = start
    
    with open(file_path, 'rb') as f:
        while i < end:
            f.seek(i)
            head_bytes = f.read(HEAD_LEN)

            if head_bytes == HEAD:
                j = i + HEAD_LEN + ENTITY_LEN
                f.seek(j)
                tail_bytes = f.read(TAIL_LEN)
                if tail_bytes == TAIL:
                    f.seek(i + HEAD_LEN)
                    entity = f.read(ENTITY_LEN)
                    name, serialnum, school, gradelevel = ENTITY_FORMAT.unpack(entity)
                    do_with_record(name, serialnum, school, gradelevel)
                    i = j + TAIL_LEN
                else:
                    i += 1
            else:
                i += 1


def read_single(file_path, index, start, end, d):

    with open(file_path, 'rb') as f:
        first_head_start_pos = start
        last_tail_end_pos = end

        for i in range(start, end):
            f.seek(i)
            head_bytes = f.read(HEAD_LEN)
            if head_bytes == HEAD:
                first_head_start_pos = i
                break

        for i in range(end, start, -1):
            f.seek(i)
            tail_bytes = f.read(TAIL_LEN)
            if tail_bytes == TAIL:
                last_tail_end_pos = i + TAIL_LEN
                break
                
        read_basic(file_path, first_head_start_pos, last_tail_end_pos)
        d[index] = {
                    'first_head_start_pos': first_head_start_pos,
                    'last_tail_end_pos': last_tail_end_pos
                   }


def read_multi_process(file_path, process_num=None):
    if process_num is None:
        process_num = cpu_count()

    m = Manager()
    d = m.dict()
    p_list = []

    with open(file_path, 'rb') as f:
        f.seek(0, 2)
        num_bytes = f.tell()

    part_num_bytes = num_bytes // process_num

    seek_list = []
    for i in range(process_num):
        seek_list.append(i * part_num_bytes)

    seek_list.append(num_bytes)

    for i in range(process_num):
        p = Process(target=read_single, args=(file_path, i, seek_list[i], seek_list[i+1], d))
        p_list.append(p)

    for p in p_list:
        p.start()

    for p in p_list:
        p.join()

    for i in range(process_num - 1):
        start = d[i]['last_tail_end_pos']
        end = d[i+1]['first_head_start_pos']
        read_basic(file_path, start, end)
```


```python
file_path = 'data.bat'
%timeit read_multi_process(file_path, process_num=4)
```

    3.66 s ± 134 ms per loop (mean ± std. dev. of 7 runs, 1 loop each)


# 结果比较
- 我的机器是2.7GHz i5的Mac pro 4核.
- 生成2百万条数据，二进制文件大小85M.
- 多进程版本用了3.66s， 不用多进程用了6.8s

# 多进程解析II
上面的平均切分然后合并被切割点隔开的字节序列有些繁琐了，不如在切分的时候就找同步头.

考虑如下字符串，如果按照长度的一半分，正好把一个H2222T分开，

'1111H2222T11H2222T1H2222T11H2222T1H2222T111'
'1111H2222T11H2222T1H2'

那么去找长度一半后的第一个H，可以避免这个问题.


```python
l = '1111H2222T11H2222T1H2222T11H2222T1H2222T111'
total_num = len(l)
part_num = total_num // 2
is_find_head = False
i = 0

split_point_list = []

while i < total_num:
    if l[i] == 'H' and i > part_num:
        split_point_list.append(i)
        part_num += part_num
        
    i += 1
split_point_list.append(total_num)
print(split_point_list)
        
    
```

    [27, 43]


# 入库使用多进程
在实践中发现，顺序解析二进制文件和解析好的数据入库相比，前者还是很快的，瓶颈主要在数据写入数据库，考虑仅入库使用多进程， 兼顾性能和简洁，用queue传递数据.
