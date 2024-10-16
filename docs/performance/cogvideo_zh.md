## CogVideo 性能表现

使用xDiT在CogVideo中的细节: [利用xDiT多GPU并行执行CogVideoX文生视频流程](https://medium.com/@xditproject/aigc%E6%8E%A8%E7%90%86%E5%8A%A0%E9%80%9F-%E5%88%A9%E7%94%A8xdit%E5%B9%B6%E8%A1%8Ccogvideox%E6%96%87%E7%94%9F%E8%A7%86%E9%A2%91%E6%B5%81%E7%A8%8B-86255f9979a9)

CogVideo 是一个文本到视频的模型。xDiT 目前整合了 USP 技术（包括 Ulysses 注意力和 Ring 注意力）和 CFG 并行来提高推理速度，同时 PipeFusion 的工作正在进行中。由于 CogVideo 在视频生成尺寸上的限制，USP 的最大并行级别为 2。因此，xDiT 可以利用最多 4 个 GPU 来执行 CogVideo，尽管机器内可能有更多的 GPU。

在配备 L40（PCIe）GPU 的计算平台上，我们对基于 `diffusers` 库的单 GPU CogVideoX 推理与我们提出的并行化版本在生成 49帧（6秒）720x480 分辨率视频时的性能差异进行了深入分析。

如图所示，对于基础模型 CogVideoX-2b，无论是采用 Ulysses Attention、Ring Attention 还是 Classifier-Free Guidance（CFG）并行，均观察到推理延迟的显著降低。值得注意的是，由于其较低的通信开销，CFG 并行方法在性能上优于其他两种技术。通过结合序列并行和 CFG 并行，我们成功提升了推理效率。随着并行度的增加，推理延迟持续下降。在最优配置下，xDiT 相对于单GPU推理实现了 3.53 倍的加速，使得每次迭代仅需 0.6 秒。鉴于 CogVideoX 默认的 50 次迭代，总计 30 秒即可完成 6 秒视频的端到端生成。

<div align="center">
    <img src="https://raw.githubusercontent.com/xdit-project/xdit_assets/main/performance/cogvideo/cogvideo-l40-2b.png" 
    alt="latency-cogvideo-l40-2b">
</div>

对于更复杂的 CogVideoX-5b 模型，虽然其增加了参数以提升视频质量和视觉效果，导致计算成本显著增加，但所有方法在该模型上仍保持了与 CogVideoX-2b 相似的性能趋势，且并行版本的加速比进一步提升。与单GPU版本相比，xDiT 实现了高达 3.91 倍的推理速度提升，将端到端视频生成时间缩短至 80 秒左右。

<div align="center">
    <img src="https://raw.githubusercontent.com/xdit-project/xdit_assets/main/performance/cogvideo/cogvideo-l40-5b.png" 
    alt="latency-cogvideo-l40-5b">
</div>

同样，在配备 A100 GPU 的系统上，xDiT 也展示了类似的加速效果。

<div align="center">
    <img src="https://raw.githubusercontent.com/xdit-project/xdit_assets/main/performance/cogvideo/cogvideo-a100-5b.png" 
    alt="latency-cogvideo-a100-5b">
</div>