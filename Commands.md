The second pivotal part of WPILib's command based structure is, well, the command.  There are a few ways of creating commands in code, but they all relate in some way to the `Command` class provided by WPILib. Commands are very powerful, and create a very safe and predictable way of controlling the robot at the same time. The reason for this being the case is this:

\[💡] __All commands must "require" the subsystem they run on. By adding the requirements to a command, we can ensure that no two commands that require the same subsystem will ever be run at the same time.__

This safety is accomplished by something called the `CommandScheduler`, which is responsible for running commands when we tell them to, and it ensures that two commands that run on the same subsystem will never be run concurrently. The way this is achieved can be further tuned by us, however, the default way this happens when nothing is provided is a command will be canceled if it is running and a new command requiring the subsystem is attempted to be run. 

# Methods of Creating Commands
## Subclassing `Command`
Though not used much by our team anymore, as we've found safer and more convenient ways of creating commands, I've chosen to look at this method of creating a command first because it most clearly shows how commands function. The `Command` class has 4 main methods that we can override to achieve the behavior we desire. These methods are the `initialize()`, `execute()`, `isFinished()`, and `end()` methods. A fifth method, `addRequirements()`, is also very important, and is how we tell the CommandScheduler that this command "requires" that subsystem, so we can achieve the behavior described above. 

`initialize()` is run once when the command is scheduled. Put any behavior in here that gets the subsystem to a required state before running the command if necessary. This can be things like calculating a target for the subsystem to reach, or reading initial values from encoders or other sensors.

`execute()` is run every loop of the robot code (which by default is every 0.02 seconds, or 50 times per second), and is where the main work of the command is done. Here is where you'll usually put things like driving a mechanism, since this loop executes over and over.

The `isFinished()` method must return a boolean, and the behavior you provide must return true when you want the command to end. Usually here you will return true if the mechanism has reached its setpoint, or something similar. The `isFinished()` method is run every loop of the program after the `execute()` method, so keep that in mind.

`end()` is run one time after `isFinished` returns true. Here you will typically stop the subsystem's motion or otherwise get it in some controlled state. Additionally, the `end()` method takes in a `boolean interrupted` parameter, which you can use to provide different behavior on command interrupt vs. it being finished through `isFinished()` returning true. The command scheduler will call `end(true)` whenever the command is interrupted in some form, either being cancelled early, or when another command requiring the subsystem is attempted to be run.

\[💡] __Although this is a very clear way to define a command, we often steer away from implementing one through subclassing `Command`. This is because for these types of commands to function, we often need public `setPower()` or similar methods. Since these methods can be run at any time from anywhere without checking subsystem requirements, this can cause unexpected behavior, which is why we generally try to steer away from this method of command implementation.__

## Returning a Command Through a Method
The preferred way of creating a command is by providing a public method in a subsystem that returns a command for that subsystem. The reason for doing this is so we can keep all potentially dangerous methods of interfacing with the subsystem private inside the subsystem, and only interact from the outside through a command. 