IOBinders and HarnessBinders
============================

In Chipyard we use special ``Parameters`` keys, ``IOBinders`` and ``HarnessBinders`` to bridge the gap between digital system IOs and TestHarness collateral.

IOBinders
---------

The ``IOBinder`` functions are responsible for instantiating IO cells and IOPorts in the ``ChipTop`` layer.

``IOBinders`` are typically defined using the ``OverrideIOBinder`` or ``ComposeIOBinder`` macros. An ``IOBinder`` consists of a function matching ``Systems`` with a given trait that generates IO ports and IOCells, and returns a list of generated ports and cells.

For example, the ``WithUARTIOCells`` IOBinder will, for any ``System`` that might have UART ports (``HasPeripheryUARTModuleImp``, generate ports within the ``ChipTop`` (``ports``) as well as IOCells with the appropriate type and direction (``cells2d``). This function returns a the list of generated ports, and the list of generated IOCells. The list of generated ports is passed to the ``HarnessBinders`` such that they can be connected to ``TestHarness`` devices.


.. literalinclude:: ../../generators/chipyard/src/main/scala/iobinders/IOBinders.scala
   :language: scala
   :start-after: DOC include start: WithUARTIOCells
   :end-before: DOC include end: WithUARTIOCells

HarnessBinders
--------------

The ``HarnessBinder`` functions determine what modules to bind to the IOs of a ``ChipTop`` in the ``TestHarness``. The ``HarnessBinder`` interface is designed to be reused across various simulation/implementation modes, enabling decoupling of the target design from simulation and testing concerns.

 * For SW RTL or GL simulations, the default set of ``HarnessBinders`` instantiate software-simulated models of various devices, for example external memory or UART, and connect those models to the IOs of the ``ChipTop``.
 * For FireSim simulations, FireSim-specific ``HarnessBinders`` instantiate ``Bridges``, which faciliate cycle-accurate simulation across the simulated chip's IOs. See the FireSim documentation for more details.
 * In the future, a Chipyard FPGA prototyping flow may use ``HarnessBinders`` to connect ``ChipTop`` IOs to other devices or IOs in the FPGA harness.

Like ``IOBinders``, ``HarnessBinders`` are defined using macros (``OverrideHarnessBinder, ComposeHarnessBinder``), and match ``Systems`` with a given trait. However, ``HarnessBinders`` are also passed a reference to the ``TestHarness`` (``th: HasHarnessSignalReferences``) and the list of ports generated by the corresponding ``IOBinder`` (``ports: Seq[Data]``).

For exmaple, the ``WithUARTAdapter`` will connect the UART SW display adapter to the ports generated by the ``WithUARTIOCells`` described earlier, if those ports are present.

.. literalinclude:: ../../generators/chipyard/src/main/scala/harness/HarnessBinders.scala
   :language: scala
   :start-after: DOC include start: WithUARTAdapter
   :end-before: DOC include end: WithUARTAdapter

The ``IOBinder`` and ``HarnessBinder`` system is designed to enable decoupling of concerns between the target design and the simulation system.

For a given set of chip IOs, there may be not only multiple simulation platforms ("harnesses", so-to-speak), but also multiple simulation strategies. For example, the choice of whether to connect the backing AXI4 memory port to an accurate DRAM model (``SimDRAM``) or a simple simulated memory model (``SimAXIMem``) is isolated in ``HarnessBinders``, and does not affect target RTL generation.

Similarly, for a given simulation platform and strategy, there may be multiple strategies for generating the chip IOs. This target-design configuration is isolated in the ``IOBinders``.
