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


An :class:`AgentVector<flamegpu::AgentVector>` is created by passing it's constructor an :class:`AgentDescription<flamegpu::AgentDescription>` and optionally the initial size of the vector which will create the specified number of default initialised agents.

The interface :class:`AgentVector<flamegpu::AgentVector>` is modelled after C++'s ``std::vector``, with elements of type :class:`AgentVector::Agent<flamegpu::AgentVector_Agent>`. However, internally data is stored in a structure-of-arrays format.  

:class:`AgentVector::Agent<flamegpu::AgentVector_Agent>` then has the standard :func:`setVariable()<flamegpu::AgentVector_Agent::setVariable>` and :func:`getVariable()<flamegpu::AgentVector_CAgent::getVariable>` methods found elsewhere in the library.

Once the :class:`AgentVector<flamegpu::AgentVector>` is ready, it can be passed to :func:`setPopulationData()<flamegpu::CUDASimulation::setPopulationData>` on the :class:`CUDASimulation<flamegpu::CUDASimulation>`. If your are using multiple agent states, it is also necessary to specify the desired agent state as the second argument.

.. tabs::

  .. code-tab:: cpp C++
    
    // Create a population of 1000 'Boid' agents
    flamegpu::AgentVector population(model.Agent("Boid"), 1000);
    
    // Manually initialise the "speed" variable in each agent
    for (flamegpu::AgentVector::Agent &instance : population) {
        instance.setVariable<float>("speed", 1.0f);
    }
    
    // Specifically set the 12th agent's variable differently
    population[11].setVariable<float>("speed", 0.0f);
    
    // Set the "Boid" population in the default state with the AgentVector
    simulation.setPopulationData(population);
    // Set the "Boid" population in the "healthy" state with the AgentVector
    // simulation.setPopulationData(population, "healthy");
  .. code-tab:: py Python
    
    # Create a population of 1000 'Boid' agents
    population = pyflamegpu.AgentVector(model.Agent("Boid"), 1000)
    
    for instance in population:
        instance.setVariableFloat("speed", 1.0)
        
    # Specifically set the 12th agent's variable differently
    population[11].setVariableFloat("speed", 0.0)
    
    # Set the "Boid" population in the default state with the AgentVector
    simulation.setPopulationData(population)
    # Set the "Boid" population in the "healthy" state with the AgentVector
    # simulation.setPopulationData(population, "healthy")
        
    
.. _Initial State From File:

From File
---------

FLAME GPU 2 simulations can be initialised from disk using either the XML or JSON format. The XML format is compatible with the previous FLAME GPU 1 input/output files, whereas the JSON format is new to FLAME GPU 2. In both cases, the input and output file formats are the same.

Loading simulation state (agent data and environment properties) from file can be achieved via either command line specification, or explicit specification within the code for the model. (See the :ref:`previous section<Configuring Execution>` for more information)

In most cases, the input file will be taken from command line which can be passed using ``-i <input file>``.

Agent IDs must be unique when the file is loaded from disk, otherwise an ``AgentIDCollision`` exception will be thrown. This must be corrected in the input file, as there is no method to do so within FLAME GPU at runtime.

In most cases, components of the input file are optional and can be omitted if defaults are preferred. If agents are not assigned IDs within the input file, they will be automatically generated.

Simulation state output files produces by FLAME GPU are compatible for use as input files. However, if working with large agent populations they are likely to be prohibitively large due to their human-readable format.


File Format
===========

=================== ============================================================================================
Block               Description
=================== ============================================================================================
``itno``            **XML Only** This block provides the step number in XML output files, it is included for backwards compatibility with FLAMEGPU 1. It has no use for input.
``config``          This block is split into sub-blocks ``simulation`` and ``cuda``, the members of each sub-block align with :class:`Simulation::Config<flamegpu::Simulation::Config>` and :class:`CUDASimulation::Config<flamegpu::CUDASimulation::Config>` members of the same name respectively. These values are output to log the configuration, and can optionally be used to set the configuration via input file. (See the :ref:`Configuring Execution` guide for details of each individual member)
``stats``           This block includes statistics collected by FLAME GPU 2 during execution. It has no purpose on input.
``environment``     This block includes members of the environment, and can be used to configure the environment via input file. Members which begin with ``_`` are automatically created internal properties, which can be set via input file.
``xagent``          **XML Only** Each ``xagent`` block represents a single agent, and the ``name`` and ``state`` values must match an agent state within the loaded model description hierarchy. Members which begin with ``_`` are automatically created internal variables, which can be set via input file.
``agents``          **JSON Only** Within the ``agents`` block, a sub block may exist for each agent type, and within this a sub-block for each state type. Each state then maps to an array of object, where each object consists of a single agent's variables. Members which begin with ``_`` are automatically created internal variables, which can be set via input file.
=================== ============================================================================================

The below code block displays example files output from FLAME GPU 2 in both XML and JSON formats, which could be used as input files.

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


Related Links
-------------
* User Guide Page: :ref:`Configuring Execution<Configuring Execution>`
* Full API documentation for :class:`RunPlan<flamegpu::RunPlan>`
* Full API documentation for :class:`AgentVector<flamegpu::AgentVector>` (``AgentVector::Agent``)
* Full API documentation for :class:`AgentVector::Agent<flamegpu::AgentVector_Agent>`
* Full API documentation for :class:`AgentVector::CAgent<flamegpu::AgentVector_CAgent>` (Read-only superclass of :class:`AgentVector::Agent<flamegpu::AgentVector_Agent>`)
* Full API documentation for :class:`CUDASimulation<flamegpu::CUDASimulation>`