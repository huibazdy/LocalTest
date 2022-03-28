## 本地与GitHub关联并进行Git管理的两种方法

### 0.准备工作

> **GitHub连接已有仓库时使用的认证方式是SSH公开密钥**

* **创建一个用于认证的SSH Key并将其添加至GitHub**

    1. 首先检查是否已经为计算机生成过SSH密钥，在Git bash命令行下先后输入以下两条命令：`cd ~/.ssh`然后`ls`若没有则显示：

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328142307963.png" alt="image-20220328142307963" style="zoom:67%;" />

        或直接一条命令：`cat ~/.ssh/id_rsa.pub`，若没有则显示：

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328142213417.png" alt="image-20220328142213417"  />

    2. 若没有生成过，使用下列命令生成：`ssh-keygen -t rsa -C "your_email@example.com"`（邮箱要用GitHub注册的邮箱！！！）

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328135624532.png" alt="image-20220328135624532"  />

        回车一次输入密码，再次回车确认密码：

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328135713636.png" alt="image-20220328135713636" style="zoom:80%;" />

        最后生成的结果是：

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328135836384.png" alt="image-20220328135836384" style="zoom: 67%;" />

        其中`id_rsa`是私有密钥，`id_rsa.pub`是共有密钥。可以在路径C:\Users\zdy\.ssh中看到生成的两个文件

        ![image-20220328140210458](https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328140210458.png)

* **在GitHub中添加共有密钥**

    在GitHub中打开一个新建的仓库，找到配置ssh的位置，你会看到要求你添加共有密钥的提示：

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328140838077.png" alt="image-20220328140838077" style="zoom:50%;" />

    点击提示的添加新的个公开密钥的链接，复制之前生成密钥路径下id_rsa.pub中的文本内容并粘贴至下图2处：（步骤1可以省略，会默认以邮箱名命名该ssh）

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328141301024.png" alt="image-20220328141301024" style="zoom: 33%;" />

    添加结束后可以在个人账户设置页面中看到以下内容：

    ![image-20220328141556015](https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328141556015.png)

    公钥文件id_rsa.pub中的文本内容可以在Git bash命令行中以命令：`cat ~/.ssh/id_rsa.pub`查看。在添加SSH Key成功后，创建GitHub账号的邮箱中会受到一封已添加新公钥的邮件。

* **完成上述设置后，本地计算机就可以利用私钥与GitHub进行认证通信了，现在进行验证**：

    键入命令`sst -T git@github.com`

    ![image-20220328144212818](https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328144212818.png)

    输入`yes`并回车，提示警告信息

    ![image-20220328144406163](https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328144406163.png)

    输入之前设置的认证密码并回车，完成连接

    ![image-20220328144542377](https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328144542377.png)

### 1. 先本地，后GitHub

* 新建一个本地仓库

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328203037683.png" alt="image-20220328203037683" style="zoom:50%;" />

* 使本地仓库受Git管理

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328203105262.png" alt="image-20220328203105262" style="zoom:50%;" />

* 向仓库中添加文件或修改

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328203349858.png" alt="image-20220328203349858" style="zoom:50%;" />

* 提交修改到暂存区

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328203530642.png" alt="image-20220328203530642" style="zoom:50%;" />

* 提交到本地仓库此处采取提交简单修改描述的方式）

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328203829610.png" alt="image-20220328203829610" style="zoom:50%;" />

* 推送到远程仓库（此处指的就是GitHub）

    1. 在GitHub上创建一个同名仓库作为本地仓库的远程仓库（最好不要勾选with a README）

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328204531623.png" alt="image-20220328204531623" style="zoom:33%;" />

    2. 添加远程仓库——`git remote add`

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328205054919.png" alt="image-20220328205054919" style="zoom:50%;" />

    3. 推送至远程仓库——`git push`

        第一次由于远程仓库为空，需要在`git push`后加上`-u`参数。其他分支也需要加上这个参数。

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328205206616.png" alt="image-20220328205206616" style="zoom:50%;" />

        之后可以在仓库中看到添加的hello.cpp文件：

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328210612671.png" alt="image-20220328210612671" style="zoom: 33%;" />





### 2. 先GitHub，后本地

* 在新建的仓库后复制仓库的ssh链接：

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328145225849.png" alt="image-20220328145225849" style="zoom: 50%;" />

* 在Git bash命令行中切换到想要保存仓库的路径下，使用`git clone`命令克隆这个仓库到本地：

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328151124958.png" alt="image-20220328151124958" style="zoom:50%;" />

* 在路径下查看克隆下来的仓库：

    ![image-20220328151249440](https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328151249440.png)

* 切换到仓库路径下，用`git status`查看仓库状态：

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328151520986.png" alt="image-20220328151520986" style="zoom:50%;" />

* 在仓库路径下用vim新建一个hello.c文件，并再次查看仓库状态可以看到仓库存在**Untracked files**：

    <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328151946687.png" alt="image-20220328151946687" style="zoom:50%;" />

* 提交相关修改（此时的修改是新建的文件hello.c）到仓库Common-Algorithms中：

    1. `gir add`——提交到暂存区

    2. `git commit`——提交到本地仓库（使文件变得可追踪）

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328152531897.png" alt="image-20220328152531897" style="zoom:50%;" />

    3. 在提交后，弹出的vim页面编辑该次提交的详细修改信息，保存并退出

        ![image-20220328152731950](https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328152731950.png)

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328152825384.png" alt="image-20220328152825384" style="zoom:50%;" />

    4. `git log`——查看仓库提交历史

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328153052500.png" alt="image-20220328153052500" style="zoom:50%;" />

    5. `git push`——推到远程仓库（此处指的是GitHub）

        <img src="https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328165441494.png" alt="image-20220328165441494" style="zoom:50%;" />

        可以观察到GitHub仓库中增加了hello.c文件：

        ![image-20220328165550722](https://gitee.com/huiba450zdy/typora-picture/raw/master/img/image-20220328165550722.png)

        