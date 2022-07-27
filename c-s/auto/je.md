---
title: 一套从音频到数字谱的自动化方案
description: 干啥啥不行，偷懒第一名
published: true
date: 2022-07-27T15:38:01.033Z
tags: 自动化, 深度学习
editor: markdown
dateCreated: 2022-07-25T10:54:42.748Z
---

# 简介
总的来说就是输入`mp3`文件，输出数字谱。大部分内容几乎都来自[这个视频](https://www.bilibili.com/video/BV1iF411F7zq?share_source=copy_web&vd_source=5ba96cafdf071d67115d1551390ede16)。

涉及到的工具主要包括以下
- [demucs *分离音乐和人声*](https://github.com/facebookresearch/demucs)
- [icassp2022-vocal-transcription *音乐转midi*](https://github.com/keums/icassp2022-vocal-transcription)
- [AcgmuseParser *midi转数字谱*](https://github.com/pluveto/AcgmuseParser)
{.links-list}

对此你主要需要掌握以下的软件及其相关知识
- git，github
- anaconda

下面就逐一介绍一下

# 具体介绍
> 注意：以下所有的git clone 操作请在同一目录下进行，否则后面写自动化脚本的时候就是自找麻烦。
{.is-info}


## 人声分离--demucs

### 安装
按照github上说的一键就能安装
前提是要有conda，不过这东西教程满天飞了，不再缀述
```
conda env update -f environment-cpu.yml  # if you don't have GPUs
conda env update -f environment-cuda.yml # if you have GPUs
conda activate demucs
pip install -e .
```
这些命令要在仓库目录下执行，也就是你得先执行
```
git clone https://github.com/facebookresearch/demucs.git
cd demucs
```
## 使用

```
demucs tracks [options]

positional arguments:
  tracks                Path to tracks

optional arguments:
  -h, --help            show this help message and exit
  -s SIG, --sig SIG     Locally trained XP signature.
  -n NAME, --name NAME  Pretrained model name or signature. Default is mdx_extra_q.
  --repo REPO           Folder containing all pre-trained models for use with -n.
  -v, --verbose
  -o OUT, --out OUT     Folder where to put extracted tracks. A subfolder with the model name will be created.
  -d DEVICE, --device DEVICE
                        Device to use, default is cuda if available else cpu
  --shifts SHIFTS       Number of random shifts for equivariant stabilization.Increase separation time but improves quality for Demucs. 10 was used in the
                        original paper.
  --overlap OVERLAP     Overlap between the splits.
  --no-split            Doesn't split audio in chunks. This can use large amounts of memory.
  --segment SEGMENT     Set split size of each chunk. This can help save memory of graphic card.
  --two-stems STEM      Only separate audio into {STEM} and no_{STEM}.
  --int24               Save wav output as 24 bits wav.
  --float32             Save wav output as float32 (2x bigger).
  --clip-mode {rescale,clamp}
                        Strategy for avoiding clipping: rescaling entire signal if necessary (rescale) or hard clipping (clamp).
  --mp3                 Convert the output wavs to mp3.
  --mp3-bitrate MP3_BITRATE
                        Bitrate of converted mp3.
  -j JOBS, --jobs JOBS  Number of jobs. This can increase memory usage but will be much faster when multiple cores are available.

```

### --segment

Personally recommend not less than 10 (the bigger the number is, the more memory is required, but quality may increase).

不加这个参数是最好的，如果运行时提示显卡内存不足再加，这个参数是越大越好，所以从8往上调吧，我这显卡似乎16往上就不行了。



### --two-stems

设成 `--two-stems=vocals` 就可以分成人声和其他



### -n

可用的模型

- `mdx`: trained only on MusDB HQ, winning model on track A at the [MDX](https://www.aicrowd.com/challenges/music-demixing-challenge-ismir-2021) challenge.
- `mdx_extra`: trained with extra training data (including MusDB test set), ranked 2nd on the track B of the [MDX](https://www.aicrowd.com/challenges/music-demixing-challenge-ismir-2021) challenge.
- `mdx_q`, `mdx_extra_q`: quantized version of the previous models. Smaller download and storage but quality can be slightly worse. `mdx_extra_q` is the default model used.
- `SIG`: where `SIG` is a single model from the [model zoo](https://github.com/facebookresearch/demucs/blob/main/docs/training.md#model-zoo).

**Faster separation:** if you want faster separation, in particular if you do not have a GPU, you can use `-n 83fc094f` for instance to use a single model, as opposed to the bag of 4 models used for the competition.

这个默认就好了

## 音频转midi
### 安装
这个用的是tf，不过安装起来没啥难度，直接用pip就行，以下是我安装所用的所有代码。
```
git clone https://github.com/keums/icassp2022-vocal-transcription.git
cd icassp2022-vocal-transcription
conda create -n icassp python=3.8
conda activate icassp
pip install -r requirements.txt
cd src
python singing_transcription.py -i ../audio/test.wav  -o ../output
```
### 使用
没啥参数可言的，`-ot`参数设成midi和fps都无所谓，两个都能得到midi文件
```
[optional arguments]
  -i path_audio           Path to input audio file. (default: '../audio/pop1.wav')
  -o pathsave             Path to folder for saving .mid file (default: '../output')
  -ot output_type        (optional) Output type: midi or frame-level pitch score(fps) (default: 'midi')
```

## AcgmuseParser
这是个图形界面工具，没啥好介绍的，选则`midi`文件，然后点转换就行。

# 自动化
## 批处理
因为我不会写批处理，所以我直接用的python，也不复杂多少
具体代码如下，注释的地方需要根据自己的电脑配置自行修改
```python
import os
from sys import argv
import shutil

INPUT_MUSIC_PATH = os.path.abspath(argv[1])
MUSIC_PATH, MUSIC_NAME = os.path.split(INPUT_MUSIC_PATH)
print(INPUT_MUSIC_PATH)

os.chdir(os.path.split(__file__)[0])
# 下面的segment参数要根据自己的GPU去调，先试式不加这个参数
# 如果不能运行再加这个参数，具体根据前面文档里的介绍来调
os.system(f'conda activate demucs & demucs "{INPUT_MUSIC_PATH}" --segment 16 --two-stems=vocals')

SEPARATE_MUSIC_PATH = f'.\\separated/mdx_extra_q\\{MUSIC_NAME.rsplit(".", 1)[0]}'
SEPARATE_VOCAL_PATH = SEPARATE_MUSIC_PATH + '\\vocals.wav'
SEPARATE_NO_VOCAL_PATH = SEPARATE_MUSIC_PATH + '\\no_vocals.wav'

os.system(f'conda activate icassp & python .\\icassp2022-vocal-transcription\\src\\singing_transcription.py\
     -i "{SEPARATE_VOCAL_PATH}" -o "{MUSIC_PATH}" -ot fps')

shutil.move(
    SEPARATE_VOCAL_PATH,
    MUSIC_PATH + '\\vocals.wav'
)
shutil.move(
    SEPARATE_NO_VOCAL_PATH,
    MUSIC_PATH + '\\no_vocals.wav'
)

# with open(os.path.join(MUSIC_PATH, 'play_music.bat'), 'w') as f:
#     f.write(
#         '''
#         start cmd /k ffplay2 vocals.wav -nodisp
#         start cmd /k ffplay no_vocals.wav -nodisp
#         '''
#     )

os.system(".\\acg_parser\\AcgmuseParser.exe")

print('over')
```
将这个`py`文件(我命名为`process.py`)，与之前提到的三个项目文件夹放在同一目录，然后运行
```
python process.py path/to/your/music.mp3
```
就可以了。

## 环境变量
每次运行都要先cd到对应目录再`python ...`的十分不方便，所以还是添加到环境变量用得舒服。
在其他路径（这个路径要设成环境变量，请自行选择，就用同一目录也行。也可以考虑直接scoop shim）创建一个`cmd`文件（`bat`也行，两个后缀是一个东西，我命名为`process_music.cmd`），内容就一句话
```
# 记得将里面的路径改成你py文件的路径
@python "\path\to\your\process.py"  %*
```
然后将这个cmd文件加入环境变量即可。

## 运行
可以将音频文件直接拖拽到批处理文件上运行，也可以在音频目录下
```
process_music.cmd music.mp3
```
然后静候一杯水的功夫，就好了。