<link href="../../Style/note.css" rel="stylesheet"></link>
定义类SrsFastLog和SrsThreadContext，实现srs_kernel_log文件中ISrsLog和ISrsThreadContext的接口。
<!-- TOC -->

- [srs_app_log](#srsapplog)
    - [class SrsThreadContext](#class-srsthreadcontext)
        - [base](#base)
            - [+ ISrsThreadContext](#isrsthreadcontext)
        - [property](#property)
            - [- std::map<st_thread_t, int> cache;](#stdmapstthreadt-int-cache)
        - [method](#method)
            - [+ int generate_id()](#int-generateid)
            - [+ int get_id()](#int-getid)
            - [+ int set_id(int v)](#int-setidint-v)
            - [+ void clear_cid()](#void-clearcid)
    - [macros](#macros)
    - [class SrsFastLog](#class-srsfastlog)
        - [base](#base-1)
            - [+ ISrsLog](#isrslog)
            - [+ ISrsReloadHandler](#isrsreloadhandler)
        - [property](#property-1)
            - [# int _level](#int-level)
            - [- char* log_data](#char-logdata)
            - [- int fd](#int-fd)
            - [- bool log_to_file_tank](#bool-logtofiletank)
            - [- bool utc](#bool-utc)
        - [method](#method-1)
            - [+ SrsFastLog()](#srsfastlog)
            - [+ virtual ~SrsFastLog()](#virtual-srsfastlog)
            - [+ int initialize()](#int-initialize)
            - [+ void verbose(const char* tag, int context_id, const char* fmt, ...)](#void-verboseconst-char-tag-int-contextid-const-char-fmt)
            - [+ void info(const char* tag, int context_id, const char* fmt, ...)](#void-infoconst-char-tag-int-contextid-const-char-fmt)
            - [+ void trace(const char* tag, int context_id, const char* fmt, ...)](#void-traceconst-char-tag-int-contextid-const-char-fmt)
            - [+ void warn(const char* tag, int context_id, const char* fmt, ...)](#void-warnconst-char-tag-int-contextid-const-char-fmt)
            - [+ void error(const char* tag, int context_id, const char* fmt, ...)](#void-errorconst-char-tag-int-contextid-const-char-fmt)
            - [- bool generate_header(bool error, const char* tag, int context_id, const char* level_name, int* header_size)](#bool-generateheaderbool-error-const-char-tag-int-contextid-const-char-levelname-int-headersize)
            - [- void write_log(int& fd, char* str_log, int size, int level)](#void-writelogint-fd-char-strlog-int-size-int-level)
            - [- void open_log_file()](#void-openlogfile)
        - [interface ISrsReloadHandler](#interface-isrsreloadhandler)
            - [+ int on_reload_utc_time();](#int-onreloadutctime)
            - [+ int on_reload_log_tank();](#int-onreloadlogtank)
            - [+ int on_reload_log_level();](#int-onreloadloglevel)
            - [+ int on_reload_log_file();](#int-onreloadlogfile)

<!-- /TOC -->

# srs_app_log

## class SrsThreadContext 
```cpp
/**
* st thread context, get_id will get the st-thread id, 
* which identify the client.
*/
class SrsThreadContext : public ISrsThreadContext
{
private:
    std::map<st_thread_t, int> cache;
public:
    SrsThreadContext();
    virtual ~SrsThreadContext();
public:
    virtual int generate_id();
    virtual int get_id();
    virtual int set_id(int v);
public:
    virtual void clear_cid();
};
```
### base
#### + ISrsThreadContext
ISrsThreadContext是在srs_kernel_log中定义的接口。  
接口方法包括 generate_id()、get_id()和set_id()。
### property
#### - std::map<st_thread_t, int> cache;

### method
#### + int generate_id()
```cpp
int SrsThreadContext::generate_id()
{
    static int id = 100;
    
    int gid = id++;
    cache[st_thread_self()] = gid;
    return gid;
}
```
#### + int get_id()
```cpp
int SrsThreadContext::get_id()
{
    return cache[st_thread_self()];
}
```
#### + int set_id(int v)
```cpp
int SrsThreadContext::set_id(int v)
{
    st_thread_t self = st_thread_self();
    
    int ov = 0;
    if (cache.find(self) != cache.end()) {
        ov = cache[self];
    }
    
    cache[self] = v;
    
    return ov;
}
```
#### + void clear_cid()
```cpp
void SrsThreadContext::clear_cid()
{
    st_thread_t self = st_thread_self();
    std::map<st_thread_t, int>::iterator it = cache.find(self);
    if (it != cache.end()) {
        cache.erase(it);
    }
}
```

## macros
```cpp
// the max size of a line of log.
#define LOG_MAX_SIZE 4096

// the tail append to each log.
#define LOG_TAIL '\n'
// reserved for the end of log data, it must be strlen(LOG_TAIL)
#define LOG_TAIL_SIZE 1
```
LOG_MAX_SIZE定义一行日志的最大长度。  
LOG_TAIL定义换行符(每条日志结尾)。  
LOG_TAIL_SIZE为日志结束符的长度。

## class SrsFastLog
```cpp
/**
* we use memory/disk cache and donot flush when write log.
* it's ok to use it without config, which will log to console, and default trace level.
* when you want to use different level, override this classs, set the protected _level.
*/
class SrsFastLog : public ISrsLog, public ISrsReloadHandler
{
// for utest to override
protected:
    // defined in SrsLogLevel.
    int _level;
private:
    char* log_data;
    // log to file if specified srs_log_file
    int fd;
    // whether log to file tank
    bool log_to_file_tank;
    // whether use utc time.
    bool utc;
public:
    SrsFastLog();
    virtual ~SrsFastLog();
public:
    virtual int initialize();
    virtual void verbose(const char* tag, int context_id, const char* fmt, ...);
    virtual void info(const char* tag, int context_id, const char* fmt, ...);
    virtual void trace(const char* tag, int context_id, const char* fmt, ...);
    virtual void warn(const char* tag, int context_id, const char* fmt, ...);
    virtual void error(const char* tag, int context_id, const char* fmt, ...);
// interface ISrsReloadHandler.
public:
    virtual int on_reload_utc_time();
    virtual int on_reload_log_tank();
    virtual int on_reload_log_level();
    virtual int on_reload_log_file();
private:
    virtual bool generate_header(bool error, const char* tag, int context_id, const char* level_name, int* header_size);
    virtual void write_log(int& fd, char* str_log, int size, int level);
    virtual void open_log_file();
};
```
### base
#### + ISrsLog
ISrsLog是定义在srs_kernel_log的日志接口类。  
接口方法包括初始化的initialize()方法和五种不同等级的日志打印方法。
#### + ISrsReloadHandler
ISrsReloadHandler是定义在srs_app_reload中的reload接口类，用于配置更新时重新加载服务器。  
该类是观察者类，SrsConfig是被观察者。SrsConfig中包含保存该类指针的向量，应该是用于重新加载时更新配置。
<p class="todo">粗略看了一下， 之后看一下所有继承这个的类</p>

### property
#### # int _level
int _level保存的是SrsLogLevel枚举类型的值。默认为SrsLogLevel::Trace。
#### - char* log_data
`log_data = new char[LOG_MAX_SIZE];`初始化了log_data，log_data是日志的缓冲区，大小为宏定义的每条日志的最大长度。
#### - int fd
指定日志文件之后，是否写入日志文件的标志。默认为-1.
#### - bool log_to_file_tank
是否把日志写入file tank? <p class="todo"/>
#### - bool utc
是否使用UTC时间。协调世界时，又称世界统一时间、世界标准时间、国际协调时间。由于英文（CUT）和法文（TUC）的缩写不同，作为妥协，简称UTC。
### method
#### + SrsFastLog()
```cpp
SrsFastLog::SrsFastLog()
{
    _level = SrsLogLevel::Trace;
    log_data = new char[LOG_MAX_SIZE];

    fd = -1;
    log_to_file_tank = false;
    utc = false;
}
```
SrsFastLog的构造函数。构造函数中对成员变量做了初始化工作，并没有真正的初始化日志所需的所有内容，这部分工作放在initialize()方法中。
#### + virtual ~SrsFastLog()
```cpp
SrsFastLog::~SrsFastLog()
{
    srs_freepa(log_data);

    if (fd > 0) {
        ::close(fd);
        fd = -1;
    }

    if (_srs_config) {
        _srs_config->unsubscribe(this);
    }
}
```
SrsFastLog的析构函数。  
srs_freepa()是srs的[srs_core](../core/srs_core.md#free-points)中定义的自动释放数组指针的方法宏。释放非数组指针的方法为srs_freep()。  
在析构函数中释放了构造函数中new的日志缓冲区。
close(fd)释放了日志文件句柄。  
解除订阅srs_config，这是观察者设计模式中的内容。在订阅srs_config之后，srs_config改动后会通知其所有订阅者并使其执行相应任务，在析构前如果不解除订阅，srs_config发布消息时会因为空指针而报错。<p class="todo">推测</p>
#### + int initialize()
```cpp
int SrsFastLog::initialize()
{
    int ret = ERROR_SUCCESS;
    
    if (_srs_config) {
        _srs_config->subscribe(this);
    
        log_to_file_tank = _srs_config->get_log_tank_file();
        _level = srs_get_log_level(_srs_config->get_log_level());
        utc = _srs_config->get_utc_time();
    }
    
    return ret;
}
```

initlialize()方法中进行了打印日志前所需的准备工作。  
首先订阅srs_config，使得在srs配置改变时自己也做出相应的调整。  
然后根据真正的配置来修改自身的成员变量。


#### + void verbose(const char* tag, int context_id, const char* fmt, ...)
```cpp
void SrsFastLog::verbose(const char* tag, int context_id, const char* fmt, ...)
{
    if (_level > SrsLogLevel::Verbose) {
        return;
    }
    
    int size = 0;
    if (!generate_header(false, tag, context_id, "verb", &size)) {
        return;
    }
    
    va_list ap;
    va_start(ap, fmt);
    // we reserved 1 bytes for the new line.
    size += vsnprintf(log_data + size, LOG_MAX_SIZE - size, fmt, ap);
    va_end(ap);

    write_log(fd, log_data, size, SrsLogLevel::Verbose);
}
```

首先进行日志类型的判断。SrsLogLevel中的日志类型为static int，因此可以进行比较。   
[generate_header](#bool-generateheaderbool-error-const-char-tag-int-contextid-const-char-levelname-int-headersize)用于产生日志头并利用传指针的方式返回日志头长度。   
[va_list](https://blog.csdn.net/qingshui23/article/details/58586545)用于C语言中解决变参问题。   
```cpp
void va_start ( va_list ap, param );
//对va_list变量进行初始化，将ap指针指向参数列表中的第一个参数
type va_arg ( va_list ap, type ); 
//获取参数，类型为 type 类型，返回值也为 type 类型
int vsprintf(char *string, char *format, va_list ap);
//将ap(通常是字符串) 按format格式写入字符串string中
void va_end ( va_list ap ); 
//回收ap指针
--------------------- 
原文：https://blog.csdn.net/qingshui23/article/details/58586545 
```
[write_log](#void-writelogint-fd-char-strlog-int-size-int-level)将日志写入文件中。
#### + void info(const char* tag, int context_id, const char* fmt, ...)
和[verbose](#void-verboseconst-char-tag-int-contextid-const-char-fmt)相同，只是日志等级不同。
#### + void trace(const char* tag, int context_id, const char* fmt, ...)
和[verbose](#void-verboseconst-char-tag-int-contextid-const-char-fmt)相同，只是日志等级不同。
#### + void warn(const char* tag, int context_id, const char* fmt, ...)
和[verbose](#void-verboseconst-char-tag-int-contextid-const-char-fmt)相同，只是日志等级不同。
#### + void error(const char* tag, int context_id, const char* fmt, ...)
和[verbose](#void-verboseconst-char-tag-int-contextid-const-char-fmt)相同，只是日志等级不同。
#### - bool generate_header(bool error, const char* tag, int context_id, const char* level_name, int* header_size)

```cpp
bool SrsFastLog::generate_header(bool error, const char* tag, int context_id, const char* level_name, int* header_size)
{
    // clock time
    timeval tv;
    if (gettimeofday(&tv, NULL) == -1) {
        return false;
    }
    
    // to calendar time
    struct tm* tm;
    if (utc) {
        if ((tm = gmtime(&tv.tv_sec)) == NULL) {
            return false;
        }
    } else {
        if ((tm = localtime(&tv.tv_sec)) == NULL) {
            return false;
        }
    }
    
    // write log header
    int log_header_size = -1;
    
    if (error) {
        if (tag) {
            log_header_size = snprintf(log_data, LOG_MAX_SIZE, 
                "[%d-%02d-%02d %02d:%02d:%02d.%03d][%s][%s][%d][%d][%d] ", 
                1900 + tm->tm_year, 1 + tm->tm_mon, tm->tm_mday, tm->tm_hour, tm->tm_min, tm->tm_sec, (int)(tv.tv_usec / 1000), 
                level_name, tag, getpid(), context_id, errno);
        } else {
            log_header_size = snprintf(log_data, LOG_MAX_SIZE, 
                "[%d-%02d-%02d %02d:%02d:%02d.%03d][%s][%d][%d][%d] ", 
                1900 + tm->tm_year, 1 + tm->tm_mon, tm->tm_mday, tm->tm_hour, tm->tm_min, tm->tm_sec, (int)(tv.tv_usec / 1000), 
                level_name, getpid(), context_id, errno);
        }
    } else {
        if (tag) {
            log_header_size = snprintf(log_data, LOG_MAX_SIZE, 
                "[%d-%02d-%02d %02d:%02d:%02d.%03d][%s][%s][%d][%d] ", 
                1900 + tm->tm_year, 1 + tm->tm_mon, tm->tm_mday, tm->tm_hour, tm->tm_min, tm->tm_sec, (int)(tv.tv_usec / 1000), 
                level_name, tag, getpid(), context_id);
        } else {
            log_header_size = snprintf(log_data, LOG_MAX_SIZE, 
                "[%d-%02d-%02d %02d:%02d:%02d.%03d][%s][%d][%d] ", 
                1900 + tm->tm_year, 1 + tm->tm_mon, tm->tm_mday, tm->tm_hour, tm->tm_min, tm->tm_sec, (int)(tv.tv_usec / 1000), 
                level_name, getpid(), context_id);
        }
    }

    if (log_header_size == -1) {
        return false;
    }
    
    // write the header size.
    *header_size = srs_min(LOG_MAX_SIZE - 1, log_header_size);
    
    return true;
}
```
用于产生日志头。
#### - void write_log(int& fd, char* str_log, int size, int level)
```cpp
void SrsFastLog::write_log(int& fd, char *str_log, int size, int level)
{
    // ensure the tail and EOF of string
    //      LOG_TAIL_SIZE for the TAIL char.
    //      1 for the last char(0).
    size = srs_min(LOG_MAX_SIZE - 1 - LOG_TAIL_SIZE, size);
    
    // add some to the end of char.
    str_log[size++] = LOG_TAIL;
    
    // if not to file, to console and return.
    if (!log_to_file_tank) {
        // if is error msg, then print color msg.
        // \033[31m : red text code in shell
        // \033[32m : green text code in shell
        // \033[33m : yellow text code in shell
        // \033[0m : normal text code
        if (level <= SrsLogLevel::Trace) {
            printf("%.*s", size, str_log);
        } else if (level == SrsLogLevel::Warn) {
            printf("\033[33m%.*s\033[0m", size, str_log);
        } else{
            printf("\033[31m%.*s\033[0m", size, str_log);
        }
        fflush(stdout);

        return;
    }
    
    // open log file. if specified
    if (fd < 0) {
        open_log_file();
    }
    
    // write log to file.
    if (fd > 0) {
        ::write(fd, str_log, size);
    }
}
```
将日志写入文件。

#### - void open_log_file()
```cpp
void SrsFastLog::open_log_file()
{
    if (!_srs_config) {
        return;
    }
    
    std::string filename = _srs_config->get_log_file();
    
    if (filename.empty()) {
        return;
    }
    
    fd = ::open(filename.c_str(), O_RDWR | O_APPEND);
    
    if(fd == -1 && errno == ENOENT) {
        fd = open(filename.c_str(), 
            O_RDWR | O_CREAT | O_TRUNC, 
            S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH
        );
    }
}
```

### interface ISrsReloadHandler
下列函数是从ISrsReloadHandler接口中继承来的方法。
#### + int on_reload_utc_time();
```cpp
int SrsFastLog::on_reload_utc_time()
{
    utc = _srs_config->get_utc_time();
    
    return ERROR_SUCCESS;
}
```
#### + int on_reload_log_tank();
```cpp
int SrsFastLog::on_reload_log_tank()
{
    int ret = ERROR_SUCCESS;
    
    if (!_srs_config) {
        return ret;
    }

    bool tank = log_to_file_tank;
    log_to_file_tank = _srs_config->get_log_tank_file();

    if (tank) {
        return ret;
    }

    if (!log_to_file_tank) {
        return ret;
    }

    if (fd > 0) {
        ::close(fd);
    }
    open_log_file();
    
    return ret;
}
```
#### + int on_reload_log_level();
```cpp
int SrsFastLog::on_reload_log_level()
{
    int ret = ERROR_SUCCESS;
    
    if (!_srs_config) {
        return ret;
    }
    
    _level = srs_get_log_level(_srs_config->get_log_level());
    
    return ret;
}
```
#### + int on_reload_log_file();
```cpp
int SrsFastLog::on_reload_log_file()
{
    int ret = ERROR_SUCCESS;
    
    if (!_srs_config) {
        return ret;
    }

    if (!log_to_file_tank) {
        return ret;
    }

    if (fd > 0) {
        ::close(fd);
    }
    open_log_file();
    
    return ret;
}
```


