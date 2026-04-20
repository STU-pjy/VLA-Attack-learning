# VLA-attack总结

## **VLA = Vision（看图）+ Language（听懂指令）+ Action（直接输出动作）**。

**给机器人一张图和一句话，它直接输出机械臂该怎么动。** 

## OpenVLA模型架构：

![](D:\Desktop\Fig\OpenVLA框架.png)

## 仿真平台：LIBERO

```bash
git clone https://github.com/Lifelong-Robot-Learning/LIBERO.git
cd LIBERO
pip install -e .
```

## VLA模型：OpenVLA、Pi0、OpenVLA-OFT

（都是7B模型，最近的几篇论文都表明OpenVLA最容易攻击，Pi0难攻击一点，需要用到24G以上的卡运行）

[OpenVLA Repository](https://github.com/openvla/openvla)

[Pi0 Repository](https://github.com/Physical-Intelligence/openpi)

[OpenVLA-OFT Repository](https://github.com/moojink/openvla-oft)

## Dataset（RLDS格式）

https://huggingface.co/datasets/openvla/modified_libero_rlds

最外层是一个 dataset，里面每一条样本不是单步，而是一个 **episode**；每个 episode 下面再挂一个 **steps** 序列。

```bash
dataset = [episode_1,episode_2,...]
episode = {
    "steps": [step_0, step_1, ..., step_T],
    # episode-level metadata
}
```

```bash
step = {
    "observation": {
        "image_primary": uint8[H, W, 3],
        "image_wrist": uint8[H, W, 3],      # 可选
        "proprio": float32[D],              # 机械臂状态/末端状态
        "state": float32[D],                # 有些数据集这样命名
    },
    "action": float32[A],            # 连续动作 [dx, dy, dz, droll, dpitch, dyaw, gripper]
    "language_instruction": "pick up the red block",   # 有时放 step 里，有时放 episode 里
    "is_first": bool,
    "is_last": bool,
    "is_terminal": bool,
}
```

## 论文1：Model-Agnostic  Adversarial  Attack and Defense for  VLA Models

GitHub：[trustmlyoungscientist/EDPA_attack_defense: Model-agnostic Adversarial Attack and Defense for Vision-Language-Action Models](https://github.com/trustmlyoungscientist/EDPA_attack_defense)（已复现补丁攻击+仿真测评，没看防御方案-防御需要Lora微调要求A800以上的卡）

##### 提出了Embedding Disruption Patch Attack (EDPA)

**优点：不需要动作输出，只是破坏视觉与语言的语义对齐，并且拉大干净图像和攻击图像在视觉表征空间的差异。**

![](D:\Desktop\Fig\EDPA.png)

## 论文2：FREEZEVLA: Action-Freezing Attacks Against Vision-Language-Action Models

GitHub：[xinwong/FreezeVLA](https://github.com/xinwong/FreezeVLA)（没有README...coming soon）

**目的：让机器人不动（不动的时候相机视角不会改变，攻击会更加稳定、持久）**

![](D:\Desktop\Fig\FreezeVLA.png)

##### Cross-prompt：现实里用户说什么任务指令，攻击者不一定知道，所以攻击图像要对不同 prompt 都有效。

##### min-max优化框架

内层：找 hardest prompts，用预训练的LLM生成一批基础prompt，然后选对freeze loss关键的词，用同义词替换这个关键词，并且检查是否会更加freeze。获得hard prompt set。

外层：对所有这些 hard prompts，一起优化一张图，让模型都倾向于输出 freeze token。

## 论文3：Exploring the Adversarial Vulnerabilities of Vision-Language-Action Models in Robotics

## 论文4：Adversarial Attacks on Robotic Vision-Language-Action Models

GitHub：[eliotjones1/robogcg: Official GitHub repository for the paper "Adversarial Attacks on Robotic Vision Language Action Models"](https://github.com/eliotjones1/robogcg)

**针对文本进行攻击**

coming soon...
