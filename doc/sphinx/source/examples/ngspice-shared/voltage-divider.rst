
.. include:: /project-links.txt
.. include:: /abbreviation.txt

.. getthecode:: voltage-divider.py
    :language: python


=============================================
 NgSpice Shared Simulation Mode Introduction
=============================================

This example explains how to plug a voltage source from Python to NgSpice.


.. code-block:: python

    import math
    
    import matplotlib.pyplot as plt
    
    import PySpice.Logging.Logging as Logging
    logger = Logging.setup_logging()
    
    from PySpice.Probe.Plot import plot
    from PySpice.Spice.Netlist import Circuit
    from PySpice.Spice.NgSpice.Shared import NgSpiceShared
    from PySpice.Unit.Units import *
    
    class MyNgSpiceShared(NgSpiceShared):
    
        def __init__(self, amplitude, frequency, **kwargs):
    
            super().__init__(**kwargs)
            
            self._amplitude = amplitude
            self._pulsation = float(frequency.pulsation)
    
        def get_vsrc_data(self, voltage, time, node, ngspice_id):
            self._logger.debug('ngspice_id-{} get_vsrc_data @{} node {}'.format(ngspice_id, time, node))
            voltage[0] = self._amplitude * math.sin(self._pulsation * time)
            return 0
    
        def get_isrc_data(self, current, time, node, ngspice_id):
            self._logger.debug('ngspice_id-{} get_isrc_data @{} node {}'.format(ngspice_id, time, node))
            current[0] = 1.
            return 0
    
    circuit = Circuit('Voltage Divider')
    
    circuit.V('input', 'input', circuit.gnd, 'dc 0 external')
    circuit.R(1, 'input', 'output', kilo(10))
    circuit.R(2, 'output', circuit.gnd, kilo(1))
    
    amplitude = 10
    frequency = Frequency(50)
    ngspice_shared = MyNgSpiceShared(amplitude=amplitude, frequency=frequency, send_data=False)
    simulator = circuit.simulator(temperature=25, nominal_temperature=25,
                                  simulator='shared', ngspice_shared=ngspice_shared)
    period = float(frequency.period)
    analysis = simulator.transient(step_time=period/200, end_time=period*2)
    
    figure1 = plt.figure(1, (20, 10))
    plt.title('Voltage Divider')
    plt.xlabel('Time [s]')
    plt.ylabel('Voltage [V]')
    plt.grid()
    plot(analysis.input)
    plot(analysis.output)
    plt.legend(('input', 'output'), loc=(.05,.1))
    plt.ylim(-amplitude*1.1, amplitude*1.1)
    
    plt.tight_layout()
    plt.show()


.. image:: voltage-divider.png
  :align: center

