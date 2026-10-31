

📱 DS-Agent: Mobile Web Reverse Engineering Framework (v2.0.0-Ultimate)

https://img.shields.io/badge/license-MIT-blue.svg
https://img.shields.io/badge/python-3.10%2B-blue
https://img.shields.io/badge/platform-Android%20%2F%20Termux-green

DS-Agent 是一个运行在 Android/Termux 上的纯协议 Web 逆向分析 Agent 框架。它通过 ADB 打通 Chrome DevTools Protocol (CDP)，让大语言模型（LLM）直接接管手机浏览器。

无需截图、无需模拟点击、无视觉依赖。仅通过原生 JavaScript 注入、Network 拦截和 Debugger 断点调试，即可让 AI 帮你完成复杂的网页逆向、加密算法提取和自动化渗透测试。

---

🌟 核心特性

· 纯协议驱动：100% 依赖 CDP 协议，后台静默运行，不占用手机屏幕，不依赖 Playwright 等笨重框架。
· 免 Root 运行：基于 Android 原生“无线调试”与 adb forward 实现底层通信。
· 全量 F12 能力映射：完美支持 DOM 读取、Network 抓包、Debugger 断点、CSS 修改、设备模拟（Emulation）。
· 无视图 Hook 注入：支持 Page.addScriptToEvaluateOnNewDocument，在页面加载前精准拦截并篡改前端加密逻辑。
· 智能自愈与持久化：自动重连断开的 CDP，AI 对话历史、抓包日志与逆向成果自动落盘（JSON/Markdown）。
· 自定义 XML 标签执行：抛弃容易出错的反引号，使用严格的 shell、javascript、cdp 标签进行指令分发，容错率极高。

---

🚀 快速开始

1. 环境准备 (Termux 与 浏览器)

Termux 官方 GitHub 仓库 (推荐直接下载 APK 安装):
https://github.com/termux/termux-app
(注：请勿使用 Google Play 商店版本，该版本已停止更新且存在兼容性问题)

Termux 软件源镜像加速 (国内用户推荐):
termux-change-repo

安装核心依赖:
pkg update && pkg upgrade -y
pkg install python android-tools -y

安装 Python 依赖库:
pip install httpx>=0.24.0 websockets>=11.0
(或者直接使用项目根目录的 requirements.txt 执行 pip install -r requirements.txt)

所需浏览器:
推荐使用 Kiwi Browser 或 Chrome。如果遇到版本限制或需要特定的内核行为，可以参考以下旧版内核浏览器：

浏览器 适用 Android 版本 Chromium 内核版本 仓库地址
Cromite Android 7.0 v119 https://github.com/uazo/cromite/releases/latest
Cromite Android 8.0+ v138 同上
Chromium (macchrome 编译) Android 7.0 v118 https://github.com/macchrome/droidchrome/releases/latest
Chromium (macchrome 编译) Android 6.0 v104 同上

---

2. ⚠️ 防杀后台保活指南 (极其重要)

在手机上跑 Agent，最怕的就是系统为了省电，把 Termux 和浏览器给偷偷杀掉了。请务必完成以下设置：

① 开启 Termux 唤醒锁
在 Termux 终端里输入并执行以下命令（这会在通知栏挂一个锁，防止 CPU 休眠）：
termux-wake-lock

② 系统电池优化设置 (必须设置为“无限制”)

· 华为/荣耀：设置 -> 电池 -> 应用耗电管理 -> 找到 Termux 和 浏览器 -> 允许后台活动。
· 小米/Redmi：设置 -> 应用设置 -> 应用管理 -> 找到 Termux 和 浏览器 -> 省电策略 -> 选择“无限制”。
· OPPO/vivo/一加：设置 -> 电池 -> 应用耗电管理 -> 允许完全后台行为。

③ 锁定后台任务卡片
打开手机的多任务（最近任务）界面，长按 Termux 和浏览器的卡片，点击“小锁”图标将其锁定。防止在你清理后台时把它们误杀。

④ 开发者选项设置
进入手机的“开发者选项”，找到并开启 “充电时不锁定屏幕”（如果你一直插着电的话）。同时关闭 “不保留活动”（如果该选项被打开，切出应用页面就会被销毁）。

---

3. 建立底层通道 (ADB)

确保手机和电脑（或本机）连接同一个 Wi-Fi。在手机“开发者选项”中开启“无线调试”。

在 Termux 中执行：
adb pair 127.0.0.1:配对端口
adb connect 127.0.0.1:连接端口
adb forward tcp:9222 localabstract:chrome_devtools_remote

(建议使用手机分屏功能，让浏览器占上半屏，Termux 占下半屏，确保浏览器处于活跃状态。)

---

4. 配置大模型

在根目录新建 api.json，填入你的模型信息（推荐使用 DeepSeek V3，性价比极高）：
{
"api_key": "sk-你的密钥",
"base_url": "https://api.deepseek.com/v1",
"model": "deepseek-chat",
"temperature": 0.2
}

---

5. 启动 Agent

赋予脚本执行权限并启动：
chmod +x again.sh
./again.sh

---

🛠️ 架构目录

---

❓ 常见问题排查 (FAQ)

· 报错 adb: no devices/emulators found：说明无线调试断开了，请重新执行 adb connect 连接端口。
· CDP 连接失败 (HTTP 500)：通常是因为你关闭了浏览器的标签页，导致 WebSocket 地址失效。重启 again.sh 即可重新获取新页面。
· API 返回 429 状态码：说明你的大模型免费额度用尽或频率受限，请充值或更换 API。
· AI 拒绝回答或报安全警告：说明触发了模型的安全对齐机制，尝试修改 system.md 或改用“兼容性测试”等话术诱导。

---

🏆 实战战绩 (Battle-Tested)

本项目已在真实的复杂前端场景中完成了降维打击：

· 破解 Bilibili WBI 签名：在 4MB 的混淆 JS 中精准定位加密函数，提取 64 位乱序索引表，还原 MD5 签名算法并输出纯 Python 脚本，成功调用接口获取视频数据。
· 绕过前端校验拦截：AI 自动分析 DOM 结构，发现未登录状态下弹幕输入框被隐藏，直接推导出了标准的 POST /x/v2/dm/post 接口参数规格（包含 CSRF token 生成逻辑）。
· 自动化业务跳转：目标接口不在当前页面时，AI 能自主读取浏览器的 localStorage 历史记录，通过注入 element.click() 自动跳转到包含目标数据的视频详情页。

---

🧠 为什么选 DS-Agent 而不是普通 ChatGPT？

特性 Web 端 ChatGPT DS-Agent (本框架)
执行能力 只能“动嘴”，给你代码让你自己去跑 直接发指令，代码瞬间在浏览器执行
环境感知 瞎猜，完全不知道网页长什么样 读取真实 DOM、网络请求和报错，按实际环境决策
自我纠错 报错了只会说“抱歉” 捕获 exceptionDetails，自动修改 Hook 代码重试
数据截断保护 无，Token 容易爆表 内置超过 3000 字符自动截断，强迫 AI 写精准查询

---

🛡️ 免责声明

本项目仅供网络安全学习、前端兼容性测试与授权渗透测试使用。请勿将本工具用于任何未授权的非法用途。使用本工具进行逆向工程所造成的任何法律后果，由使用者自行承担。

---

📬 反馈与支持

如果你在折腾的过程中遇到了 Bug，或者有更好的优化建议，欢迎发邮件交流。
邮箱：3960545079@qq.com

(PS：作者是一名热爱折腾的学生，如果你觉得这个项目帮到了你，欢迎请作者喝杯奶茶支持一下 ~)
