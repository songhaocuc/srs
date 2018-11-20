<link href="../../Style/note.css" rel="stylesheet"></link>

<!-- TOC -->

- [1. srs_app_server](#1-srs_app_server)
    - [1.1. enum SrsListenerType](#11-enum-srslistenertype)
    - [1.2. class SrsListener](#12-class-srslistener)
    - [1.3. class SrsStreamListener](#13-class-srsstreamlistener)
    - [1.4. <p class="todo">rtsp/flv listener</p>](#14-p-classtodortspflv-listenerp)
    - [1.5. class SrsUdpStreamListener](#15-class-srsudpstreamlistener)
    - [1.6. <p class="todo">udp caster listener</p>](#16-p-classtodoudp-caster-listenerp)
    - [1.7. class SrsSignalManager](#17-class-srssignalmanager)
    - [1.8. class ISrsServerCycle](#18-class-isrsservercycle)
    - [1.9. class SrsServer](#19-class-srsserver)

<!-- /TOC -->

# 1. srs_app_server
## 1.1. enum SrsListenerType
```cpp
// listener type for server to identify the connection,
// that is, use different type to process the connection.
enum SrsListenerType 
{
    // RTMP client,
    SrsListenerRtmpStream       = 0,
    // HTTP api,
    SrsListenerHttpApi          = 1,
    // HTTP stream, HDS/HLS/DASH
    SrsListenerHttpStream       = 2,
    // UDP stream, MPEG-TS over udp.
    SrsListenerMpegTsOverUdp    = 3,
    // TCP stream, RTSP stream.
    SrsListenerRtsp             = 4,
    // TCP stream, FLV stream over HTTP.
    SrsListenerFlv              = 5,
};
```
枚举类型 SrsListenerType。


## 1.2. class SrsListener
```cpp
/**
* the common tcp listener, for RTMP/HTTP server.
*/
class SrsListener
{
protected:
    SrsListenerType type;
protected:
    std::string ip;
    int port;
    SrsServer* server;
public:
    SrsListener(SrsServer* svr, SrsListenerType t);
    virtual ~SrsListener();
public:
    virtual SrsListenerType listen_type();
    virtual int listen(std::string i, int p) = 0;
};
```
纯虚类 SrsListener，主要用于tcp连接的server，例如RTMP和HTTPserver。  
<br>
保存了ip地址、端口号以及SrsServer指针。  
使用SrsServer以及listener类型进行初始化。  
SrsListenerType返回listener的类型。  
<p class="todo">之后补上具体说明</p>
listen为纯虚函数，参数为string的ip地址以及int类型的端口号。

## 1.3. class SrsStreamListener
```cpp
/**
* tcp listener.
*/
class SrsStreamListener : virtual public SrsListener, virtual public ISrsTcpHandler
{
private:
    SrsTcpListener* listener;
public:
    SrsStreamListener(SrsServer* server, SrsListenerType type);
    virtual ~SrsStreamListener();
public:
    virtual int listen(std::string ip, int port);
// ISrsTcpHandler
public:
    virtual int on_tcp_client(st_netfd_t stfd);
};
```

## 1.4. <p class="todo">rtsp/flv listener</p>

## 1.5. class SrsUdpStreamListener
```cpp
/**
 * the udp listener, for udp server.
 */
class SrsUdpStreamListener : public SrsListener
{
protected:
    SrsUdpListener* listener;
    ISrsUdpHandler* caster;
public:
    SrsUdpStreamListener(SrsServer* svr, SrsListenerType t, ISrsUdpHandler* c);
    virtual ~SrsUdpStreamListener();
public:
    virtual int listen(std::string i, int p);
};
```

## 1.6. <p class="todo">udp caster listener</p>

## 1.7. class SrsSignalManager

## 1.8. class ISrsServerCycle

## 1.9. class SrsServer