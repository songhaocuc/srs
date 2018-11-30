<link href="../../Style/note.css" rel="stylesheet"></link>

# options

## show_help()
```shell
function show_help() {
    cat << END

Options:
  -h, --help                print this message
                          
  --with-ssl                enable rtmp complex handshake, requires openssl-devel installed.
                            to delivery h264 video and aac audio to flash player.
  --with-hls                enable hls streaming, mux RTMP to m3u8/ts files.
  --with-hds                enable hds streaming, mux RTMP to f4m/f4v files.
  --with-dvr                enable dvr, mux RTMP to flv files.
  --with-nginx              enable delivery HTTP stream with nginx.
                            build nginx at: ./objs/nginx/sbin/nginx
  --with-http-callback      enable http hooks, build cherrypy as demo api server.
  --with-http-server        enable http server to delivery http stream.
  --with-stream-caster      enable stream caster to serve other stream over other protocol.
  --with-http-api           enable http api, to manage SRS by http api.
  --with-ffmpeg             enable transcoding tool ffmpeg.
                            build ffmpeg at: ./objs/ffmpeg/bin/ffmpeg
  --with-transcode          enable transcoding features.
                            user must specifies the transcode tools in conf.
  --with-ingest             enable ingest features.
                            user must specifies the ingest tools in conf.
  --with-stat               enable the data statistic, for http api.
  --with-librtmp            enable srs-librtmp, library for client.
  --with-research           build the research tools.
  --with-utest              build the utest for SRS.
  --with-gperf              build SRS with gperf tools(no gmc/gmp/gcp, with tcmalloc only).
  --with-gmc                build memory check for SRS with gperf tools.
  --with-gmp                build memory profile for SRS with gperf tools.
  --with-gcp                build cpu profile for SRS with gperf tools.
  --with-gprof              build SRS with gprof(GNU profile tool).
  --with-arm-ubuntu12       cross build SRS on ubuntu12 for armhf(v7cpu).
                          
  --without-ssl             disable rtmp complex handshake.
  --without-hls             disable hls, the apple http live streaming.
  --without-hds             disable hds, the adobe http dynamic streaming.
  --without-dvr             disable dvr, donot support record RTMP stream to flv.
  --without-nginx           disable delivery HTTP stream with nginx.
  --without-http-callback   disable http, http hooks callback.
  --without-http-server     disable http server, use external server to delivery http stream.
  --without-stream-caster   disable stream caster, only listen and serve RTMP/HTTP.
  --without-http-api        disable http api, only use console to manage SRS process.
  --without-ffmpeg          disable the ffmpeg transcode tool feature.
  --without-transcode       disable the transcoding feature.
  --without-ingest          disable the ingest feature.
  --without-stat            disable the data statistic feature.
  --without-librtmp         disable srs-librtmp, library for client.
  --without-research        do not build the research tools.
  --without-utest           do not build the utest for SRS.
  --without-gperf           do not build SRS with gperf tools(without tcmalloc and gmc/gmp/gcp).
  --without-gmc             do not build memory check for SRS with gperf tools.
  --without-gmp             do not build memory profile for SRS with gperf tools.
  --without-gcp             do not build cpu profile for SRS with gperf tools.
  --without-gprof           do not build srs with gprof(GNU profile tool).
  --without-arm-ubuntu12    do not cross build srs on ubuntu12 for armhf(v7cpu).
                          
  --prefix=<path>           the absolute install path for srs.
  --static                  whether add '-static' to link options.
  --jobs[=N]                Allow N jobs at once; infinite jobs with no arg.
                            used for make in the configure, for example, to make ffmpeg.
  --log-verbose             whether enable the log verbose level. default: no.
  --log-info                whether enable the log info level. default: no.
  --log-trace               whether enable the log trace level. default: yes.

Presets:
  --x86-x64                 [default] for x86/x64 cpu, common pc and servers.
  --osx                     for osx(darwin) system to build SRS.
  --pi                      for raspberry-pi(directly build), open features hls/ssl/static.
  --cubie                   for cubieboard(directly build), open features except ffmpeg/nginx.
  --arm                     alias for --with-arm-ubuntu12, for ubuntu12, arm crossbuild
  --mips                    alias for --with-mips-ubuntu12, for ubuntu12, mips crossbuild
  --fast                    the most fast compile, nothing, only support vp6 RTMP.
  --pure-rtmp               only support RTMP with ssl.
  --rtmp-hls                only support RTMP+HLS with ssl.
  --disable-all             disable all features, only support vp6 RTMP.
  --dev                     for dev, open all features, no nginx/gperf/gprof/arm.
  --fast-dev                for dev fast compile, the RTMP server, without librtmp/utest/research.
  --demo                    for srs demo, @see: https://github.com/ossrs/srs/wiki/v1_CN_SampleDemo
  --full                    enable all features, no gperf/gprof/arm.
  
Conflicts:
  1. --with-gmc vs --with-gmp: 
        @see: http://google-perftools.googlecode.com/svn/trunk/doc/heap_checker.html
  2. --with-gperf/gmc/gmp vs --with-gprof:
        gperftools not compatible with gprof.
  3. --arm vs --with-ffmpeg/gperf/gmc/gmp/gprof:
        the complex tools not available for arm.

Experts:
  --use-sys-ssl                     donot compile ssl, use system ssl(-lssl) if required.
  --memory-watch                    enable memory watch to detect memory leaking(hurts performance).
  --export-librtmp-project=<path>   export srs-librtmp to specified project in path.
  --export-librtmp-single=<path>    export srs-librtmp to a single file(.h+.cpp) in path.

Workflow:
  1. apply "Presets". if not specified, use default preset.
  2. apply "Options". user specified option will override the preset.
  3. check conflicts. @see Conflicts section.
  4. generate detail features.

END
}
```
cat << END意思是在遇见END字符串之前的字符串都会作为整体传递给cat打印出来。

## parse_user_option()
```cpp
function parse_user_option() {
    case "$option" in
        -h)                             help=yes                    ;;
        --help)                         help=yes                    ;;
        
        --with-ssl)                     SRS_SSL=YES                 ;;
        --with-hls)                     SRS_HLS=YES                 ;;
        --with-hds)                     SRS_HDS=YES                 ;;
        --with-dvr)                     SRS_DVR=YES                 ;;
        --with-nginx)                   SRS_NGINX=YES               ;;
        --with-ffmpeg)                  SRS_FFMPEG_TOOL=YES         ;;
        --with-transcode)               SRS_TRANSCODE=YES           ;;
        --with-ingest)                  SRS_INGEST=YES              ;;
        --with-stat)                    SRS_STAT=YES                ;;
        --with-http-callback)           SRS_HTTP_CALLBACK=YES       ;;
        --with-http-server)             SRS_HTTP_SERVER=YES         ;;
        --with-stream-caster)           SRS_STREAM_CASTER=YES       ;;
        --with-http-api)                SRS_HTTP_API=YES            ;;
        --with-librtmp)                 SRS_LIBRTMP=YES             ;;
        --with-research)                SRS_RESEARCH=YES            ;;
        --with-utest)                   SRS_UTEST=YES               ;;
        --with-gperf)                   SRS_GPERF=YES               ;;
        --with-gmc)                     SRS_GPERF_MC=YES            ;;
        --with-gmp)                     SRS_GPERF_MP=YES            ;;
        --with-gcp)                     SRS_GPERF_CP=YES            ;;
        --with-gprof)                   SRS_GPROF=YES               ;;
        --with-arm-ubuntu12)            SRS_ARM_UBUNTU12=YES        ;;
        --with-mips-ubuntu12)           SRS_MIPS_UBUNTU12=YES       ;;
                                                                 
        --without-ssl)                  SRS_SSL=NO                  ;;
        --without-hls)                  SRS_HLS=NO                  ;;
        --without-hds)                  SRS_HDS=NO                  ;;
        --without-dvr)                  SRS_DVR=NO                  ;;
        --without-nginx)                SRS_NGINX=NO                ;;
        --without-ffmpeg)               SRS_FFMPEG_TOOL=NO          ;;
        --without-transcode)            SRS_TRANSCODE=NO            ;;
        --without-ingest)               SRS_INGEST=NO               ;;
        --without-stat)                 SRS_STAT=NO                 ;;
        --without-http-callback)        SRS_HTTP_CALLBACK=NO        ;;
        --without-http-server)          SRS_HTTP_SERVER=NO          ;;
        --without-stream-caster)        SRS_STREAM_CASTER=NO        ;;
        --without-http-api)             SRS_HTTP_API=NO             ;;
        --without-librtmp)              SRS_LIBRTMP=NO              ;;
        --without-research)             SRS_RESEARCH=NO             ;;
        --without-utest)                SRS_UTEST=NO                ;;
        --without-gperf)                SRS_GPERF=NO                ;;
        --without-gmc)                  SRS_GPERF_MC=NO             ;;
        --without-gmp)                  SRS_GPERF_MP=NO             ;;
        --without-gcp)                  SRS_GPERF_CP=NO             ;;
        --without-gprof)                SRS_GPROF=NO                ;;
        --without-arm-ubuntu12)         SRS_ARM_UBUNTU12=NO         ;;
        --without-mips-ubuntu12)        SRS_MIPS_UBUNTU12=NO        ;;
        
        --jobs)                         SRS_JOBS=${value}           ;;
        --prefix)                       SRS_PREFIX=${value}         ;;
        --static)                       SRS_STATIC=YES              ;;
        --log-verbose)                  SRS_LOG_VERBOSE=YES         ;;
        --log-info)                     SRS_LOG_INFO=YES            ;;
        --log-trace)                    SRS_LOG_TRACE=YES           ;;
        
        --x86-x64)                      SRS_X86_X64=YES             ;;
        --osx)                          SRS_OSX=YES                 ;;
        --arm)                          SRS_ARM_UBUNTU12=YES        ;;
        --mips)                         SRS_MIPS_UBUNTU12=YES       ;;
        --pi)                           SRS_PI=YES                  ;;
        --cubie)                        SRS_CUBIE=YES               ;;
        --dev)                          SRS_DEV=YES                 ;;
        --fast-dev)                     SRS_FAST_DEV=YES            ;;
        --demo)                         SRS_DEMO=YES                ;;
        --fast)                         SRS_FAST=YES                ;;
        --disable-all)                  SRS_DISABLE_ALL=YES         ;;
        --pure-rtmp)                    SRS_PURE_RTMP=YES           ;;
        --rtmp-hls)                     SRS_RTMP_HLS=YES            ;;
        --full)                         SRS_ENABLE_ALL=YES          ;;
        
        --use-sys-ssl)                  SRS_USE_SYS_SSL=YES         ;;
        --memory-watch)                 SRS_MEM_WATCH=YES           ;;
        --export-librtmp-project)       SRS_EXPORT_LIBRTMP_PROJECT=${value}     ;;
        --export-librtmp-single)        SRS_EXPORT_LIBRTMP_SINGLE=${value}      ;;

        *)
            echo "$0: error: invalid option \"$option\""
            exit 1
        ;;
    esac
}
```
解析configure指令后的参数，将相应的宏值为YES。

## parse_user_option_to_value_and_option()
```cpp
function parse_user_option_to_value_and_option() {
    case "$option" in
        -*=*) 
            value=`echo "$option" | sed -e 's|[-_a-zA-Z0-9/]*=||'` 
            option=`echo "$option" | sed -e 's|=[-_a-zA-Z0-9/.]*||'`
        ;;
           *) value="" ;;
    esac
}
```
将`-option=value`形式的选项解析为value和option