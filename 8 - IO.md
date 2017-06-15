theme: Simple
autoscale: true


# IO

---

## Review

- __File__
    - some data
    - can be empty
    - can't contain other files / dirs
- __Directory__
    - _file_ containing a list of files / dirs
    - can contain other files / dirs
    - can be empty

---

```
[dfreniche@Tesla log]$ tree /var/log
.
├── Bluetooth
├── CDIS.custom
├── CoreDuet
├── DiagnosticMessages
│   ├── 2017.04.19.asl
│   └── StoreData
├── alf.log
├── apache2
├── appfirewall.log
├── asl
│   ├── 2017.04.12.G80.asl
│   ├── BB.2018.04.30.G80.asl
│   ├── Logs
│   │   ├── aslmanager.20170417T193440+02
│   │   └── aslmanager.20170419T084317+02
│   └── StoreData
├── com.apple.revisiond [error opening dir]
├── com.apple.xpc.launchd
├── corecaptured.log
├── cups
│   ├── access_log
│   ├── error_log
│   └── page_log
├── daily.out
├── wifi.log.0.bz2
└── wifi.log.1.bz2

```

---

## File System: Paths

- __relative__

 `mydir`, `./thisDir`, `bin/bash`

- __absolute__: start with the root directory

 `c:\\windows`, `/bin`

---

## File: `java.io.File`

- reads information about existing files & directories
- needed to create, delete, rename files, check directory contents... 
- needs a Path to be created

```java
File f = new File("/bin");

System.out.println("Getname:" + f.getName());
System.out.println("Canwrite: " + f.canWrite());
System.out.println("Exists:" + f.exists());
```

---

## Absolute vs Relative paths

- relative Path to our project / app

```java
File f = new File("bin");   
```

- absolute Path

```java
File f = new File("/bin");   
```

- we can create a file starting with a base path

```java
File root = new File("/");
File f = new File(root, "bin");
```

---

## System-independent paths

- `File.separator`: character used to separate parts of a relative / absolute path
    - on UNIX systems, __"/"__, on Windows __"\\"__
    - `/bin/ls`, `c:\\windows\\system32`
- `File.pathSeparator`: character used to separate directories in the `PATH` environment variable 😱
    - on UNIX systems, __":"__, on Windows __";"__

```java
File root = new File(File.separator);
File f = new File(root, "bin");
```

---

## Useful methods on File class

|   |   |
|---|---|
| exists() |          delete() |
| getName() |              lastModified() |
| getAbsolutePath() |      renameTo(File) |
| isDirectory()         | mkdir() |
| isFile()              | mkdirs() |
| length                | getParent() |
| listFiles()

---

## Stream magic

```java
Arrays.stream(f.listFiles()).sorted().forEach(System.out::println);
```

---

## Reading from the keyboard: old way

- `System.in` is a `java.io.BufferedInputStream`
    - raw stream of bytes
- to read from it, we need an `InputStreamReader`

```java
InputStreamReader in = new InputStreamReader(System.in);
int c = 0;
do {
    c = in.read();
    System.out.println(c);
} while (c != 32);
```

---

## Reading from the keyboard: old way

- this reads a _stream_ of bytes, one at a time. We need to read complete lines!
- to read complete lines, we use a `BufferedReader`

```java
BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));

String input = reader.readLine();
```

--- 

## Reading from the keyboard: "new" way

- since Java 6. New way... 🤔
- `Console` singleton class
- `Console` can be null!

```java
Console console = System.console();
if (console == null) {
    System.out.println("Null console");
    return;
}

String input =  console.readLine();
```

---

## Reader & Writer

- reader() ~= System.in

Retrieves the unique Reader object associated with this console.

```java
Console con = System.console();
 if (con != null) {
     Scanner sc = new Scanner(con.reader());
     ...
 }
```
---

## Console: format & printf

```java
console.writer().println("Hello");
console.writer().format("World %s", "!");
console.printf("This %s", "rules");
```

---

## flush, readLine, readPassword

- Flushes the console and forces any buffered output to be written immediately .


---

## Streams (file streams)

- a list of data elements
- if _Stream_ is in the name: all types of binary / byte data
    - `FileInputStream`
- if _Reader / Writer_ is in the name: chars / Strings
    - `FileReader`
    - is a _stream_ of chars / Strings (although Stream does not appear in the name)


---


## InputStream

- abstract class
- all children include "InputStream" in the name

```
                        +-------------+
                        | InputStream |
                        | <abstract>  |
                        +-------------+
                               ^
       +-----------------------|---------------------------+
       |                       |                           |
+-----------------+    +-------------------+    +-------------------+
| FileInputStream |    | FilterInputStream |    | ObjectInputStream |
| <low level>     |    |                   |    |                   |
+-----------------+    +-------------------+    +-------------------+
                                ^
                                |
                      +---------------------+
                      | BufferedInputStream |
                      |                     |
                      +---------------------+
```

---

## OutputStream

```
                        +--------------+
                        | OutputStream |
                        |  <abstract>  |
                        +--------------+
                               ^
       +-----------------------|---------------------------+
       |                       |                           |
+------------------+    +--------------------+    +--------------------+
| FileOutputStream |    | FilterOutputStream |    | ObjectOutputStream |
| <low level>      |    |                    |    |                    |
+------------------+    +--------------------+    +--------------------+
                                ^
                      ----------+-------------
                      |                      |
          +----------------------+      +-------------+
          | BufferedOutputStream |      | PrintStream |
          |                      |      +-------------+
          +----------------------+
```



---

## Reader

---

## Writer

---

## Serializable

```java
class Person implements Serializable {
    public static final long serialVersionUID = 1L;
    
    private String name;
    private transient String myId;

    public String getName() {
        return name;
    }

    public Person setName(String name) {
        this.name = name;
        return this;
    }

    public String getMyId() {
        return myId;
    }

    public Person setMyId(String myId) {
        this.myId = myId;
        return this;
    }
}

```


---