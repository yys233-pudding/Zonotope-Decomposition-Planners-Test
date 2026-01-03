# 分解算法测试框架实现方案

## 目标
创建一个测试框架，比较不同空间分解算法的性能，包括：
- ConvexPartition_HM (Hertel-Mehlhorn)
- ConvexPartition_OPT (Keil-Snoeyink)  
- TriangularDecomposition (OMPL的三角化)

## 测试指标
1. **分解时间**：执行分解算法所需的时间
2. **路径规划时间**：使用分解结果进行路径规划的时间
3. **总时间**：分解时间 + 路径规划时间
4. **分解质量**：
   - 分解部分数（越少越好）
   - 分解区域的平均面积
   - 其他质量指标

## 实现方案

### 方案概述
由于OMPL的geometric planners（RRT等）不直接使用分解结果，我们需要：
1. 直接测试分解算法的性能（分解时间和质量）
2. 对于路径规划，使用标准的OMPL planners，但记录路径规划时间
3. 创建一个新的benchmark程序，类似于pdt的benchmark.cpp，但专注于分解算法测试

### 需要创建的文件

#### 1. 测试程序主文件
- `pdt/src/experiments/src/decomposition_benchmark.cpp`
  - 主测试程序
  - 对每个分解算法执行测试
  - 记录分解时间、路径规划时间
  - 调用报告生成器

#### 2. 报告类
- `pdt/src/reports/include/pdt/reports/decomposition_benchmark_report.h`
- `pdt/src/reports/src/decomposition_benchmark_report.cpp`
  - 生成PDF报告
  - 比较各分解算法的性能

#### 3. 配置文件
- `pdt/parameters/demo/decomposition_benchmark_demo.json`
  - 测试参数配置

#### 4. CMakeLists.txt修改
- `pdt/src/experiments/CMakeLists.txt`：添加新的可执行文件
- `pdt/src/reports/CMakeLists.txt`：添加报告源文件

## 技术细节

### 分解算法集成

#### HM和OPT算法
- 源文件：`hybzono-planner/extern/polypartition/src/polypartition.cpp`
- 头文件：`hybzono-planner/extern/polypartition/src/polypartition.h`
- 需要编译polypartition.cpp到测试程序中

#### TriangularDecomposition
- 使用OMPL的TriangularDecomposition类
- 需要包含OMPL的头文件

### 路径规划
- 使用标准的OMPL geometric planners（如RRTConnect）
- 虽然不直接使用分解结果，但可以记录规划时间作为参考
- 未来可以扩展为使用分解结果的自定义planner

## 测试流程

1. 读取配置文件
2. 对每个分解算法：
   - 执行分解（记录分解时间）
   - 记录分解质量指标（部分数等）
   - 执行路径规划（记录规划时间）
   - 计算总时间
3. 生成报告，比较所有算法的性能

## 实现步骤

### 步骤1：修改CMakeLists.txt
- `pdt/src/experiments/CMakeLists.txt`：添加decomposition_benchmark可执行文件
  - 包含polypartition.cpp源文件
  - 添加polypartition头文件路径
  - 链接必要的库

### 步骤2：创建测试程序
- `pdt/src/experiments/src/decomposition_benchmark.cpp`
  - 测试HM、OPT、TriangularDecomposition三种算法
  - 记录分解时间、路径规划时间、总时间
  - 记录分解质量指标

### 步骤3：创建报告类
- `pdt/src/reports/include/pdt/reports/decomposition_benchmark_report.h`
- `pdt/src/reports/src/decomposition_benchmark_report.cpp`
- 修改`pdt/src/reports/CMakeLists.txt`

### 步骤4：创建配置文件
- `pdt/parameters/demo/decomposition_benchmark_demo.json`

## 实现注意事项

1. **分解算法接口**：
   - HM/OPT：使用polypartition库的ConvexPartition_HM和ConvexPartition_OPT
   - TriangularDecomposition：使用OMPL的TriangularDecomposition类

2. **路径规划**：
   - 使用标准的OMPL geometric planners（如RRTConnect）
   - 记录规划时间作为参考指标

3. **测试数据**：
   - 支持从polypartition/test目录读取测试数据
   - 支持随机生成测试多边形

## 下一步
开始实现代码。
