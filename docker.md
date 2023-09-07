# 镜像设置

setting中修改docker engine中的json

# 命令

```java
docker build -t onebookoneqrcodeclient .  // build
docker run -d -p 8081:80 --name my-container your-image  // 启动
docker logs my-container  // log查看
docker exec -it my-container bash  // 进入container的linux环境
```
# vue3+nginx使用docker

其实就是将项目搬到linux上运行

首先我们需要明确，我们需要什么东西：dist文件夹、nginx.conf、cert文件夹（里面是https的pem/key文件或者是crt/key文件）

1. 使用打包工具生成dist，后续docker containers使用的是dist里面的代码，不是源代码
2. 创建一个Dockerfile文件（注意：Dockerfile文件没有后缀名），这个文件放到哪里其实没有明确的规定，放什么地方都可以，但是在不同地方里面的配置的路径要做出相应的调整。通常这个文件放在vue项目的更目录处（就是dist同级目录）
```javascript
// Dockerfile中的配置，包括但不限于
FROM nginx:latest  // 说明我们这个images的模板是nginx
// 由于Dockerfile在dist同级目录，所有这里就是将dist中的文件复制到/usr/share/nginx/html
// 到这里我们要知道docker好像是在linux上的，所以这个目录也像是linux的目录，等到使用docker命令时会感受明显
COPY ./dist /usr/share/nginx/html
COPY ./nginx.conf /etc/nginx/nginx.conf
注意：通常我们本地nginx的目录结构是这样
所以我们的nginx.conf关于SSL的路径是
// ssl_certificate      ../cret/9435692_zzh.m-fuwu.cn.pem;
// ssl_certificate_key  ../cret/9435692_zzh.m-fuwu.cn.key;
│  nginx.exe
│
├─conf
│      nginx.conf
│
├─contrib
│      。。。
│
├─cret
│      9435692_zzh.m-fuwu.cn.key
│      9435692_zzh.m-fuwu.cn.pem
│
├─docs
│      。。。
│
├─html
│      。。。
│
├─logs
│      。。。
│
└─temp
    ├─。。。
但是以下代码copy到containers后目录结构就不同了变成了
│  nginx.conf
│
├─cret
    ├─9435692_zzh.m-fuwu.cn.key
    ├─9435692_zzh.m-fuwu.cn.pem
要么你改变nginx.conf的配置为
// ssl_certificate      ./cret/9435692_zzh.m-fuwu.cn.pem;
// ssl_certificate_key  ./cret/9435692_zzh.m-fuwu.cn.key;
要么你改变这里几个copy代码
COPY ./cret/9435692_zzh.m-fuwu.cn.key /etc/nginx/cret/9435692_zzh.m-fuwu.cn.key
COPY ./cret/9435692_zzh.m-fuwu.cn.pem /etc/nginx/cret/9435692_zzh.m-fuwu.cn.pem
// build完成后输出的内容
RUN echo 'echo init ok!!'
```
3. cd到Dockerfile目录，打开命令行执行
```javascript
// onebookoneqrcodeclient是给镜像取的名字可以更改、
// 注意：最后的.千万不要漏了，它表示在当前目录中执行
docker build -t onebookoneqrcodeclient .
```
4. 等待build完成后在docker desktop的images中就可以看到onebookoneqrcodeclient这个名字的image，让后运行容器
```javascript
docker run --name 取个容器名称 -d -p 9020:80 上一步取的镜像名称
```
5. 注意nginx.conf中配置
```java
location / {
    root   /usr/share/nginx/html;
    add_header Access-Control-Allow-Origin *;
    index  index.html index.htm;
}
// 这里的root的值要正确因为，我们存放dist文件在linux环境的默认路径，如果使用windows下的location /的默认配置root html就会错，应该改成上文的样子。或者dockerfile文件中将dist文件copy到别的地方去
```
# window下的命令

1. 查看container或者image的目录结构
当容器运行时

```xml
docker exec -it container-name bash   进入指定名字的容器
ls /etc/nginx/../cret                 让后通过linux命令操作，这里是查看对应文件目录
```
当容器没有运行，只有images时
```xml
docker run -it --rm image-name sh     进入指定image，这会启动一个临时容器（甚至你能在docker desktop的containers面板看到它），在退出时关闭它
ls /etc/nginx/../cret                 进行某些linux命令
```
2. docker port 容器id查看容器暴露了那些端口
