# SUPERVISOR进程管理器配置指南 #

# 1. supervisor简介 #

## 1.1. 官网 ##

    http://supervisord.org/

## 1.2. 介绍 ##

Supervisor是一个进程控制系统. 它是一个C/S系统(注意: 其提供WEB接口给用户查询和控制), 它允许用户去监控和控制在类UNIX系统的进程. 它的目标与launchd, daemontools和runit有些相似, 但是与它们不一样的是, 它不是作为init(进程号pid是1)运行. 它是被用来控制进程, 并且它在启动的时候和一般程序并无二致. 

# 2. 安装和配置 #

## 2.1. 安装 ##

    [root@Siffre /]# yum install python-setuptools -y
    [root@Siffre /]# easy_install supervisor
    [root@Siffre /]# echo_supervisord_conf > /etc/supervisord.conf
    
## 2.2. 配置 ##

    [root@Siffre /]# vim /etc/supervisord.conf   
    ; http://supervisord.org/configuration.html    
    [rpcinterface:supervisor]
    ;exitcodes=0,2 ; 'expected' exit codes for process (default 0,2)
    ;stopsignal=QUIT   ; signal used to kill process (default TERM)
    ;stopwaitsecs=10   ; max num secs to wait b4 SIGKILL (default 10)
    ;stopasgroup=false ; send stop signal to the UNIX process group (default false)
    ;killasgroup=false ; SIGKILL the UNIX process group (def false)
    ;user=chrism   ; setuid to this UNIX account to run the program
    ;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
    ;stderr_logfile_backups=10 ; # of stderr logfile backups (default 10)
    ;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
    ;autorestart=unexpected; whether/when to restart (default: unexpected)
    ;startsecs=1   ; number of secs prog must stay running (def. 1)
    ;startretries=3; max # of serial start failures (default 3)
    ;exitcodes=0,2 ; 'expected' exit codes for process (default 0,2)
    ;stopsignal=QUIT   ; signal used to kill process (default TERM)
    ;stopwaitsecs=10   ; max num secs to wait b4 SIGKILL (default 10)
    ;stopasgroup=false ; send stop signal to the UNIX process group (default false)
    ;killasgroup=false ; SIGKILL the UNIX process group (def false)
    ;user=chrism   ; setuid to this UNIX account to run the program
    ;redirect_stderr=true  ; redirect proc stderr to stdout (default false)
    ;stdout_logfile=/a/path; stdout log path, NONE for none; default AUTO
    ;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
    ;stdout_logfile_backups=10 ; # of stdout logfile backups (default 10)
    ;stdout_events_enabled=false   ; emit events on stdout writes (default false)
    ;stderr_logfile=/a/path; stderr log path, NONE for none; default AUTO
    ;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
    ;stderr_logfile_backups; # of stderr logfile backups (default 10)
    ;stderr_events_enabled=false   ; emit events on stderr writes (default false)
    ;environment=A="1",B="2"   ; process environment additions
    ;serverurl=AUTO; override serverurl computation (childutils)
    
    ; The below sample group section shows all possible group values,
    ; create one or more 'real' group: sections to create "heterogeneous"
    ; process groups.
    
    ;[group:thegroupname]
    ;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
    ;priority=999  ; the relative start priority (default 999)
    
    ; The [include] section can just contain the "files" setting.  This
    ; setting can list multiple files (separated by whitespace or
    ; newlines).  It can also contain wildcards.  The filenames are
    ; interpreted as relative to this file.  Included files *cannot*
    ; include files themselves.
    
    ;[include]
    ;files = relative/directory/*.ini
    
## 2.3. (program)配置模板 ##

    [program:cat]  #应用进程
    command=/bin/cat
    process_name=%(program_name)s
    numprocs=1
    directory=/tmp
    umask=022
    priority=999
    autostart=true
    autorestart=true
    startsecs=10
    startretries=3
    exitcodes=0,2
    stopsignal=TERM
    stopwaitsecs=10
    user=chrism
    redirect_stderr=false
    stdout_logfile=/a/path
    stdout_logfile_maxbytes=1MB
    stdout_logfile_backups=10
    stdout_capture_maxbytes=1MB
    stderr_logfile=/a/path
    stderr_logfile_maxbytes=1MB
    stderr_logfile_backups=10
    stderr_capture_maxbytes=1MB
    environment=A="1",B="2"
    serverurl=AUTO
    
    简化模板
    
    [program:test]   #应用进程
    command=python test.py
    directory=/home/supervisor_test/
    autorestart=true
    stopsignal=INT
    user=root
    stdout_logfile=test_out.log
    stdout_logfile_maxbytes=1MB
    stdout_logfile_backups=10
    stdout_capture_maxbytes=1MB
    stderr_logfile=test_err.log
    stderr_logfile_maxbytes=1MB
    stderr_logfile_backups=10
    stderr_capture_maxbytes=1MB
    
## 2.4. (program)配置说明 ##
    
    [program:应用名称]    #必须填写项
    ;[program:cat]
    
    命令路径,如果使用python启动的程序应该为 python /home/test.py, 
    不建议放入/home/user/, 对于非user用户一般情况下是不能访问
    ;command=/bin/cat
    
    当numprocs为1时,process_name=%(program_name)s
    当numprocs>=2时,%(program_name)s_%(process_num)02d
    ;process_name=%(program_name)s
    
    进程数量
    ;numprocs=1
    
    执行目录,若有/home/supervisor_test/test1.py
    将directory设置成/home/supervisor_test
    则command只需设置成python test1.py
    否则command必须设置成绝对执行目录
    ;directory=/tmp
    
    掩码:--- -w- -w-, 转换后rwx r-x w-x
    ;umask=022
    
    优先级,值越高,最后启动,最先被关闭,默认值999
    ;priority=999
    
    如果是true,当supervisor启动时,程序将会自动启动
    ;autostart=true
    
    自动重启   #必填
    ;autorestart=true
    
    启动延时执行,默认1秒
    ;startsecs=10
    
    启动尝试次数,默认3次
    ;startretries=3
    
    当退出码是0,2时,执行重启,默认值0,2
    ;exitcodes=0,2
    
    停止信号,默认TERM
    中断:INT(类似于Ctrl+C)(kill -INT pid),退出后会将写文件或日志(推荐)
    终止:TERM(kill -TERM pid)
    挂起:HUP(kill -HUP pid),注意与Ctrl+Z/kill -stop pid不同
    从容停止:QUIT(kill -QUIT pid)
    KILL, USR1, USR2其他见命令(kill -l),说明1
    ;stopsignal=TERM
    
    ;stopwaitsecs=10
    
    以root用户执行
    ;user=root
    
    重定向
    ;redirect_stderr=false
    
    ;stdout_logfile=/a/path
    ;stdout_logfile_maxbytes=1MB
    ;stdout_logfile_backups=10
    ;stdout_capture_maxbytes=1MB
    ;stderr_logfile=/a/path
    ;stderr_logfile_maxbytes=1MB
    ;stderr_logfile_backups=10
    ;stderr_capture_maxbytes=1MB
    
    环境变量设置
    ;environment=A="1",B="2"
    
    ;serverurl=AUTO
    
## 2.5. (inet_ http_server)配置说明 ##

可以使用浏览器查看和控制进程状态
    
    ;[inet_http_server] ; inet (TCP) server disabled by default
    ;port=0.0.0.0:9001  ; (ip_address:port specifier, *:port for all iface)
    ;username=user  ; 用户名 (default is no username (open server))
    ;password=123   ; 密码 (default is no password (open server))

# 3. 启动与关闭 #

## 3.1. 启动supervisord ##

    [root@Siffre /]# supervisord -c /etc/supervisord.conf
    
## 3.2. 关闭supervisord ##
    
    [root@Siffre /]# supervisorctl shutdown

## 3.3. 重新载入配置 ##
    
    [root@Siffre /]# supervisorctl reload
    说明1:
    
    常用信号说明-原文
    
    信号名称	数字表示	说明
    SIGHUP	1	终端挂起或控制进程终止。当用户退出Shell时，由该进程启动的所有进程都会收到这个信号，默认动作为终止进程。
    SIGINT	2	键盘中断。当用户按下组合键时，用户终端向正在运行中的由该终端启动的程序发出此信号。默认动作为终止进程。
    SIGQUIT	3	键盘退出键被按下。当用户按下或组合键时，用户终端向正在运行中的由该终端启动的程序发出此信号。默认动作为退出程序。
    SIGFPE	8	发生致命的运算错误时发出。不仅包括浮点运算错误，还包括溢出及除数为0等所有的算法错误。默认动作为终止进程并产生core文件。
    SIGKILL	9	无条件终止进程。进程接收到该信号会立即终止，不进行清理和暂存工作。该信号不能被忽略、处理和阻塞，它向系统管理员提供了可以杀死任何进程的方法。
    SIGALRM	14	定时器超时，默认动作为终止进程。
    SIGTERM	15	程序结束信号，可以由 kill 命令产生。与SIGKILL不同的是，SIGTERM 信号可以被阻塞和终止，以便程序在退出前可以保存工作或清理临时文件等。
    $ kill -l
     1) SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL 5) SIGTRAP
     6) SIGABRT 7) SIGBUS 8) SIGFPE 9) SIGKILL10) SIGUSR1
    11) SIGSEGV12) SIGUSR213) SIGPIPE14) SIGALRM15) SIGTERM
    16) SIGSTKFLT17) SIGCHLD18) SIGCONT19) SIGSTOP20) SIGTSTP
    21) SIGTTIN22) SIGTTOU23) SIGURG24) SIGXCPU25) SIGXFSZ
    26) SIGVTALRM27) SIGPROF28) SIGWINCH29) SIGIO30) SIGPWR
    31) SIGSYS34) SIGRTMIN35) SIGRTMIN+136) SIGRTMIN+237) SIGRTMIN+3
    38) SIGRTMIN+439) SIGRTMIN+540) SIGRTMIN+641) SIGRTMIN+742) SIGRTMIN+8
    43) SIGRTMIN+944) SIGRTMIN+1045) SIGRTMIN+1146) SIGRTMIN+1247) SIGRTMIN+13
    48) SIGRTMIN+1449) SIGRTMIN+1550) SIGRTMAX-1451) SIGRTMAX-1352) SIGRTMAX-12
    53) SIGRTMAX-1154) SIGRTMAX-1055) SIGRTMAX-956) SIGRTMAX-857) SIGRTMAX-7
    58) SIGRTMAX-659) SIGRTMAX-560) SIGRTMAX-461) SIGRTMAX-362) SIGRTMAX-2
    63) SIGRTMAX-164) SIGRTMAX