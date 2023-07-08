.. _ensembles:

运行多个模拟
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

FLAME GPU 2 提供 :class:`CUDAEnsemble<flamegpu::CUDAEnsemble>` 作为执行模型多个配置的批量运行的工具。


创建 a CUDAEnsemble
-----------------------

集成是一组批量执行的模拟，可以选择使用所有可用的GPU。要使用集成，请构造:class:`RunPlanVector<flamegpu::RunPlanVector>`和:class:`CUDAEnsemble<flamegpu::CUDAEnsemble>`而不是 :class:`CUDASimulation<flamegpu::CUDASimulation>`。

首先，您必须像往常一样定义模型，然后创建 :class:`CUDAEnsemble<flamegpu::CUDAEnsemble>`：

.. tabs::

  .. code-tab:: cuda CUDA C++
  
    flamegpu::ModelDescription model("example model");
    
    // 完全定义模型
    ...
    
    // 创建 CUDAEnsemble
    flamegpu::CUDAEnsemble ensemble(model);
    // 处理任何运行时参数
    ensemble.initialise(argc, argv);

  .. code-tab:: python
  
    model = pyflamegpu.ModelDescription("example model")
    
    # 完全定义模型
    ...
    
    # 创建 CUDAEnsemble
    ensemble = pyflamegpu.CUDAEnsemble(model)
    # 处理任何运行时参数
    ensemble.initialise(sys.argv)


创建 RunPlanVector
------------------------

:class:`RunPlanVector<flamegpu::RunPlanVector>`是一种数据结构，可用于构建运行配置，比如模拟速度、步骤和初始化环境属性。 这些是``std::vector`` 的:class:`RunPlan<flamegpu::RunPlan>` （在前一章中介绍过:ref:`previous chapter<RunPlan>`），其中包含一些附加方法，可以轻松配置批量运行。

对向量执行的操作将应用于所有元素，而单个元素也可以直接更新。 

还可以为要发送到的特定运行的日志输出指定子目录，这在构建大批量运行或参数扫描时非常有用：


.. tabs::

  .. code-tab:: cpp C++
  
    // 创建模板运行计划
    flamegpu::RunPlanVector runs_control(model, 128);
    // 确保重复运行在运行计划中使用相同的随机值
    runs_control.setRandomPropertySeed(34523);
    {  // 初始化整个向量的值
        // 所有跑步都需要 3600 步
        runs_control.setSteps(3600);
        // 每次运行的随机种子应采用值（12、13、14、15 等）
        runs_control.setRandomSimulationSeed(12, 1);
        // 使用 1 到 128 之间均匀分布的值初始化环境属性“lerp_float” 
        runs_control.setPropertyLerpRange<float>("lerp_float", 1.0f, 128.0f);
        
        // 使用均匀分布在 [0, 10] 范围内的值初始化环境属性“random_int” 
        runs_control.setPropertyUniformRandom<int>("random_int", 0, 10);
        // 使用正常分布中的值初始化环境属性“random_float”（平均值：1，stddev：2）
        runs_control.setPropertyNormalRandom<float>("random_float", 1.0f, 2.0f);
        // 使用日志正态分布中的值初始化环境属性“random_double”（平均值：2，stddev：1）
        runs_control.setPropertyLogNormalRandom<double>("random_double", 2.0, 1.0);
        
        // 使用 [1, 3, 5] 初始化环境属性数组 'int_array_3'
        runs_control.setProperty<int, 3>("int_array_3", {1, 3, 5});
        
        // 迭代向量以手动分配属性
        for (RunPlan &plan:runs_control) {
            // 例如 手动将所有“manual_float”设置为 32
            plan.setProperty<float>("manual_float", 32.0f);
        }        
    }
    // 创建一个空的 RunPlanVector，我们将通过多次变异和复制 running_control 来构造它
    flamegpu::RunPlanVector runs(model, 0);
    for (const float &mutation : {0.2f, 0.5f, 0.8f, 1.5f, 1.9f, 2.5f}) {
        // 动态生成突变子目录的名称 char subdir[24]；
        sprintf(subdir, "mutation_%g", mutation);
        runs_control.setOutputSubdirectory(subdir);
        // 填写专门参数
        runs_control.setProperty<float>("mutation", mutation);                    
        // 附加到主运行计划向量
        runs += runs_control;
    }

  .. code-tab:: py Python
  
    # 创建模板运行计划
    runs_control = pyflamegpu.RunPlanVector(model, 128)
    # 确保重复运行在运行计划中使用相同的随机值
    runs_control.setRandomPropertySeed(34523)
    # 初始化整个向量的值
    # 所有跑步都需要 3600 步
    runs_control.setSteps(3600)
    # 每次运行的随机种子应采用值（12、13、14、15 等）
    runs_control.setRandomSimulationSeed(12, 1)
    # 使用 1 到 128 之间均匀分布的值初始化环境属性“lerp_float”
    runs_control.setPropertyLerpRangeFloat("lerp_float", 1.0, 128.0)
    
    # 使用均匀分布在 [0, 10] 范围内的值初始化环境属性“random_int” 
    runs_control.setPropertyUniformRandomInt("random_int", 0, 10)
    # 使用正常分布中的值初始化环境属性“random_float”（平均值：1，stddev：2）
    runs_control.setPropertyNormalRandomFloat("random_float", 1.0, 2.0)
    # 使用日志正态分布中的值初始化环境属性“random_double”（平均值：2，stddev：1）
    runs_control.setPropertyLogNormalRandomDouble("random_double", 2.0, 1.0)
    
    #  使用 [1, 3, 5] 初始化环境属性数组 'int_array_3'
    runs_control.setPropertyArrayInt("int_array_3", (1, 3, 5))
    
    # 迭代向量以手动分配属性
    for plan in runs_control:
        # 例如 手动将所有“manual_float”设置为 32
        plan.setPropertyFloat("manual_float", 32.0)
  
    # 创建一个空的 RunPlanVector，我们将通过多次变异和复制 running_control 来构造它
    runs = pyflamegpu.RunPlanVector(model, 0)
    for mutation in [0.2, 0.5, 0.8, 1.5, 1.9, 2.5]:
        # 动态生成突变子目录的名称
        runs_control.setOutputSubdirectory("mutation_%g"%(mutation))
        # 填写专门参数
        runs_control.setPropertyFloat("mutation", mutation)
        # 附加到主运行计划向量
        runs += runs_control
    
创建日志记录配置
--------------------------------
:class:`CUDAEnsemble<flamegpu::CUDAEnsemble>`.

接下来，您需要决定将收集哪些数据，因为不可能从:ref:`previous chapter<Configuring Data to be Logged>`导出完整的代理状态。

下面显示了一个简短的示例，但是您应该参考前一章以获得全面的指南。

使用:class:`CUDAEnsemble<flamegpu::CUDAEnsemble>` 进行实验的好处之一是，特定的:class:`RunPlan<flamegpu::RunPlan>`数据包含在每个日志文件中，允许自动处理它们并用于可重复的研究。 但是，这并不能识别模型的特定版本或版本。

.. tabs::

  .. code-tab:: cpp C++
  
    // 指定所需的 LoggingConfig 或 StepLoggingConfig
    flamegpu::StepLoggingConfig step_log_cfg(model);
    {
        // 记录每个步骤（不适用于 LoggingConfig，用于退出日志）
        step_log_cfg.setFrequency(1);
        step_log_cfg.logEnvironment("random_float");
        step_log_cfg.agent("boid").logCount();
        step_log_cfg.agent("boid").logMean<float>("speed");
    }
    flamegpu::LoggingConfig exit_log_cfg(model);
    exit_log_cfg.logEnvironment("lerp_float");
    
    // 
    cuda_ensemble.setStepLog(step_log_cfg);
    cuda_ensemble.setExitLog(exit_log_cfg);

  .. code-tab:: py Python
  
    # 指定所需的 LoggingConfig 或 StepLoggingConfig
    step_log_cfg = pyflamegpu.StepLoggingConfig(model);

    # 记录每个步骤（不适用于 LoggingConfig，用于退出日志）
    step_log_cfg.setFrequency(1);
    step_log_cfg.logEnvironment("random_float");
    step_log_cfg.agent("boid").logCount();
    step_log_cfg.agent("boid").logMeanFloat("speed");

    exit_log_cfg = pyflamegpu.LoggingConfig (model)
    exit_log_cfg.logEnvironment("lerp_float")
    
    # 将日志配置传递给 CUDAEnsemble
    cuda_ensemble.setStepLog(step_log_cfg)
    cuda_ensemble.setExitLog(exit_log_cfg)
    
配置和运行 Ensemble
----------------------------------

Now you can execute the :class:`CUDAEnsemble<flamegpu::CUDAEnsemble>` from the command line, using the below parameters, it will execute the runs and log the collected data to file.

============================== =========================== ========================================================
完整的参数                      缩写的参数                   描述
============================== =========================== ========================================================
``--help``                     ``-h``                      打印命令行指南并退出。
``--devices`` <device ids>     ``-d`` <device ids>         用于执行集成的 GPU ID 的逗号分隔列表。
                                                           默认情况下将使用所有设备。
``--concurrent`` <runs>        ``-c`` <runs>               每个 GPU 运行的并发模拟数量。
                                                           默认情况下，每个 GPU 将运行 4 个并发模拟。
``--out`` <directory> <format> ``-o`` <directory> <format> 用于集成日志记录的目录和格式 (JSON/XML)。
``--quiet``                    ``-q``                      不要将集成进度打印到控制台。
``--verbose``                  ``-v``                      将配置、进度和计时 (-t) 信息打印到控制台。
``--timing``                   ``-t``                      在退出时将计时信息输出到控制台。
``--silence-unknown-args``     ``-u``                      在此标志之后传递的未知参数的静默警告。
``--error``                    ``-e`` <error level>        :enum:`ErrorLevel<flamegpu::CUDAEnsemble::EnsembleConfig::ErrorLevel>` 用于: 0, 1, 2, "off", "slow" or "fast".
                                                           默认情况下 :enum:`ErrorLevel<flamegpu::CUDAEnsemble::EnsembleConfig::ErrorLevel>` 设置为 "slow" (1).
``--standby``                                              允许操作系统在整体执行期间进入待机状态。
                                                           待机阻止功能目前仅在 Windows 上受支持，并且默认情况下处于启用状态。
============================== =========================== ========================================================

您可能还希望通过在调用:func:`initialise()<flamegpu::CUDAEnsemble::initialise>`之前设置值来指定自己的默认值：

.. tabs::

  .. code-tab:: cpp C++
  
    // 完全声明 ModelDescription、RunPlanVector 和 LoggingConfig/StepLoggingConfig
    ...
    
    // 创建 CUDAEnsemble 来运行 RunPlanVector
    flamegpu::CUDAEnsemble ensemble(model);
    
    // 覆盖配置默认值
    ensemble.Config().out_directory = "results";
    ensemble.Config().out_format = "json";
    ensemble.Config().concurrent_runs = 1;
    ensemble.Config().timing = true;
    ensemble.Config().error_level = CUDAEnsemble::EnsembleConfig::Fast;
    ensemble.Config().devices = {0};
    
    // 处理任何运行时参数
    // 如果在覆盖默认值之前执行此操作，则命令行将忽略覆盖的参数
    ensemble.initialise(argc, argv);
    
    // 将日志配置传递给 CUDAEnsemble
    cuda_ensemble.setStepLog(step_log_cfg);
    cuda_ensemble.setExitLog(exit_log_cfg);
    
    // 使用指定的 RunPlan 执行集成
    const unsigned int errs = ensemble.simulate(runs);

  .. code-tab:: py Python
    
    # 完全声明 ModelDescription、RunPlanVector 和 LoggingConfig/StepLoggingConfig
    ...
    
    # 创建 CUDAEnsemble 来运行 RunPlanVector
    ensemble = pyflamegpu.CUDAEnsemble(model);
    
    # 覆盖配置默认值
    ensemble.Config().out_directory = "results"
    ensemble.Config().out_format = "json"
    ensemble.Config().concurrent_runs = 1
    ensemble.Config().timing = True
    ensemble.Config().error_level = pyflamegpu.CUDAEnsembleConfig.Fast
    ensemble.Config().devices = pyflamegpu.IntSet([0])
    
    # 处理任何运行时参数
    # 如果在覆盖默认值之前执行此操作，则命令行将忽略覆盖的参数
    ensemble.initialise(sys.argv)
    
    # 将日志配置传递给 CUDAEnsemble
    cuda_ensemble.setStepLog(step_log_cfg)
    cuda_ensemble.setExitLog(exit_log_cfg)
    
    # 使用指定的 RunPlan 执行集成
    errs = ensemble.simulate(runs)
    
集成内的错误处理
-------------------------------

:class:`CUDAEnsemble<flamegpu::CUDAEnsemble>` 有三个错误处理级别。

====== ===== ==========================================================================================================
层级   名称   描述
====== ===== ==========================================================================================================
0      Off   失败的运行不会导致引发异常。
1      Slow  如果任何运行失败，则在尝试所有运行后将引发:class:`EnsembleError<flamegpu::exception::EnsembleError>`。
2      Fast  一旦检测到失败的运行，就会引发:class:`EnsembleError<flamegpu::exception::EnsembleError>`，并取消剩余的运行。
====== ===== ==========================================================================================================

默认错误级别为“慢”(1)，如果任何模拟无法完成，这将导致引发异常。 但是，将首先尝试所有模拟，因此将提供部分结果。

或者，当错误级别设置为“关闭”(0) 时，对 :func:`simulate()<flamegpu::CUDAEnsemble::simulate>`的调用会返回错误数。 因此，可以通过检查simulate()的返回值不等于0来手动探测失败的运行。

  
相关链接
-------------
* 用户指南: :ref:`Overriding the Initial Environment<RunPlan>` (:class:`RunPlan<flamegpu::RunPlan>` 指南)
* 用户指南: :ref:`Configuring Data to be Logged<Configuring Data to be Logged>`
* 完整的 API 文档 :class:`CUDAEnsemble<flamegpu::CUDAEnsemble>`
* 完整的 API 文档 :class:`CUDAEnsemble::EnsembleConfig<flamegpu::CUDAEnsemble::EnsembleConfig>`
* 完整的 API 文档 :class:`CUDASimulation<flamegpu::CUDASimulation>`
* 完整的 API 文档 :class:`RunPlanVector<flamegpu::RunPlanVector>`
* 完整的 API 文档 :class:`RunPlan<flamegpu::RunPlan>`
* 完整的 API 文档 :class:`LoggingConfig<flamegpu::LoggingConfig>`
* 完整的 API 文档 :class:`AgentLoggingConfig<flamegpu::AgentLoggingConfig>`
* 完整的 API 文档 :class:`StepLoggingConfig<flamegpu::StepLoggingConfig>`
