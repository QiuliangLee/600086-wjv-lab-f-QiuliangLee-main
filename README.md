# 600086-Lab-F

This lab requires you to create a simple particle simulation

## Q1. Particles

Open the empty `particles` project.

Define the following `struct`:

- `Particle` that contains `x` and `y` data members.
- `ParticleSystem` that contains a vector of `Particles`.

Implement a `new` function for each of the data structures.

A suggested starting point would be 100 particles limited in position to a 10 x 10 enclosure.

**Hint**: You may find constants useful for defining these parameters e.g.

```rust
const NUM_OF_THREADS: usize = 4;
```

Now add a function to `ParticleSystem` that moves each particle a random distance within the enclosure
You may find the following function useful (It returns a random floating point number in the range 0 to 1):

```rust
    rand::random::<f32>()
```

Remember to add `rand="*"` as a dependency in the `.toml` file

Add appropriate test code to ensure that the particle positions are both initialised correctly and updated when the move function is called.

Now add a function to `ParticleSystem` that contains a loop that repeatedly calls your move function, for approximately 10 seconds.

By default you have been building and running your code in debug mode.  Try switching to release mode, using

```system
cargo build --release
cargo run --release
```

Finally add the following macro on the line above your `particle` struct.

```rust
#[derive(Debug, Copy, Clone)]
pub struct Particle {
```

This macro instructs the compiler to implement a debug, copy and clone trait for your new struct.

## Q2. Threaded Particles

Make a copy of your `particle` project and name it `particle_threaded`

**IMPORTANT** please read all of this question before starting on the implementation.

The aim of this exercise is to spread the work of your move function across a number of threads.  How to sharing mutable data across threads requires careful thought and the solution is very much application dependent.

In our case we have two options:

1. Allow all threads access to the list of particles, and then lock each particle as it is updated.
2. Allocate each thread a different chunk of the list of particle, avoiding the need for locks.

The second option is the suggested approach as it is far more efficient.

Add a `thread_main` function to your project.  This should call your move function, implemented in the previous exercise. A suggested prototype for the function is:

```rust
fn pub thread_main (list: &mut [Particle], enclosure_size: f32);
```

The next problem is that we need to split the list of particles into sub lists, one for each thread.  This can be achieve in a number of different ways.  However the greater problem is one of ownership.  If this sub-list is moved into a thread, as we have done previously, then we will lose ownership of it and we'll not be able to use the sub-list within our print_all function.

The thread controls we have used so far in the module, create a thread and then allow it to run until the join.  But we have seen that the compiler does not recognise the join as the end of the thread's ownership.  What we need is another thread control mechanism that allows us to create a thread that only exists with a defined scope, recognised by the compiler.  To do this we use a `scoped_threadpool`, which as the name suggests allows us to force threads to exist only within a specific scope.

Add `scoped_threadpool="*"` to your `.toml`

The example below starts n threads that execute only in the scoped block of code.  At the end of the block, they are implicitly joined and then go out of scope.  This then allows us to regain ownership of any borrows.

```rust
let mut pool = scoped_threadpool::Pool::new(NUM_OF_THREADS as u32);

// Limit the scope of the reads to this section of code
pool.scoped(|scope| {
    for i in 0..PARTICLES_PER_THREAD {
        scope.execute(move || thread_main());
    }
});
// Implicit join here, where all threads go out of scope.
```

To split up our list of particles into sub-lists we can make use of the `chunk` functionality that splits a list into a number of mutable sub-lists

```rust
for slice in list.chunks_mut(NUMBER_OF_CHUNKS) {
    // slice is a mutable sub-list of list.
}
```

Use the `chunk` and `scope_threadpool` functionality to implement you solution to the problem of sharing the particle simulation load.

Once you have working code, test it in both release and debug mode.

What do you notice about the performance of the threaded versus non-threaded code ?
