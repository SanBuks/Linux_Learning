# 1. 信息存储
## 十六进制表示法
- n = i+4j  快速转换法 如 2^33 = 0x2 0000 0000
- 1100 - C - 12 | 1101 - D - 13 | 1110 - E - 14

## 字数据大小 
- 机器字长 指明 指针数据大小, 指明 虚拟地址空间大小
- gcc -m32/-m64 指明编译的字长
- 整型分有有符号无符号, 还有大小确定的类型如 `uint64_t / int64_t`

| 数据类型       | 32   | 64   | minimal |
| -------------- | ---- | ---- | ------- |
| short          | 2B   | 2B   | 2B      |
| int            | 4B   | 4B   | 2B      |
| long           | 4B   | 8B   | 4B      |
| long long(cpp) | 8B   | 8B   | 8B      |
| float          | 4B   | 4B   | -       |
| double         | 8B   | 8B   | -       |

## 寻址
- 字节为最小可寻址的内存单位
- 小端法 : 从最低有效位到最高有效位 按地址从低到高排序
- 大端法 : 从最高有效位到最底有效位 按地址从低到高排序
- 强制类型转换 : `isLittleEdian()` 与`show_bytes(byte_pointer p, size_t len)`
> 注意 网络传输 和 阅读反汇编时的 大小端区别

## 字符和代码
- 字符串 以 `\0` 结尾, 每个字符用ASCII表示  0 - 48 A - 65 a - 97
- 字符串存储从前往后 按地址从底到高排序
- 代码为二进制编码与文本类型不一样


## 布尔代数
- 布尔环 : 用 `^` , `&` 和 `~` 运算
- 加法(`^`)逆元, 交换方法 `inplace_swap()`
- `&`和`|`的分配律
- 用途 : 掩码, 表示集合

|      | 0    | 1    |
| ---- | ---- | ---- |
| &    | 置零 | 保留 |
| \|   | 保留 | 置一 |
| ^    | 保留 | 求反 |

- bis(x, y ) == x or y 
- bic(x, y) == x & ~y 
- x ^ y == (x & ~y) | (~x & y)

## 移位运算
- 左移补零
- 算术右移补符号位
- 逻辑右移补零
- 位移 k = k mod w 换句话说, k 一定小于位长

# 2.2 整数表示
## 编码
> `<climits> <cfloat>` 含有相关最大值最小值的宏
> 单射 : 原不同则像不同,  满射 : 像皆有原

- 无符号数 : B2Uw 
- 补码 : B2Tw 不对称范围
- 反码 : B2Ow 对称双零
- 原码 : B2Sw 对称双零

## 有符号和无符号转换
- 有符号转无符号 T2Uw (负数就是加上两倍最高位权) (负数情况下,有符号越靠近负零转换的越大)
> note : 无符号数是带来意外的源泉

- 无符号转有符号 U2Tw (超过Tmax就是减去两倍最高位权) (负数情况下,无符号越大转换的有符号越大)
- 有符号数与无符号数相运算, 有符号数会优先转换成无符号数

## 扩展与截断
- 无符号扩展 加零 有符号扩展 加符号位
- C 语言规则 规定 小整型运算时会先整型提升, 然后进行转换
- 无符号截断为k位 : x mod 2^k
- 有符号截断为k为 : U2T( T2U mod 2^k )

# 2.3 整数运算
## 无符号加法
- 模数加法构成阿贝尔群, 存在加法逆元
    - x+y = x+y (没溢出)
    - x+y = x+y-2^w ( 溢出 )
```c
// 无符号溢出判断
int uadd_ok(unsigned x, unsigned y){
    unsigned u=x+y;
    return u>=x;
}
```

- 无符号求反 ( 加法逆元 ) :  -x = 2^w-x  (x != 0)

## 补码加法
- 与无符号加法有相同的位级表示
    - x+y=x+y-2^w (正溢出)
    - x+y=x+y (没溢出)
    - x+y=x+y+2^w (负溢出)
- 补数的非
    - 求补+1 
    - 求反直到遇到最后一个1为止(最后一个保留)
    - 注意 :  -TMIN=TMIN 
```c++
// 补数相加溢出判断 注意模数加减法构成阿贝尔群, x+y-x === y
int tadd_ok(int x, int y){
	int t=x+y;
	return !(x>0 && y>0 && t<0) && !(x<0 && y<0 && t>=0); 
}
// 补数相减溢出判断 注意 -TMIN=TMIN
int tsub_ok(int x, int y){
	if(y==INT_MIN) return x<0 ? 0 : 1;
	return tadd_ok(x, -y);
}
```

## 无符号和补码乘法
- 无符号乘法和补码乘法有位级等价性
    - T2Bw( x \* y ) = U2Bw( T2Uw(x) \* T2Uw(y) ) = U2Bw( (x\*y) mod 2^w )
    - x \* y = U2T( (x\*y) mod 2^w )

```c
// 判断补码乘法是否溢出
int tmul_ok(int x, int y){
	int t=x*y;
	return !x || t/x==y;
}
// 或者对比 用双倍大小存放正确结果 与 拓展计算结果 对比是否相等
int tmul_ok_64(int32_t x, int32_t y){
    int64_t t1= ((int64_t)x) * ((int64_t)y);
    int32_t t2= x*y;
    return int32_t(t1)==t2;
}
```

- 无符号数 \* 2^k 等价 <\<k , 补码同理
- 乘以常数K(0111110) 有两种转换方法 :  
    - 位权相加 : K=2^k1 + 2^k2 +...+ 2^kn 
    - 连续1转换为两个位权相减 : K=2^6 - 2^1

## 无符号与补码除法
- 整数除法向零取整 ( 正数向下取整 , 负数向上取整 )
- 无符号数/2^k 等价 >\>k , 向下取整
- 补码 / 2^k 等价 >\>k : 
    - 正数向下取整
    - 负数也向下取整, 需要调整 ( 汇编处理对程序员透明 ) 
- 负数补码整除调整 : 
	- [ x / y ] 上取整 = [ (x + y - 1) / y ] 下取整 可以调换
    - 偏置法每次向左移动移位, 舍掉1 末位+1 , 舍掉0 末位+0
    - \>\>k == (x+2^k-1) / 2^k ( x< 0 )原理 
```c
int div16(int x){
	// return ( (x > 0) ? ( x >> 4 ) : ( x + ( 1 << 4 - 1)  >> 4 ) );
    // (x < 0) (x>>31 & 0xf) => ( 0xffffffff & 0xff = 000000ff )
    return x+(x>>31&0xF)>>4;
}
```

# 2.4 浮点数
> 近似表示, 精度有限, 不可结合

## IEEE 浮点表示
- 表示方式 : V = (-1)^s \* 2^E \* M 分别为 符号, 阶码和尾数
- 分布 : 1+8+23 / 1+11+52 
- 三种特殊浮点 : 
    - 规格化的 : 不为0, 不为255
    - 非规格化的 : E 都为 0
    - 特殊情况 : E 都为 1
      - M 为 0 , 根据 s 分为 正无穷和负无穷
      - M 为 非0, NaN

## 阶码部分
| 阶码位数 | 阶码作为无符号数范围 | bias : 2^(k-1) - 1 | 阶码实际范围 |
| --- | --- | --- | --- |
| 8 | 1 ~ 254 | 127 | -126 ~ 127 |
| 11 | 1 ~ 2046 | 1023 | -1023 ~ 1024 |

> 非规格化数 阶码为 1 - bias 而非 0 - bias,  实现非规格化数的平滑过度
> 非规格化数提供逐渐溢出, 均匀分布在 0 的两端

## 尾数部分
- 规格化数, 有隐藏的 1 , 如 10010101 实际为 1.10010101
- 非规格化数, 没有隐藏的 1, 如 10010101 实际为 0.10010101

## 舍入
- 舍入类型 : 
    - 向上舍入
    - 向下舍入 
    - 向零舍入 
    - 向偶数舍入 (减小误差 )
- 十进制小数的舍入只有在正中央时参照最后一位是否为偶数进行舍入
- 二进制小数的舍入只有在正中央时参照最后一位是否为零进行舍入

## 浮点运算
- 每一步骤可看作 Round(x ◦ y), 没有可结合性, 编译器倾向于优化保守
- 满足单调性及不等式性质, 具有交换性
- 除以零会产生无穷, NAN + x  = NAN, 无穷 ◦ 无穷 = NAN

## C语言浮点数转换
- 整型转浮点存在舍入
- double 转 float 存在 舍入 或 溢出为无穷
- 浮点数 转 int 存在合适值则向零舍入, 如果超出范围则会指定为 [1000...0]w 的位模式 
- 浮点数 转 unsigned 与 转 int 不一样需要具体看

# 2.5 习题
2.35 判断补码相乘溢出的实质  
`x * y = t * 2 ^ w + p => p = x * y - t * 2 ^ w`
2.36 `~ x = -x -1` ( x 是有符号数)
2.46 十进制小数 转 二进制小数, 循环替换
2.49 23 位尾数从1开始连续增加的最大正整数为 1.0000000 1 \* 2^24 = 2^24 + 1

