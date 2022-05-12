# Quantum Defects Lab - Qudi Experiments

Documentation for using `qudi` @ QDL

Qudi is https://github.com/Ulm-IQO/qudi-core

This repository contains a collection of instructions and
`qudi` configuration files used to build various experiments

## QT3 Lab CW ODMR

May 11 2022

CW ODMR was built using the [qudi-core](https://github.com/Ulm-IQO/qudi-core) and 
[qudi-iqo-modules](https://github.com/Ulm-IQO/qudi-iqo-modules) packages 
developed by Institute for Quantum Optics (Ulm Universitaet).

### Hardware

* NI 6363 
* Excelitas SPCM - AQ4C
* Windfreak SynthHD v2.06 RF Generator
* DELL PC - Windows 11

### Installation

* Install the NI card drivers and software
* [Follow instructions to install qudi-core](https://github.com/Ulm-IQO/qudi-core)
  * NB:
    * If you are using Anaconda, you should build an environment without the default packages
      * `conda create --name myenv python=3.8 --no-default-packages`
    * If you are using miniconda, you can omit `--no-default-packages`
      * `conda create --name myenv python=3.8`
* install `qudi-iqo-modules`
  * `git clone https://github.com/Ulm-IQO/qudi-iqo-modules.git`
  * `cd qudi-iqo-modules`
  * `pip install -e .`

The `pip install -e .` command installs the local source code into your python environment via soft links. 
This allows you to edit the source code if needed and then see those changes in 
your environment after relaunching a python shell (or immediately if you use the ipython/jupyter 
%autoreload magic).

#### Attempt to find the Windfreak SynthHD

Additional drivers and software may be needed in order to control the RF generator. We can attempt 
to find out before running qudi direcly. Start a python shell and run the following.

```python
import pyvisa
rm = pyvisa.ResourceManager()
```

At this point, if no error is reported, you should have the VISA/IVI libraries properly installed.
If an error is returned, one of suggested solution was to install `pyvisa-py`. However, that was not successful
for me. Instead, I installed the 
[National Instruments VISA drivers](https://www.ni.com/en-us/support/downloads/drivers/download.ni-visa.html#)

Next, once you have the VISA drivers installed, determine the name of the serial port for the device.

```python
rm.list_resources()
```

An example output

```
('TCPIP0::128.95.31.254::inst0::INSTR',
 'TCPIP0::128.95.31.186::inst0::INSTR',
 'ASRL3::INSTR',
 'ASRL5::INSTR')
```

Using the Device Manager in Windows, determine which address matches the address for the RF generator. 
In my case, `ASRL5::INSTR` matches `COM5` USB port. 

### Qudi Configuration File

Qudi operation requires a configuration file to inform the software which GUI to create,
the logic behind the GUI and the hardware devices that are being used.  The GUI, logic
and hardware code are defined in the `qudi-iqo-modules` repository. Information about building custom
GUIs, logic and hardware are found in that repository's documentation.

The configuration file used here is [cw_odmr_scan_configuration_file.cfg](cw_odmr_scan_configuration_file.cfg).

Besides the use of `ASRL5::INSTR` instead of `COM5` to configure the RF generator (line 59), the other non-standard 
configuration was the `default_scan_mode: EQUIDISTANT_SWEEP` specification in the ODMR Logic configuration (line 52. 

ToDo: It may be 
possible to use the `JUMP_LIST` scan type, but we'd need to see if adding the `SamplingOutputMode.JUMP_LIST` 
scan mode to the 
[source code](https://github.com/Ulm-IQO/qudi-iqo-modules/blob/main/src/qudi/hardware/microwave/mw_source_windfreak_synthhdpro.py#L84)
would work.

NB: Qudi configuration files support multiple gui-logic-hardware trees within the same configuration file. Thus, you 
could combine configuration files into a single file in order to ease switching between types of experiments.

### Usage

These instructions assume usage of Anaconda and the PowerShell application. 
* Launch Anaconda
* Activate the environment in which `qudi` was installed.
* Launch the PowerShell (the shell prompt should indicate you are running in the environment)
* Run qudi from the command-line
  * `> qudi`
* Click on the `cw_odmr_gui`, adjust parameters and start the scan.
