# 破甲 · ArmorBreaker（SillyDroid / SillyTavern 扩展）

给安卓酒馆整合包 SillyDroid（以及任何 SillyTavern 1.10+ 实例）用的破甲扩展。纯前端 JS 扩展，不依赖任何 ES import，放进扩展目录即可加载。

## 功能

1. 5 套内置破甲模板：轻度角色契约 / 深度无限制叙事 / 小说文学模式 / NSFW 成人解锁 / 开发者模式，另加自定义正文。
2. 三种注入位置：聊天内（可调 depth）、系统提示、主提示前；可指定注入角色 system / user / assistant。
3. 三重生效：setExtensionPrompt 原生注入 + API 层强插（Chat Completion）+ 尾部夹心强制（Text Completion），标记去重，不重复、不污染上下文。
4. 预填充：自动在 assistant 位置塞一句前缀，让模型无法开场拒绝。
5. 拒绝自动重 roll：命中拒绝话术自动重新生成，可设次数上限。
6. 一键强化、复制注入内容、斜杠命令 /pojia。

## 安装（SillyDroid 安卓）

### 方式 A：直接解压（推荐）

1. 把整包解压得到一个文件夹，名字保持 armorbreaker。

2. 放进下面任意一个目录（两个都行，取决于你的整合包版本）：

        ~/SillyTavern/public/scripts/extensions/third-party/armorbreaker/
        ~/SillyTavern/data/default-user/extensions/armorbreaker/

   老版本整合包用第一个；1.12 以上用第二个。目录里应该直接看到 index.js、manifest.json、style.css。

3. 重启酒馆：先停掉终端里的 node 进程，重新运行启动脚本（start.sh / 一键启动）。

4. 打开 UI，点顶栏的拼图/扩展图标，在「扩展」面板里能看到「破甲 · ArmorBreaker」卡片即成功。

### 方式 B：手机文件管理器

用 MT 管理器或 ZArchiver 打开 zip，全选解压到上述目录，解压后确认层级是 .../third-party/armorbreaker/index.js，不要出现 armorbreaker/armorbreaker/index.js 这种双层嵌套。

### 方式 C：Git 安装（有仓库时）

扩展面板 -> 安装扩展 -> 填你的仓库 URL -> 重启。

## 使用

1. 扩展面板里打开右上角开关。
2. 模板选「深度 · 无限制叙事」，位置「聊天内」，深度 0，角色 system。
3. 想更狠就点「一键强化」：深度模板 + 聊天内深度 0 + 尾部夹心 + 预填充全开。
4. 写成人内容时切到「NSFW · 成人解锁」，或直接选「自定义」把自己调好的文案粘进去。
5. 仍然被拒绝时：把「拒绝自动重 roll」勾上（最大 2 次），或把深度改成 2-4 再试。

斜杠命令：

    /pojia on            开启
    /pojia off           关闭
    /pojia boost         一键强化
    /pojia deep          切到深度模板（gentle/novel/explicit/dev/custom 同理）
    /pojia depth 2       改注入深度
    /pojia status        查看状态

## 参数建议

- 深度 0：贴在用户最新一条消息前面，最狠，但可能压过角色卡。
- 深度 2-4：留出对话缓冲，角色卡更稳。
- 位置「系统提示」：适合长上下文，模型遵守度高但攻势弱一些。
- 同时开「API 层强插」+「尾部夹心」最稳；如果上下文吃紧，关掉其中一个。

## 常见问题

- 面板不出现：确认路径层级、确认重启了 node 进程、看浏览器控制台有没有 404 或语法错误。
- 注入了但没效果：先确认模板不是空的；换「深度」模板；把注入位置改成聊天内深度 0；确认后端模型本身没有独立的内容过滤（云端 API 自带过滤时，破甲只能缓解不能全解）。
- 双重注入：不会。核心块带 [[AB_CORE]] 标记，尾部带 [[AB_TAIL]] 标记，命中即跳过。
- 关掉后仍残留：切一次聊天或重新生成即可，关闭时 setExtensionPrompt 会写入空串清空。

## 文件

    manifest.json   扩展声明
    index.js        主逻辑（模板 + 注入 + 面板 + 命令）
    style.css       面板样式
    README.md       本说明
    prompts/        可选的模板文件（core.txt / anchor.txt）

---

## v1.3.1：模板改不了怎么办

现在有**三条**互不依赖的修改通道，任何一条坏掉都不影响另外两条。

### 通道一：设置抽屉里的编辑器（原来那条）

v1.3.1 修了两个会让它「改不了」的毛病：

- 之前只要面板里任何一个控件触发刷新，正在输入的光标所在的框会被回写覆盖，打着打着跳回原文。现在有焦点保护，正在打字的框不覆盖。
- ST 重绘设置抽屉时会把我们整块删掉，之后再也长不回来。现在挂了 MutationObserver，被删就自动重建。

### 通道二：左下角浮动按钮「破甲」

不用打开设置抽屉，屏幕左下角常驻一个红色小按钮，点开是全屏编辑器：上面是模板正文，下面一段是常驻锚，底下四个键——保存并生效 / 恢复默认 / 复制 / 关闭。

这是给安卓准备的：全屏输入框，不会被抽屉的滚动卡住，长按粘贴也正常。

### 通道三：直接改文件

扩展目录里有个 prompts 文件夹：

    armorbreaker/prompts/core.txt      模板正文（改这个）
    armorbreaker/prompts/anchor.txt    常驻锚

用 MT 管理器改完，回到面板勾上「启动读 prompts/core.txt」，重启 node。启动时会读这两个文件并覆盖当前模板。

文件不存在或为空时什么都不做，不会把你面板里配好的东西冲掉。

### 改源码不生效的话

改 index.js 之后必须做两件事，少一件都白改：

1. 停掉终端里的 node 进程，重新启动（只刷新网页不够，文件是启动时读的）。
2. 浏览器强制刷新：安卓浏览器一般是下拉刷新或菜单里的「重新加载」，SillyDroid 里退出再进也行。

还不行就看浏览器控制台有没有 404——多半是目录层级成了 armorbreaker/armorbreaker/index.js。

### 配置搬家

面板底部有「导出配置」和「导入配置」。导出会下载一个 json 并同时复制到剪贴板；换手机、重装整合包时导入回去，模板正文和所有开关都跟着走。

### 状态自查

面板最下面那行会显示：注入多少字 · 位置 · 深度 · 角色 · 后端类型 · 常驻锚是否启用。
如果显示「模板正文为空，当前不注入任何内容」，就是模板被清空了，去通道二粘回来。
---

## v1.3.0 变更（针对"卡里写好了、酒馆里还是收着写"）

| 项               | v1.2 的做法                | v1.3 改成                                        |
| --------------- | ----------------------- | ---------------------------------------------- |
| 注入位置            | 插在第一个 system 之后，离生成很远   | 贴到最后一条 user 消息**之前**                           |
| Claude / Gemini | 中途插 system，会 400        | 并入最后一条 user，不报错                                |
| 卡的尺度条目          | 靠世界书关键词触发，不触发就没有        | 新增**常驻锚**，一段永远注入                               |
| 拒绝检测            | 只看 mes 前 400 字          | 扫头 1500 + 尾 500，另扫 reasoning，词表 40 条，加回避式短回复判定 |
| 写到一半断气          | 没有处理                    | **软截断续写**，自动 continue，可设次数                     |
| 持久化             | 以 localStorage 为主，清缓存就丢 | 以 extension_settings 为主                        |

### 配合角色卡的推荐配法

1. 模板选「场景 · 成人叙事（短 · 最稳）」——规则写太多，模型反而会去讨论规则。
2. 位置「聊天内」，深度 **0**，角色 system。
3. **常驻锚**打开，把你卡里「94 情欲尺度」那一段整块粘进去。这样世界书触发不触发，尺度都在上下文里。
4. 「API 层强插」+「尾部夹心强制」都开。
5. 「软截断续写」开，最大续写 3。
6. 「拒绝自动重 roll」平时关着，遇到连续拒绝再开——它会把拒绝存进聊天记录，翻起来难看。
7. **预填充默认不要开**。真要用，就把文本框换成你卡的下一句开头，比如「雨还没停。」——让它从正文中间接上，而不是从"好的，我明白了"接上。

### 新斜杠命令

    /pojia go        续写（模型没拒绝，只是断在半句）
    /pojia rewrite   重写本段
    /pojia anchor on 打开常驻锚
    /pojia status    看当前后端类型和注入量

### 仍然会有上限的地方

云端 API 自带的服务端过滤（部分 Gemini / OpenAI 端点）在前端注入之外，插件只能压低触发概率。遇到这种，换本地模型或换端点比改插件有用。

## ⚠️ 免责声明 / Disclaimer


> [!IMPORTANT]
> **使用本扩展前请仔细阅读以下条款，下载、安装或使用本项目即视为你已完全理解并同意本声明的全部内容。**
> 

1. **工具性质**：本项目（ArmorBreaker）仅为 SillyTavern 的纯前端界面与提示词拼接辅助扩展，不包含、不提供任何大语言模型权重、后端推理服务或 API 密钥。
2. **遵守第三方条款**：用户在使用本工具连接任何第三方商业 API（包括但不限于 OpenAI、Anthropic、Google、DeepSeek 等）或本地模型时，必须自行查阅并严格遵守对应服务商的使用条款（Terms of Service）与内容政策。因违反第三方平台规则导致的账号受限、封禁或经济损失，均由用户本人承担。
3. **内容责任归属**：本扩展仅在用户本地浏览器环境中处理文本结构，所有输入指令及模型输出内容均由用户自主触发。开发者不对模型生成内容的准确性、合法性或适用性作任何明示或暗示的担保，因生成或传播相关内容产生的一切法律与道德责任均由使用者自行承担。
4. **合规与年龄限制**：本项目仅供达到所在国家/地区法定成年年龄的用户用于技术研究、界面交互测试及合法的私人虚构创作。严禁将本工具用于任何违反当地法律法规、侵害他人权益或危害公共安全的用途。



---

**Disclaimer (English)**


1. **Nature of Software**: This project (ArmorBreaker) is strictly a client-side frontend UI and prompt-formatting extension for SillyTavern. It does not include, host, or distribute any LLM weights, inference backends, or API keys.
2. **Third-Party Terms of Service**: When connecting to third-party APIs (such as OpenAI, Anthropic, Google, DeepSeek, etc.) or local models, users are solely responsible for complying with the respective provider's Terms of Service and Acceptable Use Policies. The author assumes no liability for account suspensions, bans, or losses resulting from policy violations.
3. **User Responsibility**: All prompt injections execute locally in the user's browser. Users bear full legal and ethical responsibility for their inputs and any content generated by the underlying AI models. This software is provided "AS IS", without warranty of any kind.
4. **Age & Legal Compliance**: This tool is intended solely for adults of legal age in their respective jurisdictions for technical experimentation and lawful private creative writing. Any illegal use is strictly prohibited.
