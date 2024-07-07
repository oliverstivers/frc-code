***
## Basic Subsystem Theory

The most basic element in your robot's code is a subsystem. Simply put, a subsystem can be thought of as a collection of all physical robot hardware, such as motors, sensors, servos, etc., which work together in order to accomplish that subsystem's task. 

Some examples of subsystems in 4089's 2024 robot are the shooter, which includes the 2 motors controlling the shooter wheels, the rotator, which includes the motors to control the rotation of the big arm, and the intake, which controls the motor to run the intake rollers. 
 
\[💡] __It is very important to think of how you need to separate the hardware of the robot into different subsystems during the initial code creation. This will be explained more in depth in the [[Commands]] page, but _most_ of the time, you cannot have two things running on the same subsystem at once, so you don't want to encapsulate too much hardware into one subsystem.__


## Organization

On 4089, we organize all of our subsystems into a subdirectory inside the `robot` folder. If the subsystem contains multiple files that depend on each other, we may further organize those files into another subdirectory, titled something that fits with what the subsystem does.

![subsystemorg.png](subsystemorg.png)

_Basic example of subsystem organization. Since the `RotatorSubsystem` only contains one file, it is directly inside the `subsystems` directory. The shooter subsystem contains multiple necessary files for its proper function, so it is further organized into the `shooter` subdirectory._

