---
title: Ubuntu离线apt安装应用
subtitle:
date: 2024-01-02T09:39:23+08:00
draft: false
description: "Ubuntu离线apt安装应用"
tags: [ "ubuntu", "python" ]
categories: [ "学习笔记" ]
---

# 1 服务器准备

查询离线服务器Ubuntu版本，然后准备一个Ubuntu版本相同的能联网的服务器

```shell
cat /proc/version
```

# 2 执行脚本

联网的Ubuntu需要安装`apt-rdepends`和python环境

执行脚本输出所有的deb包

例如要安装nginx，则执行命令 `python 脚本.py nginx`，结果会输出到`result.sh`

```python
import subprocess
import sys
import re


def get_dependencies(package):
    try:
        # 运行 apt-rdepends 命令
        result = subprocess.check_output(["apt-rdepends", package], text=True)
    except subprocess.CalledProcessError as e:
        print("Error occurred while running apt-rdepends:", e)
        return []

    # 使用正则表达式匹配依赖
    dependencies = set()
    matches = re.findall(r"Depends: ([^\n(]+)", result)
    for match in matches:
        deps = [dep.strip() for dep in match.split(',')]
        dependencies.update(deps)

    # 添加原始包
    dependencies.add(package)

    return sorted(dependencies)


def create_script(dependencies, filename="result.sh"):
    with open(filename, "w") as file:
        for dep in dependencies:
            file.write(f"apt install -y --reinstall --download-only {dep}\n")


if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python script.py [package]")
        sys.exit(1)

    package = sys.argv[1]
    dependencies = get_dependencies(package)
    create_script(dependencies)
```

# 3 下载deb包

执行 `sh result.sh` 下载deb包，下载的包都在 `/var/cache/apt/archives` 目录下

# 4 离线安装

将deb包全部上传到离线服务器上，执行 `dpkg -i *.deb` 安装完成

完成后用 `systemctl status 应用名` 查看应用状态 
