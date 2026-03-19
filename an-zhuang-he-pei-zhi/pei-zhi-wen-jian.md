# 配置文件

DDICOPY的配置文件支持yaml或json格式，主要分为2类：全局配置文件和拷贝配置文件，其中全局配置文件用来定义各个模块的相关配置，拷贝配置文件用来定义文件拷贝的相关规则信息（如源地址、目的地址、路径匹配信息等），二者是引用关系，全局配置文件需要在其拷贝模块的配置中引用拷贝配置文件的路径。

## 全局配置文件

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

*   general

    用于定义一些全局的通用配置

<table data-full-width="true"><thead><tr><th width="170.39996337890625">配置项</th><th width="80.20001220703125">类型</th><th width="100.99993896484375">是否必填</th><th width="129.9998779296875">默认值</th><th>说明</th></tr></thead><tbody><tr><td>instance_name</td><td>str</td><td>是</td><td>无</td><td>实例名称，用于区分不同实例，⚠️同一台主机上的不同实例必需使用<strong>不同</strong>的instance_name。</td></tr><tr><td>log_dir      </td><td>str</td><td>否</td><td>${BASE_DIR}/${INSTANCE_NAME}/logs</td><td>日志目录，DDICOPY支持为不同的<a href="ri-zhi.md#ri-zhi-lei-xing">日志类型</a>指定不同的目录，但为了统一方便管理，建议配置log_dir的值，所有日志会统一放到这个目录下。</td></tr></tbody></table>

* watch

&#x20;     watchdog监听的相关配置，注意此部分不是必需的，但watch和http\_server二者必需设置其一

<table><thead><tr><th width="207.79998779296875">配置项</th><th width="80.20001220703125">类型</th><th width="114.59991455078125">是否必填</th><th width="127.5999755859375">默认值</th><th>说明</th></tr></thead><tbody><tr><td>watch_dirs</td><td>str</td><td>是</td><td>无</td><td>watchdog监听的目录，可以指定多个，⚠️运行前会检查目录是否存在，不存在会报错。</td></tr><tr><td>watch_include_pattern</td><td>str</td><td>否</td><td>无</td><td>正则表达式模式，watchdog监听后，匹配这个模式的加入队列</td></tr><tr><td>watch_exclude_pattern</td><td>str</td><td>否</td><td>无</td><td>正则表达式模式，watchdog监听后，匹配这个模式不加入队列</td></tr></tbody></table>

* copy

&#x20;     用于定义拷贝模块的相关配置

<table><thead><tr><th width="207.79998779296875">配置项</th><th width="77">类型</th><th width="114.59991455078125">是否必填</th><th width="127.5999755859375">默认值</th><th>说明</th></tr></thead><tbody><tr><td>copy_worker_numbers</td><td>int</td><td>否</td><td>4</td><td>拷贝程序的线程数，即拷贝的并发数，如果队列繁忙，可以适当调整该值。</td></tr><tr><td>copy_config_path</td><td>str</td><td>是</td><td>无</td><td>拷贝配置文件的路径，该值可以为目录或具体配置文件，如果指定为目录，则会遍历目录下的的yml或json文件，⚠️运行前会校验配置文件的有效性。</td></tr><tr><td>keep_file_attribute</td><td>bool</td><td>否</td><td>true</td><td>是否保留文件的属性，文件复制后，是否修改文件的属性标记（Windows系统：去掉文件的归档属性；Linux系统：给文件增加sticky bit标志位）</td></tr></tbody></table>

* http\_server

&#x20;     用于定义http\_server模块的相关配置，当使用watchdog时，可以不配置，但为了方便管理服务进程，建议配置，加入使用sftpgo的webhook发送文件信息，则必需配置。

<table><thead><tr><th width="207.79998779296875">配置项</th><th width="75.39996337890625">类型</th><th width="114.59991455078125">是否必填</th><th width="127.5999755859375">默认值</th><th>说明</th></tr></thead><tbody><tr><td>listen_host</td><td>str</td><td>否</td><td>127.0.0.1</td><td>监听的IP，为保证安全，⚠️建议配置127.0.0.1。</td></tr><tr><td>listen_port</td><td>int</td><td>否</td><td>9696</td><td>监听的端口，⚠️注意，多个实例时端口不能冲突。</td></tr><tr><td>http_include_pattern</td><td>str</td><td>否</td><td>无</td><td>正则表达式模式，http_server接收到文件信息后，匹配这个模式的加入队列</td></tr><tr><td>http_exclude_pattern</td><td>str</td><td>否</td><td>无</td><td>正则表达式模式，ttp_server接收到文件信息后，匹配这个模式不加入队列</td></tr></tbody></table>

## 拷贝配置文件

### 概述

&#x20;      我们将整个拷贝信息抽象成”源地址“和”目的地址“，地址信息又包含”地址类型“和”连接信息“，拷贝配置文件即包含多个拷贝信息的yml或json文件。

### 连接信息

除了本地地址以外，sftp、ftp、oss、s3都需要配置连接信息，以连接到对应的地址

#### SFTP

<table><thead><tr><th width="207.79998779296875">配置项</th><th width="80.20001220703125">类型</th><th width="104.19989013671875">是否必填</th><th width="88.39990234375">默认值</th><th>说明</th></tr></thead><tbody><tr><td>hostname</td><td>str</td><td>是</td><td>无</td><td>连接地址</td></tr><tr><td>port</td><td>int</td><td>是</td><td>无</td><td>端口号</td></tr><tr><td>username</td><td>str</td><td>是</td><td>无</td><td>用户名</td></tr><tr><td>password</td><td>str</td><td>否</td><td>无</td><td>密码，⚠️密码和密钥文件需指定一个</td></tr><tr><td>key_file</td><td>str</td><td>否</td><td>无</td><td>密钥文件路径，⚠️密码和密钥文件需指定一个</td></tr><tr><td>pass_pharse</td><td>str</td><td>否</td><td>无</td><td>密钥文件密码</td></tr><tr><td>user_proxy</td><td>bool</td><td>否</td><td>false</td><td>是否使用代理</td></tr><tr><td>proxy_url</td><td>str</td><td>否</td><td>无</td><td><p>代理地址：一般代理的格式，如 http://[username:password]@127.0.0.1:10808</p><p>⚠️当user_proxy为true时，必需指定<br>⚠️SFTP支持socks和http代理</p></td></tr></tbody></table>

#### FTP

<table><thead><tr><th width="207.79998779296875">配置项</th><th width="80.20001220703125">类型</th><th width="104.19989013671875">是否必填</th><th width="88.39990234375">默认值</th><th>说明</th></tr></thead><tbody><tr><td>hostname</td><td>str</td><td>是</td><td>无</td><td>连接地址</td></tr><tr><td>port</td><td>int</td><td>是</td><td>无</td><td>端口号</td></tr><tr><td>username</td><td>str</td><td>是</td><td>无</td><td>用户名</td></tr><tr><td>password</td><td>str</td><td>是</td><td>无</td><td>密码</td></tr><tr><td>user_proxy</td><td>bool</td><td>否</td><td>false</td><td>是否使用代理</td></tr><tr><td>proxy_url</td><td>str</td><td>否</td><td>无</td><td><p>代理地址：一般代理的格式，如 http://[username:password]@127.0.0.1:10808</p><p>⚠️当user_proxy为true时，必需指定<br>⚠️FTP只支持socks代理</p></td></tr></tbody></table>

#### S3

<table><thead><tr><th width="207.79998779296875">配置项</th><th width="80.20001220703125">类型</th><th width="104.19989013671875">是否必填</th><th width="88.39990234375">默认值</th><th>说明</th></tr></thead><tbody><tr><td>s3_access_key_id</td><td>str</td><td>否</td><td>无</td><td>aws的access key id：<br>1、可以直接指定该值<br>2、如果不配置，则会按照aws本身的配置方式去认证（角色认证或者本地配置文件）</td></tr><tr><td>s3_secret_access_key</td><td>str</td><td>否</td><td>无</td><td><p>aws的secret access key：</p><p>配置方式同上 </p></td></tr><tr><td>s3_region</td><td>str</td><td>是</td><td>无</td><td>S3 区域</td></tr><tr><td>s3_bucket_name</td><td>str</td><td>是</td><td>无</td><td>S3 桶名称</td></tr><tr><td>user_proxy</td><td>bool</td><td>否</td><td>false</td><td>是否使用代理</td></tr><tr><td>proxy_url</td><td>str</td><td>否</td><td>无</td><td><p>代理地址：一般代理的格式，如 http://[username:password]@127.0.0.1:10808</p><p>⚠️当user_proxy为true时，必需指定<br>⚠️S3只支持http代理</p></td></tr></tbody></table>

#### OSS

<table><thead><tr><th width="207.79998779296875">配置项</th><th width="80.20001220703125">类型</th><th width="104.19989013671875">是否必填</th><th width="88.39990234375">默认值</th><th>说明</th></tr></thead><tbody><tr><td>oss_access_key_id</td><td>str</td><td>否</td><td>无</td><td>阿里云的access key id：<br>1、可以直接指定该值<br>2、支持通过环境变量ALIBABA_CLOUD_ACCESS_KEY_ID指定</td></tr><tr><td>oss_access_key_secret</td><td>str</td><td>否</td><td>无</td><td><p>阿里云的secret access key：</p><p>1、可以直接指定该值<br>2、支持通过环境变量ALIBABA_CLOUD_ACCESS_KEY_SECRET指定</p></td></tr><tr><td>oss_region</td><td>str</td><td>是</td><td>无</td><td>OSS 区域</td></tr><tr><td>oss_bucket_name</td><td>str</td><td>是</td><td>无</td><td>OSS 桶名称</td></tr><tr><td>oss_endpoint</td><td>str</td><td>是</td><td>无</td><td>OSS 端点信息</td></tr><tr><td>user_proxy</td><td>bool</td><td>否</td><td>false</td><td>是否使用代理</td></tr><tr><td>proxy_url</td><td>str</td><td>否</td><td>无</td><td><p>代理地址：一般代理的格式，如 http://[username:password]@127.0.0.1:10808</p><p>⚠️当user_proxy为true时，必需指定<br>⚠️OSS只支持http代理</p></td></tr></tbody></table>

### 源

<table><thead><tr><th width="204.60003662109375">配置项</th><th width="81.5999755859375">类型</th><th width="102.20001220703125">是否必填</th><th width="95">默认值</th><th>说明</th></tr></thead><tbody><tr><td>source_type</td><td>str</td><td>否</td><td>local</td><td>源地址类型。⚠️枚举值：必需是local、sftp、oss、s3种的一种。</td></tr><tr><td>source_path_match</td><td>str</td><td>是</td><td>无</td><td>源地址匹配，当传入的源地址与该配置以及源地址类型匹配时，才会执行复制，可以是一个路径，也可以包含通配符。如：/data/src匹配/data/src开头的文件，/data/*/online匹配/data/DDI/online和/data/FTP/online</td></tr><tr><td>source_split_length</td><td>int</td><td>是</td><td>无</td><td>对源路径按照路径分隔符拆分的索引，会将拆分后的部分与目标路径组合成新的路径。如：<br>源文件地址为 /data/FTP/online/a.csv，指定source_split_length为1，则会将源地址拆分成"/data"和"/FTP/online/a.csv"，假如目标地址为"/dest/“，那么最终的文件目标路径为"/dest/FTP/online/a.csv"<br>⚠️2种特殊情况<br>1. 如果设置的值大于按照文件路径分隔符分隔的最大长度，如上面路径source_split_length设置了大于5的值，则将source_split_length设置为5。<br>2.source_split_length可以设置为-1，此时表示只取文件名，然后与目标地址组合，如上面例子最终组成的路径为/dest/a.csv。</td></tr><tr><td>source_include_pattern</td><td>str</td><td>否</td><td>无</td><td>正则表达式模式，匹配该模式的文件才会复制</td></tr><tr><td>source_exclude_pattern</td><td>str</td><td>否</td><td>无</td><td>正则表达式模式，不匹配该模式的文件才会复制</td></tr><tr><td>source_include_pattern_or_part_file</td><td>str</td><td>否</td><td>无</td><td>这是一个文件路径，这个文件中每一行一个简单的正则表达式，当程序启动是，会将这些正则表达式通过"|"连接起来，传递给上面的source_include_pattern，适用于多个正则的情况，便于维护</td></tr><tr><td>source_exclude_pattern_or_part_file</td><td>str</td><td>否</td><td>无</td><td>类似上面，将文件中的单行正则通过"|"组合传递给source_exclude_pattern</td></tr><tr><td>source_connection_config</td><td>class</td><td>否</td><td>无</td><td>具体的连接配置信息</td></tr></tbody></table>

### 目标

### 配置实例
