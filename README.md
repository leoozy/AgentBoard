<div align="center">
<h1> AgentBoard: An Analytical Evaluation Board of Multi-turn LLM Agents </h1>
</div>

<div align="center">
  <!-- <a href="#model">Model</a> • -->
  📚 <a href="https://drive.google.com/file/d/1h8fQlQi-Xk-dmKpUlKbwTZi3vLg3jXVv/view?usp=sharing">Anonymous Data Link</a> |
  📊 <a href="https://wandb.ai/agentboard-anon/evaluate-mistral-7b">Anonymous Panel Link</a>

</div>


## Introduction

AgentBoard emphasizes **analytical evaluation** for Large Language Models (LLMs) as generalist agents to perceive and act within various environments. It outlines four principles for constructing a benchmark to evaluate LLMs as generalist agents:
 1. **Task Diversity**: AgentBoard incorporates 9 distinct tasks to comprehensively understand the generalist ability of LLM agents, which is built upon LLM's extensive knowledge base and exceptional scenario comprehension.
 2. **Multi-round Intercation**: AgentBoard provides multi-round interaction between agents and environment, which is necessary to reflect the evolutionary nature of human intelligence, which continuously receives information and adapts towards the environment.
 3. **Partially-Observable Environments**: In AgentBoard, the complete state of the environment is not available to the agent, which assesses agent world modeling ability as additional knowledge needs to be acquired through online
exploration.
 4. **Analytical Evaluation**: AgentBoard is a systematic evaluation platform: it includes a user-friendly script to construct goal-oriented reflex agents for a range of models, and features a panel for visualizing and interpreting results across multiple dimensions of agent proficiency, including *fine-grained progress rates, grounding accuracy, performance breakdown for hard and easy examples, long-range interactions, detailed performance across various sub-skills, and trajectory with friendly visualization*

<div align="center">

<img src="./assets/main_graph.png">
<!-- <h1> A nice pic from our website </h1> -->

</div>



## Table of Contents
<details>
<summary>
Click to expand the table of contents
</summary>

- [Introduction](#introduction)
- [🚀 Quick Start](#-quick-start)
  - [Setup Environment](#setup-environment)
  - [Evaluate Models](#evaluate-models)
  - [Launch AgentBoard Analytical Evaluation Panel](#launch-agentboard-analytical-evaluation-panel)
- [Data](#data)
  - [Data Overview](#data-overview)
  - [Download Link](#download-link)
- [Evaluation Details](#evaluation-details)
  - [Evaluation Preparation](#evaluation-preparation)
    - [Internet Access](#internet-access)
    - [Environment Preparation](#environment-preparation)
  - [Running Proprietary Models](#running-proprietary-models)
    - [For Tasks except WebShop](#for-tasks-except-webshop)
    - [For WebShop](#for-webshop)
  - [Running Open-source Models](#running-open-source-models)
  - [LLM Customization](#llm-customization)
  - [Agent Customization](#agent-customization)
  - [Runtime Estimation](#runtime-estimation)
</details>


## 🚀 Quick Start 

Here we provide a quick start guide to evaluate LLM agents on AgentBoard within 30 minutes. 

### Setup Environment
We provide both local setup (recommended) and docker as follows:
<details>
<summary>
Click to expand local setup procedures (~ 15 minutes).
</summary>

Setup with a setup.sh: 

**Step 1. Create a conda environment**
```shell
conda create -n ${YOUR_ENV_NAME} python=3.8.13  # python version should be 3.8.13
conda activate ${YOUR_ENV_NAME}
```

**Step 2. download this repo**


**Step 3. Download the data from huggingface**

Download the [data](https://drive.google.com/file/d/1h8fQlQi-Xk-dmKpUlKbwTZi3vLg3jXVv/view?usp=sharing) and put it in `AgentBoard/data`
```shell
cd AgentBoard/data
tar -zxvf data.tar.gz
```

**Step 4. Set up the environment for tasks except WebArena**
```shell
INSTALL_WEBARENA=false bash ./setup.sh

# After running the above command, the env will support other tasks than WebArena
```

**Step 5. Set up the environment for WebArena**
```shell
# Please check whether the dubs and Xvfb are installed before building it
# For Ubuntu or Debian
dpkg -l | grep dbus  # will return the info
systemctl status dbus  # will return the status(active (running))
dpkg -l | grep xvfb  # will return the info

#-----------------------------------------------------------------------#

# For CentOS
yum list installed | grep Xvfb  # will return the Xvfb info
systemctl status dbus  # will return the status(active (running))
dnf list installed | grep dbus  # will return the dbus info
```

If so, you may install the webarena environment directly.
```shell
INSTALL_WEBARENA=true bash ./setup.sh
```

If not, please jump to Step 6 or [Installation by Docker](#52-installation-by-docker)

**(Additional) Step 6. Install the dubs and Xvfb**
```shell
# You must use the sudo permission to do the following:

# For Ubuntu or Debian
# Install and start the dbus service
apt-get install dbus
/etc/init.d/dbus start

# Install ans start the Xvfb
sudo apt-get update
sudo apt-get install xvfb

INSTALL_WEBARENA=true bash ./setup.sh
#--------------------------------------------------------#

# For Centos
# Install and start the dbus service
yum install -y dbus-x11
/etc/init.d/dbus start

# Install ans start the Xvfb
yum update
yum install -y Xvfb

INSTALL_WEBARENA=true bash ./setup.sh
```
</details>

<details>
  <summary>
Click to expand docker setup procedures. (~12G, 5 minutes)
</summary>

  Docker info: CentOS

**Step 1. Pull the docker image and run docker locally**
```shell
docker pull anonymous2024/agentboard:0201
docker run -itd \
    --gpus all \
    --network host \
    --name agent_space \
    --shm-size 64gb \
    -v /MODEL_PATH:/model_download \
    -v /DATA_PATH:/data \
    anonymous2024/agentboard:0201 \
    /bin/bash
docker attach agent_space # YOUR_CONTAINER_NAME
```

**Step 2. activate the env**
```shell
conda activate agentboard
```

**Step 3. Download the code and data**
Download the repo.
Download the [data](https://drive.google.com/file/d/1h8fQlQi-Xk-dmKpUlKbwTZi3vLg3jXVv/view?usp=sharing) and put it in `AgentBoard/data`.
```shell
# Download the data and move it to the project root dir
cd AgentBoard/data
tar -zxvf data.tar.gz
```

**Step 3. Build search engine index(For WebShop)**
```shell
cd ./agentboard/environment/WebShop/search_engine
mkdir -p resources resources_100 resources_1k resources_100k
python convert_product_file_format.py # convert items.json => required doc format
mkdir -p indexes
./run_indexing.sh
cd ../../../
```

**Step 4. Start web service(For Webarena)**
```shell
/etc/init.d/dbus start  # start dbus
Xvfb :99 -screen 0 1280x720x24 &  # start xvfb display
export DISPLAY=:99
python -m playwright install
```
</details>

### Setup Environment Variables in `AgentBoard/.env`
Environment Variables needed for AgentBoard include:
```
PROJECT_PATH = {path to project}/AgentBoard

ANTHROPIC_API_KEY=...
OPENAI_API_KEY=...

TODO_KEY=...
MOVIE_KEY=...
SHEET_EMAIL=...

WANDB_API_KEY=...
```
<details>
<summary>
Click to expand API key setup procedures.
</summary>

**Variables 1: API keys for Tool tasks**

Since API keys for **Tool** tasks are private, we do not provide them in this repo.

Please follow this detailed [guide](./assets/api_keys_tool.md) to get API keys for **Tool** tasks.

**Variables 2: Weights&Bias key for AgentBoard Online Visualization**

Please paste `WANDB_API_KEY` from here [guide](https://wandb.ai/authorize) in `.env` file to login Weights&Bias for AgentBoard Visulization.


**Variables 3: API keys for Proprietary models**

**⚠️ You don't need to setup API keys for models you don't want to use.**

If you use OpenAI models, please put your API keys in `.env` file.
```shell
OPENAI_API_TYPE="open_ai"
OPENAI_API_KEY=${YOUR_OPENAI_API_KEY}
```

If you use Anthropic models, please put your API keys in `.env` file.
```shell
ANTHROPIC_API_KEY=${YOUR_ANTHROPIC_API_KEY}
```
</details>

### Evaluate Models
Example script for `GPT-3.5-Turbo`:
```
python agentboard/eval_main.py \
    --cfg-path eval_configs/main_results_all_tasks.yaml \
    --tasks alfworld \
    --model gpt-3.5-turbo-0613 \
    --wandb \
    --log_path ./results/gpt-3.5-turbo-0613 \
    --project_name evaluate-gpt-35-turbo-0613 \
    --baseline_dir ./data/baseline_results
```
We now offer configuration for 12 SOTA LLM models (`gpt-4`,`gpt-3.5-turbo-0613`, `text-davinci-003`,`claude2`,`deepseek-67b`,`lemur-70b`, `mistral-7b`,`codellama-13b(34b)`,`llama2-13b(70b)`,`vicuna-13b-16k`) and a simple reflex agent based on act-only prompting. You could also customize your own [agents](https://anonymous.4open.science/r/AgentBoard/assets/agent_customization.md) and [LLMs](https://anonymous.4open.science/r/AgentBoard/assets/llm_customization.md). Models supported by [vLLM](https://github.com/vllm-project/vllm) should be generally supported in AgentBoard, while different models may require specific prompt templates.


### Launch AgentBoard Analytical Evaluation Panel
AgentBoard integrates illustrative [Weights&Bias](https://wandb.ai/site) visualization to help researchers better systematically analyze LLM agents. You can simply turn on `--wandb` switch in the arguments and customize the `project_name` and `baseline_dir` of your wandb project as the evaluation command above.

Before running, you need to setup wandb login or environment variable as instructed in [quick-start](#setup-environment-variables-in-agentboardenv). The visualization results would be both stored offline at `\wandb`. Normally after executing the evaluation command, you can visualize the live AgentBoard panel online at `https://wandb.ai/{your_wandb_id}/{project_name}`. We provide example WandB logging pages for [mistral-7b](https://wandb.ai/agentboard-anon/evaluate-mistral-7b?workspace=user-)

Note that if your run is not logged online (on a cluster without internet), you could later sync local runs to wandb online with `wandb sync [OPTIONS] [PATH]..` as detailed in [wandb docs](https://docs.wandb.ai/ref/cli/wandb-sync). 


#### Local log files
In addition to online results viewing, local logs are automatically stored in `{log_path}`. In WebArena, we additionally support more detailed trajectory files, including web page screenshots and network traffic records.
<details>
  <summary>
    Log file organization: 
  </summary>

```
{log_path}
├── logs                    # detailed example-wise logs for each task
│  ├── webarena_tracks      # WebArena provided rendered HTML files of the execution trace and a './trace' folder which is automatically generated with Playwright
│  │  ├── traces
│  │  │  ├── 102.zip
│  │  ├── render_102.html
│  │  ├── ...
│  ├── alfworld.jsonl       # each line is a json dictionary logging the statistics, trajectory, and prompt for each example
│  ├── babyai.jsonl
│  ├── ...
├── all_results.txt         # overall metrics for each task
├── dimension.txt           # agent capability dimensional scores for current LLM agent
├── alfworld.txt            # a general log for example-wise statisitcs for each task
├── babyai.txt              
└── ...              
```
</details>


## Data

### Data Overview
AgentBoard is composed of 9 diverse tasks which can be divided into 4 types, including **Embodied AI**, **Game**, **Web**, and **Tool**:



<table align="center">
  <tbody>
    <tr align="center" valign="bottom">
      <td>
        <b>Embodied AI</b>
      </td>
      <td>
        <b>Game</b>
      </td>
      <td>
        <b>Web</b>
      </td>
      <td>
        <b>Tool</b>
      </td>
    </tr>
    <tr valign="top">
<td>

- AlfWorld
- ScienceWorld
- BabyAI

</td>

<td>

- Jericho
- PDDL

</td>

<td>

- WebShop
- WebArena

</td>

<td>

- Tool-Query
- Tool-Operation

</td>

  </tr>
  </tbody>
</table>


> Note: Please download the dataset from the link provided below for the reason that the data in Dataset Viewer is not complete.

### Download Link
You can download the whole evaluation data from the anonymous Google Drive link: [data](https://drive.google.com/file/d/1h8fQlQi-Xk-dmKpUlKbwTZi3vLg3jXVv/view?usp=sharing)

The file structure of evaluation data is as follows:
<details>
<summary>
Click to expand the file structure
</summary>

```
data
├── baseline_results
├── alfworld
│   ├── alfred.pddl # additional data for alfworld
│   ├── alfred.twl2 # additional data for alfworld
│   ├── json_2.1.1  # additional data for alfworld
│   └── test.jsonl
├── babyai
│   └── test.jsonl
├── jericho
│   ├── test.jsonl
│   └── z-machine-games-master  # additional data for jericho
├── pddl
│   └── test.jsonl
├── scienceworld
│   └── test.jsonl
├── tool-operation
│   └── test.jsonl
├── tool-query
│   ├── academia  # additional data for academia tool
│   └── test.jsonl
├── webarena
│   └── test.jsonl
└── webshop
    └── test.jsonl
```

**We also provide baseline run loggings in `data/baseline_results`, which can be used for visualization in our panel. **

</details>


## Evaluation Details
### Evaluation Preparation

#### Internet Access
For regions with Internet restrictions, to evaluate the **Tool-Query**, **Tool-Operation** and **WebArena** tasks, please make sure that the machine can access the Internet.

You can check whether you have network issues by observing the output during the execution process.

#### Environment Preparation
We provide two ways to install the environment of AgentBoard, as specified in [QuickStart](#setup-environment).


### Running Proprietary Models
In this section, we provide a script to evaluate the closed-source models on each task.

Please do not forget to set the environment variables (e.g., `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`) before running the following commands.

#### For Tasks except WebShop
We provide a quick start script to evaluate the `gpt-3.5-turbo-0613` model on `alfworld` task.

```shell
python agentboard/eval_main.py \
    --cfg-path eval_configs/main_results_all_tasks.yaml \
    --tasks alfworld \
    --model gpt-3.5-turbo-0613 \
    --wandb \
    --log_path ./results/gpt-3.5-turbo-0613 \
    --project_name evaluate-gpt-35-turbo-0613 \
    --baseline_dir ./data/baseline_results
```
Parameters:
- `--cfg-path`: The path of the config file, please refer to `eval_configs/main_results_all_tasks.yaml` for more details.
- `--tasks`: The tasks to be evaluated, e.g. `tool-query`, `tool-operation`, `webarena`, `alfworld`, `babyai`, `jericho`, `pddl`, `scienceworld`.
- `--model`: The LLM to be evaluated. We provide some LLM models, including:
  - `gpt-3.5-turbo`
  - `gpt-3.5-turbo-16k`
  - `gpt-4`
  - `text-davinci-003`
  - `claude2`
- `wandb`: Online visualization will be launched given this parameter. Remove this parameter from the script if you don't need visualization, e.g. during debugging.
- `log_path`: Path to save logs, as specified [here](#check-and-analyze-results).
- `project_name`: Project name for Weights&Bias. This parameter is not necessary when wandb parameter is not used.
- `baseline_dir`: Directory to results files of baseline models you want to compare with during the run.

#### For WebShop

First, please start the WebShop server by running the following commands:
```shell
cd ./agentboard/environment/WebShop
bash ./run_dev.sh
cd ../../..
```

Then, run the following command to evaluate the `gpt-3.5-turbo-0613` model on `webshop` task.
```shell
python agentboard/eval_main.py \
    --cfg-path eval_configs/main_results_all_tasks.yaml \
    --tasks webshop \
    --model gpt-3.5-turbo-0613 \
    --wandb \
    --log_path ./results/gpt-3.5-turbo-0613 \
    --project_name evaluate-gpt-35-turbo-0613 \
    --baseline_dir ./data/baseline_results
```

### Running Open-source Models
In AgentBoard, we have pre-supported the following 8 open-source models, by default we use `vLLM` to speed up inference.
  - `llama2-13b`
  - `llama2-34b`
  - `codellama-13b`
  - `codellama-34b`
  - `vicuna-13b-16k`
  - `lemur-70b`
  - `deepseek-67b`
  - `mistral-7b`

> Please refer to `eval_configs/main_results_all_tasks.yaml` for more details about these models.


To evaluate these models, you can run the following command:
```shell
python agentboard/eval_main.py \
    --cfg-path eval_configs/main_results_all_tasks.yaml \
    --tasks ${TASK_NAME} \
    --model ${OPEN_SOURCE_MODEL_NAME}
```
We also provide LLM customizations, please refer to [LLM Customization](#llm-customization) for more details.


### LLM Customization

Please refer to [llm_customization.md](./assets/llm_customization.md) for more details about LLM customization.


### Agent Customization

Please refer to [agent_customization.md](./assets/agent_customization.md) for more details about agent customization.


### Runtime Estimation

The evaluation runtime for a language model depends on the device/API, model, and inference architecture used. In the case of open-source LLMs, the vllm inference speed is approximately 10 times faster than the huggingface pipeline.

To estimate the total time needed for evaluation, you can run a few steps to measure the inference speed and multiply it by the total number of LLM inferences, which is within 15,000 rounds.

The general formula for estimating the total time is `4h * speed`. Here are some examples of our runtime:

|     Model     | Device/API | Inference Architecture | Inference Speed | Total-time |
|:-------------:|:----------:|:----------------------:|:---------------:|:----------:|
|      GPT4     |  azure API |            -           |    1.5s/round   |    5.5h    |
| GPT-3.5-Turbo |  azure API |            -           |     1s/round    |     3h     |
| DeepSpeed-67b |   8*V100   |          vllm          |     5s/round    |    18.5h   |
|   Llama2-70b  |   8*V100   |          vllm          |     8s/round    |     28h    |
|   Llama2-70b  |   4*A100   |          vllm          |     4s/round    |   13.5h    |

