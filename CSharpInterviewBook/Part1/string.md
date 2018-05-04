### string详解
---
#### string类型
- string为引用类型，其对象值存储在托管堆上。
- string内部为一个char集合。string的长度就是char数组的个数
- string的特性
    1. **恒定性**  
        - 字符串是不可变的，字符串一经创建，就不会改变，任何改变都会产生新的字符串。比如下面的代码： 

            ```
            string s1 = "a";  
            string s2 = s1 + "b";
            ```
            执行上述代码会在堆上产生三个字符串实例："a"，"b"，"ab"。   **字符串的一些操作函数，如str1.ToLower，Trim()，Remove(int startIndex, int count)，ToUpper()等，都会产生新的字符串。**
    2. **驻留性**  
    
        - 相同的字符串在内存（堆）中只分配一次，第二次申请字符串时，发现已经有该字符串是，直接返回已有字符串的地址，这就是驻留的基本过程。 
        
---

#### StringBuilder类型  
- StringBuilder内部同string一样，有一个char[]字符数组，负责维护字符串内容。

- **StringBuilder追加字符串的过程**：
    1. StringBuilder的默认初始容量为16；
    2. 使用stringBuilder.Append()追加一个字符串时，当字符数大于16，StringBuilder会自动申请一个更大的字符数组，一般是倍增；
    3. 在新的字符数组分配完成后，将原字符数组中的字符复制到新字符数组中，原字符数组就被无情的抛弃了（会被GC回收）；
    4. 最后把需要追加的字符串追加到新字符数组中；  
- **StringBuilder产生新字符串的情况**：
    1. 追加字符串时，当字符总长度超过了当前设置的容量Capacity，这个时候，会重新创建一个更大的字符数组，此时会涉及到分配新对象。
    2. 调用StringBuilder.ToString()，创建新的字符串。
    

>因此设置合适的初始容量是非常必要的，尽量减少内存申请和对象创建。因为StringBuilder本身是有一定的开销的，少量字符串就不推荐使用了，使用String.Concat和String.Join更合适。

---

#### 算法题：字符串反转，例如：输入"12345", 输出"54321"  
- *解法1：*

```
public static string Reverse(string str)
{
    if (string.IsNullOrEmpty(str))
    {
        throw new ArgumentException("参数不合法");
    }

    //注意：设置合适的初始长度，可以显著提高效率（避免了多次内存申请）
    StringBuilder sb = new StringBuilder(str.Length);
    for (int index = str.Length - 1; index >= 0; index--)
    {
        sb.Append(str[index]);
    }
    return sb.ToString();
}
```
- *解法2*

```
public static string Reverse(string str)
{
    char[] arr = str.ToCharArray();
    Array.Reverse(arr);
    return new string(arr);
}
```
