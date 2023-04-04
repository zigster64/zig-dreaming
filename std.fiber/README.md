# how to do coroutines in zig 0.11 without async

In Zig 0.11, coroutines can be implemented using the std.fiber package without using the async keyword. Here's an example of how to create and use a coroutine in Zig:

```
const std = @import("std");

pub fn my_coroutine(fiber: std.fiber.Fiber) !void {
    // This function will be run as a coroutine
    // Do some work here...
}

pub fn main() !void {
    // Create a coroutine
    var fiber = std.fiber.Fiber.init(my_coroutine);

    // Switch to the coroutine
    var result = fiber.switchTo();

    // Check for errors
    try result.unwrap();
}
```

In this example, my_coroutine is a function that will be run as a coroutine. The std.fiber.Fiber.init function creates a new coroutine with my_coroutine as its entry point, and returns a handle to the coroutine.

The fiber.switchTo() function switches control to the coroutine, running my_coroutine. When my_coroutine completes, it returns a result value. The try result.unwrap() line checks for any errors that might have occurred during the coroutine's execution.

Note that this is just a basic example, and there are many more features and capabilities of coroutines in Zig that you can explore.

# write a fiber coroutine library for zig that I can add to std

To create a fiber coroutine library for Zig that can be added to std, we can start by defining a set of functions and types that make it easy to work with coroutines.

Here's an example of a fiber coroutine library that provides a simple interface for creating and working with coroutines:

```
// fiber.zig
const std = @import("std");

pub const FiberError = error{
    InvalidFiber,
    StackOverflow,
};

pub const FiberFn = fn(fiber: *Fiber, arg: anytype) !void;

pub const struct Fiber {
    private var handle: std.os.Handle = undefined,
    fn_ptr: ?FiberFn = null,
}

pub fn newFiber(stack_size: usize, fn_ptr: ?FiberFn) Fiber {
    const handle = try std.os.fiber.create(stack_size, fiberMain);

    return Fiber{
        .handle = handle,
        .fn_ptr = fn_ptr,
    };
}

pub fn deleteFiber(fiber: *Fiber) !void {
    if (fiber.handle == undefined) {
        return error.InvalidFiber;
    }
    try std.os.fiber.destroy(fiber.handle);
    fiber.handle = undefined;
}

pub fn switchToFiber(fiber: *Fiber, arg: anytype) anytype! {
    if (fiber.handle == undefined) {
        return error.InvalidFiber;
    }

    const result = try std.os.fiber.switchTo(fiber.handle, arg);
    return result;
}

fn fiberMain(fiber: *std.os.Fiber, arg: anytype) !void {
    const f = @fieldParent(Fiber, handle, fiber.parent);
    if (f.fn_ptr == null) {
        return;
    }

    f.fn_ptr(&f, arg);
}
```

This library defines a Fiber struct, which contains a handle to the underlying OS fiber, as well as a function pointer to the user-defined coroutine function. The library provides three main functions for working with coroutines:

newFiber: creates a new coroutine with a given stack size and function pointer.
deleteFiber: destroys a coroutine and frees any associated resources.
switchToFiber: switches execution to a given coroutine, passing an optional argument to the coroutine function.
The library also defines a FiberError error type, which can be used to handle errors that occur during coroutine execution.

To use this library, you can simply import the fiber module and use its functions:

```
// main.zig
const std = @import("std");
const fiber = @import("fiber");

pub fn myCoroutine(f: *fiber.Fiber, arg: anytype) !void {
    // Your coroutine code goes here...
}

pub fn main() !void {
    var my_fiber = fiber.newFiber(1024, myCoroutine);

    var result = fiber.switchToFiber(&my_fiber, "hello, world");
    try result.unwrap();

    fiber.deleteFiber(&my_fiber);
}
```
In this example, we define a myCoroutine function that takes a Fiber pointer and an argument. We then create a new fiber with newFiber, passing in the size of the stack and a pointer to myCoroutine. Finally, we switch execution to the new fiber using switchToFiber, passing in an argument, and handle any errors that occur during execution. When the coroutine is finished, we delete it using deleteFiber.
