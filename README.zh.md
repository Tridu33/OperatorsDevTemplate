中文|[English](README.md)

# 针对Pybind11绑定的算子开发模板

[本项目](https://github.com/Tridu33/OperatorsDevTemplate/tree/main)分别用最基础的TensorAdd作为示例介绍“Python未皮，C++为翼”的算子开发调用流程，更多算子先考虑现有再重写：

- CPU，[更多CPU量化算子参考llamafile](https://github.com/Mozilla-Ocho/llamafile/tree/main/llama.cpp)，需了解x86的AVX指令集和arm64的NEON指令集用法等知识；
- CUDA.cu for GPU，[更多GPU推理算子参考CUDA官方samples](https://github.com/NVIDIA/cuda-samples/tree/master/Samples)、[樊哲勇老师的书籍《CUDA-Programming编程》](https://github.com/brucefan1983/CUDA-Programming)和[CUDA_kernel_Samples](https://github.com/Tongkaio/CUDA_Kernel_Samples)类似案例集，需了解CUDA和pyCUDA并行开发；
- Ascend NPU，[更多NPU推理算子参考官方案例](https://github.com/Ascend/samples/tree/master/cplusplus/level1_single_api/4_op_dev/1_custom_op)和[B站起飞的老谭](https://space.bilibili.com/668461244?spm_id_from=333.337.0.0)等资料，需了解TBE,pyACL,OMl量化推理算子库等Ascend系前置知识，按需寻找或者自行重写。

  CANN算子有几种开发方式：TBE DSL、TBE TIK与AI CPU。针对全新开发的算子，在进行代码开发前，首先需要选择合适的算子实现方式。开发或者迁移算子之前需要先查询[AI框架算子清单和CANN算子清单](https://www.hiascend.com/document/detail/zh/canncommercial/80RC1/apiref/operatorlist/operatorlist_0000.html))
- Python算子：原生python写的算子不需要pybind，[Triton官方tutorials](https://github.com/triton-lang/triton/blob/main/python/tutorials/01-vector-add.py)和[Awesome-Triton-Kernels](https://github.com/zinccat/Awesome-Triton-Kernels)。

JAX是autograd+XLA在纯函数微分编程的AI框架试验田，类比PyTorch,MindSpore,Tensorflow等存在，FP编程哲学在于可组合性足够灵活，比如[cuda+cpp写算子pybind11封装一个算子给调用](https://jax.ac.cn/en/latest/Custom_Operation_for_GPUs.html)。

实际上Python AI框架和底层算子是解耦的:

1. CopyIn任务: 输入H2D指针乱飞传递给底层异构算子;
2. Compute任务: 自动(llama.cpp等推理引擎主流做法是后端优先级根据可用性自动选择后端)或者手动(ktransformers使用yaml手工指定MoE具体结构到异构设备)指派dispatch计算图中任务到异构计算结构具体算子.so中执行;
3. CopyOut:任务 最后把计算结果通过D2H返回Python调用方即可;

本项目首先介绍pybind11调用cpp并调试；然后介绍cuda和cpp算子是如何在GPU机器上绑定并使用的；然后介绍AscendNPU算子开发入门。比如[oppenmlsys中简单介绍了MindSpore先注册一个算子接口然后用类似的方法dispatch到CPU,GPU,NPU多端实现的理论](https://github.com/openmlsys/openmlsys-zh/blob/main/chapter_programming_interface/c_python_interaction.md)，实际开发步骤可以参考[官网自定义算子的教程](https://www.mindspore.cn/docs/zh-CN/r2.5.0/model_train/custom_program/op_custom.html)：

- 算子原语注册=声明一个接口行为描述，后面对接CPU,GPU,NPU各种后端
- 书写GPU/CPU/Ascend NPU算子
- 注册算子pybind11绑定函数
