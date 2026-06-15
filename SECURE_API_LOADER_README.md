# 🔒 安全API LLM加载器 (secure_LLM_api_loader)

为避免 **api_key 被保存进工作流、再通过图片(PNG)元数据泄露**，本节点从一个
**本地 JSON 文件**按「配置名称」读取 `model_name / base_url / api_key`。
工作流中**只会保存「配置名称」这个字符串**，api_key 在运行时才从本地文件读取，
永远不会进入工作流 JSON，也就不会写进图片元数据。

## 为什么原来的节点会泄露
`☁️API LLM加载器 (LLM_api_loader)` 的 `api_key` 是一个文本输入框 (widget)。
一旦你把 key 填进去，它就会被 ComfyUI 当作节点参数存进工作流 JSON；保存图片时
ComfyUI 默认把整个工作流写进 PNG 的 metadata，于是 key 随图片外泄。

## 使用方法
1. 复制 `llm_api_keys.json.example` 为 `llm_api_keys.json`。
2. 编辑 `llm_api_keys.json`，按需添加若干配置。格式：
   ```json
   {
     "我的deepseek": {
       "model_name": "deepseek-chat",
       "base_url": "https://api.deepseek.com/v1/",
       "api_key": "sk-真实key"
     }
   }
   ```
   - 最外层的键（如 `"我的deepseek"`）是**下拉菜单里显示的配置名称**，也是工作流里唯一会被保存的内容。
   - `model_name`：真正发给 API 的模型名；省略时默认用配置名称。
   - `base_url` / `api_key`：对应渠道的地址和密钥。
3. **重启 ComfyUI**（新增节点需要重启才会加载；之后修改 JSON 只需刷新页面即可更新下拉框）。
4. 在画布添加节点：`大模型派对 → 模型加载器 → 🔒安全API LLM加载器`。
5. 在 `config_name` 下拉框选择配置，把输出的 `model` 连到 `☁️API LLM通用链路 (LLM)` 节点的 `model` 输入即可。

## 选项
- `json_path`（可选）：自定义密钥 JSON 的绝对路径。留空则用本节点目录下的 `llm_api_keys.json`。
  **建议把密钥文件放在 ComfyUI 目录之外**（例如 `D:\secrets\llm_api_keys.json`），进一步降低误分享风险。路径本身不是机密。
- `is_ollama`（可选）：本地 ollama，无需 api_key。

## 安全须知
- `llm_api_keys.json` 已加入 `.gitignore`，不会被提交。
- 该文件是**明文**保存 key 的，请勿把它打包/分享给别人；分享的只应是工作流和图片。
- 即便用了本节点，分享工作流前仍建议确认图片元数据中不含任何 key。

---

## English

The stock `LLM_api_loader` node exposes `api_key` as a text widget. Once filled, the
key is saved into the workflow JSON, and ComfyUI embeds the whole workflow into the
PNG metadata of generated images — leaking the key whenever you share an image.

`secure_LLM_api_loader` (🔒 Secure API LLM Loader) instead reads
`model_name / base_url / api_key` from a local `llm_api_keys.json` file, selected by a
**profile name dropdown**. Only the profile name is stored in the workflow; the key is
resolved at runtime and never touches the workflow JSON or PNG metadata.

Usage: copy `llm_api_keys.json.example` to `llm_api_keys.json`, fill in your keys,
restart ComfyUI, add the node from `LLM party → model loader`, pick a profile, and wire
its `model` output into the `LLM` node. An optional `json_path` lets you keep the key
file outside the ComfyUI folder. `llm_api_keys.json` is gitignored.
