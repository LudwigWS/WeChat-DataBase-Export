# WeChat-DataBase-Export

微信数据库，导出微信收藏等

<span id="home">目录</span>

* [1. 使用到的 系统/软件 版本信息](#1)

* [2. 使用lldb调试获取数据库密码](#2)

* [3. 打开数据库并查看数据](#3)

* [4. 使用python清洗数据并导出](#4)
  * [4.1 准备工作](#4-1)
  * [4.2 开始导出](#4-2)

<h2 id="1">1. 使用到的 系统/软件 版本信息</h2>

* macOS（Mojave 10.14）虚拟机安装教程[点击这里](https://github.com/Gaiokane/WeChat-DataBase-Export/blob/master/vm%20install%20macos.pdf "点击跳转")
* 微信（mac版2.4.2）
* DB Browser for SQLite（3.12）
* lldb（1001.0.13.3）（使用时可以自动安装）
* python（2.7.10）（版本无所谓）

[返回目录](#home)

<h2 id="2">2. 使用lldb调试获取数据库密码</h2>

````
1. 打开微信

2. 打开终端输入 lldb -p $(pgrep WeChat) 回车

注意：若没有安装过lldb会提示安装

3. br set -n sqlite3_key 回车

4. c 回车

5. 扫码登录微信

6. 回到终端输入 memory read --size 1 --format x --count 32 $rsi 回车

7. 应该会输出类似于如下的数据
   0x000000000000: 0xab 0xcd 0xef 0xab 0xcd 0xef 0xab 0xcd
   0x000000000008: 0xab 0xcd 0xef 0xab 0xcd 0xef 0xab 0xcd
   0x000000000010: 0xab 0xcd 0xef 0xab 0xcd 0xef 0xab 0xcd
   0x000000000018: 0xab 0xcd 0xef 0xab 0xcd 0xef 0xab 0xcd
   
8. 重新打开一个终端输入以下（3至6行自行替换）
   python
   source = """
   0x000000000000: 0xab 0xcd 0xef 0xab 0xcd 0xef 0xab 0xcd
   0x000000000008: 0xab 0xcd 0xef 0xab 0xcd 0xef 0xab 0xcd
   0x000000000010: 0xab 0xcd 0xef 0xab 0xcd 0xef 0xab 0xcd
   0x000000000018: 0xab 0xcd 0xef 0xab 0xcd 0xef 0xab 0xcd
   """
   key = '0x' + ''.join(i.partition(':')[2].replace('0x', '').replace(' ', '') for i in source.split('\n')[1:5])
   print(key)

9.  回车后返回的一串0x开头的66位字符串就是数据库密码
````

[返回目录](#home)

<h2 id="3">3. 打开数据库并查看数据</h2>

````
1. 打开终端输入以下
   ls -alh ~/Library/Containers/com.tencent.xinWeChat/Data/Library/Application\ Support/com.tencent.xinWeChat/*/*/Favorites/*.db

2. 回车后返回 收藏 数据库文件路径

3. 去到对应路径下双击favorites.db（确保已安装DB Browser for SQLite）

4. 在弹出的对话框中选择 原始密钥（raw key）、SQLCipher 3默认，在密码文本框中输入数据库密码，点击OK以连接

注意：要在Windows中打开.db文件，运行DB Browser for SQLite目录中的DB Browser for SQLCipher.exe

5. 收藏文章的标题（<pagetitle>）、链接（<link>）均在表FavoritesItemTable -> 字段xml中
````

微信还有其他本地数据库，如：

* Contact - wccontact_new2.db - 好友信息
* Group - group_new.db - 群聊和群成员信息
* Message - {msg_0.db - msg_9.db} - 聊天记录和公众号文章

[返回目录](#home)

<h2 id="4">4. 使用python清洗数据并导出</h2>

<h3 id="4-1">4.1 准备工作</h3>

使用DB Browser for SQLite打开数据库，点击任务栏中 工具>设置加密>OK，禁用数据库加密，以便之后使用python读取数据

[返回目录](#home)

<h3 id="4-2">4.2 开始导出</h3>

此处以python3为例，确保已安装python3，并添加相应环境变量

下载[WeChat_Favorites_Export.py](https://github.com/Gaiokane/WeChat-DataBase-Export/blob/master/WeChat_Favorites_Export.py "点击跳转")到本地

````
修改WeChat_Favorites_Export.py中的第7行，favorites.db文件所在路径

在WeChat_Favorites_Export.py目录中按住shift+鼠标右键打开命令提示符/powershell

执行python .\WeChat_Favorites_Export.py

导出结果在WeChat_Favorites_Export.txt中
````

注意：后期收藏内容有变，若要将db文件单独复制出来重新执行py导出操作时，需将favorites.db、favorites.db-shm、favorites.db-wal这三个文件放置在同一路径下，打开数据库并禁用密码，新增数据会保存在favorites.db-shm、favorites.db-wal中，如果只拷贝favorites.db并打开会发现还是之前的数据没有变化

[返回目录](#home)

———————————————————————————————————

参考文章：

https://www.lianxh.cn/news/d34f09cb214e0.html

https://blog.csdn.net/swinfans/article/details/88712593

https://www.v2ex.com/t/466053