覆盖初始状态
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

运行FLAME GPU模型时，通常希望覆盖部分初始环境或提供预定义的代理群体。

With Code
---------

在代码中覆盖初始状态比从状态文件中覆盖初始状态更灵活，从而允许动态控制初始状态。 例如，它可以用来执行模型作为遗传算法的一部分。

.. _RunPlan:

覆盖初始环境
==================================

覆盖模型初始状态的推荐方法是使用:class:`RunPlan<flamegpu::RunPlan>`。

该类允许您配置环境属性用以覆盖默认的环境设置。 然后将覆盖后的设置传递给:func:`simulate()<void flamegpu::CUDASimulation::simulate(const RunPlan &)>`，用以在模型中运行。

:class:`RunPlan<flamegpu::RunPlan>` 提供 :func:`setProperty()<flamegpu::RunPlan::setProperty>` 与FLAME GPU中其他地方相同格式的，来覆盖设置。
    
此外 :class:`RunPlan<flamegpu::RunPlan>` 保存了用于模拟的步骤数和随机种子，它们可以替换通过命令行或者手动设置等方式设置的任何配置，因此通常应分别用:func:`setRandomSimulationSeed()<flamegpu::RunPlan::setRandomSimulationSeed>` 和 :func:`setSteps()<flamegpu::RunPlan::setSteps>` 进行配置。
    
.. tabs::

  .. code-tab:: cpp C++
  
    // 创建一个 RunPlan 来覆盖初始环境
    flamegpu::RunPlan plan(model);
    plan.setRandomSimulationSeed(123456);
    plan.setSteps(3600);
    plan.setProperty<float>("speed", 12.0f);
    plan.setProperty<float, 3>("origin", {5.0f, 0.0f, 5.0f});
  
    // 从模型创建模拟对象
    flamegpu::CUDASimulation simulation(model);

    // 配置模拟
    ...
    
    // 使用 RunPlan 中的覆盖运行模拟
    simulation.simulate(plan);

  .. code-tab:: py Python

    # 创建一个 RunPlan 来覆盖初始环境
    plan = pyflamegpu.RunPlan(model)
    plan.setRandomSimulationSeed(123456)
    plan.setSteps(3600)
    plan.setPropertyFloat("speed", 12.0)
    plan.setPropertyArrayFloat("origin", 3, [5.0, 0.0, 5.0])
    
    # 从模型创建模拟对象
    simulation = pyflamegpu.CUDASimulation(model)

    # 配置模拟
    ...

    # 使用 RunPlan 中的覆盖运行模拟
    simulation.simulate(plan)
    
.. note::

  Python 方法 ``RunPlan::setPropertyArray()`` 目前需要数组长度的第二个参数，这与其他用途不一致。 `(issue) <https://github.com/FLAMEGPU/FLAMEGPU2/issues/831>`_
    
替代技术
~~~~~~~~~~~~~~~~~~~

您还可以通过直接在:class:`CUDASimulation<flamegpu::CUDASimulation>` 实例上调用 :func:`setEnvironmentProperty()<flamegpu::CUDASimulation::setEnvironmentProperty>` 来直接覆盖环境属性的值。 同样，这些方法与:class:`RunPlan<flamegpu::RunPlan>`, :class:`HostEnvironment<flamegpu::HostEnvironment>` 和其他地方的 ``setProperty()``具有相同的用法.

这允许比 :class:`RunPlan<flamegpu::CUDASimulation>`，更细粒度的控制，因为可以随时调用它来修改当前模拟状态（例如，如果手动步进模型，您可以在步骤之间调用它）。

.. tabs::

  .. code-tab:: cpp C++
  
    // 从模型创建模拟对象
    flamegpu::CUDASimulation simulation(model);
    
    // 覆盖一些环境属性
    simulation.setEnvironmentProperty<float>("speed", 12.0f);
    simulation.setEnvironmentProperty<float, 3>("origin", {5.0f, 0.0f, 5.0f});

    // 配置模拟的其余部分
    ...
    
    // 使用 RunPlan 中的覆盖运行模拟
    simulation.simulate(plan);

  .. code-tab:: py Python
    
    # 从模型创建模拟对象
    simulation = pyflamegpu.CUDASimulation(model)

    # 创建一个 RunPlan 来覆盖初始环境
    simulation.setEnvironmentPropertyFloat("speed", 12.0)
    simulation.setEnvironmentPropertyArrayFloat("origin", [5.0, 0.0, 5.0])
    
    # 配置模拟的其余部分
    ...

    # 使用 RunPlan 中的覆盖运行模拟
    simulation.simulate(plan)


设置初始代理数量
=================================

如果您无法在初始化函数中生成代理群体（如主机代理创建中:ref:`Host Agent Creation<Host Agent Creation>`所述），您可以为每个代理状态群体创建一个:class:`AgentVector<flamegpu::AgentVector>` ，并将它们传递给:class:`CUDASimulation<flamegpu::CUDASimulation>`。

:class:`AgentVector<flamegpu::AgentVector>` 是通过向其构造函数传递 :class:`AgentDescription<flamegpu::AgentDescription>` 以及可选的向量初始大小来创建的，这将创建指定数量的默认初始化代理。

:class:`AgentVector<flamegpu::AgentVector>` 接口是根据 C++ 的 ``std::vector`` 建模的，其元素类型为 :class:`AgentVector::Agent<flamegpu::AgentVector_Agent>`。 然而，内部数据以数组结构格式存储。

:class:`AgentVector::Agent<flamegpu::AgentVector_Agent>` 然后具有在库中其他位置找到的标准 :func:`setVariable()<flamegpu::AgentVector_Agent::setVariable>` 和  :func:`getVariable()<flamegpu::AgentVector_CAgent::getVariable>` 方法。

一旦 :class:`AgentVector<flamegpu::AgentVector>`  准备就绪，就可以将其传递给 :class:`CUDASimulation<flamegpu::CUDASimulation>` 上的 :func:`setPopulationData()<flamegpu::CUDASimulation::setPopulationData>`。 如果您使用多个代理状态，则还需要指定所需的代理状态作为第二个参数。


.. tabs::

  .. code-tab:: cpp C++
    
    // 创建 1000 个“Boid”代理群体
    flamegpu::AgentVector population(model.Agent("Boid"), 1000);
    
    // 手动初始化每个代理中的“速度”变量
    for (flamegpu::AgentVector::Agent &instance : population) {
        instance.setVariable<float>("speed", 1.0f);
    }
    
    // 具体设置第12个代理的变量不同
    population[11].setVariable<float>("speed", 0.0f);
    
    // 使用 AgentVector 将“Boid”群体设置为默认状态
    simulation.setPopulationData(population);
    // 使用 AgentVector 将“Boid”群体设置为“健康”状态
    // simulation.setPopulationData(population, "healthy");
  .. code-tab:: py Python
    
    # 创建 1000 个“Boid”代理群体
    population = pyflamegpu.AgentVector(model.Agent("Boid"), 1000)
    
    # 手动初始化每个代理中的“速度”变量
    for instance in population:
        instance.setVariableFloat("speed", 1.0)
        
    # 具体设置第12个代理的变量不同
    population[11].setVariableFloat("speed", 0.0)
    
    # 使用 AgentVector 将“Boid”群体设置为默认状态
    simulation.setPopulationData(population)
    # 使用 AgentVector 将“Boid”群体设置为“健康”状态
    # simulation.setPopulationData(population, "healthy")
        
    
.. _Initial State From File:

从文件
---------

FLAME GPU 2 模拟可以使用 XML 或 JSON 格式从磁盘初始化。 XML 格式与之前的 FLAME GPU 1 输入/输出文件兼容，而 JSON 格式是 FLAME GPU 2 的新格式。在这两种情况下，输入和输出文件格式相同。

从文件加载模拟状态（代理数据和环境属性）可以通过命令行规范或模型代码中的显式规范来实现。 （更多信息请参阅上一节）

在大多数情况下，输入文件将从命令行获取，可以使用 ``-i <input file>``传递。

从磁盘加载文件时，代理 ID 必须是唯一的，否则将引发 ``AgentIDCollision`` 异常。 这必须在输入文件中纠正，因为在运行时 FLAME GPU 中没有方法可以这样做。

在大多数情况下，输入文件的组件是可选的，如果首选默认值，则可以省略。 如果在输入文件中没有为代理分配 ID，则会自动生成它们。

FLAME GPU 生成的模拟状态输出文件兼容用作输入文件。 然而，如果与大量代理群体一起工作，由于其人类可读的格式，它们可能会大得令人望而却步。


文件格式
===========

=================== ============================================================================================
块                  描述
=================== ============================================================================================
``itno``            **XML Only** 该块提供 XML 输出文件中的步骤号，包含它是为了向后兼容 FLAMEGPU 1。它对输入没有用处。
``config``          该块被分成子块 ``simulation`` 和 ``cuda`` ，每个子块的成员分别与 :class:`Simulation::Config<flamegpu::Simulation::Config>` 和 :class:`CUDASimulation::Config<flamegpu::CUDASimulation::Config>` 同名成员对齐。 这些值被输出以记录配置，并且可以选择用于通过输入文件设置配置。（有关每个成员的详细信息，请参阅配置执行指南:ref:`Configuring Execution`） 
``stats``           该块包含 FLAME GPU 2 在执行期间收集的统计信息。 它对输入没有任何目的。
``environment``     该块包含环境成员，可用于通过输入文件配置环境。 以  ``_`` 开头的成员是自动创建的内部属性，可以通过输入文件设置。
``xagent``          **XML Only** ``xagent`` 块表示单个代理，``name`` 和 ``state`` 值必须与加载的模型描述层次结构中的代理状态匹配。 以 ``_ `` 开头的成员是自动创建的内部变量，可以通过输入文件设置。
``agents``          **JSON Only**   在 ``agents`` 块内，可以存在针对每种代理类型的子块，并且在该子块中针对每种状态类型存在子块。 然后，每个状态映射到一个对象数组，其中每个对象由单个代理的变量组成。 以 ``_`` 开头的成员是自动创建的内部变量，可以通过输入文件设置。
=================== ============================================================================================

下面的代码块显示了 FLAME GPU 2 以 XML 和 JSON 格式输出的示例文件，这些文件可以用作输入文件。

.. tabs::                           

  .. code-tab:: xml XML

    <states>
        <itno>100</itno>
        <config>
            <simulation>
                <input_file></input_file>
                <step_log_file></step_log_file>
                <exit_log_file></exit_log_file>
                <common_log_file></common_log_file>
                <truncate_log_files>true</truncate_log_files>
                <random_seed>1643029170</random_seed>
                <steps>1</steps>
                <verbosity>1</verbosity>
                <timing>false</timing>
                <console_mode>false</console_mode>
            </simulation>
            <cuda>
                <device_id>0</device_id>
                <inLayerConcurrency>true</inLayerConcurrency>
            </cuda>
        </config>
        <stats>
            <step_count>100</step_count>
        </stats>
        <environment>
            <repulse>0.05</repulse>
            <_stepCount>1</_stepCount>
        </environment>
        <xagent>
            <name>Circle</name>
            <state>default</state>
            <_auto_sort_bin_index>0</_auto_sort_bin_index>
            <_id>241</_id>
            <drift>0.0</drift>
            <x>0.8293430805206299</x>
            <y>1.5674132108688355</y>
            <z>14.034683227539063</z>
        </xagent>
        <xagent>
            <name>Circle</name>
            <state>default</state>
            <_auto_sort_bin_index>0</_auto_sort_bin_index>
            <_id>242</_id>
            <drift>0.0</drift>
            <x>23.089038848876954</x>
            <y>24.715721130371095</y>
            <z>2.3497250080108644</z>
        </xagent>
    </states>


  .. code-tab:: json JSON
  
    {
      "config": {
        "simulation": {
          "input_file": "",
          "step_log_file": "",
          "exit_log_file": "",
          "common_log_file": "",
          "truncate_log_files": true,
          "random_seed": 1643029117,
          "steps": 1,
          "verbosity": 1,
          "timing": false,
          "console_mode": false
        },
        "cuda": {
          "device_id": 0,
          "inLayerConcurrency": true
        }
      },
      "stats": {
        "step_count": 100
      },
      "environment": {
        "repulse": 0.05,
        "_stepCount": 1
      },
      "agents": {
        "Circle": {
          "default": [
            {
              "_auto_sort_bin_index": 0,
              "_id": 241,
              "drift": 0.0,
              "x": 0.8293430805206299,
              "y": 1.5674132108688355,
              "z": 14.034683227539063
            },
            {
              "_auto_sort_bin_index": 168,
              "_id": 242,
              "drift": 0.0,
              "x": 23.089038848876954,
              "y": 24.715721130371095,
              "z": 2.3497250080108644
            }
          ]
        }
      }
    }


相关链接
-------------
* 用户指南页面: :ref:`Configuring Execution<Configuring Execution>`
* 完整的 API 文档 :class:`RunPlan<flamegpu::RunPlan>`
* 完整的 API 文档 :class:`AgentVector<flamegpu::AgentVector>` (``AgentVector::Agent``)
* 完整的 API 文档 :class:`AgentVector::Agent<flamegpu::AgentVector_Agent>`
* 完整的 API 文档 :class:`AgentVector::CAgent<flamegpu::AgentVector_CAgent>` (Read-only superclass of :class:`AgentVector::Agent<flamegpu::AgentVector_Agent>`)
* 完整的 API 文档 :class:`CUDASimulation<flamegpu::CUDASimulation>`