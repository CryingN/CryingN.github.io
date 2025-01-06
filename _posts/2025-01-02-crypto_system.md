---
title: '[awdp学习笔记] 密码系统类题目'
date: 2025-01-02
permalink: /posts/2025/01/crypto_system/
excerpt: '密码系统一般是一类比较特殊的考察方式...'
tags:
  - 网络安全
  - CTF
  - 密码学
  - 算法
  - 密码系统
---

# 目录

- [第七届西湖论剑](#第六届强网杯)
    - [ezdja](#ezdja)
    - [found_cms](#found_cms)
- [网鼎杯线下赛](#网鼎杯线下赛)
    - [alicewebsite](#alicewebsite)
    - [fake](#fake)
- [十七届ciscn总决赛](#十七届ciscn总决赛)
    - [anime](#anime)
    - [ezheap](#ezheap)
    - [chr](#chr)
- [参考文献](#参考文献)

# 第六届强网杯

第六届青少赛线下AWDP, 参考末心师傅的文档[^moxin], fix部分主要题目有四道:

## ezdja

经典的python类DJango框架, 文件结构为:

- **app01**
    - 主要的app文件
- **easypy**
- **static**
    - 前端文件
- **templates**
    - 前端文件
- manage.py (服务脚本)
- requirements.txt

服务脚本用于定义Django框架, 直接找到**app01/myfunc.py**, 很容易可以找到简单的sql注入的waf:

```python
def waf(sql):
    blacklists = ['union select', 'sleep', 'benchmark', 'columns', 'load_file', 'local', 'outfile', 'dumpfile', 'file']
    for blacklist in blacklists:
        if blacklist in sql:
            print(blacklist)
            return False
        return True
```

除此之外还有其他几个waf函数, 那么要想防止注入, 需要从可能的注口下手, 在vim中直接进行`:/waf`搜索找到对应的位置, 检查到在index类中存在一段函数:

```python
sql = "select * from app01_user where username = '"+login_username+"';"
xiaoxi = ""
if waf(login_username):
    cursor.execute(sql)
    result = cursor.fettchall()
    if result:
        xiaoxi = result[0][3]
else:
    xiaoxi = "No NO N0!"
```

很暴力的一个方法, 顺便学了一手, 除了`eval`以外`execute`函数也应该是一个要被重点检查的函数.

陌心师傅提供了一个基本的修复方法, 直接增强waf(这大概需要考察web手在处理手段的基本素养了, 多出题还是有好处的):

```
def waf(sql):
    blacklists = ["union select", "sleep", "benchmark","columns","load_file","local","outfile","dumpfile","file","union","select",
"select","and","*","x09","x0a","x0b","x0c","x0d","xa0","x00","x26","x7c","or","into","from","where","join","sleexml","extractvalue","+","regex","copy","read","file","create","grand","dir","insert","link","server","drop","=",">","<",";"]
    for blacklist in blacklists:
        if blacklist in sql:
            print(blacklist)
            return False
    return True
```

上了php的通防脚本, 防御成功了, 有时间我也写点自己的逆天想法(所以现在先鸽着).

## found_cms

## easygo

golang会编译为二进制文件, 需要有强大的patch技巧, 这里只进行记录:

IDA64打开easygo文件, 找到`loc_49B5DE`处汇编显示:

```asm
loc__49B5DE:
xchg    ax, ax
call    main_backdoor
jmp     short loc_49B5E9
```

继续追踪, 发现有调用`cat /flag`和`/bin/bash`两个部分, 既然是AWDP, 不难想象出官方的EXP, 直接暴力修改db部分, 将`cat /flag`修改为`echo 1234`, 个人思考了一下, 直接用retdec逆出来然后暴力改文件可能也是可以的?

## PHP通防

```python
function wafrce($str){
	return !preg_match("/openlog|syslog|readlink|symlink|popepassthru|stream_socket_server|scandir|assert|pcntl_exec|fwrite|curl|system|eval|assert|flag|passthru|exec|chroot|chgrp|chown|shell_exec|proc_open|proc_get_status|popen|ini_alter|ini_restore/i", $str);
}

function wafsqli($str){
	return !preg_match("/select|and|\*|\x09|\x0a|\x0b|\x0c|\x0d|\xa0|\x00|\x26|\x7c|or|into|from|where|join|sleexml|extractvalue|+|regex|copy|read|file|create|grand|dir|insert|link|server|drop|=|>|<|;|\"|\'|\^|\|/i", $str);
}

function wafxss($str){
	return !preg_match("/\'|http|\"|\`|cookie|<|>|script/i", $str);
}


function waf($s){
  if (preg_match("/select|flag|union|\\$|'|"|--|#|\0|into|alert|img|prompt|set|/*|x09|x0a|x0b|x0c|x0d|xa0|%|<|>|^|x00|#|x23|[0-9]|file|=|or|x7c|select|and|flag|into|where|x26|'|"|union|`|sleep|benchmark|regexp|from|count|procedure|and|ascii|substr|substring|left|right|union|if|case|pow|exp|order|sleep|benchmark|into|load|outfile|dumpfile|load_file|join|show|select|update|set|concat|delete|alter|insert|create|union|or|drop|not|for|join|is|between|group_concat|like|where|user|ascii|greatest|mid|substr|left|right|char|hex|ord|case|limit|conv|table|mysql_history|flag|count|rpad|&|*|.|/is",$s)||strlen($s)>50){
    header("Location: /");
    die();
  }
}
```

# 网鼎杯线下赛

然后是kar3a师傅在网鼎杯的记录, 据说有点像ciscn, 有时间也看看那边的题目[^kar3a].

## alicewebsite

php题, 直接丢D盾里面, 找到可疑引用:

```php
<?php
        $action = (isset($_GET['action']) ? $_GET['action'] : 'home.php');
        if (file_exists($action)) {
            include $action;
        } else {
            echo "File not found!";
        }
?>
```

D盾其实在长城杯半决赛过程中我们有配置过, 现在看来awd和awdp还是共通的, 只不过没了搅混水的过程(不过某种意义上添加一点选手间后期的斗智斗勇会更有意思, 个人感觉), 发现用户传入的`$action`没有进行过滤, 可以直接通过目录穿越, payload:`?action=../../../../flag`

防御手段是构造一个黑名单防止目录穿越:

```php
<?php
        $action = (isset($_GET['action']) ? $_GET['action'] : 'home.php');
		$deny_ext = array("..","../");
		if (in_array($action, $deny_ext)){
            if (file_exists($action)) {
                include $action;
            } else {
                echo "File not found!";
            }
		}
?>
```

## fake

buu上有原题, `/admin`进入后台, 是目录穿越:

```php
function downloadBak() {
        $file_name = $_GET['file'];
        $file_dir = $this->config['path'];
        if (!file_exists($file_dir . "/" . $file_name)) { //检查文件是否存在
            return false;
            exit;
        } else {
            $file = fopen($file_dir . "/" . $file_name, "r"); // 打开文件
            // 输入文件标签
            header('Content-Encoding: none');
            header("Content-type: application/octet-stream");
            header("Accept-Ranges: bytes");
            header("Accept-Length: " . filesize($file_dir . "/" . $file_name));
            header('Content-Transfer-Encoding: binary');
            header("Content-Disposition: attachment; filename=" . $file_name);  //以真实文件名提供给浏览器下载
            header('Pragma: no-cache');
            header('Expires: 0');
            //输出文件内容
            echo fread($file, filesize($file_dir . "/" . $file_name));
            fclose($file);
            exit;
        }
    }
```


# 参考文献

[^moxin]: 【强网杯】第六届强网杯青少年线下赛AWDP复盘[EB/OL].个人博客.<a target="_blank" href='https://moxin1044.github.io/articles/57061.html'>https://moxin1044.github.io/articles/57061.html</a>.2024.10.28.
[^kar3a]: 一次awdplus经历（网鼎杯线下赛）[EB/OL].个人博客.<a target="_blank" href='https://www.cnblogs.com/karsa/p/14062819.html'>https://www.cnblogs.com/karsa/p/14062819.html</a>.2020.11.30
[^solutions]: 暑假作业写了没.x^2+y^2=n的整数解[EB/OL].知乎.https://zhuanlan.zhihu.com/p/668845092.2023.11.26
[^complex]: Wbuildings.国城杯2024[EB/OL].个人博客.https://wbuildings.github.io/Crypto/%E5%9B%BD%E5%9F%8E%E6%9D%AF2024/#more.2024.12.07

