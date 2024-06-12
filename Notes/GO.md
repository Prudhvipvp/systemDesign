GO

1.  Java is both compiled and interpreted language. While go is a compiled language and python is a interpreted language

Interpreted - In here the source code is directly read by interpreter line by line and is executed at the same time

Compiled - Here first code is compiled by compiler into machine code which the machine runs. Obviously this one is much faster as compilation is already done.

Java is both becoz its nature of machine independent. The java compiler compiles code to byteCode. Now this byteCode is interpreted by the JVM and runs.

GO doesnt have classes, its all types

* * *

1.  In GO, The entry point for the application is the `main` function in the `main` package <ins>as described in the specification</ins>
    
2.  We cant have an import and a variable if its not used. Its throws error
    
3.  In Go, `:=` is for declaration + assignment, whereas `=` is for assignment only.
    
    For example, `var foo int = 10` is the same as `foo := 10 -> so u can omit the var and int`
    
4.  There are no classes, just inside a file, you have the func - the methods and type -> the class objects/rather called structs
    
5.  ERROR HANDLING - There is no try and catch here, rather a method can simply return a error. So you say your method also returns a error along with an int by saying func() (int,error) - now if nothing happens u return error as nil, or else u can return your custom errors also. in the caller, you can have simply if conditions to check if error is non nil
    
6.  In go , we don't follow ascii , rather UTF-8
    
7.  ARRAYS - While declaring its better to define the size of it also, instead of later appending elements on the fly. As it not optimal, and time consuming than the other way
    

INTERFACES AND TYPES AND METHODS -

1.  You have a struct which is basically a type (in scala) -
    
    1.  ```
        type gasEngine struct {
         mileage uint8
         gallons uint8
        }
        ```
        
2.  Now to make a object of this -> gasEngine{60,20}
    
3.  What if need to have methods associated with this -
    
    1.  ```
        func (e gasEngine) milesLeft() uint8 {
          return e.gallons*e.mileage
        }
        ```
        
4.  Note that this is different from a normal function where we define the func name first and inside it the arguments that go in. This way, we are telling that this milesleft method is associated with gasengine, and we can call it like - gasEngine.milesLeft()
    

Interfaces -

1.  Create a interface type -
    
    1.  ```
        type engine interface{
        milesLeft() uint8;
        }
        ```
        
2.  Now whichever type associated func's that implement this method milesleft, are its children. So we can have a simple method -
    
    1.  ```
        func canMakeIt(eng engine, current uint8) uint8 {
         ..  .. . . . 
        
        }
        ```
        
3.  To this above method we can pass gasEngine type and any other also which implement engine.
    

POINTERS -

1.  GO, used pointers and arrays are basically pointers only. Whenever we pass a array to a method, we are passing by value and any changes in the method, wont reflect outside. So , if we want, then we need to pass the reference to the method

ROUTINES -

1.  Routines in go are a way for concurrency. Say we need to make a call dbCall() multiples. instead of simply doing serially in for loop. We can do them reactively and so concurrently i.e. separate thread calls each dbCall concurrently
    
2.  To invoke while calling the method just add 'go' keyword before the method call, and it runs in a separate go routine(rather by a separate thread)
    
3.  Go routines run in the same address space, so access to shared memory must be synchronised. The [`sync`](https://go.dev/pkg/sync/) package provides useful primitives, and to lock and unlock the shared resources.
    
4.  Also, since go routines run in background, they just triggered by a separate thread( and not the current running main thread), all the concurrent calls are made and these calls are made by separate threads which just start the method call and the main thread doesn't wait for the response or something. The main thread just spawns the concurrent go routines and exits.
    
5.  So to make the main go routine wait, we have 'WaitGroups' those can be thought of as just counters. So before calling go dbCall() - you do waitGrp.add(1) (its incrementing the counter) and inside the dbCall() method at the end you do - waitGroup.done() (its decrementing the counter). Now in your main, after all the concurrent calls are done. you can waitgrp.wait() -> which basically waits till the counter = 0
    
6.  ![Screenshot 2024-06-02 at 2.30.53 AM.png](../_resources/Screenshot%202024-06-02%20at%202.30.53 AM.png)
    

PYTHON - 

1.  In Python, you can dynamically add attributes to objects, including instances of a class. This is possible because Python classes and instances of classes both have a built-in dictionary that holds their attributes. When you do emp1.name = "hello", Python adds a new entry to the emp1 instance's attribute dictionary with "name" as the key and "hello" as the value. .  -> you can get this dict by -> obj.\__dict__
2.  So, we don't define specifically, the class's attributes in class def. In the class's methods, you directly use them like self.a, self.b. You expect, the class's instance was created by calling the constructor having all these methods.
3.  Even when calling python class's method - they can be done obj.fun() OR className.fun(obj) ->
4.  So as we saw, the instance variables are specific to a specific instance.
5.  We can have the class variables which are like the java type static class variables specific to all instances of a class. So we can have a counter = 0 in class . increment it in constructor every new instance of the class created. It will be maintained since its a class level var(Like java static var)
6.  And to alter these class variables, you have to do className.varName = '0' and no obj.varName. The latter will change the val , just for the instance and not all. 

CLASS METHODS - 

1.  The same logic can be moved to a method, and give a annotation -  @clasmethod -> it can access and alter the class variables for whole class
2.  And these class methods are also used as alternative constructors. Just like the instance methods takes in first param as 'self'. The class methods first param is 'cls' which is the keyword for accessing the class. You can do - 
    1.  ```python
        class Employee:
            def __init__(self, name, age):
                self.name = name
                self.age = age
        
            @classmethod
            def create_employee(cls, name, age):
                return cls(name, age) --> just like outside you do Employee(name,age) -  here you do cls(..)
                
            def create_employee_in_instance_style(name,age):
            	return Employee(name,age)
        
        # Create an employee object using the class method
        employee = Employee.create_employee('John Doe', 30)
        ```
        
3.  Now even create_employee_in_instance_style is a instance method and will work same as the create_employee class method. Then why do we need class methods as alternative constructors.
4.  Becoz of the functionality that by design they receive cls keyword as 1st param and while returning we do cls(...) - so this is aware of sub classes and can create sub class's instance instead of the instance method where we dont know that.
5.  So basically, these class methods are the factory methods. 

STATIC METHODS - 

1.  class methods take cls as 1st param and instance methods take self. But static methods are generic methods (but reason why there are put inside class is coz it might have some logic to class). So @staticmethod are static methods

INHERITANCE - 

1.  When a class inherits other, by default all methods including the constructor is inherited.

SPECIAL METHODS - 

1.  In python, the methods with _ \_{name}_ _  are special methods, and the underscores are called as dunder. I.e. the init method is called as dunder init.
2.  Every operation we do like +, \*, len(), are all special methods. the 1 + 2 , is same as int._ _ add _ \_(1,2). So just like python implicitly calls init when you do class instantiation, thats the same for these +, len().. etc.
3.  So we can have these special methods custom to our class, in our class, we can have _ _ add _ \_(self, other) which we can use as obj1+obj2 which calls this special method. Similarly for len, \__str__ and \__rpr_\_. When you do print(obj) - the order is first it checks if class defined str special method, and then rpr method and finally if none, then the pointer address is printed. rpr is just like str method, but used more in the case for debugging , with more data printed out

PROPERTY AND DECORATORS - 

1.  By using @property annotation on a method, you can call it as if u call an attribute.
    1.  ```
        lass Employee:
        
            def __init__(self, first, last):
                self.first = first
                self.last = last
        
            @property
            def email(self):
                return '{}.{}@email.com'.format(self.first, self.last)
        
            @property
            def fullname(self):
                return '{} {}'.format(self.first, self.last)
            
            @fullname.setter
            def fullname(self, name):
                first, last = name.split(' ')
                self.first = first
                self.last = last
            
            @fullname.deleter
            def fullname(self):
                print('Delete Name!')
                self.first = None
                self.last = None
        
        
        emp_1 = Employee('John', 'Smith')
        emp_1.fullname = "Corey Schafer"
        
        print(emp_1.first)
        print(emp_1.email)
        print(emp_1.fullname)
        
        del emp_1.fullname
        ```
        
2.  Setter - As you can see, we called full name method as an attribute although theres no such. Similarly we can set it 
3.  DELETER - We have this used to do delete some property of the object. In above you could do del self.first also instead of = None.

## DJANGO 

1.  Similar to modules in sprint boot, we call it as apps in django. But Django apps are more self-contained and are designed to be plug-and-play, meaning you can easily add or remove an app from a project. On the other hand, Spring Boot modules are more tightly integrated into the project and may have dependencies on other modules, making them less portable.