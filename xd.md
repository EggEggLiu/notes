GPU Pipeline

RHI

Shader(VertexShader画点决定形状 + FragmentShader着色[插值算法], GeometryShader决定精细程度，可以加点增强清晰度)

光栅化(确定小块的归属问题)

BRDF(积分integral)

从Camera出发，而不是光源出发，从而减少计算量

OpenGL、Vulkan、DX

对多次反射的处理
- 不处理
- 预计算静物
- 光线追踪

双重缓冲(一个用于显示的frontBuffer，一个用来画的backBuffer) / 三重缓冲(多一个备胎)

glfw
- OpenGL是显卡厂商录好的驱动，用来画图。glfw负责开窗口呈现画好的东西

glad
- 从动态库加载显卡厂商实现的OpenGL函数

静态库.lib和动态库.dll

VAO(点集规范)

VBO(点数据)

EBO(边数据)

GLSL写vs、fs送进GPU里绑定program

render loop
