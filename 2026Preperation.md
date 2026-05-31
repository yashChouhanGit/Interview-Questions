Q.1 What is the difference between struct and class in Swift?

Ans. 

The fundamental difference is value type vs reference type. Structs are copied on assignment, classes are shared by reference. So with a struct, each variable owns its data independently — with a class, multiple variables can point to the same object in memory.

Classic example — if I assign a struct to a new variable and mutate it, the original is untouched. With a class, both variables see the change because they're pointing to the same heap object.

Classes also support inheritance and have deinit, which matters for resource cleanup. Structs don't. And since classes live on the heap with ARC, you have to watch for retain cycles — something you completely avoid with structs.

In practice, I default to struct unless I specifically need shared state, inheritance, or Obj-C interop. UIKit forces classes on you, but in SwiftUI or pure Swift, structs are safer, faster, and more predictable — especially in concurrent code.


If they ask "Why are structs faster?" — Stack allocation, no ARC overhead, no retain/release calls.

If they ask "When would you use a class?" — Shared mutable state — like a ViewModel, a cache, a network manager, or anything UIKit.

If they ask "What about thread safety?" — Structs give you safety by default since each thread gets its own copy. Classes need explicit synchronization.


-------------------------------------------------------------------

Q.2 Memory Management & ARC 

Ans. 

Core Answer: ARC stands for Automatic Reference Counting — it's Swift's memory management system. Every time you create a class instance, ARC tracks how many strong references point to it. When that count drops to zero, ARC automatically deallocates the object. So unlike manual memory management in C, you're not calling malloc/free — but unlike a garbage collector, ARC is deterministic — memory is freed the moment the last reference goes away, not at some future collection cycle.
The real challenge with ARC is retain cycles — when two objects hold strong references to each other, the count never reaches zero and you leak memory. Classic example is a closure capturing self strongly, or a delegate pattern where the delegate is declared strong instead of weak. The fix is using weak or unowned references. I use weak when the reference can become nil at some point — like a delegate or a coordinator. I use unowned when I'm certain the referenced object will outlive the current one — like self inside a closure where the object owns the closure.

If they ask "What's the difference between weak and unowned?" — Both break retain cycles, but weak is optional and safely becomes nil when the object is deallocated. unowned is non-optional and crashes if you access it after deallocation — so it's only safe when lifetimes are guaranteed.

If they ask "How do you detect memory leaks?" — Instruments with the Leaks and Allocations template. Also Xcode's Memory Graph Debugger — it visually shows reference cycles at runtime. In code, I verify deinit is being called where expected.

If they ask "Does ARC apply to structs?" — No. ARC only manages class instances on the heap. Structs are value types on the stack, so they're automatically cleaned up when they go out of scope — no reference counting needed, which is one reason structs are more performant.

If they ask "What is a memory graph and how do you use it?" — It's a snapshot of all live objects and their references at a given moment in Xcode. You pause the app, hit the memory graph button, and look for objects that should've been deallocated but are still alive — usually pointing to a cycle involving closures or delegates.

Closing line: In practice, the most common mistake I've seen — even in senior codebases — is forgetting [weak self] in closures inside ViewModels or network layers. It's subtle because it doesn't crash, it just silently leaks. So I treat [weak self] as a default habit in any escaping closure, and only drop it when I'm fully confident about the object's lifetime.
