# OpenCLI Browser 发布 GitHub 包：操作备忘

适用：本机没有 `gh`、没有可用 GitHub SSH，但 Chrome 已安装 OpenCLI Browser 扩展且已登录 GitHub 时，创建公开仓库并上传一个 Skill/项目压缩包。

> 这是浏览器 UI 自动化工作流。外部发布会创建公开内容，开始前必须由用户明确确认：目标所有者、仓库名、公开/私有、上传范围。

## 0. 发布前整理

1. 在工作区创建独立发布目录，包含 README、Skill/项目文件、审计文件。
2. 使用 `.gitignore` 排除：原始出版物 PDF、OCR 全文、私密输入、缓存、`__pycache__`、临时预览。
3. 打 ZIP；对公开发布而言，可上传 `README.md + package.zip`，避免在网页端逐个上传大量文件。
4. README 必须写明：安装方式、内容范围、试测/审校边界、版权排除项。

## 1. 找到已安装的 OpenCLI

`opencli` 可能不在 PATH。先检查 daemon 与 profile：

```bash
# 例：某个已有 Node 项目中安装的 OpenCLI
OPENCLI="/path/to/node_modules/@jackwener/opencli/dist/src/main.js"
node "$OPENCLI" profile list
node "$OPENCLI" daemon status
node "$OPENCLI" github whoami -f json
```

若 daemon 返回 `extensionConnected: true`，可直接使用浏览器桥接。daemon 的本地端口需要 `X-OpenCLI: 1` 请求头；日常操作优先用 CLI，不直接调用内部 HTTP 接口。

## 2. 身份与所有者双重核验

```bash
node "$OPENCLI" github whoami -f json
```

- `whoami` 是当前登录身份；
- 创建页面中的 **Owner** 是实际仓库所有者；二者可能不同（例如有多账号或有管理权限）。
- 若二者不同，必须在网页创建页确认 Owner 显示为用户指定的目标，不能仅凭 `whoami` 就发布。

## 3. 用浏览器创建仓库

```bash
SESSION=ghpub
node "$OPENCLI" browser "$SESSION" open "https://github.com/new?owner=<OWNER>"
# 返回的 page ID 保存为 TAB
node "$OPENCLI" browser "$SESSION" state --tab "$TAB"
```

从 `state` 返回的数字引用中找到：仓库名输入框、描述框、公开性、README 开关、创建按钮。

```bash
node "$OPENCLI" browser "$SESSION" fill <repo-name-ref> "<REPOSITORY>" --tab "$TAB"
node "$OPENCLI" browser "$SESSION" fill <description-ref> "<DESCRIPTION>" --tab "$TAB"
node "$OPENCLI" browser "$SESSION" click <create-ref> --tab "$TAB"
node "$OPENCLI" browser "$SESSION" wait time 4 --tab "$TAB"
```

创建前确认：Owner、仓库名、Public/Private、是否关闭 GitHub 自动 README（已有本地 README 时关闭）。

## 4. 上传包并提交

新仓库空白页可点击 `uploading an existing file`，或直接打开：

```bash
node "$OPENCLI" browser "$SESSION" open "https://github.com/<OWNER>/<REPOSITORY>/upload" --tab "$TAB"
node "$OPENCLI" browser "$SESSION" state --tab "$TAB"
node "$OPENCLI" browser "$SESSION" upload <file-input-ref> \
  "/absolute/path/README.md" \
  "/absolute/path/package.zip" --tab "$TAB"
```

等待上传完成，再填写提交说明并提交：

```bash
node "$OPENCLI" browser "$SESSION" fill <summary-ref> "Add packaged Skill suite" --tab "$TAB"
node "$OPENCLI" browser "$SESSION" fill <description-ref> "Describe included assets and exclusions." --tab "$TAB"
node "$OPENCLI" browser "$SESSION" click <commit-ref> --tab "$TAB"
```

## 5. 发布后验收

在仓库首页核对：

1. URL、Owner 与可见性正确；
2. `README.md` 与 ZIP 均存在；
3. README 内容能说明安装、范围、边界；
4. ZIP 中不含 OCR 原书、隐私材料、缓存；
5. 提交信息与用户意图一致。

可用页面求值做最小验收：

```bash
node "$OPENCLI" browser "$SESSION" eval \
  "JSON.stringify({url:location.href,files:[...document.querySelectorAll('a')].map(a=>a.textContent.trim()).filter(x=>x==='README.md'||x.endsWith('.zip'))})" \
  --tab "$TAB"
```

## 失败处理

- **`opencli: command not found`**：从已有 Node 项目定位 `@jackwener/opencli/dist/src/main.js`，用 `node` 执行；不要假设全局安装。
- **AppleScript/Chrome 不可控**：优先转用 OpenCLI Browser bridge，不必开启 Chrome 的 Apple Events JavaScript。
- **常用 9222 无法连接**：用 `node "$OPENCLI" profile list` / `daemon status`，不要猜调试端口。
- **登录账号与目标 Owner 不同**：停下确认 Owner 下拉菜单，不向未获授权的账号创建仓库。
- **上传很多文件**：优先上传审校后的单一 ZIP + README；GitHub 网页单批上传限制通常小于 100 个文件、单文件 25 MB。
