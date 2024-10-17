在使用 `sudo` 执行命令时,不会使用用户的代理配置;
修改 `sudoers` 传递代理配置
```bash
sudo visudo

添加
Defaults    env_keep += "http_proxy https_proxy"
```