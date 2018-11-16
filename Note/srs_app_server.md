<link href="note.css" rel="stylesheet"></link>


# srs_app_server
## enum SrsListenerType
<pre>
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
</pre>
枚举类型 SrsListenerType。


## class SrsListener
<pre>
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
</pre>
纯虚类 SrsListener，主要用于tcp连接的server，例如RTMP和HTTPserver。  
<br>
保存了ip地址、端口号以及SrsServer指针。  
使用SrsServer以及listener类型进行初始化。  
SrsListenerType返回listener的类型。  
<p class="todo">之后补上具体说明</p>
listen为纯虚函数，参数为string的ip地址以及int类型的端口号。

## class SrsStreamListener
<pre>
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
</pre>

## <p class="todo">rtsp/flv listener</p>

## class SrsUdpStreamListener
<pre>
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
</pre>

## <p class="todo">udp caster listener</p>

## class SrsSignalManager

## class ISrsServerCycle

## class SrsServer