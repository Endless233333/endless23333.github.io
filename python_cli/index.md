# 通过 python -m 直接在命令行中使用 python 模块以实现一些有用的功能


在过去, 我经常使用 `python -m http.server` 用来在局域网中给别人分享一些文件, 所以我打算去翻翻自己电脑上安装的各种 python 包, 找一找能直接通过 `python -m` 使用的有用的模块。

## python -m
在 [python 的文档](https://docs.python.org/3/using/cmdline.html#cmdoption-m)中可以看到, `python -m  <module-name>` 会在 sys.path 中搜索指定模块, 并以 `__main__` 模块执行其内容。

以 `__main__` 模块执行其内容有两种情况:
- 如果是单独的 py 文件, 可以通过 `__name__ == '__main__'`, 一般是 `if __name__ == '__main__':`
- 如果是一个包, 执行其下的 `__main__.py`

因此只需要在 sys.path 中找到包含 `__main__.py` 这个文件的目录或者内容中有 `if __name__ == '__main__':` 的文件即可。

## 寻找
我电脑上的 python 包都在 /usr/lib/python3.11, cd 到此目录。

使用 `find . -name "__main__.py"` 寻找 `__main__.py`.

使用 `rg -l "if __name__ == '__main__':" | sort` 寻找内容中有 `if __name__ == '__main__':` 的文件。

## 筛选与使用
/usr/lib/python3.11 下的 site-packages 存放第三方模块。

**一般可以通过 `python -m xxx -h` 或 `python -m xxx --help` 获取帮助信息。**

### 标准库
#### ast
解析源代码成抽象语法树, 例如:
```
> echo "print('hello')" | python -m ast
Module(
   body=[
      Expr(
         value=Call(
            func=Name(id='print', ctx=Load()),
            args=[
               Constant(value='hello')],
            keywords=[]))],
   type_ignores=[])
```

#### base64
用于 base64 编解码, 例如:
```
> echo -n "hello" | python -m base64
aGVsbG8=

> echo -n "aGVsbG8=" | python -m base64 -d
hello%
```

由于 coreutils 里就有 base64, 所以直接在命令行使用这个模块意义不大。

#### compileall
可以将 .py 编译为 .pyc, 例如: `python -m compileall .`.

默认 .pyc 会放在 `__pycache__` 目录下。

#### cProfile
性能分析, 例如: `python -m cProfile -m base64 ./1.txt`.

#### curses
用于创建 TUI 的模块。

`python -m curses.has_key` 检查 has_key.

`python -m curses.textpad` 打开一个简单的写字板。

#### ensurepip
用于确保安装了 pip, 没什么用。

#### gzip
例如: `python -m gzip --best ./test.mp4`

#### http.server
起一个简单的 http 服务。

#### imghdr
确认图像类型, 例如:
```
> python -m imghdr ./Pictures/7.png
./Pictures/7.png: png
```

#### json.tool
校验并打印好看的 json, 例如: `python -m json.tool ~/data.json`.

#### lib2to3
用于将 python2 代码转化成 python3 代码, 可以直接 `2to3` 而不用 `python -m lib2to3`, 也没什么用。

#### mimetypes
显示文件的 mime 类型, 例如:
```
> python -m mimetypes 1.sh
type: application/x-sh encoding: None
```

#### pdb
python debugger, 用法: `python -m pdb your_script.py`.

#### platform
获取平台信息:
```
> python -m platform
Linux-6.6.9-arch1-1-x86_64-with-glibc2.38
```

#### profile
性能分析, 例如:
```
> python -m profile 1.py
hello
         5 function calls in 0.013 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.000    0.000 1.py:1(<module>)
        1    0.000    0.000    0.000    0.000 :0(exec)
        1    0.000    0.000    0.000    0.000 :0(print)
        1    0.013    0.013    0.013    0.013 :0(setprofile)
        1    0.000    0.000    0.013    0.013 profile:0(<code object <module> at 0x7f373a01e5d0, file "1.py", line 1>)
        0    0.000             0.000          profile:0(profiler)
```

选项 `-o` 可以保存分析结果到指定文件。

#### pstats
处理和显示 profile 的结果, 例如:
```
> python -m pstats test
Welcome to the profile statistics browser.
test%
```

#### pydoc
python 文档工具。

#### quopri
`python -m quopri xxx` 对 xxx 进行 Quoted-Printable 编码, 选项 -d 解码。

#### random
`python -m random` 测试产生随机数。

#### shlex
分词, 例如:
```
> echo -n "print('hello')" | python -m shlex
Token: 'print'
Token: '('
Token: "'hello'"
Token: ')'
```

#### site
`python -m site` 打印 sys.path 等信息。

#### sysconfig
`python -m sysconfig` 打印配置信息。

#### tabnanny
检查缩进, 例如:
```
> python -m tabnanny -v 1.py
'1.py': Clean bill of health.
```

#### tarfile
创建和解压 tar 文件, 例如:
```
> touch 1
> ls
1
> python -m tarfile -c 1.tar ./1
> ls
1  1.tar
> python -m tarfile -l 1.tar
./1
> python -m tarfile -t 1.tar
[<TarInfo './1' at 0x7f9cc5ae5cc0>]
> rm 1
> ls
1.tar
> python -m tarfile -e 1.tar
> ls
1  1.tar
```

#### tkinter
用于创建 GUI 的模块, 直接 `python -m tkinter` 的话会有一个简单的小窗口。

#### turtledemo
正如它的名字一样, 直接使用会出现 turtle 的 demo.

#### unittest
用于在命令行中直接运行单元测试。

#### uu
进行 uu 编码和解码。

#### venv
创建虚拟环境: `python -m venv aaa`
激活虚拟环境: `source aaa/bin/activate`
退除虚拟环境: `deactivate`

#### xmlrpc.server
`python -m xmlrpc.server` 起一个简单的 xmlrpc 服务。

#### zipapp
创建和执行 pyz.

创建: `python -m zipapp -o 1.pyz -m "myapp:main" ./app`

执行: `python 1.pyz`

在已存在 `__main__.py` 的目录中不可以使用 `-m` 选项指定入口点。
