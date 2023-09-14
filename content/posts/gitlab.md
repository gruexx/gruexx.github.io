---
title: Gitlab 搭建记录
subtitle: docker安装gitlab和gitlab-runner及项目自动部署
date: 2023-09-12T17:50:14+08:00
draft: false
description: Gitlab 搭建记录
tags: [ "gitlab", "docker", "gitlab-runner", "CI/CD" ]
categories: [ "学习笔记" ]
---

# 1 gitlab

## 1.1 docker中安装gitlab

执行 `docker pull gitlab/gitlab-ce:latest` 安装gitlab

执行 `docker run -d -p 443:443 -p 1028:1028 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce`
启动gitlab

通过ip和端口（这里是1028）就可以访问gitlab前端界面了

# 2 gitlab-runner

## 2.1 docker中安装gitlab-runner

执行 `docker pull gitlab/gitlab-runner:latest` 安装gitlab-runner

执行 `docker run -d --name gitlab-runner --restart always -v /home/gitlab-runner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest`
启动gitlab-runner

## 2.2 runner注册及配置

进入**管理中心**的**Runner**板块，点击右侧**注册一个实例runner**按钮，复制**token**

{{< image src="https://blog.porrizx.cc:7103/data/blog-img/gitlab/token.jpg" caption="注册一个实例runner" >}}

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
