# Code Structure

## Input_Data.py
In this file, we can setup the simulation by inputting various kinds of variables such as:

`working_dir` which will be changed according to the user's workspace


### Solver Settings

```python
working_dir = r"/home/doktorand/Software/PyAeroSweep-Stan-V3/PyAeroSweep/"
Solver_settings = Solver()

Solver_settings.working_dir = working_dir

Solver_settings.name = 'SU2'                 # SU2 or Fluent

# Solver dimensions
# 2d   or 3d        for SU2 
Solver_settings.dimensions = '2d'            

# Only available for SU2 in 3D
# defines half od the shape or a full shape analysis (Only symmetric works for now)
Solver_settings.symmetric = True             

# SST or SA for SU2
Solver_settings.turbulence_model = 'SST'

# Number of processors
Solver_settings.processors = 1

# Cauchy convergence criteria
# Could be either LIFT or DRAG
Solver_settings.monitor          = "LIFT"
Solver_settings.tolerance        = 5e-7      
Solver_settings.max_iterations   = 100
Solver_settings.save_frequency   = 100

# Warm start
# YES or NO
Solver_settings.warmstart = 'YES'

# SU2 reference config file name which will be updated
Solver_settings.config_file = 'Run_airfoil_template.cfg'
```

### Freestream Settings
The following code block is the setup of Freestream. The air speed (in Mach), altitude (meters) and angle of attack (degrees) can be set as following.
```python
Freestream = Data()
Freestream.Mach             = np.array([0.21,0.25])
Freestream.Altitude         = np.array([0,2000])                 # in meters
Freestream.Angle_of_attack  = np.array([0.0,3.0,5.0])               # in degrees
```


### Geometry Settings
```python
Geometry_data = Geometry()

# Geometry to analyze
''' Could be airfoil or wing
    Airfoils can be parametrically defined using the PARSEC methods
    Wings are defined only using the existing CAD file and work either for
    straight tapered wings with or without the kink'''

Geometry_data.type = 'airfoil'

# Reference values
Geometry_data.reference_values = {
    "Area"   : 2.62,
    "Length" : 2.62,
    "Depth"  : 1,
    "Point"  : [0.25*2.62,0,0]              # reference point about which the moment is taken
}

# Flag to use PARSEC parametrization or to use already existing airfoils
Geometry_data.PARSEC = True

# Airfoil files are used either to write PARSEC-generated airfoil and then read by Pointwise or to read directly from them
Geometry_data.airfoil_files = {
    "upper" : "main_airfoil_upper.dat",
    "lower" : "main_airfoil_lower.dat"
}


segment = Segment()
segment.tag                = 'section_1'
segment.spanwise_location  = 0 
segment.chord              = 7.760
segment.incidence          = 2 
segment.dihedral           = 3 
segment.leading_edge_sweep = 30 
segment.Airfoil.files      = {
    "upper" : "main_airfoil_upper_1.dat",
    "lower" : "main_airfoil_lower_1.dat"
}
segment.Airfoil.PARSEC     = {
                                "rle"        : 0.0084,                      # Main airfoil LE radius
                                "x_pre"      : 0.458080577545180,           # x-location of the crest on the pressure side
                                "y_pre"      : -0.04553160030118,           # y-location of the crest on the pressure side  
                                "d2ydx2_pre" : 0.554845554794938,           # curvature of the crest on the pressure side  
                                "th_pre"     : -9.649803736,                # trailing edge angle on the pressure side [deg]
                                "x_suc"      : 0.46036604,                  # x-location of the crest on the suction side 
                                "y_suc"      : 0.06302395539,               # y-location of the crest on the suction side
                                "d2ydx2_suc" : -0.361421420,                # curvature of the crest on the suction side
                                "th_suc"     : -12.391677695858             # trailing edge angle on the suction side [deg]
}
Geometry_data.Segments.append(segment)
```

### Mesh Settings
```python
Mesh_data = Mesh()

# Flag to mesh the shape or not
Mesh_data.meshing    = True

# Mesh type
Mesh_data.structured = True

# Defined the OS in which Pointwise is used
# WINDOWS or Linux
Mesh_data.operating_system = 'Linux'

# Pointwise tclsh directory used in Windows
Mesh_data.tclsh_directory =  r"/home/doktorand/Fidelity/Pointwise/Pointwise2022.1" 

# Desired Y+ value
Mesh_data.Yplus = 1.0

# Define the Glyph template to use for meshing
Mesh_data.glyph_file = "mesh_clean_airfoil_SU2.glf"

# Mesh filename for either the newly generated mesh or an eisting mesh
Mesh_data.filename = 'su2meshEx.su2'

# Define far-field 
Mesh_data.far_field = 100 * Geometry_data.reference_values["Length"]


Mesh_data.airfoil_mesh_settings = {
    "LE_spacing"             : 0.001,                       # Airfoil leading edge spacing
    "TE_spacing"             : 0.0005,                      # Airfoil trailing edge spacing
    "flap_cluster"           : 0.005,
    "connector dimensions"   : [200, 200, 8],     #         "connector_dimensions": [200, 120, 150, 150, 70, 25, 8, 8],
    "number of normal cells" : 230
}
```
### Packing all the inputs

```python
Input = Data()
Input.Solver        = Solver_settings
Input.Freestream    = Freestream
Input.Geometry      = Geometry_data
Input.Mesh          = Mesh_data
```
### Summary Code Example
At the end, the function `Input_data()` is defined as

```python
import os
import subprocess
import numpy as np
from Core.Data                          import Data
from Components.Solver                  import Solver
from Components.Geometry                import Geometry
from Components.Geometry.Wing.Segment   import Segment
from Components.Mesh                    import Mesh


from Run_aerodynamic_analysis import run_aerodynamic_analysis


def Input_data():

    working_dir = r"/home/doktorand/Software/PyAeroSweep-Stan-V3/PyAeroSweep/" 

# ------------------------------- SOLVER SETTINGS ----------------------------------------------------------- #
#

    Solver_settings = Solver()

    Solver_settings.working_dir = working_dir

    Solver_settings.name = 'SU2'                 # SU2 or Fluent

    # Solver dimensions
    # 2d   or 3d        for SU2 
    Solver_settings.dimensions = '2d'            

    # Only available for SU2 in 3D
    # defines half od the shape or a full shape analysis (Only symmetric works for now)
    Solver_settings.symmetric = True             

    # SST or SA for SU2
    Solver_settings.turbulence_model = 'SST'

    # Number of processors
    Solver_settings.processors = 1

    # Cauchy convergence criteria
    # Could be either LIFT or DRAG
    Solver_settings.monitor          = "LIFT"
    Solver_settings.tolerance        = 5e-7      
    Solver_settings.max_iterations   = 100
    Solver_settings.save_frequency   = 100

    # Warm start
    # YES or NO
    Solver_settings.warmstart = 'YES'

    # SU2 reference config file name which will be updated
    Solver_settings.config_file = 'Run_airfoil_template.cfg'



# ------------------------------- FREESTREAM SETTINGS ------------------------------------------------------- #
#
    Freestream = Data()
    Freestream.Mach             = np.array([0.21,0.25])
    Freestream.Altitude         = np.array([0,2000])                 # in meters
    Freestream.Angle_of_attack  = np.array([0.0,3.0,5.0])               # in degrees




# ------------------------------- GEOMETRY SETTINGS --------------------------------------------------------- #
#
    Geometry_data = Geometry()

    # Geometry to analyze
    ''' Could be airfoil or wing
        Airfoils can be parametrically defined using the PARSEC methods
        Wings are defined only using the existing CAD file and work either for
        straight tapered wings with or without the kink'''

    Geometry_data.type = 'airfoil'

    # Reference values
    Geometry_data.reference_values = {
        "Area"   : 2.62,
        "Length" : 2.62,
        "Depth"  : 1,
        "Point"  : [0.25*2.62,0,0]              # reference point about which the moment is taken
    }

    # Flag to use PARSEC parametrization or to use already existing airfoils
    Geometry_data.PARSEC = True

    # Airfoil files are used either to write PARSEC-generated airfoil and then read by Pointwise or to read directly from them
    Geometry_data.airfoil_files = {
        "upper" : "main_airfoil_upper.dat",
        "lower" : "main_airfoil_lower.dat"
    }


    segment = Segment()
    segment.tag                = 'section_1'
    segment.spanwise_location  = 0 
    segment.chord              = 7.760
    segment.incidence          = 2 
    segment.dihedral           = 3 
    segment.leading_edge_sweep = 30 
    segment.Airfoil.files      = {
        "upper" : "main_airfoil_upper_1.dat",
        "lower" : "main_airfoil_lower_1.dat"
    }
    segment.Airfoil.PARSEC     = {
                                    "rle"        : 0.0084,                      # Main airfoil LE radius
                                    "x_pre"      : 0.458080577545180,           # x-location of the crest on the pressure side
                                    "y_pre"      : -0.04553160030118,           # y-location of the crest on the pressure side  
                                    "d2ydx2_pre" : 0.554845554794938,           # curvature of the crest on the pressure side  
                                    "th_pre"     : -9.649803736,                # trailing edge angle on the pressure side [deg]
                                    "x_suc"      : 0.46036604,                  # x-location of the crest on the suction side 
                                    "y_suc"      : 0.06302395539,               # y-location of the crest on the suction side
                                    "d2ydx2_suc" : -0.361421420,                # curvature of the crest on the suction side
                                    "th_suc"     : -12.391677695858             # trailing edge angle on the suction side [deg]
    }
    Geometry_data.Segments.append(segment)


# ------------------------------- MESH SETTINGS ---------------------------------------------------------------- #
#

    Mesh_data = Mesh()

    # Flag to mesh the shape or not
    Mesh_data.meshing    = True

    # Mesh type
    Mesh_data.structured = True

    # Defined the OS in which Pointwise is used
    # WINDOWS or Linux
    Mesh_data.operating_system = 'Linux'

    # Pointwise tclsh directory used in Windows
    Mesh_data.tclsh_directory =  r"/home/doktorand/Fidelity/Pointwise/Pointwise2022.1" 

    # Desired Y+ value
    Mesh_data.Yplus = 1.0

    # Define the Glyph template to use for meshing
    Mesh_data.glyph_file = "mesh_clean_airfoil_SU2.glf"

    # Mesh filename for either the newly generated mesh or an eisting mesh
    Mesh_data.filename = 'su2meshEx.su2'

    # Define far-field 
    Mesh_data.far_field = 100 * Geometry_data.reference_values["Length"]


    Mesh_data.airfoil_mesh_settings = {
        "LE_spacing"             : 0.001,                       # Airfoil leading edge spacing
        "TE_spacing"             : 0.0005,                      # Airfoil trailing edge spacing
        "flap_cluster"           : 0.005,
        "connector dimensions"   : [200, 200, 8],     #         "connector_dimensions": [200, 120, 150, 150, 70, 25, 8, 8],
        "number of normal cells" : 230
    }


# ----------------------------------------------------------------------------------------------------------------------------- #
#
    # Pack all inputs
    Input = Data()
    Input.Solver        = Solver_settings
    Input.Freestream    = Freestream
    Input.Geometry      = Geometry_data
    Input.Mesh          = Mesh_data


    return Input
```

After finishing running the function `Input_data()`, function `aerodynamic_analysis(Input)` will be executed using `Input_data()` as its input to perform the simulation.

