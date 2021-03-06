Lambdas
=======
In previous chapter we thought of removing ``GroupByExperience`` class, only the method body should be given to our *group()* function and Lambdas are the best examples of implementing them. Lambdas give us the ability to encapsulate a single unit of code block and pass on to another code. It can also be considered as anonymous function which doesn't have any function name but has list of parameters, function body, returns result and even throws exceptions. Below is the code statement if we convert our ``GroupByExperience`` class to a lambda expression.

.. code:: java

	(Employee e) -> { return e.yearsOfExpr >= 7 ? 
                        "Expert" : e.yearsOfExpr >= 3 ? "Intermediet" : "Fresher"; }

Basically *Lambda* has 3 parts.

#. **A list of parameters** : In above example "Employee e"
#. **Function body**		: The behavior (right hand side of arrow)
#. **An arrow**				: Separator between parameter list and function body

.. note:: Lambda syntax follows some of the below rules.

	* Parameter types are optional.
	* If you have single parameter then both parameter type and parenthesis are optional.
	* If you have multiple parameters, then they should be enclosed with in parenthesis.
	* For multiple statements in function body should be enclosed with in courly braces.
	* If lambda body exnclosed inside courly braces then ``return`` keyward is required in case your behavior returns value.

With applying above rules our ``GroupByExperience`` class can be writtten in following ways.

.. code:: java

	e -> { return e.yearsOfExpr >= 7 ? 
                "Expert" : e.yearsOfExpr >= 3 ? 
                         "Intermediet" : "Fresher"; }
	
	e -> e.yearsOfExpr >= 7 ? 
                "Expert" : e.yearsOfExpr >= 3 ? 
                        "Intermediet" : "Fresher"

|
|
	Below are some more examples of lambda expressions.
	
	#. `BiConsumer<List, Integer> addIntoList = (List list, Integer element) -> list.add(element);`
			Adding an element to a given list.
	#. `Predicate<Employee> isJavaEmp = e -> "Java".equals(e.technology);`
			Checking an employee is is from Java technology.
	#. `Supplier<Integer> uniqueKey = () -> new Random().nextInt();`
			Generate a unique number with the help of generator.

	
We saw couple of more lambda expressions above but what are these left hand side classes (BiConsumer, Predicate etc). If you remember in behavior parameterization chapter I mentioned, what if ``Groupable`` interface were given by java it self then we don't have to write our own interfaces or abstract classes to give different different implementations. Java 8 has already came up with bundle of general purpose functional interfaces which are included as part of JDK and we are going to visit them soon.

Now we have some ideas on how lambdas look like and their syntaxes. Just look into the below two lambda expressions.

.. code:: java
	
	1. Runnable runnable	= () -> "I love Lambdas".length();
	2. Callable<Integer> ca	= () -> "I love Lambdas".length();

Both of expressions look similar except the left hand side target types. You would be thinking how does it possible to assign same object to two different types of references. Is the Callable extends Runnble or vice-versa? The answer is a big NO, this is possible due to the `type inference` feature which decides the target type depending upon the context where it is used.



Type Inferencing
^^^^^^^^^^^^^^^^
There has been much more improvent in compiler intelligence level that it takes advantage of target typing to infer the type parameters of a generic method invocation. When inially Generics introduced in JDK 1.5, the type of generic was mandatory in both side of the expression. For example: 

`List<String> list = new ArrayList<String>();` 

But in JDK 1.7 right hand side generic type become optional by changing it to the diamond(<>) operator where type is evaluated from it's left hand side target type declaration. Still there were some limitations in generic type evaluation.

.. code:: java

	1. List<String> l = Arrays.asList();

	
	2. List<String> list = new ArrayList<>();
	   list.addAll(Arrays.asList());

If you compile above code in JDK 1.7, then the statement-1 will be compiled successfully but not statement-2 and it will generate ``The method addAll(Collection<? extends String>) in the type List<String> is not applicable for the arguments (List<Object>)`` error message. So what really happened in statement-2 where as both of the statements looks similar. Just look into the signatures of these methods.

+---------------------------------------------------+ 
|     Method Signatures                             | 
+===================================================+ 
| public static <T> List<T> asList(T... a)          |
|                                                   |
| public boolean addAll(Collection<? extends E> c)  | 
+---------------------------------------------------+ 

The asList() is a type safe method which is able to infer its return type based on the given direct target type but in addAll() case, compiler didn't have idea to deduce the type when applied on method parameter as target type and asList() method returned List<Object> that is incompatible with List<String> reference. Java 8 has enhanced this `type inferencing` technique to deal with such wierd scenarios. Now let's see how type inferencing works in lambda expressions.

The type of lambda is deduced from the context where it is used. If we take our earlier example of Runnable and Callable, the signature of lambda expression matches with the singature of run() and call() methods. Runnable class run() method neither accept any argument nor return anything. Our lambda expression ``() -> "I love Lambdas".length()`` also doesn't supply any parameter.

.. code:: java

    For run() method fully described lambda expression is
    () -> {
        "I love Lambdas".length();
    }

	
    and for call() it is
    () -> {
        return "I love Lambdas".length();
    }

Java compiler always looks for a matching functional interface to associate with the lambda expression from it's surrounding context or target type. Compiler expects you to use lambda expresssion in following places such that it can determine the target type.

	- Variable declarations
	- Assignment statements
	- Return statements
	- Method or constructor arguments
	- Lambda expression bodies
	- Ternary expressions (?:) etc

For method or constructor arguments, the compiler determines the target type with two other language features: `overload resolution` and `type argument inference`. Look into the below code snippet.

.. code:: java

    public static void main(String[] args) throws Exception {
        execute(() -> "done");  // Line-1
    }

    static void execute(Runnable runnable) {
        System.out.println("Executing Runnable...");
    }

    static void execute(Callable<String> callable) throws Exception {
        System.out.println("Executing Callable...");
        callable.call();
    }

    /* static void execute(PrivilegedAction<String> action) {
        System.out.println("Executing PrivilegedAction...");
        action.run();
    } */
	
	
    Output: Executing Callable...

Here we have two overloaded methods: Runnable and Callable. When you call the execute method with the mentioned lambda, the ``execute(Callable)`` will be called because the lambda ``() -> "done"`` says `I am not accepting any argument but I will be returning` *done* and Callable.call() method can return something.

.. note:: Don't be confused with why Runnable is not called. In lambda ``() -> "done"``, "done" is a return statement not any assignment statement so that Void will be returned. Suppose lambda exxpression would be ``() -> {String s = "done";}`` then it must have called Runnable.

Now just uncomment `execute(PrivilegedAction)` method and try to reexecute and this time you will get compilation error: `The method execute(Callable<String>) is ambiguous for the type Lambdas`. The reason is both the last two execute() methods are capable to return and compiler found the ambiguous methods. So to resolve this you have to explicitly type cast the lambda expression as below.

	`execute((Callable<String>) (() -> "done"));`


Accessing outer scope variables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Some of the rules applicable for anonymous classes are also applicable to Lambdas:
	- Lambda has access to members of its enclosing scope. (see line-1)
	- Like nested class or anonymous class, it can also shadows any other declarations in the enlosing scope that is of same name. (see line-2)

.. code:: java

    public class LambdaFeatures {
        private int x = 10;

        public void example() {
            Consumer<String> funcInterface = str -> {
                System.out.println("x= " + x);  // Line-1

                int x = 50;                     // Line-2
                System.out.println("x= " + x);
            };
        }
    }
	
    Output: x= 10
            x= 50
	
Restrictions in Lambdas
^^^^^^^^^^^^^^^^^^^^^^^
Lambda has some restrictions:

	- You can't declare any static or non-static initializers.
	- It cann't access local variables in its enclosing scope that are not defined final or effectively final. This restriction exists with anonymous class also. Let's discuss why is this limitation with following code snippet.

.. code:: java

    public class LambdaFeatures {
        int y = 50;
		
        public static void main(String[] args) throws Exception {
            int x = 50;

            Thread tt = new Thread() {
                public void run() {
                    System.out.println("MyThread start.");

                    Thread.sleep(1000L);
					
                    System.out.println("MyThread end. x=" + x);
                }
            };

            t.start();
			
            x++;
            System.out.println("main end");
        }
    }

Local variables stored in the stack where as instance variables stored in heap. In the above code snippet main thread declares variable "x" and also creates a Thread which is trying to use this x variable. As we know local variables will be stored in the local stack (here stack of main) and when thread "tt" will be created it will executed separate to main thread. There might be chances that main will be completed first and the stack will be released before thread tt trying to use it. So if variable is declared final, them lambda will take a copy of it and use whenever require.

Where to use Lambdas
^^^^^^^^^^^^^^^^^^^^
We have discussed enough on lambdas and anonybmous classes. Let's discuss the scenarios where should we use them.

	- **Anonymous class:** Use it whenever you want to declare some additional fields (member variables) or methods which lambda cann't do.
	
	- **Lambda:** 
		* Use it if you want to encapsulate a single unit of behavior and pass to some other code. For example: performing certain operation on each element of collection.
		
		* Use it if you need a simple instance of a functional interface where you do not need a constructor, a named type, fields, or additional methods.