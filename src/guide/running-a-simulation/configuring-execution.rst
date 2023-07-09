.. _Configuring Execution:

配置执行
=====================

:class:`CUDASimulation<flamegpu::CUDASimulation>` 实例提供了一系列配置选项，可以直接在代码中或通过命令行界面进行控制。 下表详细介绍了代码中可用的变量以及命令行界面可用的参数。

======================= ========================== ================== ====================================================================================
配置变量                 完整的参数                  缩写的参数          描述
======================= ========================== ================== ====================================================================================
``input_file``          ``--in <file path>``       ``-i <file path>`` 用于读取输入状态（代理/环境数据）的 JSON/XML 文件的路径。 请参阅 :ref:`File Format Guide<Initial State From File>`
``step_log_file``       ``--out-step <file path>`` n/a                用于存储步骤记录数据的 JSON/XML 文件的路径。
``exit_log_file``       ``--out-exit <file path>`` n/a                用于存储退出日志记录数据的 JSON/XML 文件的路径。
``common_log_file``     ``--out-log <file path>``  n/a                用于存储步骤和退出日志记录数据的 JSON/XML 文件的路径。
``truncate_log_files``  n/a                        n/a                如果为 true，日志文件将覆盖具有相同路径/名称的任何预先存在的文件。 默认值 true。
``random_seed``         ``--random <int>``         ``-r <int>``       随机种子。 默认值是时钟的样本（例如，每次运行都会改变）。
``steps``               ``--steps <int>``          ``-s <int>``       要执行的模拟步骤数。 0 将无限期地运行，或者直到退出函数导致模拟结束。 默认值 1。    
``quiet``               ``--quiet``                ``-q``             不要将集成进度打印到控制台。
``verbose``             ``--verbose``              ``-v``             将配置、进度和计时 (-t) 信息打印到控制台。
``timing``              ``--timing``               ``-t``             将模拟时序详细信息输出到控制台。 默认值 false。
``unknown args``        ``--silence-unknown-args`` ``-u``             在此标志之后传递的未知参数的静默警告。
``console_mode``        ``--console``              ``-c``             仅构建可视化，禁用可视化。 默认值 false。
``device_id``           ``--device <device id>``   ``-d <device id>`` 要使用的 GPU 的 CUDA 设备 ID。 默认值 0 (请注意，这可以在 :class:`CUDASimulation::Config<flamegpu::CUDASimulation::Config>` 中找到)
``inLayerConcurrency``  n/a                        n/a                允许在层内使用并发性。 默认值“true”。(这可以在 :class:`CUDASimulation::Config<flamegpu::CUDASimulation::Config>` 中找到)
n/a                     ``--help``                 ``-h``             打印命令行界面的帮助并退出。
======================= ========================== ================== ====================================================================================

为了处理命令行参数 ``argc`` and ``argv`` (仅限Python: ``argv`` ) 必须传递给 :func:`initialise()<flamegpu::Simulation::initialise>`。

您可能还希望通过在调用:func:`initialise()<flamegpu::Simulation::initialise>`之前设置值来指定自己的默认值：   

.. tabs::

  .. code-tab:: cpp C++

    int main(int argc, const char **argv) {
    
        ...
        
        // 从模型创建模拟对象
        flamegpu::CUDASimulation simulation(model);
        
        // 更改默认配置
        simulation.SimulationConfig().random_seed = 12;
        
        // 使用提供的命令行参数初始化模型
        simulation.initialise(argc, argv);
        
        // 运行模拟
        simulation.simulate();
        
        ...

  .. code-tab:: py Python
  
    # 导入 sys 以访问 argv
    import sys

    # 从模型创建模拟对象
    simulation = pyflamegpu.CUDASimulation(model)
        
    # 更改默认配置
    simulation.SimulationConfig().random_seed = 12;
    
    # 使用提供的命令行参数初始化模型
    simulation.initialise(sys.argv)

    # 运行模拟
    simulation.simulate()



要在代码中配置模拟，必须通过:class:`Simulation::Config<flamegpu::Simulation::Config>`和:class:`CUDASimulation::Config<flamegpu::CUDASimulation::Config>`结构更新变量，这些变量分别通过 :class:`CUDASimulation<flamegpu::CUDASimulation>`实例上的 :func:`SimulationConfig()<flamegpu::Simulation::SimulationConfig>` 和 :func:`CUDAConfig()<flamegpu::CUDASimulation::CUDAConfig>` 访问。 随后必须调用 :func:`applyConfig()<flamegpu::Simulation::applyConfig>` 来实现对配置的任何更改。


.. tabs::

  .. code-tab:: cpp C++
     
    // 从模型创建模拟对象
    flamegpu::CUDASimulation simulation(model);
    
    // 更新配置
    simulation.SimulationConfig().steps = 100;
    simulation.SimulationConfig().input_file = "input.json";
    simulation.CUDAConfig().device = 1;

    // 应用更新的配置
    simulation.applyConfig();
    
    // 运行模拟
    simulation.simulate();

  .. code-tab:: py Python

    # 从模型创建模拟对象
    simulation = pyflamegpu.CUDASimulation(model)
    
    # 更新配置
    simulation.SimulationConfig().steps = 100
    simulation.SimulationConfig().input_file = "input.json"
    simulation.CUDAConfig().device = 1

    # 应用更新的配置
    simulation.applyConfig()

    # 运行模拟
    simulation.simulate()

相关链接
-------------
* 用户指南: :ref:`Initial State From File<Initial State From File>`
* 完整的 API 文档 :class:`CUDASimulation<flamegpu::CUDASimulation>`
* 完整的 API 文档 :class:`Simulation<flamegpu::Simulation>`
* 完整的 API 文档 :class:`Simulation::Config<flamegpu::Simulation::Config>`
* 完整的 API 文档 :class:`CUDASimulation::Config<flamegpu::CUDASimulation::Config>`