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
int _level保存的是SrsLogLevel枚举类型的值。
#### - char* log_data
#### - int fd
#### - bool log_to_file_tank
#### - bool utc
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
#### + void verbose(const char* tag, int context_id, const char* fmt, ...)
#### + void info(const char* tag, int context_id, const char* fmt, ...)
#### + void trace(const char* tag, int context_id, const char* fmt, ...)
#### + void warn(const char* tag, int context_id, const char* fmt, ...)
#### + void error(const char* tag, int context_id, const char* fmt, ...)
#### - bool generate_header(bool error, const char* tag, int context_id, const char* level_name, int* header_size)
#### - void write_log(int& fd, char* str_log, int size, int level)
#### - void open_log_file()
### interface ISrsReloadHandler
#### + int on_reload_utc_time();
#### + int on_reload_log_tank();
#### + int on_reload_log_level();
#### + int on_reload_log_file();


