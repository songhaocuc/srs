<link href="../Style/note.css" rel="stylesheet"></link>

<!-- TOC -->

- [configure](#configure)
    - [main output dir](#main-output-dir)
    - [parser user options](#parser-user-options)
    - [generate Makefile.](#generate-makefile)
    - [](#)

<!-- /TOC -->

# configure
编译的时候出现了一些错误，感觉还是需要先看一下configure文件，了解一下整个编译流程比较好。

## main output dir
定义主要的输出路径。  
```sh
#####################################################################################
# the main output dir, all configure and make output are in this dir.
#####################################################################################
# create the main objs
SRS_WORKDIR="."
SRS_OBJS_DIR="objs"
SRS_OBJS="${SRS_WORKDIR}/${SRS_OBJS_DIR}"
SRS_MAKEFILE="Makefile"

# linux shell color support.
RED="\\033[31m"
GREEN="\\033[32m"
YELLOW="\\033[33m"
BLACK="\\033[0m"
```
`SRS_WORKDIR="."`是当前目录，即"trunk/"。   


## parser user options
```sh
#####################################################################################
# parse user options, set the variables like:
# srs features: SRS_SSL/SRS_HLS/SRS_NGINX/SRS_FFMPEG_TOOL/SRS_HTTP_CALLBACK/......
# build options: SRS_JOBS
#####################################################################################
# parse options, exit with error when parse options invalid.
. auto/options.sh

# setup variables when options parsed.
. auto/setup_variables.sh

# clean the exists, when not export srs-librtmp.
# do this only when the options is ok.
if [[ -f Makefile ]]; then
make clean
fi
# remove makefile
rm -f ${SRS_WORKDIR}/${SRS_MAKEFILE}

# create objs
mkdir -p ${SRS_OBJS}

# for export srs-librtmp, change target to it.
. auto/generate-srs-librtmp-project.sh

# apply user options.
. auto/depends.sh

# the auto generated variables.
. auto/auto_headers.sh
```
首先执行[`auto/options.sh`](./auto/options.md)解析并检查configure后的参数。根据参数定义宏变量（全局）。  
然后执行`auto/setup_variables.sh`初始化关于系统的变量。   
如果Makefile存在就执行make clean  
然后移除Makefile文件  
创建objs目录，[关于mkdir](https://www.cnblogs.com/ayseeing/p/4313956.html)  
如果用户指定了generate-srs-librtmp-project选项，那么编译将只产生librtmp库  
depends.sh配置了依赖项。  
auto_headers.sh根据options自动生成srs_auto_headers.hpp  

## generate Makefile.
```sh
#####################################################################################
# generate Makefile.
#####################################################################################
# ubuntu echo in Makefile cannot display color, use bash instead
SRS_BUILD_SUMMARY="_srs_build_summary.sh"

# srs-librtmp sample entry
SrsLibrtmpSampleEntry="nossl"
if [ $SRS_SSL = YES ]; then SrsLibrtmpSampleEntry="ssl";fi
# utest make entry, (cd utest; make)
SrsUtestMakeEntry="@echo -e \"ignore utest for it's disabled\""
if [ $SRS_UTEST = YES ]; then SrsUtestMakeEntry="(cd ${SRS_OBJS_DIR}/utest; \$(MAKE))"; fi
```
## 