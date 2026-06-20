# WSL2-with-VPN

其实是Ubuntu WSL2 with VPN，不过其他发行操作版操作应该差不多。用于记录作者 Ubuntu WSL2 代理的配置，作者仅在 Ubuntu WSL + Clash 环境下测试过,本 readme 大部分内容由 AI 生成，并经过 Luka 检查。

如有问题，请以**操作+问题+显示错误的提示和图片+确认自己已根据FAQ自查**的格式提 issue。

---

## 🛠️ 前置准备 (Windows 宿主机)

在部署脚本前，请确保 Windows 端已完成以下配置：

### 1. 开启局域网访问 (Allow LAN)
打开您的代理客户端（Clash Verge / v2rayN 等），在设置中务必开启 **Allow LAN (允许局域网连接)**。
> *注：请确认您的 HTTP 代理端口和 SOCKS5 代理端口，以便在后续脚本中配置。*

### 2. 放行 Windows 防火墙
以**管理员身份**打开 Windows PowerShell，执行以下命令，永久允许 WSL 虚拟网卡访问宿主机：
```powershell
New-NetFirewallRule -DisplayName "WSL2 Allow Proxy" -Direction Inbound -InterfaceAlias "vEthernet (WSL*)" -Action Allow
```

---

## 📂 部署前准备：备份与评估

在修改 `~/.bashrc` 之前，**强烈建议先进行备份**。

根据您现有的配置情况，选择下方合适的部署方式。

### 备份当前配置
无论选择哪种部署方式，都请先执行备份：
```bash
cp ~/.bashrc ~/.bashrc.bak.$(date +%Y%m%d%H%M%S)
echo "✅ 备份完成: $(ls -t ~/.bashrc.bak.* | head -1)"
```

### 查看现有自定义配置
执行以下命令，查看您在默认配置之外**自行添加**的内容（如自定义 alias、PATH 等）：
```bash
diff /etc/skel/.bashrc ~/.bashrc
```
> - 如果**没有任何输出**：说明您的 `~/.bashrc` 文件是 Ubuntu 默认状态，可直接使用**方式一**。
> - 如果**有大量输出**：说明您已有自定义配置，请使用**方式二**。

### 恢复备份
如果在部署过程中出现任何意外，随时可以恢复备份：
```bash
# 查看所有备份文件
ls -lt ~/.bashrc.bak.*

# 恢复最近一次的备份 (将文件名替换为您实际的备份文件名)
cp ~/.bashrc.bak.20260620163000 ~/.bashrc
source ~/.bashrc
```

### 恢复出厂设置
如果文件被改得面目全非，可以从系统骨架复制一份全新的默认配置：
```bash
cp /etc/skel/.bashrc ~/.bashrc
source ~/.bashrc
```

---

## 📦 部署脚本 (二选一)

> **⚠️ 端口配置指南**：
> - **如果您使用 Clash**：通常 HTTP 和 SOCKS5 都是 `7890`（或 `7897`），保持下方默认即可。
> - **如果您使用 V2RayN**：请将 `WSL_HTTP_PORT` 改为 `10809`，`WSL_SOCKS_PORT` 改为 `10808`。

### 方式一：全新部署（适合新系统 / 纯净 `.bashrc`）

此方式会将 `~/.bashrc` 恢复为官方默认状态，然后写入代理脚本。**会清除您之前自行添加的所有 alias 和环境变量**。

```bash
# 1. 清理原有配置
cp /etc/skel/.bashrc ~/.bashrc
unset -f proxy_on proxy_off proxy_status proxy_help _auto_proxy_check update_terminal_title _init_proxy_env 2>/dev/null
unset PROMPT_COMMAND http_proxy https_proxy all_proxy no_proxy _LAST_PROXY_CHECK _PROXY_LAST_STATE 2>/dev/null
```

---

### 方式二：增量添加（适合已有自定义配置的老用户）

此方式**不会改动您原有的任何配置**，只会智能地清理旧的代理代码块（如果存在），然后将新版代理脚本**追加**到文件末尾。

```bash
# 1. 清理可能存在的旧版代理代码块 (精准删除，不影响其他内容)
sed -i '/# --- WSL2 智能自动代理配置/,$d' ~/.bashrc 2>/dev/null

# 2. 清理当前终端内存中残留的旧函数和变量
unset -f proxy_on proxy_off proxy_status proxy_help _auto_proxy_check update_terminal_title _init_proxy_env 2>/dev/null
unset PROMPT_COMMAND http_proxy https_proxy all_proxy no_proxy _LAST_PROXY_CHECK _PROXY_LAST_STATE 2>/dev/null
```

---

> 无论您选择了方式一还是方式二都请按下述内容操作。

打开 `~/.bashrc` 的**编辑界面**

```bash
cat >> ~/.bashrc << 'EOF'
```

**下载、复制** `WSL2 智能自动代理配置.bashrc` 中的内容后**粘贴**进去，**回车**

重新加载配置

```bash
source ~/.bashrc
```

---

## 🎮 使用指南

部署完成后，您的终端将获得以下命令：

| 命令 | 功能描述 |
| :--- | :--- |
| `proxy_on` | 开启代理。脚本会验证代理是否开启成功，成功则设置环境变量并变更终端标题为 🟢，否则保持直连。 |
| `proxy_off` | 关闭代理。清除环境变量，切换为物理直连，终端标题变更为 🔴。 |
| `proxy_status`| 查看当前代理状态。 |
| `proxy_help` | 呼出带有颜色高亮的帮助菜单与自动化特性说明。 |

### 自动化行为说明：

1. **触发方式**：开启 WSL 时与每次输入命令**敲击回车后**。
2. **开机行为**：WSL 启动时**不会**自动劫持网络，而是保持直连，并播报一次代理通道状态。
3. **中途断开**：如果您在 Windows 上退出了代理软件，下次在 WSL 敲击回车时，脚本会自动清除失效的代理变量，切回直连并提示您。
4. **中途恢复**：如果您在 Windows 上启动了代理软件，脚本会自动检测到通道恢复，并提示您输入 `proxy_on` 开启。

## ❓ 常见问题与排查方法 (FAQ)

**Q：为什么通过探测小米 204 接口的方式检测代理开关？**
、
A：脚本只关心 **"代理软件是否开启且能转发数据"**，将具体的流量分流规则完全交还给客户端处理。

**Q: 提示 `❌ 代理未响应`，但我明明开着软件？**

A: 请检查两点：
1. 软件是否开启了 **Allow LAN (允许局域网连接)**？
2. 在 Windows 中是否更换了网络（如从家里 Wi-Fi 换到了手机热点）？Windows 可能会将新网络识别为"公用网络"并重新启用防火墙拦截。请去 Windows 网络设置中将当前网络改为 **"专用网络 (Private)"**。

**Q: 我使用的是 V2RayN，端口怎么填？**

A: V2RayN 默认 HTTP 端口是 `10809`，SOCKS5 端口是 `10808`。请使用 `nano ~/.bashrc` 打开配置文件，找到顶部的端口配置区进行修改：
```bash
export WSL_HTTP_PORT="10809"
export WSL_SOCKS_PORT="10808"
```
保存后执行 `source ~/.bashrc` 即可。

**Q: 部署后我想修改端口或调整逻辑怎么办？**

A: 使用文本编辑器打开 `~/.bashrc`，定位到 `# --- WSL2 智能自动代理配置` 开头的代码块进行修改。修改后执行 `source ~/.bashrc` 生效。**请勿在标记区域外部添加代理相关代码**，以免造成函数重复定义。

**Q: 我部署后终端出现重复提示怎么办？**

A: 这通常是因为多次部署导致代码块重复。请执行以下命令清理所有旧代码块，然后重新从 **方式一** 或 **方式二** 开始部署：
```bash
sed -i '/# --- WSL2 智能自动代理配置/,$d' ~/.bashrc
source ~/.bashrc
```

**Q: 如何彻底卸载此脚本？**

A: 执行以下命令即可将终端配置恢复为 Ubuntu 官方出厂状态：
```bash
cp /etc/skel/.bashrc ~/.bashrc
source ~/.bashrc
```
如果您之前使用了**方式二（增量添加）** 且保留了自定义配置，请手动用 `nano ~/.bashrc` 打开文件，删除从 `# --- WSL2 智能自动代理配置` 开始到文件末尾的所有内容。
