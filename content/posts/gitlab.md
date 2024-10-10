---
title: Gitlab 搭建记录
subtitle: docker安装gitlab和gitlab-runner及项目自动部署
date: 2023-09-12T17:50:14+08:00
draft: false
description: Gitlab 搭建记录
tags: [ "gitlab", "docker", "gitlab-runner", "CI/CD" ]
categories: [ "笔记" ]
featuredImage: "https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/gitlab/moon.jpg"
featuredImagePreview: "https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/gitlab/moon.jpg"
---

# 1 gitlab

## 1.1 docker中安装gitlab

```shell
# 安装gitlab
docker pull gitlab/gitlab-ce:latest

# 启动gitlab
docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
```

通过ip和端口（这里是1028）就可以访问gitlab前端界面了

# 2 gitlab-runner

## 2.1 docker中安装gitlab-runner

```shell
# 安装gitlab-runner
docker pull gitlab/gitlab-runner:latest

# 启动gitlab-runner
docker run -d --name gitlab-runner --restart always -v /home/gitlab-runner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest
```

## 2.2 runner注册及配置

进入**管理中心**的**Runner**板块，点击右侧**注册一个实例runner**按钮，复制**token**

{{< figure src="https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/gitlab/token.jpg" caption="注册一个实例runner" >}}

执行 `docker exec -it b446f8c800d4 /bin/bash` 进入 `gitlab-runner`的容器内部

修改runner的配置文件 `/etc/gitlab-runner/config.toml`

`url`和`token`改成自己的，这是给前端Vue项目的runner，所以`image`用的`node:14.19.1`，不同类型项目使用不同的image

```toml
concurrent = 1

check_interval = 0

[session_server]
session_timeout = 1800

[[runners]]
name = "ds_front_runner"
url = "http://xxx.xxx.xxx.xxx:1028/"
token = "z_pQBBShu5e6Bx-mxaj4"
executor = "docker"
[runners.custom_build_dir]
[runners.cache]
[runners.cache.s3]
[runners.cache.gcs]
[runners.cache.azure]
[runners.docker]
tls_verify = false
image = "node:14.19.1"
privileged = false
disable_entrypoint_overwrite = false
oom_kill_disable = false
disable_cache = false
volumes = ["/cache", "/node_modules"]
pull_policy = ["if-not-present"]
shm_size = 0
```

修改完成执行`exit`退出容器并重启gitlab-runner

# 3 项目自动部署配置

在项目根目录下创建文件 `.gitlab-ci.yml`

下面是一个前端自动部署的例子

```yaml
workflow:
  rules:
    - if: $CI_COMMIT_MESSAGE =~ /docs/
      when: never
    - if: '$CI_COMMIT_BRANCH != "dev"'
      when: never
    - when: always

variables:
  DOCKER_DRIVER: overlay2
  MAVEN_CLI_OPTS: "-s .m2/settings.xml --batch-mode"
  MAVEN_OPTS: "-Dmaven.repo.local=.m2/repository -Dmaven.test.skip=true"
  server_ip: xxx.xxx.xxx.xxx
  server_port: 100
  dist_path: ruoyi-ui/dist
  upload_path: /home/ubuntu/three-can-do
  ssh_user: root
  ssh_password: *

stages:
  - build
  - upload
  - deploy

before_script:
  - whoami
  - ls -l

# 前端打包
build-dist:
  tags:
    - front
  stage: build
  image: node:14.19.1
  script:
    - cd ruoyi-ui
    - node -v
    - npm config set registry https://registry.npm.taobao.org
    - npm install
    - npm run build:prod
    - cd ..
  artifacts:
    paths:
      - $dist_path/
  cache:
    paths:
      - node_modules/


# 上传生成的 dist目录
upload-dist:
  tags:
    - front
  stage: upload
  image: ictu/sshpass
  script:
    - sshpass -p $ssh_password scp -r -P $server_port -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no $dist_path/ $ssh_user@$server_ip:$upload_path
  environment:
    name: tcd


# 重启nginx
deploy-dist:
  tags:
    - front
  stage: deploy
  image: ictu/sshpass
  script:
    - sshpass -p $ssh_password ssh -p $server_port -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no $ssh_user@$server_ip "systemctl restart nginx"
  environment:
    name: tcd

```

在项目仓库**CI/CD**界面查看流水线运行情况

# 4 gitlab升级

## 4.1 获取upgrade-path
在**仪表盘**能看到我的gitlab版本是`14.6.1`，由于gitlab不能跨大版本升级，要升级到最新版本`16.5.1`要先构建**upgrade-path**

{{< figure src="https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/gitlab/version.jpg" caption="gitlab版本" >}}

使用gitlab提供的生成**upgrade-path**的工具

https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/

输入自己当前版本以及要升级到的版本就能生成**upgrade-path**

{{< figure src="https://blog.gruex.info:9004/data/blog-img/minio-blog-pic/gitlab/upgrade-path.jpg" caption="upgrade-path" >}}

## 4.2 备份

```shell
# 备份
docker exec -ti gitlab gitlab-rake gitlab:backup:create
```
备份过程中可能会有以下警告
```text
Warning: Your gitlab.rb and gitlab-secrets.json files contain sensitive data 
and are not included in this backup. You will need these files to restore a backup.
Please back them up manually.
```
以防万一备份下 `gitlab.rb` 和 `gitlab-secrets.json`

```shell
cp gitlab.rb /home/gitlab-backup/
cp gitlab-secrets.json /home/gitlab-backup/
```

```shell
# 查看备份情况
ll /home/gitlab/data/backups/
```
备份完成会有一个tar包 `1699498444_2023_11_09_14.6.1_gitlab_backup.tar`

## 4.3 开始升级

根据**upgrade-path**顺序执行

```shell
# 14.6.1 => 14.9.5
docker stop gitlab && docker rm gitlab
docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:14.9.5-ce.0

# 14.9.5 => 14.10.5
docker stop gitlab && docker rm gitlab
docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:14.10.5-ce.0

# 14.10.5 => 15.0.5
docker stop gitlab && docker rm gitlab
docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:15.0.5-ce.0

# 15.0.5 => 15.4.6
docker stop gitlab && docker rm gitlab
docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:15.4.6-ce.0

# 15.4.6 => 15.11.13
docker stop gitlab && docker rm gitlab
docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:15.11.13-ce.0

# 15.11.13 => 16.1.5
docker stop gitlab && docker rm gitlab
docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:16.1.5-ce.0

# 16.1.5 => 16.3.6
docker stop gitlab && docker rm gitlab
docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:16.3.6-ce.0

# 16.3.6 => 16.5.1
docker stop gitlab && docker rm gitlab
docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:16.5.1-ce.0
```

自动下载太慢可以先手动拉取镜像

``` 
# 直接拉取
docker pull gitlab/gitlab-ce:17.3.5-ce.0

# dockerhub.icu加速
docker pull dockerhub.icu/gitlab/gitlab-ce:17.3.5-ce.0
docker image tag dockerhub.icu/gitlab/gitlab-ce:17.3.5-ce.0 gitlab/gitlab-ce:17.3.5-ce.0
docker rmi dockerhub.icu/gitlab/gitlab-ce:17.3.5-ce.0
```
