# 如何用起来

## 下载代码

通过 Git Clone SSH 方式（HTTPS国内不方便）
```
git clone git@github.com:jswxharry/get_jobs_plus.git
cd get_jobs_plus
```

对Git 不熟悉的小伙伴请参考教程 [git 教程 中文版](https://liaoxuefeng.com/books/git/introduction/index.html)
下载安装 [Git for windows](https://git-scm.com/install/windows)

没配置过Github SSH key的请参考下面文章

### GitHub SSH 配置
- [About SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)
- [生成新的SSH key 并且添加至你本机](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- [给你的Github账户添加新的SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- [国内用户遇到ssh: connect to host github.com port 22 Connection timed out 请参考](https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port)

## 本地环境依赖安装

### JDK21
下载链接：https://www.oracle.com/java/technologies/downloads/#java21
或者其他的21 TLS 版本

### 下载Maven
Maven 地址：https://maven.apache.org/download.cgi
3.9 版本 安装可以基于官方说明：https://maven.apache.org/install.html

### 下载 VS Code
下载链接：https://code.visualstudio.com/

### NodeJS V20.19.6 TLS
下载地址:https://nodejs.org/zh-cn/download
选windows 安装程序MSI 版，安装中选添加到PATH(默认的)，其他不安装

检查安装成功：
```
node --version
npm -version
```

## 用 VS Code 启动项目
 - 用VS Code 打开项目目录
 - 第一次配置应该会提示你安装各种Java插件配置，跟着走就行，也可以看下提示的使用教程
 - 正常的话会自动经行依赖下载，等它完成
 
 ### 启动前端配置界面
 在Terminal tab 里启动一个新的command promt
 安装依赖（install)
 启动环境（run dev)
 ```
 cd front
 npm install
 npm run dev
 ```

 ### 启动后端服务
 - 选中 src\main\java\com\getjobs\GetJobsApplication.java 点击run java


## 其他
### VS Code 插件 SQLite Viewer
可以安装 SQLite Viewer 插件看数据库里的内容

## 高级功能

### BOSS调试模式（仅收集职位信息，不投递）

BOSS平台支持调试模式，开启后只会收集职位信息并保存到数据库，不会实际投递简历。

#### 适用场景
- 职位分析和筛选：先收集数据再决定是否投递
- 测试配置效果：验证关键词、过滤条件是否生效
- 数据备份和调研：保存职位信息用于后续分析
- 避免误投：确保配置正确后再正式投递

#### 开启方法
在 `boss_config` 表中设置 `debugger` 字段为 `1`：
- 通过前端配置界面：在BOSS配置页面找到"调试模式"选项并开启（暂时没有界面）
- 或直接修改数据库：`UPDATE boss_config SET debugger = 1 WHERE id = 1`

#### 功能特点
1. **完整数据收集**：会遍历所有匹配的职位，解析职位详情并保存到 `boss_data` 表
2. **过滤规则生效**：黑名单、薪资过滤、HR活跃状态过滤等规则都会正常执行
3. **状态标记清晰**：职位会被标记为"未投递"或"已过滤"状态
4. **支持后续分析**：可在前端查看职位分析报表，支持按状态、薪资、公司等维度筛选

#### 实现原理
在 [`Boss.java`](src/main/java/com/getjobs/worker/boss/Boss.java:610-612) 中实现：
```java
// 调试模式：仅遍历不投递
if (Boolean.TRUE.equals(config.getDebugger())) {
    log.info("调试模式：仅遍历岗位，不投递 | 公司：{} | 岗位：{}", job.getCompanyName(), job.getJobName());
    return;
}
```

开启调试模式后，系统会执行完整的职位搜索和过滤逻辑，但在实际投递前会跳过投递步骤，确保只收集数据不发送简历。