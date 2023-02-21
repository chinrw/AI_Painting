
# 从零开始安装动漫风格的ControlNet

本教程基于：

```
https://github.com/lllyasviel/ControlNet/discussions/12
```

感谢作者

## 前置软件安装

# Anaconda

Anaconda是一种包管理器，并提供了虚拟环境和版本解算器用于隔离和部署。

* 安装方式

使用下面的链接下载并安装：

```
https://repo.anaconda.com/archive/Anaconda3-2022.10-Windows-x86_64.exe
```

* 安装检查

打开菜单中的```Anaconda Powershell Prompt (anaconda3)```。

此时命令行前面会出现一个```(base)```的字样代表Anaconda安装成功，代表你当前处于base虚拟环境之下。

* 添加Anaconda的国内源（可选）

完成Anaconda的安装后，还需通过执行下面的命令添加Anaconda的国内镜像源（清华）：

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/

conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/

conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
```

当从Anaconda下载的时候，会优先搜索上面添加的国内源服务器，保证下载速度。

可以通过```conda config --show channels```命令查看添加是否成功。

# Git

Git是一种版本管理工具，在本例中多用于从远程的仓库获取代码和模型数据。

* 安装方式

打开命令行工具，使用下面的命令安装：

```
winget install --id Git.Git -e --source winget
```

* 替代安装方式

可以通过下列链接下载直接安装：

```
https://github.com/git-for-windows/git/releases/download/v2.39.2.windows.1/Git-2.39.2-64-bit.exe
```

* 安装检查

安装完成后，关闭并重新打开命令行，使用下列命令检查是否安装成功：

```
git --version
```

## ControlNet代码下载与环境构建

ControlNet代码发布在下列地址：

```
https://github.com/lllyasviel/ControlNet
```

其官方模型发布在下列地址：

```
https://huggingface.co/lllyasviel/ControlNet/tree/main
```

# 下载代码并构建运行环境

打开菜单中的```Anaconda Powershell Prompt (anaconda3)```

使用```cd```命令转移到你希望放置ControlNet的目录。

全功能的ControlNet及其模型需要100GB左右的空间，请注意规划存储空间。

使用下面的命令获取ControlNet代码：

```
git clone https://github.com/lllyasviel/ControlNet.git
```

完成后，会在当前目录下创建一个ControlNet目录。

使用```cd ControlNet```命令进入该目录。

为了规避因为网络原因导致的构建失败问题，使用文本编辑器打开目录下的environment.yaml文件。

在```- pip:```下面添加代码```- -i https://pypi.tuna.tsinghua.edu.cn/simple```

注意缩进对齐，完成后应如下所示（添加的项目应该与其他子项保持相同缩进）：

```
  - pip:
     - -i https://pypi.tuna.tsinghua.edu.cn/simple
```

保存退出。

* 构建运行环境

在```(base)```前缀下，使用下列的命令构建环境：

```
conda env create -f environment.yaml
```

构建需要一定的时间，且还是会有构建失败的情况发生。

无论是哪种情况，请先使用```conda activate control```命令尝试激活刚才创建的环境。

* 构建失败

构建失败原因一般是pip工具无法正常访问软件仓库，pip工具是python自带的包管理器，本例中运行环境的构建一部分由conda提供源，一部分由pip提供源。

这个时候我们需要为pip工具添加国内镜像源服务器。

首先使用```python --version```和```pip --version```命令确保python及pip已经安装成功。

python版本应在3.8以上，pip版本应在20以上。

接下来使用下面两行代码为pip添加国内源（清华）：

```
pip config --global set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

pip config --global set install.trusted-host tuna.tsinghua.edu.cn
```

添加成功后，命令行会反馈配置写入到一个pip.ini文件中，可以打开该文件检查是否添加成功。

或者使用```pip config list```指令查看。

* 重新构建环境

虽然构建失败，但虚拟环境已经创建，且部分环境已经构建成功，此时我们仅需构建失败的部分。

确保你处于control虚拟环境之下（即命令行前缀为```(control)```）

使用下面的命令继续构建环境：

```
conda env update -f environment.yaml
```

完成构建。

# 下载官方模型

使用下面的命令激活lfs组件用于下载大体积文件：

```
git lfs install
```

进入一个空闲目录中，使用下面的命令下载模型：

```
git clone https://huggingface.co/lllyasviel/ControlNet
```

请确保网络通畅，该命令会下载90GB左右的模型文件。hugface在国内下载速度也很快，经测试带宽基本可以跑满。

如果下载速度过慢，请参考最下面的troubleshooting部分。

下载过程中命令行界面的反馈比较少，可以通过任务管理器网络通信监控下载状态。

如果你并不需要全功能的ControlNet，仅需挑选部分模型下载即可，模型的用途和位置可参考：

```
https://huggingface.co/lllyasviel/ControlNet
```

# 合并代码和模型

下载的代码和模型具有完全相同的目录结构，直接合并即可。

# 试运行

在本例中，我们仅关心ControlNet的根据姿态生成功能，即```pose2image```。

确保你有一张显存大于9GB的Nvidia显卡（如果你的显卡显存小于9GB，或者需要一次性生成多张图片，请见下文中的代码修改部分）

确保你位于```(control)```虚拟环境之下，并位于ControlNet目录中。

使用下面的命令运行ControlNet的pose2image功能。

```
python gradio_pose2image.py
```

等待初始化完成后，命令行将回馈一个网址，将其中的```0.0.0.0```改为```127.0.0.1```并使用浏览器打开。

enjoy。

* 注意：可以使用```Ctrl + C```指令退出。

## 还没结束，大奶二次元萌妹呢

为了生成大奶二次元萌妹，需要使用社区模型。

首先，下载下列两个模型：

```
https://huggingface.co/Linaqruf/anything-v3.0/resolve/main/anything-v3-full.safetensors

https://huggingface.co/runwayml/stable-diffusion-v1-5/resolve/main/v1-5-pruned.ckpt
```

下载完毕后，将它们移动到```ControlNet/models```目录中。

确保你位于```(control)```虚拟环境之下，并位于ControlNet目录中。

使用下列命令迁移生成基于上述两个模型的新模型：

```
python tool_transfer_control.py
```

运行结束后，会在models目录下生成一个新的模型```control_any3_openpose.pth```，这就是我们需要的。

此外，该模型也可以直接在下列地址下载：

```
https://huggingface.co/toyxyz/Control_any3/tree/main
```

# 编辑代码让其调用新模型

打开ControlNet目录下的```gradio_pose2image.py```文件。

修改代码有两处，如下：

* 1，在```from share import *```的下面添加：

```
from cldm.hack import hack_everything
hack_everything(clip_skip=2)
```

* 2，在```model = create_model('./models/cldm_v15.yaml').cpu()```下面修改：

```
# 上面是写实模型，下面是动漫模型，可以通过追加或删除代码前端的 # 符号调整
# model.load_state_dict(load_state_dict('./models/control_sd15_openpose.pth', location='cuda'))
model.load_state_dict(load_state_dict('./models/control_any3_openpose.pth', location='cpu'))
```

顺便可以把最后一行的```0.0.0.0```改为```127.0.0.1```。

* 开启低显存模式

如果你的显卡显存小于9GB，或者你需要一次性生成多张图片，请打开ControlNet根目录下的config.py文件，修改如下：

```
save_memory = True
```

保存退出。

# 生成妹子

确保你位于```(control)```虚拟环境之下，并位于ControlNet目录中。

再次使用```python gradio_pose2image.py```运行，同样的指令即可生成大奶萌妹。

对于姿势的干预，你需要提供一张姿态图，建议使用下面的两个网站摆好姿势，下载图片，然后再去生成。

同时也可以接受prompt词的指导。

```
二次元
https://www.vrmwebpose.app/

写实
https://webapp.magicposer.com/
```

enjoy。
