.. _Running a Simulation:

运行模拟
====================
一旦您有了完整描述的模型，就可以创建并执行模拟了。

为了执行模型，必须通过传递完整的模型描述来创建 :class:`CUDASimulation<flamegpu::CUDASimulation>` 。 然后，:class:`CUDASimulation<flamegpu::CUDASimulation>`  创建自己的:class:`ModelDescription<flamegpu::ModelDescription>`副本，因此进一步的更改不会影响模拟。

然后可以通过调用:func:`simulate()<flamegpu::CUDASimulation::simulate>`来执行模拟：


.. tabs::

  .. code-tab:: cpp C++
  
    // 完全声明模型
    flamegpu::ModelDescription model("example model");
    ...
     
    // 从模型创建模拟对象
    flamegpu::CUDASimulation simulation(model);

    // 配置模拟
    ...
    
    // 运行模拟
    simulation.simulate();

  .. code-tab:: py Python

    # 完全声明模型
    model = pyflamegpu.ModelDescription("example model")
    ...
    
    # 从模型创建模拟对象
    simulation = pyflamegpu.CUDASimulation(model)

    # 配置模拟
    ...

    # 运行模拟
    simulation.simulate()

这是最简单的情况，但通常需要配置模拟，选择模拟选项，覆盖初始状态并指定要收集的数据。

本章分为几个部分，详细介绍了这些功能：

.. toctree::
   :maxdepth: 1
   
   configuring-execution.rst
   initial-state.rst
   collecting-data.rst
   

此外，还可以配置可视化，但这在它自己的章节（:ref:`own chapter<Visualisation>`）中进行了介绍。