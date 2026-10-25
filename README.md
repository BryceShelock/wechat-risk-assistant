# WeChat Risk Assistant

类微信聊天原型：对每一条中文消息做细粒度心理风险识别，再决定由模型回复，还是转给人工坐席。

用户端是 `wechat_mock.html`（会话 + 朋友圈），坐席端是 `admin.html`（队列、接管、模型配置）。检测模型是共享编码器的多任务 BERT，可选 LoRA；回复走可配置的 OpenAI 兼容接口。

## 能力

- **识别**：`MultiTaskBertForPsychology`，三个头分别预测问题类型（S1，8 类多标签）、情绪状态（S2：depression / anxiety / none）、风险（S3：suicide_ideation / self_harm / none）
- **分流**：`risk_scoring.py` 合并风险分，`policy_engine.py` 输出 `ai_reply` 或 `handoff_to_human`
- **回复**：`chat_llm.py` 按后台保存的 base URL、模型名和采样参数调用上游 LLM，支持流式
- **实时**：FastAPI + WebSocket，会话、消息和推理日志写入 SQLite

## 运行

```bash
pip install -r requirements-pip.txt
python backend_api.py
```

浏览器打开 `wechat_mock.html`。端口不是 8010 时加上 `?api=http://127.0.0.1:<端口>`。坐席端用同样方式打开 `admin.html`。

本地试识别：

```bash
python multitask_predict.py --model_dir <best_model 目录> --text "示例中文句子"
```

训练入口是 `train_multitask_lora.py`，合并权重用 `merge_peft_to_best_model.py`。标签定义在 `multitask_config.py`。

## 布局

| 路径 | 作用 |
| --- | --- |
| `multitask_model.py` / `train_multitask_lora.py` | 模型与 LoRA 训练 |
| `multitask_predict.py` | 推理 |
| `backend_api.py` / `chat_llm.py` | API 与上游 LLM |
| `wechat_mock.html` / `admin.html` | 用户端与坐席端 |
| `后端多任务模型说明文档.md` | 检测分支设计说明 |

毕业设计原型，检测方案以仓库内说明文档为准。MIT License。
