# Custom Render Pipeline

## Renderer PipeLine Asset and Renderer Pipline

    Renderer pipeline assset 是Renderer pipline的载体，是unity生成的一个asset文件。通过Renderer pipeline asset 通过CreatePipline 创建Renderer pipline.

    Renderer Pipeline 中的Render接口是SRP的入口。

    CameraRenderer 是逻辑上的抽离，将Renderer Pipeline的逻辑剥离出来，实现解耦合。然后使用它来循环渲染所有的摄像机。

###Renderer 的基础理解
    整个渲染流程其实跟画画相似，先清理所有的痕迹，在一张白纸上来绘画。那渲染流程基本也是围绕这块。
    
1. 针对于每个摄像机剔除渲染不需要渲染的对象（请参阅 [CullingResults](https://docs.unity.cn/cn/2020.3/ScriptReference/Rendering.CullingResults.html)）
2. 设置投影矩阵（可以通过camera也可以自定义）
3. 清理深度，清理颜色，整理画板颜色
4. 设置排序参数 设置过滤对象参数 设置drawing参数
5. 提交渲染参数命令
6. 渲染天空盒
   
ps:34步骤是固定的常规渲染操作，可以通过不同的参数来渲染不透明物体和透明物体以及其他类型组合的物体。 ScriptableRenderContext.DrawRenderers会 发起一系列调用并混合ScriptableRenderContext.ExecuteCommandBuffer 调用。这些调用会设置全局着色器属性、更改渲染目标、分发计算着色器和其他渲染任务。最后要执行渲染循环的话 就调用ScriptableRenderContext.Submit。 天空盒有单独的渲染命令，不用走renderer buff。

从上面我们也就可以在最后渲染编辑器下的错误material。（粉色 材质球丢失或者shader报错），也可以渲染编辑器下摄像机的矩阵框以及Gizmos等。

### Renderer 常规术语

- [CommandBuffer](https://docs.unity.cn/cn/2020.3/ScriptReference/Rendering.CommandBuffer.html)： List of graphics commands to execute.（渲染命令列表）可以存储指定渲染状态（“设置渲染目标、绘制网格等等”）
- [ScriptableRenderContext](https://docs.unity.cn/cn/2020.3/ScriptReference/Rendering.ScriptableRenderContext.html) ：UPR的定义状态和渲染命令的上下文。SRP使用的延迟执行的方式来实现渲染，需要使用scripteableRenderContext来构建渲染命令列表（CommandBuffer），最后告诉unity底层来执行这些命令。最终unity的低级图形架构随后将指令发送到图形API
- [ScriptableRenderContext.ExecuteCommandBuffer](https://docs.unity.cn/cn/2020.3/ScriptReference/Rendering.ScriptableRenderContext.ExecuteCommandBuffer.html): 调用过程中，sripteableRenderContext 会将commandBuffer参数注册到自己要执行的命令内部列表中。这些命令要在Submit期间。并且如果绘制调用取决CommonBuffer中指定的渲染的状态，保在其他 ScriptableRenderContext 方法（如 DrawRenderers、DrawShadows）之前调用 ExecuteCommandBuffer.该方法用于立即执行CommandBuffer中的渲染命令
- [ScriptableRenderContext.Submit](https://docs.unity.cn/cn/2020.3/ScriptReference/Rendering.ScriptableRenderContext.Submit.html) Submit方法用于将CommandBuffer中的渲染命令提交到渲染队列中等待执行，适用于需要在适当的渲染阶段执行的情况。
   

    
