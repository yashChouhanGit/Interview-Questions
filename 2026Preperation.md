Q.1 What is the difference between struct and class in Swift?

Ans. 

The fundamental difference is value type vs reference type. Structs are copied on assignment, classes are shared by reference. So with a struct, each variable owns its data independently — with a class, multiple variables can point to the same object in memory.

Classic example — if I assign a struct to a new variable and mutate it, the original is untouched. With a class, both variables see the change because they're pointing to the same heap object.

Classes also support inheritance and have deinit, which matters for resource cleanup. Structs don't. And since classes live on the heap with ARC, you have to watch for retain cycles — something you completely avoid with structs.

In practice, I default to struct unless I specifically need shared state, inheritance, or Obj-C interop. UIKit forces classes on you, but in SwiftUI or pure Swift, structs are safer, faster, and more predictable — especially in concurrent code.


If they ask "Why are structs faster?" — Stack allocation, no ARC overhead, no retain/release calls.

If they ask "When would you use a class?" — Shared mutable state — like a ViewModel, a cache, a network manager, or anything UIKit.

If they ask "What about thread safety?" — Structs give you safety by default since each thread gets its own copy. Classes need explicit synchronization.
