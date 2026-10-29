# DTF-Net 部分代码公开说明

本项目用于水面波动条件下的水下对空图像恢复：相机位于水下，观测水面上方目标，
处理时变折射产生的几何畸变和局部模糊。这里只公开融合网络核心与使用示例。

## 公开范围

- `dtf_net/model.py`：由提供的实验工程原样复制，包含双编码器、三级先验门控、
  多尺度解码器、图像级混合及有界残差。保留原有模式开关，以保持权重键名兼容。
- `dtf_net/api.py`：新增的预计算结果融合接口，以及严格的权重加载。
- `dtf_net/infer.py`：读取 D、T、M、U 的图像推理示例。
- `options/`、`examples/`、`tests/`：配置、随机张量演示、结构与数值检查。

不公开：训练/验证/测试数据、真实实验图像、划分与清单、权重、TMT第三方实现、
几何配准及先验生成、训练代码与损失、对比和消融流水线、评估脚本、服务器脚本、
内部日志及论文图表。本公开包不能独立复现论文的全部流程或报告指标。

## 使用方式

从仓库根目录运行，Python >= 3.10：

```bash
python -m pip install -r requirements.txt
python -m pip install -e . --no-deps
python -m examples.synthetic_demo
python -m unittest discover -s tests -v
```

随机张量示例只验证结构。网络输出头零初始化，因此未训练网络输出两张候选图的
等权平均，这是初始化行为，不是论文恢复效果。

真实推理需要自行准备匹配的训练权重，以及来自同一窗口、同一目标帧、同一裁剪的：

| 输入 | 含义 |
| --- | --- |
| D / template_rgb.png | 几何模板 |
| T / stage1_center.png | TMT第一阶段恢复的目标帧 |
| M / flow_magnitude.png | 形变幅值先验（单通道编码图） |
| U / uncertainty.png | 对齐后时序不一致性先验（单通道编码图） |

四张图必须同尺寸。示例按8位RGB/灰度读取并除以255，与原数据加载器的基本数值
约定一致，但不自动缩放不匹配的先验图。M/U的原始生成、标定和编码未公开，
必须与训练时一致；不能随意逐图拉伸，也不能把彩色热力图作为先验输入。
U是辅助不一致性描述，不代表经过概率校准的不确定度。

```bash
python -m dtf_net.infer --config options/test/fusion.json --template datasets/example/template_rgb.png --temporal datasets/example/stage1_center.png --flow datasets/example/flow_magnitude.png --uncertainty datasets/example/uncertainty.png --checkpoint experiments/pretrained_models/your_model.pth --output results/example --device cpu
```

上述路径是你自行准备的文件示例，仓库没有附带这些实验数据或权重。GPU可改为
`--device cuda`。只有`stage1.*`权重被过滤，其余融合参数严格加载。旧检查点若没有
residual_scale字段，应以实际训练配置为准；给定源代码默认0.25。

完整研究流程的TMT读取20帧原始扰动序列。这里把已算好的T作为接口输入，内部
使用恒等适配器复用原融合代码，不代表使用一帧原始图像代替原时序分支。
联合微调后，T必须来自相应版本的TMT权重。

## 与参考仓库的关系

参考 VascHAT 的 README、配置、示例、测试、数据/权重占位目录与公开范围说明的组织方式。
没有复制其HAT网络代码，也没有套用其训练参数。
第三方方法链接见 NOTICE.md；论文作者、最终题目、DOI和正式GitHub地址待作者确认，
目前不编造引用条目或发表状态。

## 许可证

按作者选择，已公开文件采用Apache-2.0，允许按许可条款使用、修改和再分发。
该许可不覆盖未公开的数据、权重或第三方代码。
