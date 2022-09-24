People love to hate Java. One of the most common reasons people give for disliking it is that it is "verbose." I will explain why people think this. I will go over various features that make the language needlessly verbose. For a basis of comparison, I'll use Javascript, Typescript, Python, and C#.

1. [Boilerplate](#boilerplate)
2. [Import Statements](#import-statements)
2. [Getters and Setters](#getters-and-setters)
3. [No Default Arguments](#no-default-arguments)
3. [Constructors](#constructors)
3. [Initializing a Hashmap](#initializing-a-hashmap)
3. [Strict Typing](#strict-typing)
3. [Curly Braces](#curly-braces)
3. [Bad Lambda Support](#bad-lambda-support)
3. [No Operator Overloading or Indexers](#no-operator-overloading-or-indexers)

# Boilerplate

Any Java programmer has probably seen this a hundred times:

```Java
public class Main {
    public static void main(string[] args) {
        System.out.prinln("hello world");
    }
}
```

This is a ton of boilerplate just to run the simplest operation you can imagine, especially if we compare this to Python:

```Python
print "hello world"
```

All of that boilerplate makes the language seem very verbose immediately, and is even intimidating to new programmers. So why is it like this? We can go line-by-line:

```public class Main { ```

This line results from the first design principle of Java: that absolutely everything must happen inside of a class. You cannot just make functions outside of a class, even if that may be a better design for you. 

```public static void main(string[] args) {```

This line results from the next design principle of Java: that all behavior (print statements, mutation, etc.) must go inside of a function. In this case, the main function, which is the first function that gets called. That function is static and takes an array as an argument (both concepts that might be confusing to new programmers). Finally:

```System.out.println("hello world");```

There is perhaps no other action that you do as frequently as log something. In the process of debugging, many coders add and remove print statements all over the place with great frequency. Therefore, it is imperative that print statements should be as easy to write as possible. System.out.println is not short. But I guess the reason it looks like that results, in part, because of the first rule that everything needs to be encapsulated in a class. 

# Import Statements

This is an example of Java star imports:

```Java
import java.awt.*;
import java.sql.*;
import java.util.*;
```

It looks fine. Unfortunately, star imports are considered by many to be an antipattern in Java. Many IDEs/liters flag star imports as warnings. I used to assume the reason had something to do with performance. Apparently, the real issue is with naming conflicts. Adding more star imports can cause code to break because of ambiguity on which package to get things from. 

Ok, so you can't use star imports. But that means that the top of your Java files will all be littered with something like this:

```Java
import com.google.common.base.Preconditions.checkArgument;
import com.google.common.base.Preconditions.checkNotNull;
import com.google.common.base.Preconditions.checkState;
import com.google.common.graph.GraphConstants.ENDPOINTS_MISMATCH;
import com.google.common.collect.ImmutableSet;
import com.google.common.collect.Iterators;
import com.google.common.collect.Sets;
import com.google.common.collect.UnmodifiableIterator;
import com.google.common.math.IntMath;
import com.google.common.primitives.Ints;
import java.util.AbstractSet;
import java.util.Set;
import javax.annotation.CheckForNull;
```

I want to be clear that I'm not picking on the repositories I'm stealing the example code from. They may be well-done. I'm just trying to illustrate some of the pitfall of the language. 

Most IDEs automatically generate import statements, and the automatically hide them. As a general rule of thumb, when you IDE auto-generates code, and then auto-hides that code, that is a code smell for bad language design. That principle will come up again. 

Code should be written in such a way that it provides maximum information to the reader. Import statements are the first thing you see when open a file. If they are unreadable, they are failing at conveying useful information.

So is there a better way? There is. This is how you do imports in Typescript:

```Typescript
import { HttpClient, HttpErrorResponse } from "@angular/common/http";
import { Observable, throwError } from "rxjs";
import { catchError, map, tap } from 'rxjs/operators';
```

There are 7 imports total, but only 3 lines. Imports form the same source are compressed onto the same line, essentially eliminating the repeated code from the Java example. 

In Typescript, you can quickly skim over and say, "ok, this uses HTTP, and rxjs Observables, and rxjs operators." If the reader if familiar with those packages, they immediately get a vague overview of what the code will be doing. 

# Getters and Setters

A great deal of the verbosity of Java can be blamed on getters and setters.

```Java
class Demo {

    private int length; 
    private int width; 
    private int height; 

    public Demo(int length, int width, int height) {
        this.length = length;
        this.width = width;
        this.height = height;
    }

    public getLength() {
        return this.length;
    }

    public setLength(int length) {
        this.length = length;
    }

    public getWidth() {
        return this.length;
    }

    public setWidth(int width) {
        this.width = width;
    }

    public getHeight() {
        return this.height;
    }

    public setHeight(int height) {
        this.height = height;
    }
}
```

Java is a language I learned in university. I remember one day, the professor told us, "generally speaking, you should make all class variables private. There is almost never a good reason for a public field." One student chimed in, "if we're not allowed to use public variables, then we will just make getters and setters for everything." 

I think the response was something along the lines of, "You shouldn't expose everything. You should think hard before exposing something." The issue still remains, though: sometimes you do want a field exposed. You will find yourself wanting to implement a getter and setter, therefore. At that point, the variable is exposed, so why is it still private?

Getters and setters do provide some utility. You might want to do some action every time you update or access a variable (for example, log it, enforce constraints on changes, etc). Unfortunately, it is inadvisable to start with a public variable, and then later enforce the use of getters and setters should you need the functionality associated with it. That breaks backwards-compatibility. Therefore, developers opt to preemptively require the "private & getter & setter" trifecta.

Spring Boot, and probably other Java frameworks, actually require the use of getters and setters. Some developers use Lombok to auto-generate the required getters and setters at runtime. Others use their IDE to auto-generate them. These getters and setters are usually and the bottom of the class, lest anybody see them. Devs even sometimes use their IDE to automatically hide the getters and setters. This is the same code smell from before: when you are auto-generating something, and auto-hiding it, that is a sign of poor language design. 

Is there a better way? I think there is. I actually prefer how C# 3.0+ handles it.

```C#
public class Foo
{
    // getter and setter are attached directly to variable, on one line.
    public string bar { get; set; } 
}
```

Or in the case where you need the getter or setter to do something:

```C#
private string mFoo; // backing field
public string foo
{
    get { return mFoo; }
    set
    {
        mFoo = value;
        PerformSomeAction();
    }
}
```

I still don't like the fact that you need to use a backing field in order to make the getter or setter do something. However, if we need to introduce a backing variable, at least that change is internal to the class, and does not break backwards-compatibility. Therefore, this way is still superior to the Java style of making the user call getX and setX from the start.

```Java
myObject.setY(myObject.getX());
```

Hideous!

# No Default Arguments

Java has function overloading, which allows you to do the following:

```Java
public myFunction(int arg1, string arg2, boolean arg2) {
    // do something
}

public myFunction(int arg1, string arg2) {
    this.myFunction(arg1, arg2, true);
}

public myFunction(int arg1) {
    this.myFunction(arg1, "automate", true);
}

public myFunction() {
    this.myFunction(12, "automate", true);
}
```

Although it's good you can do this, it is verbose. Here is how you would do the same thing in Python:

```Python
def my_function(arg1=12, arg2="automate", arg3=True):
    # do something
```

# Constructors

```Java
class Person {
    
    String firstName;
    String lastName; 
    Birthday birthday;
    String nationality;
    int heightInInches;
    List<String> siblings;

    public Person(String firstName, String lastName, Birthday birthday, String nationality, int hightInInches, List<String> siblings) {
        this.firstName = firstName; 
        this.lastName = lastName; 
        this.birthday = birthday; 
        this.nationality = nationality;
        this.heightInInches = heightInInches; 
        this.siblings = siblings; 
    }

    // getters and setters
}

Person ben = new Person("Benjamin", "Keller", new Birthday(1, 2, 1992), "USA", 70, 
    new ArrayList<>(Arrays.asList("Sarah", "David", "Kelly")));
```

I have not explicitly defined the class Birthday, but I think it's clear how it works. 

Here's a question: how many times does the word "birthday" appear in the above code? I count 6 times, including the types. This violates the DRY principle.

The above code is second nature to most programmers. However, it is extremely confusing to beginners. I know that because I was once a beginner. I hate to admit it, but Java is the first object-oriented language I learned. The first thing I learned about the language, before even how to add and subtract, was class initialization with constructors. It was very confusing to me what everything did and why I had to repeat the same thing 6 times. 

In Python, it's a little better. We only repeat "birthday" 3 times. However, Python also isn't asking for the [type](#strict-typing), which is a loss.

```Python
class Person:
    def __init__(self, first_name, last_name, birthday, nationality, heightInInches, siblings):
        self.firstName = firstName
        self.lastName = lastName
        self.birthday = birthday
        self.nationality = nationality
        self.heightInInches = heightInInches
        self.siblings = siblings

ben = new Person("Benjamin", "Keller", new Birthday(1, 2, 1992), "USA", 70, ["Sarah", "David", "Kelly"])

```

The fundamental issue here, I think, is that we are trying to use a class as a structured store of data, much like a JSON object, and therefore we are trying to initialize it with a lot of data. What I would do in Typescript is not use a class at all, but rather, create an interface.

```Typescript
interface Person {
    firstName: string,
    lastName: string,
    birthday: Birthday,
    nationality: string,
    siblings: string[]
}

// use
const ben: Person = {
    firstName: "Benjamin", 
    lastname: "Keller", 
    birthday: {
        day: 1,
        month: 2,
        year: 1992
    }, 
    nationality: "USA", 
    heightInInches: 70,
    siblings: ["Sarah", "David", "Kelly"]
}
```

Unlike passing objects in through the constructor, each item is labeled, eg, "Benjamin" is labeled "firstName" so we know what it is. 

When passing arguments into a function/constructor, some IDEs (like IntelliJ) create [parameter hints](https://www.jetbrains.com/help/webstorm/viewing-method-parameter-information.html): artificial labels which are not actually part of the code. When your IDE has to insert phantom text to make your code readable, that's a code smell for poor language design. 

The ability to create JSON objects puts JS/TS, in my opinion, far above Java.

# Initializing a HashMap

When programming, I often want to create a Hashmap/dictionary/etc, and instantiate it with multiple key-value pairs at the same time. The trouble is Java can't decide on the standard way to do this. [This](https://www.baeldung.com/java-initialize-hashmap) tutorial page lists various options. I hate all of them. 

**Option 1**

```Java
Map<String, Integer> myKeyValueParisMap = new HashMap<>();
myKeyValueParisMap.put("key1", 1);
myKeyValueParisMap.put("key2", 2);
```

**Option 2** comes with a stern warning to never do it. It "it creates an anonymous extra class at every usage, holds hidden references to the enclosing object, and might cause memory leak issues." It's a shame because this is syntactically the best. 

```Java
Map<String, Integer> myKeyValueParisMap = new HashMap<String, String>() {{
    put("key1", 1);
    put("key2", 2);
}};
```

**Option 3** 

```Java
Map<String, Integer> myKeyValueParisMap = Stream.of(new Object[][] { 
     { "key1", 1 }, 
     { "key2", 2 }, 
}).collect(Collectors.toMap(data -> (String) data[0], data -> (Integer) data[1]));
```

**Option 4** only works for up to 10 elements.

```Java
Map<String, Integer> myKeyValueParisMap = Map.of(
    "key1", 1,
    "key2", 2
);
```

**Option 5**

```Java
import static java.util.Map.entry;
Map<String, Integer> myKeyValueParisMap = Map.ofEntries(
    entry("key1", 1),
    entry("key2", 1)
);
```

**Option 6**

```Java
Map<String, Integer> myKeyValueParisMap = Stream.of(
    new AbstractMap.SimpleEntry<>("key1", 1), 
    new AbstractMap.SimpleEntry<>("key2", 2))
    .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
```

I like option 5 best overall. However, what I really like best is to use another language, like JS/TS:

```Javascript 
const myKeyValueParisMap = {
    "key1": 1,
    "key2": 2
};
```

The difference is clear. Java is worse for instantiating lists/arrays, too. 

# Strict Typing

Let me start by saying that strict typing isn't bad. However, I can't omit it from this list, because it does indeed make Java more verbose. Referring back to the [Constructors](#constructors) section: almost half of the "redundancy" comes from the fact that we have to list the type. For instance: 

```Java
Writer myWriter = new Writer();
```

The word "writer" is repeated 3 times. We just made the reader read the same thing thrice.

Can this be better? I think there when you are creating and instantiating a constant on the same line, lack of typing is clearly superior. In Typescript, types are optional. I think this is great, because although you want to use types in most cases, sometimes they are clutter. For example: 

```Typescript
const myString = "hello world" // not typed
const myString: string = "hello world" // typed
```

Which of those two is better? Your linter may even flag the second line. It will say that the type "string" is redundant, because it is. We already *know* it's a string because quotes are there, and everyone knows quotes indicate string. The same goes for other types.

```Javascript
const myWriter = new Writer()
const myNumber = 12
```

Adding types would improve nothing. You know what type they are just by looking at their creation. 

Notice, however, that in these examples, the variables are constants. When you are mutating variables, it may help to add typing, to prevent reassignments into improper types. Personally, when I write code in Typescript, I almost always initialize variables with the ``const`` keyword. I almost never use the ``let`` or ``var`` keywords, because I like to follow functional design patterns, where variables do not change. This is in fact one of the things that I like about functional design patterns: in many cases, it obviates the need for typing. 

One case I am uncertain about is as follows:

```Typescript
const myArrayOfNumbers: number[] = new [1, 2, 4, "6", 8, 12, 145] // error!
```

I think it is useful to add the type here, so that it catches errors like the above. 

However, there is another case where I think typing is not necessary. If the variable has a very small scope, putting the creation close enough to object use, we can forgoe the safeguard of typing. For example, in the the case of array functions like the below, the typing in the reduce is too verbose and shouldn't be used.

```Typescript
const newArray: number[] = oldArray.reduce((previousArray: number[], currentArray: number[]) => {
    // do something
}, [])
```

# Curly Braces

You may prefer curly braces to just indenting. I'm not here to argue with you. Out of all of the items on this list, this item is the most stylistic and subjective. However, the requirement of curly braces does make Java code longer than Python code. The reason I think this matters is that I like for code to be compact. I like to be able to see my whole file of code without scrolling or using a vertical monitor. Closing curly braces take up an extra line. 

Here is some Java code: 

```Java
int solution(int[][] grid) {
    int sum = 0;
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == 1 && !visited.contains(i * grid.length + j)) {
                sum++;
                this.helper(grid, i, j);
            }
        }
    }
    return sum; 
}
```

Actually, you could have omitted the curly braces for the if statements, because the contents fit on one line. However, I consider this bad form, because it has a high risk of introducing bugs (if you need multiple lines in the if statement block, a lack of curly braces will introduce an error), and it creates inconsistent style.

The same code in Python:

```Python
def solution(grid):
    sum = 0
    for i, row in enumerate(grid):
        for j, val in enumerate(row):
            if val == 1 and (i, j) not in visited:
                sum += 1
                self.helper(grid, i, j)
    return sum
```

People complain about Python whitespace, but I have no idea what those people are talking about. What, are they coding in Notepad? I don't know about you, but I use a code editor that takes care of that for me. 

In addition to the lack of curly braces, I also want point out the quality-of-life improvements in Python. The loops are simpler to write and read, and the syntax for the if statement is simpler. 

# Bad Lambda Support

This is how you sort a list of objects by user-defined criteria in Java:

```Java
Collections.sort(users, new Comparator<User>() {
    @Override
    public int compare(User u1, User u2) {
        return u2.getCreatedOn().compareTo(u1.getCreatedOn());
    }
});
```

The alternative to this is to make user implement ``Comparable<User>``, and then define the compareTo method. This approach isn't scalable, though, in that it only allows you one way to sort the object. 

This is how you sort a list of object by user-defined criteria in Javascript:

```Javascript
users.sort((u1, u2) => u2.createdOn() - u1.createdOn());
```

I don't know about you, but that is much better. Javascript can do this because it has built-in support for lambdas, which allows for built-in support for array functions (ie, sort, forEach, map, filter, reduce, etc). When we want to do custom sorts, the functional approach is superior, but it is very clunky in Java, because Java's strictly object-oriented mentality is restrictive. 

# No Operator Overloading or Indexers

In Java, syntax exists that the users cannot implement their own uses of. In the following cases, it feels like Java doesn't trust its own users to create new implementations, lest they confuse themselves.

**Operator Overloading**

In C#, you can do the following

```C#
Vector3 newVec = new Vector3(10f, 105, 5f) + new Vector3(10f, 5f, 5f);
// newVec == new Vector3(20f, 15f, 10f)
Vector3 newVec = new Vector3(10f, 105, 5f) - new Vector3(10f, 5f, 0f);
// newVec == new Vector3(0f, 5f, 5f)
Vector3 newVec = new Vector3(20f, 10f, 5f) * new Vector3(10f, 5f, 5f);
// newVec == new Vector3(200f, 50f, 25ff)
Vector3 newVec = new Vector3(10f, 15f, 5f) / new Vector3(10f, 5f, 1f);
// newVec == new Vector3(1f, 3f, 5f)
```

You can do this in C# because it allows you to overload the operators ``+``, ``*``, etc. In Java however, you cannot do this. At best, you can do:

```Java
Vector3 newVec = new Vector3(10f, 105, 5f).add(new Vector3(10f, 5f, 5f));
```

I personally find that MUCH less clear. The syntax of operators is clarifying, and more readable via the use of whitespace.

For Java to disallow operator overloading is incompatible with the decision for Java to allow function overloading. Operators are functions. Java's not going all the way with its design philosophy. 

**Indexers**

To get and update the value of an ArrayList in Java, one must do:

```Java
// Get the value of an ArrayList
int val = myList.get(index);
// Update the value of an ArrayList
myList.set(index, element);
```

But in most languages, you can do something like the following:

```Python
# Get value of variable-length-list
val = my_list[index]
# Update the value of variable-length-list
my_list[index] = element
```

Of course, Java uses indexers for arrays. Java programmers know how to use indexers, because they exist in the language, but exclusively for that one data structure. Other data structures (ArrayLists, HashMaps, LinkedLists, etc.) cannot use them. I consider this an inconsistency. 

I use Python in the that example, but C# allows you to access the value of ArrayLists with almost the same syntax. This is how you set up an "indexer" in C#, allowing you to access items through [square brackets]:

```C#
// Defining Linked List
class LinkedList {  
   Node head;  
   // Setting up indexers
   public int this[int i]
   {
      // get indexer allows square brackets to read data
      get => this.getAt(i);
      // set indexer allows square brackets to change data
      set => this.setAt(i, value);
   }
}
```

Java syntax may have seemed fine so far, but it gets very unwieldy. Java HashMaps, an extremely commonly used data structure, do not use indexers. 

```Java
// Add to the value in a HashMap in Java
myHashMapWithLongName.put(myKey, myHashMapWithLongName.get(myKey) + amountToAdd);
// Subtract from the value in a HashMap in Java
myHashMapWithLongName.put(myKey, myHashMapWithLongName.get(myKey) - amountToAdd);
// Bonus: add to value in 2d ArrayList in Java
my2dArrayList.get(i).put(j, my2dArrayList.get(i).get(j) + toAdd);
```
I believe this violates the DRY principle because you repeat almost everything twice. The first two lines could be very easily mixed up if not for the adjacent placement. Compare to other languages:

```Python
# Add to the value in a hashmap/dictionary/object in other languages
myHashMapWithLongName[myKey] += amountToAdd
# Subtract from the value in a hashmap/dictionary/object in other languages
myHashMapWithLongName[myKey] -= amountToAdd
# Bonus: add to value in 2d variable-length-array in other languages
my2dArray[i][j] += toAdd
```

In the Java example I get confused by what's going on. I especially found it weird as a beginner. Liberal use of the [square brackets] give other languages a ton of simplicity. 
