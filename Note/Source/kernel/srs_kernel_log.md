<link href="../../Style/note.css" rel="stylesheet"></link>
srs_kernel_log文件中主要是log和threadcontext的接口，以及print方法的定义。具体实现为空。
<!-- TOC -->

- [1. srs_kernel_log](#1-srs_kernel_log)
    - [1.1. class SrsLogLevel](#11-class-srsloglevel)
    - [1.2. class ISrsLog](#12-class-isrslog)
    - [1.3. class ISrsThreadContext](#13-class-isrsthreadcontext)
    - [1.4. instance](#14-instance)
    - [1.5. print method](#15-print-method)

<!-- /TOC -->

# 1. srs_kernel_log

## 1.1. class SrsLogLevel
```cpp
/**
* the log level, for example:
* if specified Debug level, all level messages will be logged.
* if specified Warn level, only Warn/Error/Fatal level messages will be logged.
*/
class SrsLogLevel
{
public:
    // only used for very verbose debug, generally, 
    // we compile without this level for high performance.
    static const int Verbose = 0x01;
    static const int Info = 0x02;
    static const int Trace = 0x03;
    static const int Warn = 0x04;
    static const int Error = 0x05;
    // specified the disabled level, no log, for utest.
    static const int Disabled = 0x06;
};
```

## 1.2. class ISrsLog
```cpp
/**
* the log interface provides method to write log.
* but we provides some macro, which enable us to disable the log when compile.
* @see also SmtDebug/SmtTrace/SmtWarn/SmtError which is corresponding to Debug/Trace/Warn/Fatal.
*/ 
class ISrsLog
{
public:
    ISrsLog();
    virtual ~ISrsLog();
public:
    /**
    * initialize log utilities.
    */
    virtual int initialize();
public:
    /**
    * log for verbose, very verbose information.
    */
    virtual void verbose(const char* tag, int context_id, const char* fmt, ...);
    /**
    * log for debug, detail information.
    */
    virtual void info(const char* tag, int context_id, const char* fmt, ...);
    /**
    * log for trace, important information.
    */
    virtual void trace(const char* tag, int context_id, const char* fmt, ...);
    /**
    * log for warn, warn is something should take attention, but not a error.
    */
    virtual void warn(const char* tag, int context_id, const char* fmt, ...);
    /**
    * log for error, something error occur, do something about the error, ie. close the connection,
    * but we will donot abort the program.
    */
    virtual void error(const char* tag, int context_id, const char* fmt, ...);
};
```
ISrsLog是srs的log接口。srs_app_log中的SrsFastLog继承了该接口。


## 1.3. class ISrsThreadContext
```cpp
/**
 * the context id manager to identify context, for instance, the green-thread.
 * usage:
 *      _srs_context->generate_id(); // when thread start.
 *      _srs_context->get_id(); // get current generated id.
 *      int old_id = _srs_context->set_id(1000); // set context id if need to merge thread context.
 */
// the context for multiple clients.
class ISrsThreadContext
{
public:
    ISrsThreadContext();
    virtual ~ISrsThreadContext();
public:
    /**
     * generate the id for current context.
     */
    virtual int generate_id();
    /**
     * get the generated id of current context.
     */
    virtual int get_id();
    /**
     * set the id of current context.
     * @return the previous id value; 0 if no context.
     */
    virtual int set_id(int v);
};
```
ISrsThreadContext是srs中的线程上下文类。srs_app_log.hpp中的SrsThreadContext实现该接口。

## 1.4. instance
```cpp
// user must provides a log object
extern ISrsLog* _srs_log;

// user must implements the LogContext and define a global instance.
extern ISrsThreadContext* _srs_context;
```
声明了_srs_log和_srs_context两个全局变量，在srs_main_server中进行定义。

## 1.5. print method
```cpp
#define srs_verbose(msg, ...) _srs_log->verbose(NULL, _srs_context->get_id(), msg, ##__VA_ARGS__)
#define srs_info(msg, ...)    _srs_log->info(NULL, _srs_context->get_id(), msg, ##__VA_ARGS__)
#define srs_trace(msg, ...)   _srs_log->trace(NULL, _srs_context->get_id(), msg, ##__VA_ARGS__)
#define srs_warn(msg, ...)    _srs_log->warn(NULL, _srs_context->get_id(), msg, ##__VA_ARGS__)
#define srs_error(msg, ...)   _srs_log->error(NULL, _srs_context->get_id(), msg, ##__VA_ARGS__)
```