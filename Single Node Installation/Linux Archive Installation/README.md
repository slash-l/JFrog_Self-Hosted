# Single Node Installation
## Linux Archive Installation

### 前置条件

1. 演示环境[准备](https://github.com/j1an5/JFrog_Self-Hosted#%E6%BC%94%E7%A4%BA%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)

### Artifactory
1. 设置环境变量,JFrog主目录 JFORG_HOME(Set the JFrog Home environment variable)
    ```shell
    echo "export JFROG_HOME=/opt/jfrog" >> /etc/profile
    source /etc/profile
    ```
2. 创建主目录,进入主目录并下载需要的版本包,本文使用的版本是7.33.9(Create a JFrog Home directory and move the downloaded installer archive into that directory, for example:)
    ```
    mkdir -p $JFROG_HOME && cd $JFROG_HOME/
    wget https://releases.jfrog.io/artifactory/artifactory-pro/org/artifactory/pro/jfrog-artifactory-pro/7.33.9/jfrog-artifactory-pro-7.33.9-linux.tar.gz
    rtV=7.33.9
    ```
3. 提取压缩包并重命名(Extract the contents of the compressed archive and move it into the artifactory directory.)
    ```
    tar -xvf jfrog-artifactory-pro-${rtV}-linux.tar.gz
    mv artifactory-pro-${rtV} artifactory
    ```
4. 修改配置文件-必要(Customize the product configuration (optional) including database, Java Opts, and filestore.)
    ```terminal
    # cp $JFROG_HOME/artifactory/var/etc/{system.full-template.yaml,system.yaml}
    # vim $JFROG_HOME/artifactory/var/etc/system.yaml
    shared:
    ....
      node:
        id: "art1"
        ip: "192.168.xx.xx”
    ....
    ```
5. 运行Artifactory(Run Artifactory)
    | Daemon Process |
    | ---- |
    | $JFROG_HOME/artifactory/app/bin/artifactoryctl start |
6. 检查日志(Check Artifactory Log.)
    ```
    # tail -f $JFROG_HOME/artifactory/var/log/console.log
    ....
    ###############################################################
    ###   All services started successfully in xx.xxx seconds   ###
    ###############################################################
    ....
    ```
7. 访问Artifactory(Access Artifactory from your browser.)
    > http://SERVER_HOSTNAME:8082
### Xray
1. 设置环境变量(Set the JFrog Home environment variable)
    ```shell
    echo "export JFROG_HOME=/opt/jfrog" >> /etc/profile
    source /etc/profile
    ```
2. 创建主目录,进入主目录并下载需要的版本包,本文使用的版本是3.43.1(Create a JFrog Home directory and move the downloaded installer archive into that directory, for example:
    ```shell
    mkdir -p $JFROG_HOME && cd $JFROG_HOME/
    wget https://releases.jfrog.io/artifactory/jfrog-xray/xray-linux/3.43.1/jfrog-xray-3.43.1-linux.tar.gz
    ```
3. 提取压缩包并重命名(Extract the contents of the compressed archive and move it into xray directory.)
    ```shell
    tar -xvf jfrog-xray-3.43.1-linux.tar.gz
    mv jfrog-xray-3.43.1-linux xray
    ```
4. 前置条件(Prerequisites)
    1. PostgreSQL [安装](https://www.postgresql.org/download/linux/redhat/)
        ```terminal
        yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
        yum install -y postgresql12-server
        /usr/pgsql-12/bin/postgresql-12-setup initdb
        systemctl enable postgresql-12 && systemctl start postgresql-12
        ```
    2. PostgreSQL 信任设置
        ```terminal
        # vim /var/lib/pgsql/12/data/pg_hba.conf
            # "local" is for Unix domain socket connections only
            local   all             all                                     peer
            # IPv4 local connections:
            host    all             all             127.0.0.1/32            trust
            # IPv6 local connections:
            host    all             all             ::1/128                 trust
            # Allow replication connections from localhost, by a user with the
            # replication privilege.
            local   replication     all                                     peer
            host    replication     all             127.0.0.1/32            md5
            host    replication     all             ::1/128                 md5
        # systemctl restart postgresql-12
        ```
    3. 创建Xray数据库
        ```
        # su - postgres //切换用户并执行创建数据库脚本
        $ POSTGRES_PATH=$(dirname $(readlink -f $(which psql))) $JFROG_HOME/xray/app/third-party/postgresql/createPostgresUsers.sh
        ....main] - Waiting for Postgres to get ready using the commands: "/usr/pgsql-12/bin/psql --host=localhost --port=5432 --version" & "/usr/pgsql-12/bin/psql --host=localhost --port=5432 -l"
        ....main] - Postgres is ready. Executing commands
        ....main] - Postgres setup is now complete
        $ exit
        ```
    4. 安装rabbitmq依赖 - 压缩包内有提供,版本可能不同
        ```shell
        rpm -ivh --replacepkgs $JFROG_HOME/xray/app/third-party/rabbitmq/socat-1.7.3.2-2.el7.x86_64.rpm
        rpm -ivh --replacepkgs $JFROG_HOME/xray/app/third-party/rabbitmq/erlang-23.2.3-1.el7.x86_64.rpm
        ```
    5. Db-Utils - 压缩包内有提供,版本可能不同
        ```shell
        yum install -y $JFROG_HOME/xray/app/third-party/misc/libdb-utils-5.3.21-25.el7.x86_64.rpm
        ```
5. 修改配置文件(Customize the product configuration)
    >![Artifactory Join Key 1](https://github.com/j1an5/JFrog_Self-Hosted/blob/main/resource/images/Artifactory%20Join%20Key%201.png?raw=true)
    ![Artifactory Join Key 2](https://github.com/j1an5/JFrog_Self-Hosted/blob/main/resource/images/Artifactory%20Join%20Key%202.png?raw=true)

    ```
    # cp $JFROG_HOME/xray/var/etc/{system.full-template.yaml,system.yaml}
    # vim $JFROG_HOME/xray/var/etc/system.yaml
    configVersion: 1
      shared:
        jfrogUrl: http://192.168.xx.xx:8082/
        security:
          joinKey: "xxxxxxxxxxxxxxxx"
        node:
          id: "xray1"
          ip: "192.168.xx.xx"
        database:
          type: postgresql
          driver: org.postgresql.Driver
          url: "postgres://localhost:5432/xraydb?sslmode=disable"
          username: xray
          password: xxxxxxxxxxxx
    ```
6. 启动Xray(Start and manage the Xray service as the user who extracted the tar)
    | Daemon Process |
    | ---- |
    | $JFROG_HOME/xray/app/bin/xray.sh start |
7. 检查启动日志(Check Xray Log.)
    ```termminal
    # tail -n100 -f $JFROG_HOME/xray/var/log/console.log
    ###############################################################
    ###   All services started successfully in xx.xxx seconds   ###
    ###############################################################
    ```
8. 访问Xray(Access Xray from your browser)
    >http://jfrogUrl
