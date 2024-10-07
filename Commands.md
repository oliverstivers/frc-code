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
The preferred way of creating a command is by providing a public method in a subsystem that returns a command for that subsystem. This is called a **command factory.** The reason for doing this is so we can keep all potentially dangerous methods of interfacing with the subsystem private inside the subsystem, and only interact from the outside through a command.  We can specify all the behavior of the command from within this method, such as `initialize`, `execute`, `isFinished`, and `end` methods. Let's look at an example from our 2024 code, in the [`RotatorSubsystem.java`](https://github.com/Stealth-Robotics/Stealth2024/blob/main/src/main/java/frc/robot/subsystems/RotatorSubsystem.java): When controlling any mechanism, you usually want to think "What is the *simplest* thing I could tell the subsystem to do from the outside while still being able to accomplish everything I need?" This gets into the concept of **abstraction**, where you try to hide the unnecessary details that make the code more complex in the subsystem they're a part of, and only worrying about simpler things from the outside. In the context of the rotator, we really only need to tell it to go to a position, and want a command returned that will do that, and then finish when the rotator reaches its target position. Take a look at the code below: 

```java
public Command rotateToPositionCommand(DoubleSupplier rotations) {
	return this.runOnce(() -> setMotorTargetPosition(rotations.getAsDouble()))
        .andThen(new WaitUntilCommand(() -> this.isMotorAtTarget()));
}
```

You first see that we have a public method that returns a command, and it takes a `DoubleSupplier` parameter. This is important, as this position we command it to go to will change depending on different states in the code, and we want to be able to tell it to go to the correct position during runtime. Thus, we use a `DoubleSupplier` and the `getAsDouble()` method. 

Next, we jump into the single line in the method. We call `this.runOnce()`, which returns an `InstantCommand`, a command that ends instantly. We set the target position of the motor which is sent to the motor controller and will run with PID control. For more information, check out the [[Motors]]() section.

Since setting the motor target position ends instantly, we want to somehow "extend" the command until the motor is at the target position. That's where the next part of the method comes into play. By adding a `.andThen()` to the `InstantCommand` returned through `this.runOnce()`, we can specify something to run after, which will extend the command until whatever runs finishes. We create a new `WaitUntilCommand`, which will take a `BooleanSupplier` parameter. Basically, think of Tthis a method that will be called every loop, and when that method returns true, the command will end. We have a method, `isMotorAtTarget()`, which returns a boolean. However, we need a `BooleanSupplier`. To turn this method into a boolean supplier, we can use a  lambda expression, by putting a 
`() ->` in front of the method call. This now makes the `WaitUntilCommand` call whatever is after the `() ->` to check the boolean every loop. 
