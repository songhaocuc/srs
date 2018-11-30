<link href="../../Style/note.css" rel="stylesheet"></link>

<!-- TOC -->

- [srs_app_config](#srsappconfig)
    - [class SrsConfDirective](#class-srsconfdirective)
        - [properties](#properties)
            - [+ int conf_line](#int-confline)
            - [+ std::string name](#stdstring-name)
            - [+ std::vector\<std::string\> args](#stdvectorstdstring-args)
        - [constructors](#constructors)
            - [+ SrsConfDirective()](#srsconfdirective)
            - [+ virtual ~SrsConfDirective()](#virtual-srsconfdirective)
        - [args](#args)
        - [directives](#directives)
        - [help utilities](#help-utilities)
        - [parse utilities](#parse-utilities)
        - [private parse](#private-parse)
    - [class SrsConfig](#class-srsconfig)
        - [user command](#user-command)
        - [golbal env variables](#golbal-env-variables)
        - [config section](#config-section)
        - [reload section](#reload-section)
        - [constructors](#constructors-1)
        - [dolphin](#dolphin)
        - [reload](#reload)
    - [[_srs_internal]class SrsConfigBuffer](#srsinternalclass-srsconfigbuffer)
    - [deep compare directive](#deep-compare-directive)
    - [helper utilities](#helper-utilities)
    - [global config](#global-config)

<!-- /TOC -->

# srs_app_config

## class SrsConfDirective
srs配置指令，配置文件将会被解析为一组指令。  
所有的指令都有name、args和child-directives。
```cpp
/**
* the config directive.
* the config file is a group of directives,
* all directive has name, args and child-directives.
* for example, the following config text:
        vhost vhost.ossrs.net {
            enabled         on;
            ingest livestream {
                enabled      on;
                ffmpeg       /bin/ffmpeg;
            }
        }
* will be parsed to:
*       SrsConfDirective: name="vhost", arg0="vhost.ossrs.net", child-directives=[
*           SrsConfDirective: name="enabled", arg0="on", child-directives=[]
*           SrsConfDirective: name="ingest", arg0="livestream", child-directives=[
*               SrsConfDirective: name="enabled", arg0="on", child-directives=[]
*               SrsConfDirective: name="ffmpeg", arg0="/bin/ffmpeg", child-directives=[]
*           ]
*       ]
* @remark, allow empty directive, for example: "dir0 {}"
* @remark, don't allow empty name, for example: ";" or "{dir0 arg0;}
*/
class SrsConfDirective
{
public:
    /**
    * the line of config file in which the directive from
    */
    int conf_line;
    /**
    * the name of directive, for example, the following config text:
    *       enabled     on;
    * will be parsed to a directive, its name is "enalbed"
    */
    std::string name;
    /**
    * the args of directive, for example, the following config text:
    *       listen      1935 1936;
    * will be parsed to a directive, its args is ["1935", "1936"].
    */
    std::vector<std::string> args;
    /**
    * the child directives, for example, the following config text:
    *       vhost vhost.ossrs.net {
    *           enabled         on;
    *       }
    * will be parsed to a directive, its directives is a vector contains 
    * a directive, which is:
    *       name:"enalbed", args:["on"], directives:[]
    * 
    * @remark, the directives can contains directives.
    */
    std::vector<SrsConfDirective*> directives;
public:
    SrsConfDirective();
    virtual ~SrsConfDirective();
// args
public:
    /**
    * get the args0,1,2, if user want to get more args,
    * directly use the args.at(index).
    */
    virtual std::string arg0();
    virtual std::string arg1();
    virtual std::string arg2();
// directives
public:
    /**
    * get the directive by index.
    * @remark, assert the index<directives.size().
    */
    virtual SrsConfDirective* at(int index);
    /**
    * get the directive by name, return the first match.
    */
    virtual SrsConfDirective* get(std::string _name);
    /**
    * get the directive by name and its arg0, return the first match.
    */
    virtual SrsConfDirective* get(std::string _name, std::string _arg0);
// help utilities
public:
    /**
    * whether current directive is vhost.
    */
    virtual bool is_vhost();
    /**
    * whether current directive is stream_caster.
    */
    virtual bool is_stream_caster();
// parse utilities
public:
    /**
    * parse config directive from file buffer.
    */
    virtual int parse(_srs_internal::SrsConfigBuffer* buffer);
// private parse.
private:
    /**
    * the directive parsing type.
    */
    enum SrsDirectiveType {
        /**
        * the root directives, parsing file.
        */
        parse_file, 
        /**
        * for each direcitve, parsing text block.
        */
        parse_block
    };
    /**
    * parse the conf from buffer. the work flow:
    * 1. read a token(directive args and a ret flag), 
    * 2. initialize the directive by args, args[0] is name, args[1-N] is args of directive,
    * 3. if ret flag indicates there are child-directives, read_conf(directive, block) recursively.
    */
    virtual int parse_conf(_srs_internal::SrsConfigBuffer* buffer, SrsDirectiveType type);
    /**
    * read a token from buffer.
    * a token, is the directive args and a flag indicates whether has child-directives.
    * @param args, the output directive args, the first is the directive name, left is the args.
    * @param line_start, the actual start line of directive.
    * @return, an error code indicates error or has child-directives.
    */
    virtual int read_token(_srs_internal::SrsConfigBuffer* buffer, std::vector<std::string>& args, int& line_start);
};
```
### properties
#### + int conf_line
#### + std::string name
#### + std::vector\<std::string\> args
### constructors
#### + SrsConfDirective()
#### + virtual ~SrsConfDirective()
### args
```cpp
// args
public:
    /**
    * get the args0,1,2, if user want to get more args,
    * directly use the args.at(index).
    */
    virtual std::string arg0();
    virtual std::string arg1();
    virtual std::string arg2();
```
### directives
```cpp
// directives
public:
    /**
    * get the directive by index.
    * @remark, assert the index<directives.size().
    */
    virtual SrsConfDirective* at(int index);
    /**
    * get the directive by name, return the first match.
    */
    virtual SrsConfDirective* get(std::string _name);
    /**
    * get the directive by name and its arg0, return the first match.
    */
    virtual SrsConfDirective* get(std::string _name, std::string _arg0);
```
### help utilities
```cpp
// help utilities
public:
    /**
    * whether current directive is vhost.
    */
    virtual bool is_vhost();
    /**
    * whether current directive is stream_caster.
    */
    virtual bool is_stream_caster();
```
### parse utilities
```cpp
// parse utilities
public:
    /**
    * parse config directive from file buffer.
    */
    virtual int parse(_srs_internal::SrsConfigBuffer* buffer);
```
### private parse
```cpp
// private parse.
private:
    /**
    * the directive parsing type.
    */
    enum SrsDirectiveType {
        /**
        * the root directives, parsing file.
        */
        parse_file, 
        /**
        * for each direcitve, parsing text block.
        */
        parse_block
    };
    /**
    * parse the conf from buffer. the work flow:
    * 1. read a token(directive args and a ret flag), 
    * 2. initialize the directive by args, args[0] is name, args[1-N] is args of directive,
    * 3. if ret flag indicates there are child-directives, read_conf(directive, block) recursively.
    */
    virtual int parse_conf(_srs_internal::SrsConfigBuffer* buffer, SrsDirectiveType type);
    /**
    * read a token from buffer.
    * a token, is the directive args and a flag indicates whether has child-directives.
    * @param args, the output directive args, the first is the directive name, left is the args.
    * @param line_start, the actual start line of directive.
    * @return, an error code indicates error or has child-directives.
    */
    virtual int read_token(_srs_internal::SrsConfigBuffer* buffer, std::vector<std::string>& args, int& line_start);
```

## class SrsConfig
```cpp
/**
* the config service provider.
* for the config supports reload, so never keep the reference cross st-thread,
* that is, never save the SrsConfDirective* get by any api of config,
* for it maybe free in the reload st-thread cycle.
* you can keep it before st-thread switch, or simply never keep it.
*/
```
### user command
```cpp
// user command
private:
    /**
     * whether srs is run in dolphin mode.
     * @see https://github.com/ossrs/srs-dolphin
     */
    bool dolphin;
    std::string dolphin_rtmp_port;
    std::string dolphin_http_port;
    /**
    * whether show help and exit.
    */
    bool show_help;
    /**
    * whether test config file and exit.
    */
    bool test_conf;
    /**
    * whether show SRS version and exit.
    */
    bool show_version;
```

### golbal env variables
```cpp
// global env variables.
private:
    /**
    * the user parameters, the argc and argv.
    * the argv is " ".join(argv), where argv is from main(argc, argv).
    */
    std::string _argv;
    /**
    * current working directory.
    */
    std::string _cwd;
```

### config section
```cpp
// config section
private:
    /**
    * the last parsed config file.
    * if reload, reload the config file.
    */
    std::string config_file;
    /**
    * the directive root.
    */
    SrsConfDirective* root;
```

### reload section
```cpp
// reload section
private:
    /**
    * the reload subscribers, when reload, callback all handlers.
    */
    std::vector<ISrsReloadHandler*> subscribes;
```
### constructors
```cpp
public:
    SrsConfig();
    virtual ~SrsConfig();
```

### dolphin
```cpp
// reload section
private:
    /**
    * the reload subscribers, when reload, callback all handlers.
    */
    std::vector<ISrsReloadHandler*> subscribes;
public:
    SrsConfig();
    virtual ~SrsConfig();
```

### reload
```cpp
// reload
public:
    /**
    * for reload handler to register itself,
    * when config service do the reload, callback the handler.
    */
    virtual void subscribe(ISrsReloadHandler* handler);
    /**
    * for reload handler to unregister itself.
    */
    virtual void unsubscribe(ISrsReloadHandler* handler);
    /**
    * reload the config file.
    * @remark, user can test the config before reload it.
    */
    virtual int reload();
```
关于reload部分的内容太多，而且大都雷同，通知所有订阅者进行更改。具体看到再仔细了解。

## [_srs_internal]class SrsConfigBuffer
```cpp
/**
* the buffer of config content.
*/
class SrsConfigBuffer
{
protected:
    // last available position.
    char* last;
    // end of buffer.
    char* end;
    // start of buffer.
    char* start;
public:
    // current consumed position.
    char* pos;
    // current parsed line.
    int line;
public:
    SrsConfigBuffer();
    virtual ~SrsConfigBuffer();
public:
    /**
    * fullfill the buffer with content of file specified by filename.
    */
    virtual int fullfill(const char* filename);
    /**
    * whether buffer is empty.
    */
    virtual bool empty();
};
```

## deep compare directive
```cpp
/**
* deep compare directive.
*/
extern bool srs_directive_equals(SrsConfDirective* a, SrsConfDirective* b);
```


## helper utilities
```cpp
/**
 * helper utilities, used for compare the consts values.
 */
extern bool srs_config_hls_is_on_error_ignore(std::string strategy);
extern bool srs_config_hls_is_on_error_continue(std::string strategy);
extern bool srs_config_ingest_is_file(std::string type);
extern bool srs_config_ingest_is_stream(std::string type);
extern bool srs_config_dvr_is_plan_segment(std::string plan);
extern bool srs_config_dvr_is_plan_session(std::string plan);
extern bool srs_config_dvr_is_plan_append(std::string plan);
extern bool srs_stream_caster_is_udp(std::string caster);
extern bool srs_stream_caster_is_rtsp(std::string caster);
extern bool srs_stream_caster_is_flv(std::string caster);
```

## global config
```cpp
extern SrsConfig* _srs_config;
```