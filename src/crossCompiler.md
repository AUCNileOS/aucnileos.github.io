As with any other OS development it is necessary to use a cross compiler unless you are developing on your own operating system.

If you are new to OS development (and find the previous sentence confusing) we highly recommend the following readings and tutorials:
- [Getting Started](https://wiki.osdev.org/Getting_Started)
- [Beginner Mistakes](https://wiki.osdev.org/Beginner_Mistakes)
- [Why do I need a Cross Compiler?](https://wiki.osdev.org/Why_do_I_need_a_Cross_Compiler%3F)
- [Bare Bones](https://wiki.osdev.org/Bare_Bones)

# Migrating from Docker Containers to Nix Flakes
Previously, the cross compiler lived in a docker container which was then used to compile NileOS.

However, other than performance issues caused by the overhead of a docker container, there were also issues of not having access to your development environment or the rest of your directory while in the container. Thus we decided to improve this using [Nix Flakes](https://nix.dev/concepts/flakes.html)

The reason we chose to use a nix flake is because of their ability to guarantee reproducibility across various devices (removing the possibility of works on my device issues), while having none of the performance disadvantages and directory limitations associated with Docker containers.

For more details on the specifics of the nix flake we used see [Nix](nix.md).

# Upgrading to gcc 14
Through the efforts of Dr Karim, NileOS was updated to use gcc 14. See this [pull request](https://github.com/AUCNileOS/bosml_ipioe/pull/3) for details on the implementation.

The entire issue was caused by a change in how the gcc compiler manages the base pointer for stacks that clashed with how NileOS used to manage the stack pointer.

This entire experience taught us two things:
- Sometimes issues can be caused by a single line of assembly that requires intense sessions of emulation (due to the need to see every single line of assembly running) in order to discover (or the help of Dr Karim).
- Do not needlessly change the major gcc version without ensuring that none of the changes made to gcc cause new problems to appear.