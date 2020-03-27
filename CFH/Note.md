- [崔福红的笔记本](#%e5%b4%94%e7%a6%8f%e7%ba%a2%e7%9a%84%e7%ac%94%e8%ae%b0%e6%9c%ac)
  - [下载代码](#%e4%b8%8b%e8%bd%bd%e4%bb%a3%e7%a0%81)
  - [编译其他](#%e7%bc%96%e8%af%91%e5%85%b6%e4%bb%96)
  - [GDB](#gdb)
  - [TCMalloc](#tcmalloc)
  - [编译](#%e7%bc%96%e8%af%91)
  - [运行](#%e8%bf%90%e8%a1%8c)
  - [环境变量](#%e7%8e%af%e5%a2%83%e5%8f%98%e9%87%8f)
  - [pprof](#pprof)
    - [例子](#%e4%be%8b%e5%ad%90)
  - [输出信息](#%e8%be%93%e5%87%ba%e4%bf%a1%e6%81%af)
    - [解释](#%e8%a7%a3%e9%87%8a)
  - [一些有用的正则表达式](#%e4%b8%80%e4%ba%9b%e6%9c%89%e7%94%a8%e7%9a%84%e6%ad%a3%e5%88%99%e8%a1%a8%e8%be%be%e5%bc%8f)
***
# 崔福红的笔记本
## 下载代码
> CN3.0  
- repo init -u ssh://cuifh@192.168.3.39:39999/manifest/cns3 && repo sync  
- repo forall -c git checkout VW.03.680304-00.20.10_00-CA  
- repo forall -c git checkout cns3_R_Lane_Demo_Branch  
- repo forall -c git checkout cns3_SVW-VW_SOP1  
- repo forall -c git checkout cns3_R_Lane_Demo  
- git push --progress "origin" cns3_develop:refs/for/cns3_develop  
- repo forall -c git reset --hard HEAD
> 37W
- repo init -u ssh://cuifh@192.168.3.39:39999/manifest/cns3   
- cd .repo/manifests  
- git checkout 37W_develop  
- cd ../..   
- repo sync   
- repo forall -c git checkout 37W_develop  
> MA
- repo init -u ssh://cuifh@192.168.3.39:39999/manifest/cns3  
- cd .repo/manifests  
- git checkout cns3_sop2_develop  
- cd ../..   
- repo sync  
- repo forall -c git branch cns3_sop2_F_alignment_MA
- origincns3_sop2_F_alignment_MA  
- repo forall -c git checkout cns3_sop2_F_alignment_MA  
***
## 编译其他
- make -f makefileLNXPC Sanitizer=address  
- make -f makefileLNX Sanitizer=address  
- valgrind  --num-callers=20 --leak-check=full --undef-value-errors=no --log-file=valgrind_output ./MXNavi  
- 要想有符号，你用未去符号的.dbg文件mv成可执行文件
***
## GDB
- . /opt/pcc-zr3-sdk/environment-setup-aarch64-linux-gnu  
- aarch64-linux-gnu-gdb MXNavi.dbg fatcore  
- set sysroot /opt/pcc-zr3-sdk/aarch64-linux-gnu
***
## TCMalloc
1. 确保/var/MXNavi下包含以下文件，不需要手动copy，确认即可
libprofiler.so libprofiler.so.0 libprofiler.so.0.4.18libtcmalloc.so libtcmalloc.so.4 libtcmalloc.so.4.5.3
    > mount-read-write.sh
2. 修改/usr/lib/systemd/system/MXNavi.service
    > Environment=HEAPPROFILE=/var/MXNavi/heap_dump  
    > Environment=HEAP_PROFILE_ALLOCATION_INTERVAL=0  
    > Environment=HEAP_PROFILE_INUSE_INTERVAL=52428800  
3. chmod -R 0777 /var/MXNavi
4. 重启
内存每增长一个阈值，tcmalloc会生成一个文件；测试完毕后把    heap_dump*拷出来
备注：默认情况下，每次上电后都会清空上一次的heap_dump。如想保   存上一次的dump，则需要设Environment=HEAP_PROFILE_CLEANUP=0，  但设置之后，前后两的dump会掺杂在一起，所以要特别注意  
## 编译
-fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free
-ltcmalloc

## 运行
HEAPPROFILE=profile_prefix your_program

## 环境变量
- HEAP_PROFILE_ALLOCATION_INTERVAL
default: 1073741824 (1 Gb)
累积alloc多少内存后，就生成profile文件

- HEAP_PROFILE_INUSE_INTERVAL
default: 104857600 (100 Mb)
使用的内存每增长多少，就生成profile文件

- HEAP_PROFILE_TIME_INTERVAL
default: 0
每隔多久，就生成profile文件

- HEAPPROFILESIGNAL
default: disabled
收到指定的信号，就生成profile文件

## pprof
pprof [options] program heap_profile_file
    
- --add_lib=xxx   Read additional symbols and line info from the given library
- --lib_prefix=xx Comma separated list of library path prefixes
- --tools=xxx     $PATH for object tool pathnames
- --text      生成文本文件
- --pdf           生成PDF文件
- --stacks    查看每一个点的栈
- --show_bytes  以字节为单位查看(默认M)
- --base=<base> 对比查看两个版本
- --lines     粒度为每一行
- --functions   粒度为每个函数(默认)

### 例子
-  google-pprof --tools=/opt/pcc-zr3-sdk-20200210/toolchain/bin/aarch64-linux-gnu- --lib_prefix=/opt/pcc-zr3-sdk-20200210/aarch64-linux-gnu/usr/lib/,/opt/pcc-zr3-sdk-20200210/toolchain/aarch64-linux-gnu/libc/lib,/opt/pcc-zr3-sdk-20200210/toolchain/aarch64-linux-gnu/libc/usr/lib --text --stacks --functions MXNavi.dbg heap_dump.0010.heap > dump.txt

- pprof --tools=/opt/pcc-zr3-sdk-0770/toolchain/bin/aarch64-linux-gnu- --lib_prefix=/opt/pcc-zr3-sdk-0770/aarch64-linux-gnu --text --base=fengxw.0260.heap MXNavi fengxw.0282.heap > 2.txt

## 输出信息
|   1   |   2   |   3   |   4   |   5   |                                          |
| :---: | :---: | :---: | :---: | :---: | ---------------------------------------- |
| 255.6 | 24.7% | 24.7% | 255.6 | 24.7% | GFS_MasterChunk::AddServer               |
| 184.6 | 17.8% | 42.5% | 298.8 | 28.8% | GFS_MasterChunkTable::Create             |
| 176.2 | 17.0% | 59.5% | 729.9 | 70.5% | GFS_MasterChunkTable::UpdateState        |
| 169.8 | 16.4% | 75.9% | 169.8 | 16.4% | PendingClone::PendingClone               |
| 76.3  | 7.4%  | 83.3% | 76.3  | 7.4%  | __default_alloc_template::_S_chunk_alloc |
| 49.5  | 4.8%  | 88.0% | 49.5  | 4.8%  | hashtable::resize                        |

### 解释

1. 直接使用的内存
2. 第一列的百分比
3. 第二列的累积和(Fibonacci)
4. 该点以及该点所有子函数所使用的内存
5. 第四列的百分比

## 一些有用的正则表达式
- ^\d{8,}\s+\(.\*\).\*$