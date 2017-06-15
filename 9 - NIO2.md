theme: Simple
autoscale: true


# NIO2: New I/O api, v2

---

## NIO2: New I/O api, v2

- introduced in Java 1.4 as replacement for Streams
    - not used by anyone
- Java 1.7 brings up NIO2
    - still nobody uses it

---

## Path

- interface `java.nio.file.Path`
- understands symbolic links
- to create:

```java
Path bin = Paths.get("/bin");

// using varargs so no need to change between / & \
// the correct system-dependent path.separator gets added

Path bin = Paths.get("/", "bin");

```

---

## Path from URIs

```java
Path bin = null;
try {
    bin = Paths.get(new URI("file:///bin"));
} catch (URISyntaxException e) {
    e.printStackTrace();
}

System.out.println(bin.getFileName());

```

---

## FileSystem

- get a Path

```java
Path bin = FileSystems.getDefault().getPath("/bin");

System.out.println(bin.getFileName());
```

- all mounted volumes

```java
FileSystems.getDefault().getFileStores().forEach(System.out::println);
```

---

## Converting Path <--> File

```java
Path bin = FileSystems.getDefault().getPath("/bin");

System.out.println(bin.getFileName());

File f = bin.toFile();  // converts from "new" Path to "old" File

Path newBin = f.toPath();

System.out.println(newBin);

```
---

## Splitting the path into components

```java
Path bin = FileSystems.getDefault().getPath("/usr/local/bin");

for (int i = 0; i < bin.getNameCount(); i++) {
    System.out.println(bin.getName(i));
}

usr
local
bin
```

---

## Some utilities

```java
Path bin = FileSystems.getDefault().getPath("/usr/local/bin");

System.out.println(bin.getFileName());  // bin
System.out.println(bin.getRoot());      // /
System.out.println(bin.getParent());    // /usr/local

System.out.println(bin.isAbsolute());   // true
```

---

## Getting absolute path from relative

```java
Path bin = FileSystems.getDefault().getPath("hello.txt");

System.out.println(bin.toAbsolutePath());
```

---

## Subpath

```java
Path bin = FileSystems.getDefault().getPath("/usr/local/bin/this/that");

System.out.println(bin.subpath(0, 2));
System.out.println(bin.subpath(3, 5));

usr/local
this/that
```

---

## Testing file exists

```java
 Path bin = FileSystems.getDefault().getPath("/usr/local/bin/this/that");

System.out.println(Files.exists(bin));
```

---

## `Files`: Creating directories

```java
try {
    Path hello = Paths.get("hello");
    if (Files.exists(hello)) {
        Files.delete(hello);
    } else {
        Files.createDirectory(hello);
    }
} catch (IOException e) {
    e.printStackTrace();
}

```

---

## `Files`: Deleting directories

```java
Path hello = Paths.get("hello");
            
Files.deleteIfExists(hello);
```

---

## Other `Files` methods

- isSameFile
- copy
- delete
- move

---

## Printing all lines from a file

```java
try {
    Files.readAllLines(Paths.get("/Users/dfreniche/.bash_history")).
        stream().
        forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}

```

---

## Walking a directory

- print all files in `/usr/local`

```java
try {
    Files.walk(Paths.get("/usr/local")).forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}

```

---

- now just the MarkDown files, please

```java
try {
    Files.walk(Paths.get("/usr/local")).filter(f -> f.getFileName().toString().endsWith(".md")).forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}

```

---

- only directories

```java
try {
    Files.walk(Paths.get("/usr/local")).
        filter(f -> Files.isDirectory(f)).
        forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}
```

```java
try {
    Files.walk(Paths.get("/usr/local")).
        filter(f -> Files.isDirectory(f) && f.getParent().toString().equals("/usr/local") ).
        forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}
```

---

## Find files

- Predicate takes: Path & File Attributes

```java
Path home = Paths.get("/Users/dfreniche");

Stream<Path> foundJavas = null;
try {
    foundJavas = Files.find(home, 4, (p, a) -> p.toString().endsWith(".java"));
} catch (IOException e) {
    e.printStackTrace();
}
foundJavas.forEach(System.out::println);

```