### 镜像制作过程有时候我们需要做定制化的开发，但是有时候我们也需要小心定制开发的过程中文件参数。


## 接下来我们先来了解一下DockerFile的参数还有书写规则吧！
#### 构建指令（用于构建镜像，其指定的操作不会在由镜像运行的容器上执行）

（1） FROM（指定基础镜像）

- FROM必须是第一条非注释指令，它指定基础镜像且指定的镜像是已存在的。
  
- 格式：

    FROM <image>   # 指定基础镜像

    FROM <image>:<tag> # 指定一个tag版本的基础镜像

例子：
- 
    FROM ubuntu:14.04


（2）MAINTAINER（指定镜像创建者信息）

- 指定镜像的作者和联系方式信息。
- 格式：

    MAINTAINER <author> "e-mail"  # 指定作者名和E-mail

（3）RUN（安装软件用） 

    # RUN <command> 

    # shell模式，以#/bin/sh -c command 形式执行, 如RUN echo hello 
                                                                                
    # RUN ["executable", "param1", "param2" ... ] 

    # exec模式，指定其他形式的shell来运行指令 ,如RUN ["/bin/bash" ,“-c”,“echo  hello" ]

例子：

    # RUN apt-get update && apt-get install -y nginx # 合并执行命令

（4）ENV（用于设置环境变量）

在镜像中设置一个环境变量。

格式:

    > ENV  <key>  <value>
    > # 设置后，后续RUN命令都可使用环境变量所输出的指令，容器启动ENV <key>=<value>后，通过#docker inspect查看环境变量，也可以通过 # docker run --env key=value设置或修改环境变量。     
    
例子：



`> ENV  JAVA_HOME  /path/to/java/dirent` 
`> 在容器中安装JAVA程序，设置JAVA_HOME` 



（5）ADD COPY（将本地文件或目录复制到由dockerfile构建的镜像中）

所有拷贝到container中的文件和文件夹权限为0755，uid和gid为0；如果是一个目录，那么会将该目录下的所 有文件拷贝到目录下，不包括目录；如果是文件且中不使用斜杠结束，则会将视为文件，的内容会写入；如果是文件且中使用斜杠结束，则会文件拷贝到目录下。
格式:
                                      
            
    ADD  <src>  <dest> 
    
    # <src>可以是文件或目录的本地地址（即Dockerfile所在目录的相对路径），也可以是一个远程的url（docker不推荐，更建议使用wget或curl获取文件）; <dest>是镜像中的绝对路径 
    
    ADD  ["<src>" "<dest>"]
    
    # 适用于文件路径中有空格的情况，同理有COPY的情况
    
    COPY <src>  <dest> 
    
    COPY ["<src>" "<dest>"]

- ADD与COPY的区别：ADD指令包含类似tar的解压功能，而COPY只单纯复制文件。

- 例子 ：

    `# vim   index.html  # 编辑一个index.html文件`

	`# 在Dockfile中写下`

`COPY  index.html   /usr/share/nginx/html/`   

`# copy Dockerfile所在目录的文件到容器的绝对路径下，使用docker build 指令后，运行容器，查看端口映射，并# curl http：//127.0.0.1：host_port 可以看到index.html的内容`   

### 设置指令（用于设置image的属性，其指定的操作将在由image运行的容器中执行）            

  （6）CMD（提供容器运行的默认命令）
提供容器运行的默认命令。与RUN类似，但RUN指定的命令是在容器构建过程中运行，而CMD指定的命令是在容器运行过程中运行。该指令只能在文件中存在一次，如果有多个，则只执行最后一条。

- 格式：

`CMD   ["executable","param1","param2"] # exec模式`  
`CMD   command param1 param2# shell模式` 
`CMD   ["param1","param2"]`                      
`# 当Dockerfile指定了ENTRYPOINT时使用，其作为ENTRYPOINT指令的默认参数` 

例子：

    # docker  run  --name  test  -d   repository  cmd   //构建容器中的CMD指令被run的cmd覆盖，不会执行
    # docker  run  --name  test  -d   repository //执行构建容器中的CMD指令

（7）ENTRYPOINT（提供容器运行的默认命令）

- 指定容器运行时执行的命令，可以多次设置，但是只有最后一个有效。
- 格式:

`ENTRYPOINT ["executable", "param1", "param2"]` 

#### exec模式

`ENTRYPOINT command param1 param2 `  

#### shell模式 

例子：

- 该指令的使用分为两种情况，一种是独自使用，另一种和CMD指令配合使用。


- a. 当独自使用时，如果你还使用了CMD命令且CMD是一个完整的可执行的命令，那CMD指令和ENTRYPOINT会互相覆盖，只有最后一个CMD或者ENTRYPOINT有效。构建容器中的ENTRYPOINT指令不会被run的cmd覆盖，ENTRYPOINT指令仍会执行。

    CMD   echo “Hello, World!”

    ENTRYPOINT  ls  -l 

    CMD指令将不会被执行，只有ENTRYPOINT指令被执行

- b. 与CMD指令配合使用来指定ENTRYPOINT的默认参数。ENTRYPOINT指令只能使用exec模式，指定执行命令，而不能指定参数；CMD指令不是一个完整的可执行命令，仅仅是参数部分。

- 在Dockerfile中，写如下内容

    FROM ubuntu  
    ENTRYPOINT ["/usr/bin/nginx"]  
    CMD ["-h"] 



在命令行中，执行以下命令

    # docker build -t="locutus1/nginx"  .# 构建docker镜像
    
    # docker run -p 80 --name nginx_test2 -d locutus1/nginx -g "daemon  off;" 
    
    # docker ps -l

命令# docker run 的-g “daemon off;” 指令将CMD中的-h指令覆盖。

（8）USER（设置container容器的用户）
设置启动容器的用户，默认是root用户。

格式：

    USER  user 
    USER  uid
    USER  user：group
    USER  uid：gid
    USER  user：gid
    USER  uid：group 

例子：

    ENTRYPOINT ["memcached"]  # 指定指令memcached的运行用户为daemon  
    USER daemon   
	或 
    ENTRYPOINT ["memcached", "-u", "daemon"]

（9）EXPOSE（指定容器需要映射到宿主机的端口）

- 该指令会将容器中的端口映射成宿主机中的某个端口。当你需要访问容器的时候，可以不使用容器的IP地址而是使用宿主机的IP地址和映射后的宿主机端口。要完成整个操作需要两个步骤：
- 1. 在Dockerfile使用EXPOSE设置需要映射的容器端口
- 2. 在运行容器的时候指定-p选项和EXPOSE设置的端口，这样EXPOSE设置的端口号会被随机映射成宿主机器中的一个端口号。
- 格式:

`EXPOSE <port> [<port>...]` 

例子:

    EXPOSE port1 // 映射一个端口  
    # docker run -p port1 image  // 运行容器使用的相应命令 



    EXPOSE port1 port2 port3 //映射多个端口  
    
    # docker  run  -p  port1  -p  port2-p   port3  image   
    
    // 运行容器使用的相应命令   
    
    # docker run -p host_port1:port1 -p host_port2:port2 -p host_port3:port3  image
     //指定需要映射到宿主机上的某个端口号  
    
    # docker port port1 containerID   
    
    //对于一个运行的容器，可以使用docker port加上容器中需要映射的端口和容器的ID来查看该端口号在宿主机器上的映射端口




端口映射是docker比较重要的一个功能，原因在于我们每次运行容器时容器IP地址不能指定而是在桥接网卡的地址范围内随机生成的。宿主机器的IP地址是固定的，我们可以将容器的端口的映射到宿主机的一个端口，免去每次访问容器中的某个服务时都要查看容器的IP的地址。


（10）VOLUME（指定挂载点）

使容器中的一个目录具有持久化存储数据的功能，该目录可以被容器本身使用，也可以共享给其他容器使用。我们知道容器使用的是AUFS，这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失。当容器中的应用有持久化数据的需求时，可以在Dockerfile中使用该指令。
格式:

`VOLUME ["<mountpoint>"]`

例子：

`VOLUME ["/tmp/data"]` 



运行通过该Dockerfile构建镜像的容器，/tmp/data目录中的数据在容器关闭后，里面的数据还存在。
如果另一个容器也有持久化数据的需求，且想使用上述容器共享的/tmp/data目录，那么可以运行下面的命令启动一个容器。


    # docker run -it -rm -volumes-from container1 image2   /bin/bash   
    //container1为第一个容器的ID，image2为第二个容器运行image的名字。


运行通过该Dockerfile构建镜像的容器，/tmp/data目录中的数据在容器关闭后，里面的数据还存在。
如果另一个容器也有持久化数据的需求，且想使用上述容器共享的/tmp/data目录，那么可以运行下面的命令启动一个容器。

    # docker run -it -rm -volumes-from container1 image2   /bin/bash   
    
    //container1为第一个容器的ID，image2为第二个容器运行image的名字。

（11）WORKDIR（切换目录）

可以多次切换(相当于cd命令)，对RUN，CMD，ENTRYPOINT生效。

格式:

    WORKDIR /path/to/workdir
    
    // 一定要使用绝对路径，如果使用相对路径，那么路径会传递下去

    WORKDIR   a
    WORKDIR   b
    WORKDIR   c   
    
    // 相当于#cd /a/b/c  

例子：在Dockerfile中写入以下

    WORKDIR /path/to/p1   
    WORKDIR p2   
    RUN  vim a.txt
    
    // 在/p1/p2下，执行# vim a.txt

（12）ONBUILD（镜像触发器）

当一个镜像被其他镜像作为基础镜像执行时，新的镜像会在构建过程中插入ONBUILD触发器中的指令

格式：

    ONBUILD <Dockerfile关键字>
    
    // ONBUILD指定的命令在由Dockerfile构建原镜像时并不执行，而是在由原镜像被其他镜像作为基础镜像构建时执行 

例子：

    # vim Dockerfile
    ...
    ONBUILD COPY index.html /usr/share/nginx/html/   
    ...
    
    # docker build -t="locutus1/nginx4"  .  //构建原镜像locutus1/nginx4
    
    # docker run -p 81 --name nginx4 locutus1/nginx4
    
    # curl http://127.0.0.1:host_port 
    // 依旧是原来nginx的内容，说明在构建时虽然显示，但没有执行 ONBUILD  COPY  index.html   /usr/share/nginx/html/
    
    
    # vim Dockerfile  
    
    FROM locutus1/nginx4  
    // 原镜像locutus1/nginx4作为locutus1/nginx5的基础镜像
    ... 
    
    # docker build -t="locutus1/nginx5" . 
    //在step0后，执行了触发器的指令COPY 
    
    # docker run -p 82 --name nginx4 locutus1/nginx5
    
    # curl http://127.0.0.1:host_port   
    //是COPY指令所复制的index.html的内容了


### 特别说明一下！

我在centos作为基础镜像制作nginx服务时,编写CMD执行路径脚本时，发现制作的nginx服务虽然启动执行命令，但是命令结束后也就会自动退出界面。


我们需要了解一种容器知识点，就是容器中的服务需要跑在前台才会在容器启动后保证容器正常使用，否则容器在执行完命令后就会自动退出。返回码为0.这是正常现象，所以，一般启动后台脚本的话我们使用如下方式。


《第一本Docker书》里面，讲到Docker容器启动web服务时，都指定了前台运行的参数，例如apache:


    ENTRYPOINT [ "/usr/sbin/apache2" ]
    CMD ["-D", "FOREGROUND"]


又例如nginx：

    ENTRYPOINT [ "/usr/sbin/nginx", "-g", "daemon off;" ]

为什么要这么做呢？因为Docker容器仅在它的1号进程（PID为1）运行时，会保持运行。如果1号进程退出了，Docker容器也就退出了。 