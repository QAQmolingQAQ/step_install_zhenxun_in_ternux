# step_install_zhenxun_in_ternux
本文是通过napcat脚本安装后的步骤,但有部分修改，系统换成ubuntu。
以下是原版，
NapCat.Termux - 安卓 Termux 部署 recommend
```bash
curl -o napcat.termux.sh https://nclatest.znin.net/NapNeko/NapCat-Installer/main/script/install.termux.sh && bash napcat.termux.sh
```
仓库地址: NapCat.installer

已修改：将 debian 改为 ubuntu 
我的可以在仓库下载，也可以你手动修改。

'execute_command "proot-distro install ubuntu --override-alias napcat" "安装napcat容器"'
运行后，会安装napcat和proot.

之后安装sql,我安装的是PostgreSQL数据库。
先不进虚拟环境，在termux环境下，
```bash
pkg install git
```

```bash
pkg update && pkg upgrade -y
pkg install postgresql
pg_ctl -D $PREFIX/var/lib/postgresql -l logfile start
psql -d postgres
```

#-- 1. 创建专用数据库（比如叫 zhenxun）
```bash
CREATE DATABASE zhenxun;
```
-- 2. 创建一个专用用户（比如叫 zhenxun），并设置密码,记得替换你的密码。
```bash
CREATE USER zhenxun WITH PASSWORD '你的强密码';
```
-- 3. 将该用户设为 bot_db 数据库的所有者
CONNECT：允许用户连接到数据库。
USAGE ON SCHEMA：允许用户访问该 schema 中的对象。
SELECT, INSERT, UPDATE, DELETE：对现有表的数据操作权限。
ALTER DEFAULT PRIVILEGES：确保未来创建的新表也自动授予相同权限。
```bash
ALTER DATABASE zhenxun OWNER TO zhenxun;
\c zhenxun
GRANT CONNECT ON DATABASE zhenxun TO zhenxun;
GRANT USAGE ON SCHEMA public TO zhenxun;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO zhenxun;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO zhenxun;
```
-- 4. 退出 psql
```
\q
```
## 根据需要决定
# 1修改 postgresql.conf 监听所有地址（可不修改）
找到并编辑配置文件：

```bash
nano $PREFIX/var/lib/postgresql/postgresql.conf
```
找到 #listen_addresses = 'localhost' 这一行（通常在文件较前的位置），取消注释（删除行首的 #），并将 localhost 改为 '*'：
crtl+o,然后回车写入，ctrl+x退出。

# 2修改 pg_hba.conf 允许容器 IP 连接
编辑客户端认证文件：

```bash
nano $PREFIX/var/lib/postgresql/pg_hba.conf
```
在文件末尾添加一行（允许局域网内任意 IP 连接，仅用于测试环境）：
```
host    all             all             192.168.3.0/24            trust
```
或者更宽松（如果还有其他设备需要连接，不建议）：
```
host    all             all             0.0.0.0/0            trust
```
# 3：重启 PostgreSQL 使配置生效
```bash
pg_ctl -D $PREFIX/var/lib/postgresql restart
```
---
## 现在进入容器，
```bash
proot-distro login napcat
```
先例行更新
```bash
apt update
apt install git -y
```
然后
安装uv
# 方法一
```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/astral-sh/uv/releases/latest/download/uv-installer.sh | sh
source $HOME/.local/bin/env
```
# 方法二
```bash
pip install uv
```
```bash
apt install pipx
pipx install uv
```

# 下载真寻，方法一
```bash
cd ~
git clone https://github.com/HibiKier/zhenxun_bot.git
cd zhenxun_bot
uv venv --python 3.10
source ~/zhenxun_bot/.venv/bin/activate
```
# 下载真寻，方法二
```bash
cd ~
mkdir mkdir ~/uv_temp
cd ~/uv_temp
uv venv --python 3.10
source ~/uv_temp/.venv/bin/activate
uv pip install nb-cli
uv pip install nb-cli-plugin-zhenxun
```
选择目录，安装，cd的~替换为你的目录
```
cd ~
source ~/uv_temp/.venv/bin/activate
nb zx
```
安装，先选1，然后git和下载zip都行，依赖选择n，后面安装。然后可以选择是否删除~/uv_temp目录，删除命令是 rm -rf ~/uv_temp 。

## 安装
，
```bash
cd ~/zhenxun_bot 
deactivate
uv venv --python 3.10
source .venv/bin/activate
export UV_LINK_MODE=copy
uv sync
```
再修复后续部分依赖安装不上。

由于Playwright 在 Ubuntu 26.04 上会报错，我们需要让它“以为”自己在 Ubuntu 24.04 上。
在终端执行：
```bash
export PLAYWRIGHT_HOST_PLATFORM_OVERRIDE=ubuntu24.04-arm64
playwright install chromium
unset PLAYWRIGHT_HOST_PLATFORM_OVERRIDE
```

```bash
apt update
apt install -y build-essential python3-dev
```
遇到地区选asia回车，选时区往下翻shanghai，回车。


再次补充依赖
```bash
uv pip uninstall psutil -y
rm -rf .venv/lib/python3.10/site-packages/psutil*
rm -rf .venv/build
rm -rf ~/.cache/pip
export CFLAGS="-O2 -march=armv8-a"
export LDFLAGS="-L/lib/aarch64-linux-gnu -L/usr/lib/aarch64-linux-gnu"
export LD_LIBRARY_PATH="/lib/aarch64-linux-gnu:/usr/lib/aarch64-linux-gnu"
uv pip install --no-cache-dir --no-binary psutil psutil==5.9.8
unset CFLAGS
unset LDFLAGS
unset LD_LIBRARY_PATH
```

最后运行真寻
```bash
cd ~/zhenxun_bot
source ~/zhenxun_bot/.venv/bin/activate
uv run zx
```
