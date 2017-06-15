theme: Simple
autoscale: true


# Exceptions

```
Object
  |
Throwable
  |----------------|
  |                |
Exception         Error
  |
RuntimeException
```

---

## Checked vs Unchecked

- all exceptions occur at runtime
- checked: handle or declare: compiler forces us
- unchecked or _runtime_: we don't need to handle or declare
    - because they're too common: NullPointerException, ArrayIndexOutOfBoundsException, etc.

---

## Should I "catch" Errors? 🤔

- never, ever
- never
- really
- don't do it
- ever
- I'm serious
- don't


---

## Exception on OCP

- study! (page 288)

---

## Try-catch-finally

```java

try {
	// do something that can throw
} catch (Exception e) { 
    // do something if this exception is injected
} finally {
    // do this always
}

```

---

## Try-catch gotchas

* Error! Needs a block

```java
try
	int i = 0;
catch (Exception e) ;
```

---

## Try-catch gotchas

* Error! Try need a catch / finally

```java
try  {
    System.out.println("2");
}   // error!
```

* This is fine

```java

try {
	int i = 0;
} catch (Exception e) { }
```

---
## Try-catch gotchas

Just __one__ try, please

```java

try {
		
} try {
		
} catch (Exception e) {}
```


---

## Try-catch gotchas

```java
// we can have a try-catch with Throwable
try {
	int i = 0;
} catch (Throwable t) {
    //  Pokémon catch: Gotta catch'em all!
}

try {
	int i = 0;
} catch (Exception t) {
	// i++;               Error: i - local to try block
}

```

---

## Try-catch gotchas


```java 
// can have more than one catch
try {
	int i = 0;
} catch (Exception e) {

} catch (Throwable t) {

}

try {
	int i = 0;
} catch (Throwable t) {

} catch (Exception e) { // error: Exception has already been caught

}
```

---

## Finally always gets called

```java
static void print(String s) {
    try  {
        System.out.println("2");
        if (s == null) return;
    } finally {
        System.out.println("3");
    }
    System.out.println("1");

    System.out.println("2");
}
```

---

## Creating your own exceptions


```java
// Runtime, non-checked Exception 
class NonCheckedException extends RuntimeException {
    public NonCheckedException() {  // typical constructors
    }

    public NonCheckedException(String message) {
        super(message);
    }

    public NonCheckedException(Exception cause) {
        super(cause);
    }
}

// Compile-time, checked exception, handle or declare rule
class CheckedException extends Exception {

}

```

---

## Catch-Or-Declare rule (Checked exceptions)

```java 
public class Main {
    public static void main(String[] args) {
        throwsNonChecked(); // no problem with non-checked
        try {
            throwsChecked();
        } catch (CheckedException e) {
            e.printStackTrace();
        }
    }

    static void throwsNonChecked() {
        throw new NonCheckedException();
    }

    static void throwsChecked() throws CheckedException {
        throw new CheckedException();
    }
}

```

--- 

## What does this prints? 😈

```java 
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        try {
            System.out.println("DogeBegin");
            new Main().saveToDisk();
        } catch (IOException e) {
            System.out.println(e.getMessage());
        } finally {
            System.out.println("DogeFinally");
        }
    }

    void saveToDisk() throws IOException{
        throw new IOException("Such litte space. Much files. Wow");
    }
}
```

---

## What does this prints?

```
DogeBegin
Such litte space. Much files. Wow
DogeFinally	
```

---
## What does this prints? 😈

```java 
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        try {
            System.out.println("DogeBegin");
            new Main().saveToDisk();
        } catch (IOException e) {
            System.out.println("IODoge" + e.getMessage());
        } catch (Exception e) {
            System.out.println("ExceptionDoge"+ e.getMessage());
        } finally {
            System.out.println("DogeFinally");
        }
    }

    void saveToDisk() throws IOException{
        throw new IOException("Such litte space. Much files. Wow");
    }
}
```

---

## What does this prints?

```
DogeBegin
IODogeSuch litte space. Much files. Wow
DogeFinally

```


---


## What does this prints? 😈

```java
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        new Main().run();
    }

    void run() {
        try {
            System.out.println("DogeBegin");
            saveToDisk();
        } catch (IOException e) {
            System.out.println("IODoge" + e.getMessage());
            return;
        } finally {
            System.out.println("DogeFinally");
        }
        System.out.println("DogeReturn");
    }

    void saveToDisk() throws IOException{
        throw new IOException("Such litte space. Much files. Wow");
    }
}
```
---

## What does this prints?

```

DogeBegin
IODogeSuch litte space. Much files. Wow
DogeFinally
```

---
## What does this prints? 😈

```java
import java.io.IOException;

public class Main {
    public static void main(String[] args) {
        new Main().run();
    }

    void run() {
        try {
            System.out.println("DogeBegin");
            saveToDisk();
        } catch (IOException e) {
            System.out.println("IODoge" + e.getMessage());
            return;
        } finally {
            System.out.println("DogeFinally");
        }
        System.out.println("DogeReturn");
    }

    void saveToDisk() throws IOException{
        throw new RuntimeException("Such litte space. Much files. Wow");
    }
}

```

---

```
Exception in thread "main" java.lang.RuntimeException: Such litte space. Much files. Wow
DogeBegin
	at Main.saveToDisk(Main.java:22)
DogeFinally
	at Main.run(Main.java:11)
	at Main.main(Main.java:5)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:134)
```

---

## Multicatch

```java
try {
            
} catch (ArrayIndexOutOfBoundsException | ArithmeticException e) {  
    // can't use parent & child here
} finally {

}

try {

} catch (Exception | ArithmeticException e) {   // won't compile

} finally {

}

```

---

## Multicatch is effectively final

```java
try {

} catch (ArithmeticException  | ArrayIndexOutOfBoundsException e) {
    e = new ArithmeticException();  // error: e is final
} 
```

- but this compiles!

```java
try {

} catch (ArithmeticException  e) {
    e = new ArithmeticException();
} finally {

}
```

---


## Remember we can catch something that can be thrown

```java
static void print(String s) {
    try {
        thrower();
    } catch (SQLException e) {

    } 
}

static void thrower() throws ArrayIndexOutOfBoundsException {
    
}
```

---

## Try-with-resources

- need that Connection and Statement implement AutoCloseable
- AutoCloseable implemented by more that 150 classes!

```java
try (Connection connection = DriverManager.getConnection(url);
    Statement statement = connection.createStatement()) {

}
```

---

## Try-with-resources

- no need for that try
- here's how this magic works

```java
try (Connection connection = DriverManager.getConnection(url);
    Statement statement = connection.createStatement()) {

} finally {
    statement.close();  // these are the 1st two lines in the finally block
    connection.close(); // like super / this in a constructor

    // can't use the statement here: closed!
}
```

--- 

## AutoCloseable

```java
try(MyFile file = new MyFile()) {

}
// ...
class MyFile implements AutoCloseable {
    public MyFile() {
        super();

        System.out.println("Creating " + this);
    }

    @Override
    public void close() {
        System.out.println("Closing " + this);
    }
}

```

---

## Autocloseable is a functional interface...

```java
try (AutoCloseable c = new AutoCloseable() {
    @Override
    public void close() {

    }
}) {

} catch (Exception e) {
    
}
```

- as lambda

```java
try (AutoCloseable c = () -> {

}) {

} catch (Exception e) {

}

```

---

## Assertions

- conditions that can never happen while the program is running
- they're absurd, but we want to catch them while developing
- we need to enable assertions (-ea) Edit Configurations > VM Options

---

## Useful asserts

- we're on the main thread, or in a background thread
- check exhaustive switch (with assert that fails in default)
- check an else is never met
- check some object that shouldn't be null is never null...

```java
// check we're running on Main Thread

assert (Thread.currentThread().getName().equals("main"));
```