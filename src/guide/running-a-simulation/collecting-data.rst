.. _Collecting Data:

收集数据
^^^^^^^^^^^^^^^

模拟完成后，您可能想要取回一些数据。 FLAME GPU 提供了两种实现此目的的方法：记录日志，可以将大型模拟状态减少到最重要的数据点，以及导出整个模拟状态。

记录
-------

由于模拟可能涉及数千到数百万个代理，因此记录完整的模拟状态将产生大量输出，其中大部分是不需要的。 FLAME GPU提供了指定在每一步模拟时或在模拟结束时记录的群体级统计归约和环境属性集合的功能。

.. _Configuring Data to be Logged:

配置要记录的数据
=============================

要定义日志记录配置，您应该创建:class:`LoggingConfig<flamegpu::LoggingConfig>`或  :class:`StepLoggingConfig<flamegpu::StepLoggingConfig>`，并将 :class:`ModelDescription<flamegpu::ModelDescription>`传递给构造函数。 这两个类共享相同的接口来指定要记录的数据，但是:class:`StepLoggingConfig<flamegpu::StepLoggingConfig>` 还允许您调用 :func:`setFrequency()<flamegpu::StepLoggingConfig::setFrequency>`来调整记录步骤的频率（默认为 1，每个步骤）。

环境属性通过:func:`logEnvironment()<flamegpu::LoggingConfig::logEnvironment>`进行记录，并指定属性的名称。 与大多数 FLAME GPU 变量和属性方法不同，这里不需要指定类型。

代理数据根据代理状态进行记录，因此具有多个状态的代理必须为需要记录的每个状态指定配置。 可以使用 :func:`logCount()<flamegpu::AgentLoggingConfig::logCount>` 记录处于该状态的代理总数。 或者可以记录变量减少；  :func:`logMean()<flamegpu::AgentLoggingConfig::logMean>`、:func:`logStandardDev()<flamegpu::AgentLoggingConfig::logStandardDev>`、:func:`logMin()<flamegpu::AgentLoggingConfig::logMin>`、:func:`logMax()<flamegpu::AgentLoggingConfig::logMax>`、:func:`logSum()<flamegpu::AgentLoggingConfig::logSum>`指定变量的名称和类型（与整个 API 中使用的形式相同）。

设置完成后，日志记录配置将使用:func:`setStepLog()<flamegpu::CUDASimulation::setStepLog>`  和  :func:`setExitLog()<flamegpu::CUDASimulation::setExitLog>`传递到 :class:`CUDASimulation<flamegpu::CUDASimulation>`。

.. tabs::

  .. code-tab:: cpp C++
    
    
    flamegpu::ModelDescription model("example model");
    
    // 完全定义模型
    ... 
    
    // 指定所需的 LoggingConfig 或 StepLoggingConfig
    flamegpu::StepLoggingConfig step_log_cfg(model);
    {
        // 记录每个步骤（不适用于 LoggingConfig，用于退出日志）
        step_log_cfg.setFrequency(1);
        // 在记录的数据中包含环境属性“env_prop”
        step_log_cfg.logEnvironment("env_prop");
        // 包括默认状态下“boid”代理的当前数量
        step_log_cfg.agent("boid").logCount();
        // 包括当前处于“活动”状态的“boid”代理数量
        step_log_cfg.agent("boid", "alive").logCount();
        // 包括默认状态下 boid 代理群体的变量“速度”的平均值
        step_log_cfg.agent("boid").logMean<float>("speed");
        // 在默认状态下包含 boid 代理群体变量“速度”的标准差
        step_log_cfg.agent("boid").logStandardDev<float>("speed");
        // 包括默认状态下 boid 代理群体变量“速度”的最小值和最大值
        step_log_cfg.agent("boid").logMin<float>("speed");
        step_log_cfg.agent("boid").logMax<float>("speed");
        // 包括“活跃”状态下 boid 代理群体的变量“健康”总和
        step_log_cfg.agent("boid", "alive").logSum<int>("health");
    }
    
    // 创建 CUDASimulation 实例
    flamegpu::CUDASimulation cuda_sim(model);
    
    // 附加日志记录配置
    cuda_sim.setStepLog(step_log_cfg);
    // cuda_sim.setExitLog(exit_log_cfg);
    
    // 正常运行模拟
    cuda_sim.simulate();

  .. code-tab:: py Python    
    
    model = pyflamegpu.ModelDescription("example model")
    
    # 完全定义模型
    ...
    
    # 指定所需的 LoggingConfig 或 StepLoggingConfig
    step_log_cfg = flamegpu.StepLoggingConfig(model)
    # 记录每个步骤（不适用于 LoggingConfig，用于退出日志）
    step_log_cfg.setFrequency(1)
    # 在记录的数据中包含环境属性“env_prop”
    step_log_cfg.logEnvironment("env_prop")
    # 包括默认状态下“boid”代理的当前数量
    step_log_cfg.agent("boid").logCount()
    # 包括当前处于“活动”状态的“boid”代理数量
    step_log_cfg.agent("boid", "alive").logCount()
    # 包括默认状态下 boid 代理群体的变量“速度”的平均值
    step_log_cfg.agent("boid").logMeanFloat("speed")
    # 在默认状态下包含 boid 代理群体变量“速度”的标准差
    step_log_cfg.agent("boid").logStandardDevFloat("speed")
    # 包括默认状态下 boid 代理群体变量“速度”的最小值和最大值
    step_log_cfg.agent("boid").logMinFloat("speed")
    step_log_cfg.agent("boid").logMaxFloat("speed")
    # 包括“活跃”状态下 boid 代理群体的变量“健康”总和
    step_log_cfg.agent("boid", "alive").logSumInt("health")
    
    # 创建 CUDASimulation 实例
    cuda_sim = flamegpu.CUDASimulation(model)
    
    # 附加日志记录配置
    cuda_sim.setStepLog(step_log_cfg)
    # cuda_sim.setExitLog(exit_log_cfg)
    
    # 正常运行模拟
    cuda_sim.simulate()


访问收集的数据
========================

将 :class:`CUDASimulation<flamegpu::CUDASimulation>` 配置为使用特定的日志记录配置并执行模拟后，可以使用 :func:`getRunLog()<flamegpu::Simulation::getRunLog>` 通过代码访问日志。 这将返回一个 :class:`RunLog<flamegpu::RunLog>`，其中包含所请求的步骤和退出日志数据。

性能数据始终附加到请求的日志中，因此可以根据需要进行访问。

.. tabs::

  .. code-tab:: cpp C++
    
    // 附加日志记录配置
    cuda_sim.setStepLog(step_log_cfg);
    cuda_sim.setExitLog(exit_log_cfg);
    
    // 正常运行模拟
    cuda_sim.simulate();
    
    // 获取记录的数据
    flamegpu::RunLog run_log = cuda_sim.getRunLog();
    
    // 获取使用的随机种子
    uint64_t rng_seed = run_log.getRandomSeed();
    // 获取步数记录频率
    unsigned int step_log_freqency = run_log.getStepLogFrequency();
    
    // 访问步骤和退出日志数据
    // 如果未指定相应的日志记录配置，则步骤和退出日志将为空。
    flamegpu::LogFrame exit_log = run_log.getExitLog();
    std::list<flamegpu::LogFrame> step_log = run_log.getStepLog();
    
    // 迭代步骤日志并将一些信息打印到控制台
    for (auto &log:step_log) {
        // 获取步骤索引
        unsigned int step_count = log.getStepCount();
        // 获取记录的环境属性
        int env_prop = log.getEnvironmentProperty<int>("env_prop");
        // 从默认状态获取记录的 boid 代理属性减少数据
        unsigned int agent_count = log.getAgent("boid").getCount();
        // Reduce 运算符将返回类型向上转换为最合适的类型，以免丢失数据
        double agent_speed_mean = log.getAgent("boid").getMean("speed");
        // 将数据打印到控制台
        printf("#%u: %u, %f\n", step+count, agent_count, agent_speed_mean);
    }

  .. code-tab:: py Python
  
    # 附加日志记录配置
    cuda_sim.setStepLog(step_log_cfg)
    cuda_sim.setExitLog(exit_log_cfg)
    
    # 正常运行模拟
    cuda_sim.simulate()
    
    # 获取记录的数据
    run_log = cuda_sim.getRunLog();
    
    # 获取使用的随机种子
    rng_seed = run_log.getRandomSeed();
    # 获取步数记录频率
    step_log_freqency = run_log.getStepLogFrequency();
    
    # 访问步骤和退出日志数据
    # 如果未指定相应的日志记录配置，则步骤和退出日志将为空。
    exit_log = run_log.getExitLog();
    step_log = run_log.getStepLog();
    
    # 迭代步骤日志并将一些信息打印到控制台
    for log in step_log:
        # 获取步骤索引
        unsigned int step_count = log.getStepCount();
        # 获取记录的环境属性
        int env_prop = log.getEnvironmentPropertyInt("env_prop")
        # 从默认状态获取记录的 boid 代理属性减少数据
        unsigned int agent_count = log.getAgent("boid").getCount()
        # Reduce 运算符将返回类型向上转换为最合适的类型，以免丢失数据
        double agent_speed_mean = log.getAgent("boid").getMean("speed")
        # 将数据打印到控制台
        print("#%u: %u, %f"%(step+count, agent_count, agent_speed_mean))
        

将收集的数据写入文件
==============================

您可以将其存储到文件中以便稍后进行后处理，而不是在运行时处理记录的数据。

通常，您可以通过:ref:`earlier section<Configuring Execution>` 来处理此问题，如前面部分所述。 但是，您也可以在 :class:`CUDASimulation<flamegpu::CUDASimulation>`上调用 :func:`exportLog()<flamegpu::Simulation::exportLog>`来手动触发导出。

.. tabs::

  .. code-tab:: cpp C++
    
    // 附加日志记录配置
    cuda_sim.setStepLog(step_log_cfg);
    cuda_sim.setExitLog(exit_log_cfg);
    
    // 正常运行模拟
    cuda_sim.simulate();
    
    // 将记录的数据导出到文件
    cuda_sim.exportLog(
      "log.json", // 要输出的文件（必须以“.json”或“.xml”结尾）
      true,       // 步骤日志是否应包含在日志文件中
      true,       // 退出日志是否应包含在日志文件中
      true,       // 日志文件中是否应包含步骤时间（如果不包含步骤日志，则视为 false）
      true,       // 是否应将模拟时间包含在日志文件中（如果不包含退出日志则视为 false）
      false       // 是否应缩小日志文件
    );

  .. code-tab:: py Python
  
    # 附加日志记录配置
    cuda_sim.setStepLog(step_log_cfg)
    cuda_sim.setExitLog(exit_log_cfg)
    
    # 正常运行模拟
    cuda_sim.simulate()
        
    # 将记录的数据导出到文件
    cuda_sim.exportLog(
      "log.json", # 要输出的文件（必须以“.json”或“.xml”结尾）
      True,       # 步骤日志是否应包含在日志文件中
      True,       # 退出日志是否应包含在日志文件中
      True,       # 日志文件中是否应包含步骤时间（如果不包含步骤日志，则视为 false）
      True,       # 是否应将模拟时间包含在日志文件中（如果不包含退出日志则视为 false）
      False)      # 是否应缩小日志文件
  

访问完整的代理状态
----------------------------------

在某些有限的情况下，您可能希望直接访问完整的代理群体。 这只能通过代码来实现，可以通过直接访问代理数据或手动触发导出到文件。


与指定初始代理群体类似，您可以将代理状态群体获取到 :class:`AgentVector<flamegpu::AgentVector>`.

.. tabs::

  .. code-tab:: cpp C++
  
    flamegpu::ModelDescription model("example model");
    flamegpu::AgentDescription boid_agent = model.newAgent("boid");
    
    // 完全定义模型并设置 CUDASimulation
    ...
    
    // 正常运行模拟
    // step() 还可以用于在每个步骤的基础上访问代理状态
    cuda_sim.simulate();
    
    // 将 boid 代理数据从默认状态复制到代理向量
    flamegpu::AgentVector out_pop(boid_agent);
    cuda_sim.getPopulationData(out_pop);
    
    // 迭代代理并打印它们的速度
    for (flamegpu::AgentVector::Agent &boid : out_pop) {
        printf("Speed: %f\n", boid.getVariable<float>("speed"));
    }
    
  .. code-tab:: py Python
  
    model = pyflamegpu.ModelDescription("example model");
    boid_agent = model.newAgent("boid");
    
    # 完全定义模型并设置 CUDASimulation
    ... 
    
    # 正常运行模拟
    # step() 还可以用于在每个步骤的基础上访问代理状态
    cuda_sim.simulate()
    
    # 将 boid 代理数据从默认状态复制到代理向量
    out_pop = pyflamegpu.AgentVector(boid_agent)
    cuda_sim.getPopulationData(out_pop)
    
    # 迭代代理并打印它们的速度
    for boid in out_pop:
        print("Speed: %f"%(boid.getVariableFloat("speed"))

或者，可以调用:func:`exportData()<flamegpu::Simulation::exportData>`将完整的模拟状态导出到文件（所有代理变量和环境属性）。

.. tabs::

  .. code-tab:: cpp C++
  
    flamegpu::ModelDescription model("example model");
    flamegpu::AgentDescription boid_agent = model.newAgent("boid");
    
    // 完全定义模型并设置 CUDASimulation
    ...
    
    // 正常运行模拟
    // step() 还可以用于在每个步骤的基础上访问代理状态
    cuda_sim.simulate();
    
    // Log the simulation state to JSON/XML file
    cuda_sim.exportData("end.json");
    
  .. code-tab:: py Python
  
    model = pyflamegpu.ModelDescription("example model");
    boid_agent = model.newAgent("boid");
    
    // 完全定义模型并设置 CUDASimulation
    ...
    
    # 正常运行模拟
    # step() 还可以用于在每个步骤的基础上访问代理状态
    cuda_sim.simulate()
    
    # Log the simulation state to JSON/XML file
    cuda_sim.exportData("end.json")

附加说明
----------------

在撰写本文时，无法记录或导出环境宏属性，这样做需要通过 init、step 或 exit 函数手动输出它们。


相关链接
-------------
* 用户指南页面: :ref:`Configuring Execution<Configuring Execution>`
* 完整的 API 文档 :class:`LoggingConfig<flamegpu::LoggingConfig>`
* 完整的 API 文档 :class:`AgentLoggingConfig<flamegpu::AgentLoggingConfig>`
* 完整的 API 文档 :class:`StepLoggingConfig<flamegpu::StepLoggingConfig>`
* 完整的 API 文档 :class:`RunLog<flamegpu::RunLog>`
* 完整的 API 文档 :class:`AgentVector<flamegpu::AgentVector>`
* 完整的 API 文档 :class:`AgentVector::Agent<flamegpu::AgentVector_Agent>`
* 完整的 API 文档 :class:`AgentVector::CAgent<flamegpu::AgentVector_CAgent>` (Read-only superclass of :class:`AgentVector::Agent<flamegpu::AgentVector_Agent>`)
* 完整的 API 文档 :class:`CUDASimulation<flamegpu::CUDASimulation>`
* 完整的 API 文档 :class:`Simulation<flamegpu::Simulation>`
* 完整的 API 文档 :class:`Simulation::Config<flamegpu::Simulation::Config>`