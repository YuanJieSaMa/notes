# Ranger是什么
zsh:1: unknown file attribute: h
![](https://cdn.jsdelivr.net/gh/YuanJieSaMa/personal-photo-house/img/202310021544114.png)

其默认操作和Vim的默认操作基本一致.

# Ranger的体用与个性化

## 体用

1. 你可以在ranger内选择打开一种文件的方式，通过字母*r* :
2. ![](https://cdn.jsdelivr.net/gh/YuanJieSaMa/personal-photo-house/img/202310021623951.png)
  然后可以根据编号选择.
3. Ranger支持通过[]来上下移动**父文件夹**  
4. *H*和*L*分别可以**退回历史记录** 和 **撤回退回操作** 
5. 同样支持通过设置快捷键来**指定跳转的文件夹**，例如：通过{key}跳转到Study文件夹....
6. 通过<C-h>或者*zh* 显示或隐藏**隐藏文件** 
7. *O*可以自由选择更改排序方式：
  ![](https://cdn.jsdelivr.net/gh/YuanJieSaMa/personal-photo-house/img/202310021640789.png)
8. */* 可以用来在当前文件夹中**搜索** :
  ![](https://cdn.jsdelivr.net/gh/YuanJieSaMa/personal-photo-house/img/202310021703068.png)
  使用*n* 和*N* 来查找下一个和返回上一个
9. 从该文件夹进入终端为*S* 
10. ranger有**fzf** 的支持，可以在fzf中找文件，再通过ranger打开，具体操作在后
11. ranger支持**直接复制文件路径** 通过*yp*,不过ranger内置了剪切板，具体操作在后
12. *cw* 可以重命名文件，*a* 可以直接编辑当前文件名(在文件名后插入)，*i* 在文件名前插入，*A* 在后缀后插入.
13. *v*可以选中所有文件，*<speace>* 可以选中单个文件，在选中后:
```
:bulkrename
```
随后可以在编辑器中更改他们的名字，建议搭配(n)vim使用.

13. 复制文件和粘贴文件和vim的操作一样，重名时ranger会在文件后缀名后方加上**下划线** ，使用*po* 可以覆盖.剪切为*dd*，删除为*dD* 
14. *dU* 可以查看**文件夹大小**: 
![](https://cdn.jsdelivr.net/gh/YuanJieSaMa/personal-photo-house/img/202310022055402.png)
15. 在复制过程中，按下dd可以取消任务.

> 除此以外，还有很多..... 但是真正好用的地方在与ranger的**可定制性极高**.

## 个性化配置

ranger的配置文件在][](~/.config/ranger/)
在配置前，我们需要先初始化ranger,使用：
```
ranger --copy-config=all
```
![](https://cdn.jsdelivr.net/gh/YuanJieSaMa/personal-photo-house/img/202310022117045.png)
随后我们需要添加一个ranger的环境变量：
```
export RANGER_LOAD_DEFAULT_RC=FALSE
```

或者[永久变量](https://blog.csdn.net/yi412/article/details/11523525)
![](https://cdn.jsdelivr.net/gh/YuanJieSaMa/personal-photo-house/img/202310022128668.png)

ranger的具体信息和用法都可以在[ranger的wiki](https://github.com/ranger/ranger/wiki)中找到
![](https://cdn.jsdelivr.net/gh/YuanJieSaMa/personal-photo-house/img/202310022128668.png)

1. commands.py 是添加自己的命令的，里面有示例，官方也提供了很多脚本
2. rc.conf是主要的配置文件
3. rifle.conf存储文件打开方式
4. scope.sh为ranger的切入点

在ranger的wiki中找到*plugins*
![](https://cdn.jsdelivr.net/gh/YuanJieSaMa/personal-photo-house/img/202310022137017.png)
我们可以在里面找到许多有趣的插件：

1. [ranger_devicons](https://github.com/alexanderjeurissen/ranger_devicons)可以给ranger中显示的文件加上**图标** 


我们同样可以在wiki中[Custom commands](https://github.com/ranger/ranger/wiki/Custom-Commands)中找到一些有趣的命令：


### 一些重要配置
1. 我们打开rc.conf，找到*set vcs_aware* , 改为true. 他可以协助管理Git.
2. 要实现ranger中预览图片，首先我们安装w3m,然后在rc.conf中找到**preview_images** ,改为true.再找到*preview_images_method* 改为w3m. 
3. 使用**cw** 进行批量重命名需要在*rc.conf* 中添加：
```
map cw eval fm.execute_console("bulkrename") if fm.thisdir.marked_items else fm.open_console("rename ")
```
并删除原来的map映射.

4. 在配置完我们的需求后，我们可以进入*scope.sh* 来，更改打开方式（反注释）
5. 建立映射如：
```
map f console scout -ftsea%space

```
可以使用f来查找文件，并只显示包含查找字符的文件.

6. 下载youtube-dl并使用：
```
map ytv console shell youtube-dl -ic%space
map yta console shell youtube-dl -xic%space

```
来下载youtube视频.

7. 使用：
在commands.py中加入：
```
import os
from ranger.core.loader import CommandLoader

class extract_here(Command):
    def execute(self):
        """ extract selected files to current directory."""
        cwd = self.fm.thisdir
        marked_files = tuple(cwd.get_selection())

        def refresh(_):
            cwd = self.fm.get_directory(original_path)
            cwd.load_content()

        one_file = marked_files[0]
        cwd = self.fm.thisdir
        original_path = cwd.path
        au_flags = ['-x', cwd.path]
        au_flags += self.line.split()[1:]
        au_flags += ['-e']

        self.fm.copy_buffer.clear()
        self.fm.cut_buffer = False
        if len(marked_files) == 1:
            descr = "extracting: " + os.path.basename(one_file.path)
        else:
            descr = "extracting files from: " + os.path.basename(
                one_file.dirname)
        obj = CommandLoader(args=['aunpack'] + au_flags
                            + [f.path for f in marked_files], descr=descr,
                            read=True)

        obj.signal_bind('after', refresh)
        self.fm.loader.add(obj)
```
和
```
import os
from ranger.core.loader import CommandLoader

class compress(Command):
    def execute(self):
        """ Compress marked files to current directory """
        cwd = self.fm.thisdir
        marked_files = cwd.get_selection()

        if not marked_files:
            return

        def refresh(_):
            cwd = self.fm.get_directory(original_path)
            cwd.load_content()

        original_path = cwd.path
        parts = self.line.split()
        au_flags = parts[1:]

        descr = "compressing files in: " + os.path.basename(parts[1])
        obj = CommandLoader(args=['apack'] + au_flags + \
                [os.path.relpath(f.path, cwd.path) for f in marked_files], descr=descr, read=True)

        obj.signal_bind('after', refresh)
        self.fm.loader.add(obj)

    def tab(self, tabnum):
        """ Complete with current folder name """

        extension = ['.zip', '.tar.gz', '.rar', '.7z']
        return ['compress ' + os.path.basename(self.fm.thisdir.path) + ext for ext in extension]
```

再

```
map C console compress%space
```
压缩文件

8. 找到VIM-like,按，需求配置上下左右键位，我的：
```
copymap <UP>       i   
copymap <DOWN>     k
copymap <LEFT>     j
copymap <RIGHT>    l
copymap <HOME>     gg
copymap <END>      G
map K move down=5
map I move up=5
#copymap <PAGEDOWN> <C-E>
#copymap <PAGEUP>   <C-U>
```
然后I键位和K键位有重合，记得要修改
