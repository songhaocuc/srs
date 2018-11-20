<link href="../../Style/note.css" rel="stylesheet"></link>

<!-- TOC -->

- [1. kernel module](#1-kernel-module)
    - [1.1. ISrsLog* _srs_log](#11-isrslog-_srs_log)
    - [1.2. ISrsThreadContext* _srs_context](#12-isrsthreadcontext-_srs_context)
- [2. app module](#2-app-module)
    - [2.1. SrsConfig* _srs_config](#21-srsconfig-_srs_config)
    - [2.2. SrsServer* _srs_server](#22-srsserver-_srs_server)
- [3. macro features](#3-macro-features)
    - [3.1. void show_macro_features()](#31-void-show_macro_features)
    - [3.2. void check_macro_features()](#32-void-check_macro_features)
- [4. main entrance](#4-main-entrance)
    - [4.1. int main()](#41-int-main)
        - [4.1.1. endian检查](#411-endian检查)
        - [4.1.2. 解析](#412-解析)
        - [4.1.3. 修改cwd](#413-修改cwd)
        - [4.1.4. 初始化日志](#414-初始化日志)
        - [4.1.5. 日志trace一些信息](#415-日志trace一些信息)
        - [4.1.6. 检查宏特性](#416-检查宏特性)
        - [4.1.7. 服务器初始化并运行](#417-服务器初始化并运行)
    - [4.2. int run()](#42-int-run)
        - [4.2.1. 检测daemon进程](#421-检测daemon进程)
        - [4.2.2. 创建daemon进程](#422-创建daemon进程)
    - [4.3. int run_master()](#43-int-run_master)
        - [4.3.1. 初始化StateThreads](#431-初始化statethreads)
        - [4.3.2. 初始化signal_manager](#432-初始化signal_manager)
        - [4.3.3. 获取pid文件](#433-获取pid文件)
        - [4.3.4. 开始侦听](#434-开始侦听)
        - [4.3.5. 注册信号](#435-注册信号)
        - [4.3.6. HTTP处理](#436-http处理)
        - [4.3.7. 开始吸收](#437-开始吸收)
        - [4.3.8. 开始循环](#438-开始循环)

<!-- /TOC -->

# 1. kernel module

## 1.1. ISrsLog* _srs_log
``` cpp
ISrsLog* _srs_log = new SrsFastLog();
```

## 1.2. ISrsThreadContext* _srs_context
``` cpp
ISrsThreadContext* _srs_context = new SrsThreadContext();
```

# 2. app module

## 2.1. SrsConfig* _srs_config
``` cpp
SrsConfig* _srs_config = new SrsConfig();
```

## 2.2. SrsServer* _srs_server
``` cpp
SrsServer* _srs_server = new SrsServer();
```

# 3. macro features

## 3.1. void show_macro_features()

## 3.2. void check_macro_features()

# 4. main entrance

## 4.1. int main()

### 4.1.1. endian检查
``` cpp
bool srs_is_little_endian()
{
    // convert to network(big-endian) order, if not equals, 
    // the system is little-endian, so need to convert the int64
    static int little_endian_check = -1;
    
    if(little_endian_check == -1) {
        union {
            int32_t i;
            int8_t c;
        } little_check_union;
        
        little_check_union.i = 0x01;
        little_endian_check = little_check_union.c;
    }
    
    return (little_endian_check == 1);
}
```
检查系统字节序,网络字节序为大端，如果系统字节序为小端，需要进行转换。  
如果是小端，assert。
<p class="todo">linux和windows都是小端，可能有处理</p>

### 4.1.2. 解析
``` cpp
// never use srs log(srs_trace, srs_error, etc) before config parse the option,
// which will load the log config and apply it.
if ((ret = _srs_config->parse_options(argc, argv)) != ERROR_SUCCESS) {
    return ret;
}
```

### 4.1.3. 修改cwd
``` cpp
// change the work dir and set cwd.
string cwd = _srs_config->get_work_dir();
if (!cwd.empty() && cwd != "./" && (ret = chdir(cwd.c_str())) != ERROR_SUCCESS) {
    srs_error("change cwd to %s failed. ret=%d", cwd.c_str(), ret);
    return ret;
}
if ((ret = _srs_config->initialize_cwd()) != ERROR_SUCCESS) {
    return ret;
}
```

### 4.1.4. 初始化日志
``` cpp
// config parsed, initialize log.
if ((ret = _srs_log->initialize()) != ERROR_SUCCESS) {
    return ret;
}

// we check the config when the log initialized.
if ((ret = _srs_config->check_config()) != ERROR_SUCCESS) {
    return ret;
}
```
日志级别
+ verbose `log for verbose, very verbose information.`
+ info `log for debug, detail information.`
+ trace `log for trace, important information.`
+ warn `log for warn, warn is something should take attention, but not a error.`
+ error `log for error, something error occur, do something about the error, ie. close the connection, but we will donot abort the program.`

### 4.1.5. 日志trace一些信息
``` cpp
srs_trace(RTMP_SIG_SRS_SERVER", stable is "RTMP_SIG_SRS_PRIMARY);
srs_trace("license: "RTMP_SIG_SRS_LICENSE", "RTMP_SIG_SRS_COPYRIGHT);
srs_trace("primary/master: "RTMP_SIG_SRS_PRIMARY);
srs_trace("authors: "RTMP_SIG_SRS_AUTHROS);
srs_trace("contributors: "SRS_AUTO_CONSTRIBUTORS);
srs_trace("uname: "SRS_AUTO_UNAME);
srs_trace("build: %s, %s", SRS_AUTO_BUILD_DATE, srs_is_little_endian()? "little-endian":"big-endian");
srs_trace("configure: "SRS_AUTO_USER_CONFIGURE);
srs_trace("features: "SRS_AUTO_CONFIGURE);

srs_trace("conf: %s, limit: %d", _srs_config->config().c_str(), _srs_config->get_max_connections());
```

### 4.1.6. 检查宏特性
``` cpp
// features
check_macro_features();
show_macro_features();
```

### 4.1.7. 服务器初始化并运行
``` cpp
/**
* we do nothing in the constructor of server,
* and use initialize to create members, set hooks for instance the reload handler,
* all initialize will done in this stage.
*/
if ((ret = _srs_server->initialize(NULL)) != ERROR_SUCCESS) {
    return ret;
}

return run();
```
在服务器的构造函数中什么都不做，使用initialize来创建成员、为reload handler实例设置hooks。

## 4.2. int run()

### 4.2.1. 检测daemon进程
Unix/Linux中的<a href="https://www.cnblogs.com/minico/p/7702020.html">daemon进程</a>类似于Windows中的后台服务进程，一直在后台运行运行，例如http服务进程nginx，ssh服务进程sshd等。
注意，其英文拼写为daemon而不是deamon。
``` cpp
// if not deamon, directly run master.
if (!_srs_config->get_deamon()) {
    return run_master();
}
```
如果不是虚拟光驱，直接运行 `run_master();`

### 4.2.2. 创建daemon进程
如果是后台进程，创建后台进程。
``` cpp
srs_trace("start deamon mode...");

int pid = fork();

if(pid < 0) {
    srs_error("create process error. ret=-1"); //ret=0
    return -1;
}

// grandpa
if(pid > 0) {
    int status = 0;
    if(waitpid(pid, &status, 0) == -1) {
        srs_error("wait child process error! ret=-1"); //ret=0
    }
    srs_trace("grandpa process exit.");
    exit(0);
}

// father
pid = fork();

if(pid < 0) {
    srs_error("create process error. ret=0");
    return -1;
}

if(pid > 0) {
    srs_trace("father process exit. ret=0");
    exit(0);
}

// son
srs_trace("son(deamon) process running.");

return run_master();
```
<p class="todo">这些和linux系统的知识有关，先略过</p>

## 4.3. int run_master()

### 4.3.1. 初始化StateThreads
http://state-threads.sourceforge.net/
``` cpp
if ((ret = _srs_server->initialize_st()) != ERROR_SUCCESS) {
    return ret;
}
```

### 4.3.2. 初始化signal_manager
``` cpp
if ((ret = _srs_server->initialize_signal()) != ERROR_SUCCESS) {
    return ret;
}
```

### 4.3.3. 获取pid文件
<a href="https://blog.csdn.net/yinqingwang/article/details/52841744">linux/unix下 pid文件作用浅析</a>
``` cpp
if ((ret = _srs_server->acquire_pid_file()) != ERROR_SUCCESS) {
    return ret;
}
```

### 4.3.4. 开始侦听
``` cpp
if ((ret = _srs_server->listen()) != ERROR_SUCCESS) {
    return ret;
}
```

### 4.3.5. 注册信号
``` cpp
if ((ret = _srs_server->register_signal()) != ERROR_SUCCESS) {
    return ret;
}
```

### 4.3.6. HTTP处理
``` cpp
if ((ret = _srs_server->http_handle()) != ERROR_SUCCESS) {
    return ret;
}
```

### 4.3.7. 开始吸收 
<p class="todo">搞明白再写</p>

``` cpp
if ((ret = _srs_server->listen()) != ERROR_SUCCESS) {
    return ret;
}
```

### 4.3.8. 开始循环
``` cpp
if ((ret = _srs_server->cycle()) != ERROR_SUCCESS) {
    return ret;
}
```
