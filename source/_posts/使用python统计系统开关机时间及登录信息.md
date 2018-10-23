---
title: 使用python统计系统开关机时间及登录信息
date: 2018-07-20 05:30:15
tags:
categories: python
thumbnail: http://editerupload.eepw.com.cn/201411/48601416902987.jpg
---

    日常开发中使用python可以做一些小工具，相对C语言和shell脚本来说使用起来很简单，而且是解释执行的，所以开发和使用效率比较高。

## 需求

审计一下linux（或者说是类unix系统）下系统的开关机、登录、注销情况。

## 分析

linux下系统的开关机、登录和注销行为都有日志记录，该日志文件位于/var/log/wtmp, 我们只需要监测日志文件的变化，通过对比变化前后的内容就可以知道我们要监测变更信息。

另外这个日志文件是二进制形式的，不能直接打开查看，可以使用last命令查看，另外通过添加以下last命令的参数可以更好地获取里面的信息。

## 实践

这里是通过c文件调用python脚本获取python脚本执行的输出信息来进行行为审计的，重点将以下python部分，也就是处理日志信息的那里。

整个工作流程是这样的，c文件负责定时调用python脚本，python脚本负责获取当前last命令下输出的内容信息，然后对比本地缓存的之前的last信息，通过差异项来判断那些是登录操作，哪些是注销操作，是本地行为还是远程链接等等。

通过下面的命令可以获取到最近的30条系统登录信息：
```
last -F -x -n 30
```
比如本机的

```
第一列    第二列        第三列            第四列                      第五列                      第六列
wst      pts/1        192.168.8.167    Fri Jul 20 16:01:10 2018   still logged in
wst      pts/1        192.168.8.167    Fri Jul 20 15:55:41 2018 - Fri Jul 20 15:56:01 2018  (00:00)
wst      pts/0        192.168.8.167    Fri Jul 20 15:39:27 2018   still logged in
wst      pts/0        192.168.8.167    Fri Jul 20 13:50:08 2018 - Fri Jul 20 13:52:47 2018  (00:02)
wst      pts/8        :0.0             Thu Jul 19 15:19:43 2018 - Thu Jul 19 15:34:02 2018  (00:14)
wst      pts/8        :0.0             Thu Jul 19 14:31:28 2018 - Thu Jul 19 14:31:31 2018  (00:00)
wst      pts/4        :0.0             Thu Jul 19 11:55:53 2018 - Thu Jul 19 15:34:00 2018  (03:38)
wst      pts/4        :0.0             Thu Jul 19 11:51:13 2018 - Thu Jul 19 11:51:28 2018  (00:00)
wst      pts/3        :0.0             Thu Jul 19 11:23:10 2018 - Thu Jul 19 15:33:56 2018  (04:10)
wst      pts/2        192.168.8.167    Thu Jul 19 11:15:22 2018 - Thu Jul 19 15:52:59 2018  (04:37)
wst      pts/1        192.168.8.167    Thu Jul 19 10:43:45 2018 - Thu Jul 19 15:53:03 2018  (05:09)
wst      pts/0        192.168.8.167    Thu Jul 19 09:39:35 2018 - Thu Jul 19 15:53:02 2018  (06:13)
wst      pts/0        192.168.8.167    Wed Jul 18 10:57:37 2018 - Wed Jul 18 18:21:56 2018  (07:24)
wst      pts/4        192.168.8.167    Tue Jul 17 14:39:01 2018 - Tue Jul 17 17:05:50 2018  (02:26)
wst      pts/3        192.168.8.167    Tue Jul 17 13:44:47 2018 - Tue Jul 17 17:06:02 2018  (03:21)
wst      pts/2        192.168.8.167    Tue Jul 17 13:33:03 2018 - Tue Jul 17 17:05:33 2018  (03:32)
wst      pts/1        192.168.8.167    Tue Jul 17 11:16:58 2018 - Tue Jul 17 17:05:57 2018  (05:48)
wst      pts/0        192.168.8.167    Tue Jul 17 09:36:20 2018 - Tue Jul 17 17:49:40 2018  (08:13)
wst      pts/7        :0.0             Mon Jul 16 15:09:10 2018   still logged in
wst      pts/6        :0.0             Mon Jul 16 15:08:07 2018 - Thu Jul 19 15:35:23 2018 (3+00:27)
wst      pts/5        :0.0             Mon Jul 16 15:01:04 2018 - Thu Jul 19 15:23:10 2018 (3+00:22)
wst      :0           :0               Mon Jul 16 15:00:53 2018   still logged in
wst      pts/4        192.168.8.167    Mon Jul 16 14:06:51 2018 - Mon Jul 16 15:52:46 2018  (01:45)
wst      pts/3        192.168.8.167    Mon Jul 16 11:16:07 2018 - Mon Jul 16 18:06:51 2018  (06:50)
wst      pts/1        192.168.8.167    Mon Jul 16 10:54:49 2018 - Mon Jul 16 18:06:42 2018  (07:11)
wst      pts/2        192.168.8.167    Mon Jul 16 10:38:56 2018 - Mon Jul 16 18:06:56 2018  (07:28)
wst      pts/1        192.168.8.167    Mon Jul 16 09:54:30 2018 - Mon Jul 16 10:42:00 2018  (00:47)
wst      pts/0        192.168.8.167    Mon Jul 16 09:46:38 2018 - Mon Jul 16 18:06:58 2018  (08:20)
(unknown :0           :0               Mon Jul 16 09:44:14 2018 - Mon Jul 16 15:00:52 2018  (05:16)
runlevel (to lvl 5)   3.10.0.3a2000+   Mon Jul 16 09:44:13 2018   still running
reboot   system boot  3.10.0.3a2000+   Mon Jul 16 17:44:04 2018   still running
shutdown system down  3.10.0.3a2000+   Fri Jul 13 17:42:14 2018 - Mon Jul 16 17:44:04 2018 (3+00:01)
wst      pts/3        :0.0             Fri Jul 13 17:07:58 2018 - Fri Jul 13 17:42:02 2018  (00:34)
wst      pts/4        192.168.8.167    Fri Jul 13 09:40:06 2018 - Fri Jul 13 16:32:28 2018  (06:52)
wst      pts/3        192.168.8.167    Fri Jul 13 09:27:39 2018 - Fri Jul 13 14:54:36 2018  (05:26)
wst      pts/5        192.168.8.167    Thu Jul 12 19:06:20 2018 - Thu Jul 12 19:21:56 2018  (00:15)
wst      pts/4        192.168.8.167    Thu Jul 12 18:58:16 2018 - Thu Jul 12 19:21:49 2018  (00:23)
wst      pts/3        192.168.8.179    Thu Jul 12 17:07:13 2018 - Thu Jul 12 19:18:49 2018  (02:11)
wst      pts/2        :0.0             Thu Jul 12 16:22:08 2018 - Fri Jul 13 17:42:07 2018 (1+01:19)
wst      pts/1        :0.0             Thu Jul 12 15:04:06 2018 - Fri Jul 13 17:42:09 2018 (1+02:38)
wst      pts/1        192.168.8.167    Thu Jul 12 10:09:17 2018 - Thu Jul 12 14:26:53 2018  (04:17)
wst      pts/0        :0.0             Thu Jul 12 09:30:10 2018 - Fri Jul 13 17:42:09 2018 (1+08:11)
wst      :0           :0               Thu Jul 12 09:26:36 2018 - Fri Jul 13 17:42:13 2018 (1+08:15)
(unknown :0           :0               Thu Jul 12 09:26:14 2018 - Thu Jul 12 09:26:36 2018  (00:00)
runlevel (to lvl 5)   3.10.0.3a2000+   Thu Jul 12 09:26:13 2018 - Fri Jul 13 17:42:14 2018 (1+08:16)
reboot   system boot  3.10.0.3a2000+   Thu Jul 12 17:26:04 2018 - Fri Jul 13 17:42:14 2018 (1+00:16)
wst      pts/1        192.168.8.165    Tue Jul 10 19:27:05 2018 - Tue Jul 10 20:20:12 2018  (00:53)
wst      pts/1        192.168.8.165    Tue Jul 10 14:59:36 2018 - Tue Jul 10 19:26:28 2018  (04:26)
wst      pts/2        192.168.8.167    Mon Jul  9 18:52:45 2018 - Mon Jul  9 18:56:30 2018  (00:03)
wst      pts/2        192.168.8.167    Mon Jul  9 15:42:22 2018 - Mon Jul  9 18:50:20 2018  (03:07)

wtmp 开始 Wed Jun  8 17:16:26 2016

```

第一列是用户名，第一列是reboot的表示系统启动，第二列是表示虚拟终端登录信息，比如图形界面登录有的是“：0”，有的是“ttyx”,不同发行版的系统是有细微差别的。

第三列是用户的登录IP获取内核信息，如果是第一列是reboot，那么第三列就是内核信息，如果是正常用户登录，那么第三列就是本地或者远程IP地址。

再接下来第四列是开始时间，即系统的启动时间或者登录时间。

第五列是结束时间，可能是系统的关机时间或者是用户的注销时间，当然如果是still running，则代表该系统还在运行中，我们可以通过第四列判断出系统的开机时间。

如果是still logged in，则表示用户还在连接中，我们可以通过第四列判断出用户的登录时间。

第六列是运行时间统计，这里有个问题需要注意以下，有时候这个值是负的，这很神奇？时间怎么可能是负的，其实是因为系统开机时记录的时间不准确造成的，记录的时间比实际时间晚了8个小时。

在获取系统开机时间时，我们需要将这个八个小时减去才能获取到正确的时间。具体要不要减去这8个小时要根据你电脑的世纪情况来判断，基本方法就是第六列有负值，那么肯定就需要减去这8个小时。

这里面其实还是需要你仔细分析差异点，然后找出规律，最后通过不同的条件判断出系统的行为。

## 关键代码

```
if __name__ == '__main__':
    now_str = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    # 通过subprocess包执行linux命令，然后获取返回值
    ret, result = subprocess.getstatusoutput('last -F -x -n 30')
    if ret != 0:
        #print('error')
        os._exit(0)
    # 按行将日志信息存储起来
    contents_new = result.split('\n')
    contents_old = []
    # 读取之前缓存的日志信息到contents_old，然后对比contents_old和contents_new
    if os.path.isfile(FILENAME):
        with open(FILENAME, 'rb') as f:
            for line in f.readlines():
                line = base64.b64decode(line).decode('utf-8')
                contents_old.append(line)
    else:
        with open(FILENAME, 'wb') as f:
            for content_new in contents_new :
                content_new = content_new.strip('\n')
                temp = base64.b64encode(bytes(content_new, 'utf-8'))
                f.write(temp)
                f.write(b'\n')
        os._exit(0)
        
    # 处理新产生的日志文件
    with open(FILENAME, 'wb') as f :
        for content_new in contents_new :
            if (content_new and ('wtmp' not in content_new) and
                ('down' not in content_new) and ('crash' not in content_new) and
                ('shutdown' not in content_new) and ('runlevel' not in content_new)
                ):
                # 如果新查询到的日志在原来的缓存文件中没有，那么就代表系统有新的日志产生，我们处理的就是这部分日志信息。
                if content_new not in contents_old :
                    #print('that is what we should deal with')
                    #print(content_new)
                    # 分析日志信息的函数
                    deal_log(content_new, now_str)
            else :
                #print('no use')
                pass
            # 使用base64进行加密缓存日志信息（项目需要，其实就是不想直接暴露这些信息给用户）
            content_new = base64.b64encode(bytes(content_new, 'utf-8'))
            f.write(content_new)
            f.write(b'\n')


```

deal_log的内容：
```
def deal_log(content_new, now_str):
    content_words = content_new.split(' ')
    while '' in content_words :
        content_words.remove('')
    ret, plat = subprocess.getstatusoutput('uname -m')
    if plat == 'aarch64' :
        #print('kylin')
        deal_kylin_log(content_new, content_words)
    elif plat == 'i686':
        #print('cods')
        deal_cods_log(content_new, content_words, now_str)
    elif plat == 'mips64' :
        #print('neo')
        deal_neo_log(content_new, content_words)

```

其实就是根据具体不同的平台来进行不同的处理。

比如kylin平台：

```
def fix_time(time) :
    format_time = datetime.datetime.strptime(time[3] + '-' + time[0] + '-' +
    time[1] + ' ' + time[2], '%Y-%b-%d %H:%M:%S')
    time_right = format_time + datetime.timedelta(hours = -8)

    time_right_str = time_right.strftime('%Y-%m-%d %H:%M:%S')

    return time_right_str


def formate_time(time) :
    #print(time)
    format_time = datetime.datetime.strptime(time[3] + '-' + time[0] + '-' +
        time[1] + ' ' + time[2], '%Y-%b-%d %H:%M:%S')
    format_time_str = format_time.strftime('%Y-%m-%d %H:%M:%S')

    return format_time_str


def deal_kylin_log(content_new, content_words) :
    user = content_words[0]
    # login
    if (('still logged in' in content_new) or ('gone - no logout' in content_new)) :
        if (':0.0' not in content_words[2]) :
            action = '登录'
            login_time = formate_time(content_words[4:8])
            if ':0' in content_words[2] :
                location = '本地'
            else :
                location = '远程(' + content_words[2] + ')'
            #print('user: ' +  user + ' action: ' + action + ' locaton: ' + 
            #    location + ' login_time: ' + login_time)
            print('login|登录|<Object class="时间" name="' + login_time + '"><登录用户>' +
                user + '</登录用户><登录点>' + location + '</登录点><登录时间>' + login_time +
                '</登录时间><登出时间/><状态>登录</状态></Object>|<Reason/>|2|审计|成功')
    # boot
    elif ('still running' in content_new) and ('reboot' in content_new) :
        boot_time = fix_time(content_words[5:9])
        action = '开机'
        #print('user: ' +  user + ' action: ' + action + ' boot_time: ' + boot_time)
        print('boot|开机|<Object class="时间" name="' + boot_time + '"><开机时间>' + boot_time +
            '</开机时间><状态>开机</状态></Object>|<Reason/>|2|审计|成功')

    # shutdown
    elif ('reboot' in content_new) and ('still running' not in content_new) :
        shutdown_time = formate_time(content_words[11:15])
        action = '关机'
        #print('user: ' +  user + ' action: ' + action + ' shutdown_time: ' + shutdown_time)
        print('shutdown|关机|<Object class="时间" name="' + shutdown_time + '"><关机时间>' +
            shutdown_time + '</关机时间>' + '<状态>关机</状态></Object>|<Reason/>|2|审计|成功')

    elif ( ':0.0' not in content_words[2]):
        action = '注销'
        logout_time = formate_time(content_words[10:14])
        if ':0' in content_words[2] :
            location = '本地'
        else :
            location = '远程(' + content_words[2] + ')'
        #print('user: ' +  user + ' action: ' + action + ' location:' + location + 
        #    ' logout_time: ' + logout_time)
        print('logout|注销|<Object class="时间" name="' + logout_time + '"><注销用户>' +
            user + '</注销用户><注销位置>' + location + '</注销位置><注销时间>' + logout_time +
            '</注销时间><登出时间/><状态>注销</状态></Object>|<Reason/>|2|审计|成功')

    else :
        #print('can not recognize')
        pass
```

最后把日志组织成相应的格式输出，由c程序捕获这些输出就可以了。

## 总结

总体思路就是用python做一个中间处理的小工具，通过调用这个小工具可以将固定格式的日志信息处理之后组织成我们想要的信息返回回来，这里我是通过直接打印然后让c程序获取python的打印信息。

我知道c和python可以通过相应的api互相调用，暂时没有研究，回头研究一下看能不能做个改进。

[项目地址] [1]

2018.7.20 北京 晴

[1]: https://github.com/jiaoqiyuan/audit_sys_info.git