# 配置文件

程序程序程序程序    DDICOPY的配置文件支持yaml或json格式，主要分为2类：全局配置文件和拷贝配置文件，其中全局配置文件用来定义各个模块的相关配置，拷贝配置文件用来定义文件拷贝的相关规则信息（如源地址、目的地址、路径匹配信息等），二者是引用关系，全局配置文件需要在其拷贝模块的配置中引用拷贝配置文件的路径。

1. 全局配置文件

以下用一个实际生产中用到的实际配置文件加以说明：

```yml
general:
  instance_name: LOCAL2SAAS_4_203
  log_dir: /data/ddicopylogs/ddicopyv2/instances/LOCAL2SAAS_4_203/logs
  
watch:
  watch_dirs:
    - /data/SFTPDATA.temp/FYYY
    - /data/SFTPDATA.temp/DIAO

copy:
  copy_worker_numbers: 8
  copy_config_path: /data/ddicopylogs/ddicopyv2/instances/LOCAL2SAAS_4_203/configs/copyconfigs
  copy_record_file_backup: false
  keep_file_attribute: false
  
http_server:
  listen_host: 127.0.0.1
  listen_port: 9601
```

* general

&#x20;      一些全局通用的设置：

&#x20;       1\. instance\_name: 必需定义，实例名称，DDICOPY可以运行多个实例(进程)，用于区分不同实例，同一台主机上的不同实例必需使用**不同**的instance\_name。

&#x20;       2\. log\_dir: 日志目录，DDICOPY支持为不同的[日志类型](ri-zhi.md)指定不同的目录，但为了统一方便管理，建议配置log\_dir的值，所有日志会统一放到这个目录下。另外log\_dir非必需，如果没有设置，会在"基目录"自动创建一个名为"logs"的目录，Linux系统会在家目录的"\~/.ddiccopy/${instance\_name}/logs"，Windows系统会在代码所在目录的上级"../.ddiccopy/${instance\_name}/logs"



* watch
* copy
* http\_server
