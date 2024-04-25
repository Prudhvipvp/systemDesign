

Dynamic Binding vs Static binding -

- In Java, static binding is used for overloaded methods, while dynamic binding is used for overridden methods. This means that when a method is overloaded, the compiler will determine which method to call at compile time. However, when a method is overridden, the compiler will not determine which method to call until runtime.
- Now consider the below overridden case -

```
method drawShape(shape: Shape) is
    shape.draw();
```

- the `draw` method defined in `Shape` class. Wait a sec, but there are also four subclasses that override this method. Can we safely decide which of the implementations to call here? It doesn’t look so. So its decided at runtime. Note that the draw method has been overridden thats why its dynamic binding.
- Now consider this case -

```
method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);
```

- The exporter class has multiple overloaded methods with same name export as - export(s:Shape), export(c:Circle), export(r :Rectangle) etc
- This is method overloading -> static binding and why its static because What if there’s a shape class that doesn’t have appropriate `export` method in `Exporter` class? For instance, an `Ellipse` object. The compiler can’t guarantee that the appropriate overloaded method exists in contrast with overridden methods. The ambiguous situation arises which a compiler can’t allow.
- So, if you call exporter.export(new Circle()); -> the exporter' class's export(s:Shape) will be called and Not export(c:Circle). Becoz - Compiler know that `Exporter` class has a basic implementation of the export that supports `Shape` class: `export(s: Shape)`.That’s the only implementation that can be safely linked to a given code without making things ambiguous. That’s why even if we pass a `Rectangle` object into `exportShape`, the exporter will still call an `export(s: Shape)` method.
- Therefore, compiler developers use a safe path and use the early (or static) binding for overloaded methods.
- Now, to enforce to use export(Circle) always, we have a pattern - [Double Dispatch.](https://refactoring.guru/design-patterns/visitor-double-dispatch)This pattern is the trick that allows using dynamic binding alongside with overloaded methods

No, you cannot directly assign a `List<ParentClass>` variable to an `ArrayList<ChildClass>`. This is because of generics in Java and how it ensures type safety.

If `ChildClass` is a subclass of `ParentClass`, you can do the reverse assignment:

java

`List<ParentClass> list = new ArrayList<>(); list.add(new ChildClass()); // This is allowed`

This is because the `List<ParentClass>` variable can hold objects of `ChildClass` due to polymorphism. However, assigning an `ArrayList<ChildClass>` to a `List<ParentClass>` directly is not allowed.

If you need a list that can contain both `ParentClass` and its subclasses, you can use a wildcard (`? extends ParentClass`). For example:

java

`List<? extends ParentClass> list = new ArrayList<ChildClass>();`

This allows the list to hold objects of type `ChildClass` or any other subclass of `ParentClass`. Keep in mind that with this approach, you have limited ability to modify the list (e.g., add elements) because the compiler doesn't know the exact type of the elements in the list.

&nbsp;

1\. local variables are also thread-safe because each thread has there own copy and using local variables is a good way to write thread-safe code in Java.

2\. In Java, when you make a variable final, i.e. final arr = new Array();  it means the arr cant re referenced to any object at all. Although the contents inside arr can be changed, i.e. arr.add() can be done

3\. In Java lambdas, need not be always multi threaded, the normal stream() is sequential and single threaded. while parallelStream() isnt

4\. Whatever the stream is, when you are using any local variable(defined out of lambda) inside lambda, java rule is it should be final/effectively final(i.e. in code anywhere it shouldnt be reassigned again).

5\. This is becoz, when lambda is created in jvm while code execution, it captures the local variable and by value whatever is its value, its captured and kept in lambda. Now lambda's can be passed to methods/stored for later execution. Now when its later executed, by then if some code changes local var, it doesnt reflect in lambda and can cause unexpected results. So its reqd to be final as a rule of thumb. Note that I can use int\[\] arr = {0} in lambda becoz, arr is here a reference and not a variable and the arr isnt re referenced again later ( and so it became effectively final), if u reassign arr again , then you cant use it in lambda either.

6\. Also if arr is final, i can still do arr.add(1) but if a is final, i cant do a+=1. This is becoz, a+=1, is actually doing a reassignment to a and not just changing the value of underlying object the pointer points to. In this case, the underlying object is 1 and since java integers is immutable, its value cant be changed. So, when you do a+=1; it doesnt update 1 to 2. it creates a new object 2 and assigns it to a and throws away 1. So youa re re assigning a final var which is wrong.

&nbsp;

```
public class TaskService{


 public synchronized void modifyTask(Task task){
        validateTask(task);
        tasksRepository.modifyTask(task);
        auditsRepository.addAction(task.getTaskId(), Action.MODIFY);
    }

public synchronized void removeTask(Integer taskId){
        tasksRepository.removeTask(taskId);
        auditsRepository.addAction(taskId,Action.REMOVE);

    }
}
```

No, if a thread is currently executing the modifyTask method, another thread cannot execute the removeTask method at the same time. This is because both methods are marked with the synchronized keyword. In Java, the synchronized keyword is used to create a critical section around a block of code or method. This means that only one thread can execute the critical section at a time. If a thread is currently executing a synchronized method, all other threads that attempt to execute any synchronized method on the same object will be blocked until the first thread exits the method. So in your case, if a thread is executing the modifyTask method, any other threads that attempt to execute modifyTask or removeTask on the same TasksService object will be blocked until the first thread finishes executing modifyTask. This ensures that modifyTask and removeTask cannot run concurrently on the same TasksService object.

&nbsp;

- HashMap isn’t thread-safe at all. Thus, it is non-synchronized in nature. The ConcurrentHashMap, on the other hand, is thread-safe.
- Due to non-synchronization, the performance of HashMap is relatively higher, and various threads are capable of performing simultaneously. A similar case is not possible with ConcurrentHashMap. Its performance gets comparatively lower because some of the threads need to wait.
- If the other threads try to modify or add the contents to an object while a thread iterates the HashMap object, we receive a run-time exception that says ConcurrentModificationException. On the other hand, the ConcurrentHashMap doesn’t generate any such exception when we perform any kinds of moderations during iteration.

private Map&lt;String, Map<Long, AtomicLong&gt;> UserTimeMap = new ConcurrentHashMap<>();

Its used in rate limiter, key is userId and innerMap's key is timestamp and value is number of requests at that timestamp.

Note that its AtomicLong -> i.e. thread safe synchronised Long. So crud on it happen with thread safety.Note that <span style="color: #ececec;">while multiple threads can access and perform operations on an</span> `AtomicLong`<span style="color: #ececec;">, the operations themselves are atomic and thread-safe, ensuring consistency and avoiding race conditions. Atomic means, if a thread is doing a set on AtomicLong, other threads will see it as a single atomic operation and <span style="color: #ececec;">it means that the new value is immediately visible to other threads. So, no issues. Basically, use it when high concurrency is expected</span></span>