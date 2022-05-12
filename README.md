# Qudi Experiments @ QT3

[Qudi](https://github.com/Ulm-IQO/qudi-core) is a tool for running QDM
experiments and supports a wide range of hardware.

The [QT3 Lab Setup section](#qt3-lab-setup) describes the general hardeware and software setup.

[Experiment configurations](#experiments) describe specific configuration and usage instructions.

  * [CW ODMR](#cw-odmr)
  * [Pulsed ODMR](#pulsed-odmr)

# QT3 Lab Setup

This setup was completed May 11 2022.

## Hardware

* NI 6363
* Excelitas SPCM - AQ4C
* Windfreak SynthHD v2.06 RF Generator
* DELL PC - Windows 11

## Software

The following is a record of the software that was installed to run `qudi`
successfully. We chose to install and configure our python environment with Anaconda.
However, there are other possible ways to achieve this, such as installing
python from www.python.org and using `venv` or `virtualenv`.

Qudi comes in two separate code bases -- `qudi-core` and `qudi-iqo-modules` -- which
were installed separately. This modularity facilitates community-built software to
support a wider range of hardware.


* Installed the NI card drivers and software
* Installed Anaconda
* [Installed qudi-core](https://ulm-iqo.github.io/qudi-core/setup/installation.html)
  * Opened PowerShell via the Anaconda application
  * Created new environment
    * `conda create --name qudi-qdl python=3.8 --no-default-packages`
  * Activated the environment
    * `conda activate qudi-qdl`
  * Installed `qudi-core`
    * `pip install qudi-core`
* Installed `qudi-iqo-modules`
  * `cd C:\Users\QT3\projects`
  * `git clone https://github.com/Ulm-IQO/qudi-iqo-modules.git`
  * `cd qudi-iqo-modules`
  * `pip install -e .`

The `pip install -e .` command installs the local source code into the python environment via soft links.
This allows for editing of the source code without having to re-install `qudi-iqo-modules` to see changes.

#### Attempt to find the Windfreak SynthHD

Additional drivers and software were needed to control the RF generator. This was
discovered by realizing `pyvisa` was unable to connect to the hardware.
The following code resulted in an error.

```python
import pyvisa
rm = pyvisa.ResourceManager() #this line resulted in an error
rm.list_resources()
```

We installed the
[National Instruments VISA drivers](https://www.ni.com/en-us/support/downloads/drivers/download.ni-visa.html#)
as suggested.

Subsequently, the output from `rm.list_resources()` was

```
('TCPIP0::128.95.31.254::inst0::INSTR',
 'TCPIP0::128.95.31.186::inst0::INSTR',
 'ASRL3::INSTR',
 'ASRL5::INSTR')
```

Using the Device Manager in Windows, it was determined that `ASRL5::INSTR` matches `COM5` USB port.


# Experiments

#### Configuration Files

Qudi operation requires a configuration file to inform the software which GUI to create,
the logic behind the GUI and the hardware devices being used.  The GUI, logic
and hardware code are defined in the `qudi-iqo-modules` repository.

Below are configuration files and instructions for running qudi-based experiments.


## CW ODMR

The configuration file
[cw_odmr_scan_configuration_file.cfg](cw_odmr_scan_configuration_file.cfg) sets
up CW ODMR.

##### Non-Standard Configuration

Besides the use of `ASRL5::INSTR` instead of `COM5` to configure the RF generator
(line 59), the other non-standard configuration was the `default_scan_mode: EQUIDISTANT_SWEEP`
specification in the ODMR Logic configuration (line 52).

ToDo: It may be possible to use the `JUMP_LIST` scan type, but we'd need to see
if adding the `SamplingOutputMode.JUMP_LIST` scan mode to the
[source code](https://github.com/Ulm-IQO/qudi-iqo-modules/blob/main/src/qudi/hardware/microwave/mw_source_windfreak_synthhdpro.py#L84)
would work.


### Usage

* Launch Anaconda
* Activate the environment in which `qudi` was installed (currently `qudi-qdl`).
* Launch the PowerShell
* Run qudi from the command-line with path to configuration
  * `> qudi -c  C:\Users\QT3\projects\qudi-qdl-experiments\cw_odmr_scan_configuration_file.cfg`
* Click on the `cw_odmr_gui`, adjust parameters and start the scan.


## Pulsed ODMR
