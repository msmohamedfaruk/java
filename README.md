<style>
    red {color:red}
    green {color:green}
</style>


## Time to have a quick notes on what we've learned

1. [Constructor Overloading](#constructor-overloading)
1. [Interfaces](#interfaces)
    1. [Default Methods](#default-methods)
1. [Jvm Internals](#jvm-internals)
1. [Exceptions](#exceptions)
1. [I/O](#i/o)

### Constructor Overloading
_Thumb Rule:_
+ this() invocation 
    - should be done only via **this** operator not by _classname_
    - should be appeared first
    - should be appeared only once within the constructor context
    - shouldn't carry instance variable as one of the args
    - shouldn't be recursive either directly or indirectly 
### Interfaces _Interesting Facts:_
+ When the subclass implements the interface, It need to provide only the definitions of un-implementated methods over the hierarchy.
    * For e.g: 
    ``` JAVA
    public interface A {
        void foo();
        void bar();
    }
    // skeleton implementation
    public abstract class AbstractA {
        public void bar() {
            System.out.println("AbstractA: bar()");
        }
    }
    public class SubClass extends AbstractA implements A {
        public void foo() {
            System.out.println("SubClass: foo()");
        }
        // No-Compilation Error, since bar() implementation will be used up from its inheritance tree (Abstract)
    }
    ```

+ #### Default Methods
    1. can be implemented since **Java 1.8** 
    1. is binary compatible (i.e no need to recompile the implementation code) rather _source compatbile_
    1. can be invoked only via instance not via Reference type
    1. **_Thumb rule_** on managing conflict resolution strategy across different context:
        - ##### method overriding
            + Always class wins over interface 
            + Hierarchically sub type of interface always wins over super type.
        - ##### method invocation
            ``` JAVA
            public interface A {
                void foo();
            }
            public interface B {
                void bar();
            }
            public interface C extends A {
                void fooBar();
                default void go() {
                    sysout("C: go");
                }
            }
            // skeleton implementation
            public abstract class AbstractA {
                public void foo() {
                    sysout("AbstractA: foo");
                }
                public void go() {
                    sysout("AbstractA: go");
                }
            }
            public class X extends AbstractA implements A, B, C {
                public void bar() {
                    sysout("X: bar");
                }
                public void fooBar() {
                    sysout("X: fooBar");
                }
            } 
            public class AppMain() {
                public static void main(String[] args) {
                    C c = new X();
                    c.foo();
                    c.bar();
                    c.fooBar();
                    c.go();
                }
            }
            ```

### Jvm Internals
+ Class Loader
    1. Parent delegation model
        1. Bootstrap ClassLoader (primordial)
        1. Extension ClassLoader
        1. Application ClassLoader
        1. User-Defined ClassLoader
+ Linking
    - Verification
    - Preparation
    - Resolution
        * Eager
        * Lazy Loading
+ Initializer

+ Runtime memory areas
    - Heap
        1. Class Object resides on Heap and it refers the Class data unidirectionally to utilize method table on the time of method ivocation.
    - Method Area
        1. Class data (Meta information) resides in Method area, bidirectional linkage made b/w them for following use cases:
            + Class data itself holds the **reference of Class Object** to reporting back the status (reference) to JVM when it is accessed for subsequent times.
            + Class object retrieves the meta information from class data while dealing with reflection
        1. Run time Constant Pool
        1. Field Info (Name, Type & Modifiers like final, static, public)
        1. Method bytecode
    - Method Table
        1. An array object contains the reference of starting addresses of all the methods available in this class via method bytecode
    - Stack
        1. Local variables
        1. Operand Stack
            * Familiarized Opcodes:
                - load
                - store
                - dup
                - ...
    - PC

+ Garbage collection

    - __Thumb Rule__:
        * Identify the live objects
        * With minimal *stop-the-world*

    - Mark & Sweep
    - Mark-Sweep-Compact
    - Mark & Copy

    - Generational collections
        * minor GC
            1. eden
            1. from
            1. to
        * full GC
            1. run after n-th (default=15) iteration of minor GC
        * Further optimizing Pause-time via
            Types | Young Gen| Tenured Gen |
            :---|:---|:---|
            SingleGC | Mark & Copy | Mark-Sweep-Compact|
            MultipleGC | Mark & Copy | Mark-Sweep-Compact|
            CMS | Mark & Copy | Concurrent Mark-Sweep (Mostly)|

### Exceptions
+ Types
    * __Checked__
    * _UnChecked_
+ Inheritance Tree Structure
    * Throwable
        - Exception
            1. __IOException__
                1. FileNotFoundException
            1. _Runtime Exception_
                1. NullPointerException
                1. ArrayOutOfBoundException
                1. IllegalAccessException
                1. ClassCastException
        - _Errors_
            1. VirtualMachineError
                1. OutOfMemoryError
            1. LinkageError
                1. NoClassDefFoundError

+ **Thumb rule** on Method Overriding
    * *SuperClass throws CheckedExceptions*, then
        <ol>
        <green>
        1. SubClass throws CheckedExceptions | SubType of CheckedExceptions | No Exceptions<br/>
        2. SubClass throws UnCheckedExceptions<br/>
        </green>
        <red>
        1. SubClass does not throw ParentType of CheckedExceptions otherwise it violates the contract
        </red>
        </ol>

    * *SuperClass throws UnCheckedExceptions*, then
        <ol>
        <green>
        1. SubClass throws UnCheckedExceptions | SubType of UnCheckedExceptions | No Exceptions<br/>
        </green>
        <red>
        1. SubClass doesn't throw neither ParentType of UnCheckedException nor CheckedExceptions otherwise it breach the contract
        </red><br/>
            but for convention, UnChecked Ex should be added in DOCs not in Declaration.
        </ol>

+ try-with-resources
    ``` JAVA
    try (final obj = java.lang.AutoCloseable) {}
    ```
    * Needs to clarify/recap: calrified
        1. what would happen if close() throws IOException which is not being captured in the upper hierarchy (invoking method)
            - Ans: Since IOException is checkedException, compiler will generate error neither the exception is handled nor it is declared as throws clause.
    * Items from Effective JAVA about Exception: 
        - should be used only in handling Exceptional situation
        - shouldn't be ignored by placing empty body
        - UnChecked Exception should be documented properly with possible use cases.
        - should be used in form of more specific rather than Generic types
        - should be wrapped up with Higher level and Lower level environment details (by means of translation & chaining)

### I/O
+ Encoding-Scheme
    - Unicode
        1. UTF-8
        1. UTF-16
        1. UTF-16BE
        1. UTF-16LE
    - BMP (Basic Multilingual Plane)
        - represented by 1 code point (16 bit)
    - Supplementary (Non-BMP)
        - represented by 2 code point (4 bytes)

+ Streams Classification over source/sink:
    - Byte Streams
        * InputStream
            1. FileInputStream
            1. FilterInputStream
                1. BufferedInputStream
            1. DataInputStream
        * OutputStream
            1. FileOutputStream
            1. FilterOutputStream
                1. BufferedOutputStream
            1. DataOutputStream        
    - Character Streams
        * Reader
            1. FileReader(Reader)
            1. InputStreamReader(InputStream)
            1. BufferedReader(Reader)
        * Writer
            1. FileWriter(Writer)
            1. OutputStreamWriter(OutputStream)
            1. BufferedWriter(Writer)

+ (De) Serialization:
    * __Thumb Rule__:\
        - Serialization
            1. The object being serialized must implement Serializable
            1. Also the same is applicable for it's Object graph
            1. Multiple references of the same object over the graph path will be serialized only once.
                1. Serialization happened only at the first attempt
                1. Reset of them uses the same serialized reference. 
        - DeSerialization:
            <ol><green>
            1. add/delete instance variables/methods<br/>
            2. change access modifiers<br/>
            3. change static member to instance member
            </green></ol>
            <ol><red>
            1. shouldn't change the field type<br/>
            2. shouldn't remove the Serializable implements over the object graph path<br/>
            </red></ol>
    * Helper Class:
        - ObjectInputStream implements ObjectInput
            1. is.readObject();
        - ObjectOutputStream implements ObjectOutput
            1. os.writeOject(Object instanceof primitive|Arrays|String|Enum|Serializable);
        1. *Symmentric Iteration order* needs to be maintained over read and write operation
    * Interesting Facts:
        - DeSerialization won't invoke constructor unless untill any of the Ancestors doesn't implement Serializable
    
