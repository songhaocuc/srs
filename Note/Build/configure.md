<link href="../Style/note.css" rel="stylesheet"></link>

# configure
编译的时候出现了一些错误，感觉还是需要先看一下configure文件，了解一下整个编译流程比较好。

## main output dir
定义主要的输出路径。  
```shell
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
```shell
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
首先执行[`auto/options.sh`](./auto/options.md)解析并检查configure后的参数。  
然后执行`auto/setup_variables.sh`根据