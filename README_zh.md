# SchGen：基于语义锚定代码表示的 PCB 原理图生成

SchGen 是一个面向特定领域的大语言模型与数据集框架，用于根据自然语言描述自动生成 PCB 原理图。它提出了一种可扩展的原理图代码表示方法，并构建了一个采集自真实开源硬件设计的 PCB 原理图数据集，从而支持对用于原理图合成的 LLM 进行监督训练与评估。

## 主要特性

- 从自然语言生成 PCB 原理图
- 语义锚定的代码表示
- 可编辑的 KiCad 原理图生成
- 用于数据集构建的智能体式草图流水线
- 面向 GPT-oss 模型的 LoRA 微调流水线

![Pipeline](./assets/pipeline.png)

数据集地址：[microsoft/SchGen_dataset](https://huggingface.co/datasets/microsoft/SchGen_dataset)。

模型地址：[microsoft/SchGen](https://huggingface.co/microsoft/SchGen)。

如需引用本项目及对应论文，请使用以下 Bib 条目：

```
@misc{luo2026schgenpcbschematicgeneration,
      title={SchGen: PCB Schematic Generation with Semantic-Grounded Code Representations}, 
      author={Qinpei Luo and Ruichun Ma and Xinyu Zhang and Lili Qiu},
      year={2026},
      eprint={2605.30345},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={https://arxiv.org/abs/2605.30345}, 
}
```

## 开始使用

### 前置条件

1. **LLM 模型访问**

    **OpenRouter**：在 ``./config.py`` 中，将变量 ``openrouter_api_key`` 替换为你自己的 API key。

2. **Python 环境**
    <details>
    <summary> 说明 </summary>
    
    (1) 为项目配置 Python 虚拟环境，建议使用 Python 3.10 和 Conda。可参考[教程](https://code.visualstudio.com/docs/python/environments)。

    (2) 进入虚拟环境，并使用以下命令安装 Python 软件包：
    
    `pip install torch==2.8.0 torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128`

    `pip install -r ./requirements.txt`

    (3) 在虚拟环境中设置项目路径环境变量

    ``conda env config vars set PROJECT_PATH={YOUR_PROJECT_PATH} && conda deactivate && conda activate {YOUR_CONDA_ENV}``

    (4) 配置 GPT 微调，请参考[此博客](https://cookbook.openai.com/articles/gpt-oss/fine-tune-transfomers)。


    (5) KiCad 在不同系统上使用的 Python 解释器路径在 ``./config.py`` 中指定。相关配置基于各操作系统的常见默认设置，但可能需要根据用户实际的安装路径进行调整。


    **简要说明**
    以下是环境配置所需执行的全部命令
    ```
    # 1) 创建并激活环境
    conda create -n [YOUR_CONDA_ENV] python=3.10 -y
    conda activate [YOUR_CONDA_ENV]

    # 2) 安装依赖
    pip install --upgrade pip
    pip install -r requirements.txt

    # 3) 设置 Conda 环境的 PROJECT_PATH
    conda env config vars set PROJECT_PATH={YOUR_PROJECT_PATH} && conda deactivate && conda activate {YOUR_CONDA_ENV}

    # 4) 用于 GPT 微调

    pip install "trl>=0.20.0" "peft>=0.17.0" "transformers>=4.55.0" trackio
    pip install -U flash-attn

    # 可选：登录 Hugging Face
    from huggingface_hub import notebook_login
    notebook_login()
    ```

</details>

3. **安装 KiCad v8**
    <details>
    <summary> 说明 </summary>

    从 [Github KiCad releases](https://github.com/KiCad/kicad-source-mirror/releases) 安装 8.0.9 版本

    直接下载安装程序：  
    [Windows](https://github.com/KiCad/kicad-source-mirror/releases/download/8.0.9/kicad-8.0.9-x86_64.exe)  
    [Mac](https://github.com/KiCad/kicad-source-mirror/releases/download/8.0.9/kicad-unified-universal-8.0.9.dmg)

    在 Ubuntu 上安装 KiCad v8
    ```
        sudo add-apt-repository --yes ppa:kicad/kicad-8.0-releases
        sudo apt update
        sudo apt install --install-recommends kicad
    ```

   </details>

### 测试

要测试环境是否已正确配置：

1. 运行 `./modules/utils/llm_interface.py`，测试 LLM 模型访问  
2. 运行 `./modules/kicad_sch_interface.py`，测试基于 Python 的 KiCad 原理图编辑功能。

这些脚本都实现了用于测试的 main 函数。

### KiCad 使用方法

1. 点击项目文件打开 KiCad 项目。例如：  
   `./KiCAD_Project/example_project.kicad_pro`

2. 此时会显示 KiCad 主项目窗口。在该窗口中，点击 KiCad 原理图文件，即可在独立窗口中查看当前原理图。例如：  
   `example_project.kicad_sch`

3. SchGen 依赖 KiCad 自带的 Python 环境来进行 PCB 操作。``./config.py`` 中针对不同系统指定了默认设置，但你可能需要检查这些设置并进行必要修改。

## 分步指南

以下所有命令均在你指定的 ``PROJECT_PATH`` 下执行。

### 1. 数据集构建

![Pipeline](./assets/dataset.png)

## 数据集

训练数据集由开源 PCB 设计和参考原理图构建，主要基于以 CC BY-SA 4.0 许可证发布的 SparkFun 资源。

数据集包括：
- KiCad 原理图
- 语义代码表示
- 合成的用户请求
- 思维链推理轨迹


#### 1.1 准备符号上下文

使用以下命令，从 KiCad 的 ``.kicad_sym`` 文件中准备符号和封装信息：

```
mkdir export
python ./modules/utils/kicad_scan_lib.py
```

你应当能在 ``./export`` 文件夹下看到 ``organized_fp.json`` 和 ``organized_lib.json`` 两个文件。

#### 1.2 智能体式草图

执行以下命令，根据用户请求和图像源生成原理图草图。
```
python ./dataset_construction/agentic_sketch.py --model {MODEL_NAME} --save_path ./dataset_construction/sch_sketch --schematic_name {SCHEMATIC_NAME} --sch_request "{USER_REQUEST}" --img_ref_path {IMAGE_REFERENCE}
```

#### 1.3 人工对齐

KiCad 原理图草图位于 ``./dataset_construction/sch_sketch/{schematic_name}``，用户可以将其与参考图像进行比较，以确保二者对齐。

#### 1.4 代码转换

运行以下命令，将 KiCad 原理图转换为指定表示层级对应的 Python 代码。
使用 `dataset_construction/kicad_read_sch.py` 的轻量级 CLI。短参数可以使命令更加简洁：

```bash
python ./dataset_construction/kicad_read_sch.py \
    -m <module_name> \
    -s <path/to/schematic.kicad_sch> \
    -r <L1|L2|L3>
```

说明：
- 输出文件会写入原理图文件所在目录，并命名为 `{schematic_stem}_{repr}.py`，例如 `test_L1.py`。
- 如果省略 `-r`，则默认使用 `L1`。
- 如需运行内置调试示例，请使用 `--debug`。

示例（显式指定 L1）：
```bash
python ./dataset_construction/kicad_read_sch.py -m test -s ./dataset_construction/sch_sketch/test.kicad_sch -r L1
# -> ./dataset_construction/sch_sketch/test_L1.py
```

### 1.5 Jsonl 数据集生成

运行以下命令，根据一个原理图文件生成一条 JSONL 训练数据集条目。

```bash
python ./dataset_construction/make_dataset.py \
    -s <path/to/schematic.kicad_sch> \
    -o <path/to/output.jsonl>
```

说明：
- 该单原理图模式会将样本写入 `-o` 指定的 JSONL 文件。
- 脚本会根据原理图文件名后缀自动推断表示层级，例如 `*_L1.kicad_sch`、`*_L2.kicad_sch` 或 `*_L3.kicad_sch`。
- 如果省略 `-s` 和 `-o`，脚本将回退到批处理模式，并处理 `./dataset_construction/make_dataset.py` 中 `BASE_DIR` 下的完整数据集。

示例：
```bash
python ./dataset_construction/make_dataset.py \
    -s ./dataset_construction/sch_sketch/test_L1.kicad_sch \
    -o ./jsonl_dataset/new_form/test.jsonl
```

在批处理模式下，脚本会写入 `./dataset_construction/make_dataset.py` 中配置的默认数据集路径。

### 2. 模型训练

运行以下命令，使用 JSONL 数据集和 LoRA 微调来训练原理图生成模型：

```bash
python ./training/train.py \
    --out_dir <output_directory> \
    --data_file <path/to/dataset.jsonl>
```

说明：
- `--out_dir`：用于保存训练后模型权重和检查点的目录。
- `--data_file`：JSONL 训练数据集的路径。
- 训练在基础 GPT-oss-20b 模型选定的专家层（7、15、23）上使用 LoRA 适配器。
- 训练进行 2 个 epoch，并启用梯度检查点和仅 assistant 损失。
- 训练前会自动过滤长度超过 13312 个 token 的样本。

示例：
```bash
python ./training/train.py \
    --out_dir models/my_experiment \
    --data_file str(Path(project_path) / "dataset_construction" / "jsonl_dataset" / "test.jsonl")
```

### 3. 原理图生成

我们提供两种生成 PCB 原理图的方式。
1. 从数据集条目生成
```
python ./schematic_generation/generate.py --test_dataset --index {DATASET_INDEX}
```
数据集可在 Hugging Face 获取：[microsoft/SchGen_dataset](https://huggingface.co/datasets/microsoft/SchGen_dataset)。

2. 直接输入用户请求
```
python ./schematic_generation/generate.py --test_raw --prompt {User_Request}
```

原理图生成模型可以从 [microsoft/SchGen](https://huggingface.co/microsoft/SchGen) 加载，也可以从本地已训练模型路径加载。（默认从在线 Hugging Face 仓库加载，但你可以通过指定 ``--model_path {YOUR_MODEL_PATH (Optional)}``，将其修改为自己的模型路径。）
生成的原理图 Python 代码表示默认保存在 ``./schematic_generation/generated.py``，也可以替换为你自己的路径。

生成代码后，可以执行以下命令创建项目和对应原理图

```
python ./init_project.py {YOUR_PROJECT_NAME} {YOUR_CODE_PATH}
```

#### 用户验证

尽管 SchGen 可以根据自然语言请求自动生成 PCB 原理图，但生成的设计仍可能包含错误连接、组件选择错误或布局不一致等问题。

在继续生成 PCB 布局之前，用户应在 KiCad 中检查生成的原理图。   
如有必要，用户可以修改请求，或手动修改生成的原理图/代码，然后重新运行该工作流。


### 示例：生成基于 AP2112K 的 3.3V 稳压器

此示例通过一个简单的用户请求演示端到端工作流：

```text
我想要一个基于 AP2112K 的 3.3V 稳压器
```

#### 1. 生成原理图代码

```bash
python ./schematic_generation/generate.py \
    --test_raw \
    --prompt "我想要一个基于 AP2112K 的 3.3V 稳压器" \
    --model_path "/home/ruichunma/workspace/SchGen/models/gpt-oss-20b-pcb-finetune-L1" (Optional)
```

生成的 Python 原理图代码将保存到默认路径：

```bash
./schematic_generation/generated.py
```

#### 2. 创建 KiCad 项目和原理图

```bash
python ./init_project.py voltage_regulator ./schematic_generation/generated.py
```

该命令会创建一个名为 `voltage_regulator` 的 KiCad 项目，并执行生成的原理图代码，以生成对应的 `.kicad_sch` 文件。

#### 3. 用户验证

在 KiCad 中打开生成的原理图并检查设计是否符合要求。如果不符合，请修改 prompt，或在继续之前手动编辑生成的原理图/代码。

## 免责声明

生成的原理图和 PCB 布局在制造或部署前，应始终由有经验的工程师进行审查。

SchGen 旨在用于快速原型开发和研究，不保证电气正确性或可制造性。制造或部署前必须进行人工审查。

## 许可证

本项目采用 MIT 许可证发布。
