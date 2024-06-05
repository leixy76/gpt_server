# gpt_server

本项目依托fastchat的基础能力来提供**openai server**的能力.
1. **在此基础上完美适配了更多的模型**，**优化了fastchat兼容较差的模型**
2. 重新适配了vllm对模型适配较差，导致解码内容和hf不对齐的问题。
3. 支持了**vllm**、**LMDeploy**和**hf**的加载方式
4. 支持所有兼容sentence_transformers的语义向量模型（Embedding和Reranker）
5. Chat模板支持了**function**角色，使其完美支持了**LangGraph Agent**框架
6. **降低了模型适配的难度和项目使用的难度**(新模型的适配仅需修改低于5行代码)，从而更容易的部署自己最新的模型。

（仓库初步构建中，构建过程中没有经过完善的回归测试，可能会发生已适配的模型不可用的Bug,欢迎提出改进或者适配模型的建议意见。）

<br>

## 项目实时进展
已经实现**LMDeploy**后端，其中包括lmdeploy的Pytorch后端和TurboMind后端。

LMDeploy TurboMind 引擎拥有卓越的推理能力，在各种规模的模型上，每秒处理的请求数是 vLLM 的 1.36 ~ 1.85 倍。

## 更新信息

```plaintext
6-5   支持了 glm4-9b系列（hf和vllm）
4-27  支持了 LMDeploy 加速推理后端
4-20  支持了 llama-3
4-13  支持了 deepseek
4-4   支持了 embedding模型 acge_text_embedding (通过测试)
3-9   支持了 reranker 模型 （ bge-reranker，bce-reranker-base_v1）
3-3   支持了 internlm-1.0 ,internlm-2.0
3-2   支持了 qwen-2 0.5B, 1.8B, 4B, 7B, 14B, and 72B
2-4   支持了 vllm 实现
1-6   支持了 Yi-34B
12-31 支持了 qwen-7b, qwen-14b
12-30 支持了 all-embedding(理论上支持所有的词嵌入模型)
12-24 支持了 chatglm3-6b 
```

## 支持的模型以及推理后端

**推理速度：** LMDeploy TurboMind > vllm > LMDeploy PyTorch > HF

| Models / BackEnd                 | HF | vllm |LMDeploy TurboMind|LMDeploy PyTorch|
| :--: | :--: | :--: |:--:|:--:|
| chatglm4-9b             | √ | √   |×  |×   |
| chatglm3-6b             | √ | √   |√   |√   |
| Qwen (7B, 14B, etc.)) | √ | √   |√   |√   |
| Qwen-2 (0.5B--72B) | √   |   √   |√   |√   |
| Yi-34B                 | √ | √   |√   |√   |
| Internlm-1.0                 | √ | √   |√   |√   |
| Internlm-2.0                 | √ | √   |√   |√   |
| Deepseek                 | √ | √   |√   |√   |
| Llama-3                 | √ | √   |√   |√   |

-----

<br>

**原则上支持所有的Embedding/Rerank 模型**
<br>
以下模型经过测试：
| Embedding/Rerank  | HF |
|----|---|
|bge-reranker|√|
|bce-reranker|√|
|bge-embedding|√|
|bce-embedding|√|
|piccolo-base-zh-embedding|√|
|acge_text_embedding|√| 


目前 **acge_text_embedding** MTEB榜单排行第一, 超越付费的闭源服务，阿里云OpenSearch服务(第二)和baichuan-text-embedding(第五)

## 启用方式

### Python启动
#### 1. 配置python环境
```bash
# 1. 创建conda 环境
conda create -n gpt_server python=3.10

# 2. 激活conda 环境
conda activate gpt_server

# 3. 安装依赖
pip install -r requirements.txt
```

#### 2. 修改启动配置文件

[config.yaml](https://github.com/shell-nlp/gpt_server/blob/main/gpt_server/script/config.yaml "配置文件")

```bash
cd gpt_server/script
vim config.yaml
```

```yaml
serve_args:
  host: 0.0.0.0
  port: 8082

models:
  - chatglm3:  #自定义的模型名称
      alias: null # 别名     例如  gpt4,gpt3
      enable: false  # false true
      model_name_or_path: /home/dev/model/chatglm3-6b/
      model_type: chatglm  # qwen  chatglm3 yi internlm
      work_mode: vllm  # vllm hf lmdeploy-turbomind  lmdeploy-pytorch
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        # - 1
        - 0
      # - gpus:
      #   - 0
  - chatglm4:  #自定义的模型名称
      alias: null # 别名     例如  gpt4,gpt3
      enable: true  # false true
      model_name_or_path: /home/dev/model/THUDM/glm-4-9b-chat/
      model_type: chatglm  # qwen  chatglm3 yi internlm
      work_mode: vllm  # vllm hf lmdeploy-turbomind  lmdeploy-pytorch
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        # - 1
        - 0
  - qwen:  #自定义的模型名称
      alias: gpt-4,gpt-3.5-turbo,gpt-3.5-turbo-16k # 别名     例如  gpt4,gpt3
      enable: true  # false true
      model_name_or_path: /home/dev/model/qwen/Qwen1___5-14B-Chat/ 
      model_type: qwen  # qwen  chatglm3 yi internlm
      work_mode: vllm  # vllm hf lmdeploy-turbomind  lmdeploy-pytorch
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 1
      # - gpus:
      #   - 3

  - mixtral:  #自定义的模型名称
      alias: null # 别名     例如  gpt4,gpt3
      enable: false  # false true
      model_name_or_path: /home/dev/model/NousResearch/Nous-Hermes-2-Mixtral-8x7B-SFT/
      model_type: qwen  # qwen  chatglm3 yi internlm
      work_mode: vllm  # vllm hf lmdeploy-turbomind  lmdeploy-pytorch
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 3
        - 0
    

  - llama3:  #自定义的模型名称
      alias: null # 别名     例如  gpt4,gpt3
      enable: false  # false true
      model_name_or_path: /home/dev/model/unsloth/unsloth/llama-3-8b-Instruct/
      model_type: llama  # qwen  chatglm3 yi internlm
      work_mode: hf  # vllm hf lmdeploy-turbomind  lmdeploy-pytorch
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 0

  - yi:  #自定义的模型名称
      alias: null # 别名     例如  gpt4,gpt3
      enable: false  # false true
      model_name_or_path: /home/dev/model/01ai/Yi-34B-Chat/
      model_type: yi  # qwen  chatglm3 yi internlm
      work_mode: hf  # vllm hf lmdeploy-turbomind  lmdeploy-pytorch
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 2
        # - 0

  - internlm:  #自定义的模型名称
      alias: null # 别名     例如  gpt4,gpt3
      enable: false  # false true
      model_name_or_path: /home/dev/model/Shanghai_AI_Laboratory/internlm2-chat-7b/
      model_type: internlm  # qwen  chatglm3 yi internlm
      work_mode: hf  # vllm hf lmdeploy-turbomind  lmdeploy-pytorch
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 0
    
  # Embedding 模型
  - piccolo-base-zh:
      alias: null # 别名   
      enable: true  # false true
      model_name_or_path: /home/dev/model/assets/embeddings/sensenova/piccolo-base-zh/
      model_type: embedding
      work_mode: hf
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 2

  - bce-embedding-base_v1:
      alias: text-embedding-ada-002 # 别名   
      enable: true  # false true
      model_name_or_path: /home/dev/model/maidalun1020/bce-embedding-base_v1/
      model_type: embedding
      work_mode: hf
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 2

  - bce-reranker-base_v1:
      alias: null # 别名   
      enable: false  # false true
      model_name_or_path: /home/dev/model/maidalun1020/bce-reranker-base_v1/
      model_type: embedding
      work_mode: hf
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 0
  - bge-reranker-base:
      alias: null # 别名   
      enable: true  # false true
      model_name_or_path: /home/dev/model/Xorbits/bge-reranker-base/
      model_type: embedding
      work_mode: hf
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 2
  - bge-base-zh:
      alias: null # 别名   
      enable: true  # false true
      model_name_or_path: /home/dev/model/Xorbits/bge-base-zh-v1___5/
      model_type: embedding
      work_mode: hf
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 2
  
  - acge_text_embedding:
      alias: null # 别名   
      enable: true  # false true
      model_name_or_path: /home/dev/model/aspire/acge_text_embedding
      model_type: embedding
      work_mode: hf
      device: gpu  # gpu / cpu
      workers:
      - gpus:
        - 2

```

#### 3. 运行命令

[start.sh](https://github.com/shell-nlp/gpt_server/blob/main/gpt_server/script/start.sh "服务主文件")

```bash
sh start.sh
```


#### 4. 使用 openai 库 进行调用

**见 gpt_server/tests 目录 样例测试代码**
<br>
https://github.com/shell-nlp/gpt_server/tree/main/tests


### Docker安装
待填充


## 致谢

    FastChat : https://github.com/lm-sys/FastChat

      vLLM   : https://github.com/vllm-project/vllm

    LMDeploy ： https://github.com/InternLM/lmdeploy

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=shell-nlp/gpt_server&type=Date)](https://star-history.com/#shell-nlp/gpt_server&Date)
