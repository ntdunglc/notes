---
title: "Book summary: Working effectively with legacy code"
date: 2021-02-21
draft: false
---
# Part 1: The mechanic of change

## Chapter 1: Changing software
My take is to not do more than 1 change at the same commit/merge request, you either add a feature or refactor, but not both

|                        | Adding a Feature | Fixing a Bug | Refactoring | Optimizing |
|------------------------|------------------|--------------|-------------|------------|
| Structure              | Y                | Y            | Y           | -          |
| New funtionality       | Y                | -            | -           | -          |
| Existing Functionality | -                | Y            | -           | -          |
| Resource usage         | -                | -            | -           | Y          |


## Chapter 2: Working with feedback
We want to quickly have feedback when editing code, for example, run unit tests every time editing code assuming the tests are fast enough  
The Legacy Code Dilemma: when we change code, we should have tests in place. To put tests in place, we often have to change code.  
1. Identify change points.
2. Find test points (sometimes test point is not the same with change point, for eg we can test an outer function that use the function we change, see chapter 11, 12)
3. Break dependencies (all other chapters are about this).
4. Write tests.
5. Make changes and refactor.

## Chapter 3: Sensing and separation
1. Sensing: We break dependencies to sense when we can’t access values
our code computes. In the example, our code call display.showLine(line) but
we can not get value of that line
2. Separation: We break dependencies to separate when we can’t even get
a piece of code into a test harness to run. In the example, display is
a real hardware that need to fake
Fake/Mock is important tool to break dependency

## Chapter 4: The seam model
A seam is a place where you can alter behavior in your program without editing in
that place.  Every seam has an `enabling point`, a place where you can make the decision to use
one behavior or another.  
The goal of finding seam is to add tests without editing (too much) the production code  
- Preprocessor seam: C++ macros to run different code for prod and test environment
- Link seam: C/C++ compiler, Java classpath. Separation is often a reason to use a link seam (ie, can not get a class under test so we fake its dependency)
- Object seam: lots of chapters are about object seams, a common technique is using inheritance
to overwrite some behavior

## Chapter 5: Tools
- Automated Refactoring tools: for eg, extract method
- Mock object: is fake object + assertion
- Unit test harness
- General/Integration test harness

# Part 2: Changing software

## Chapter 6: I don't have much time and I have to change it
This chapter tries to convince you to write the test for future code reading and less frustration  
Easy techniques:
- sprout method/class: write new feature in new method/class (with test) and call from existing code, instead of editing existing code.
- wrap method: rename old method, create a new method with the same signature that call old method + new logic 
- wrap class: similar to decorator pattern, except we rename old class and give that name to new class

## Chapter 7: I takes forever to make a change
Lag time (ie: feedback time): fastet feedback enable different workflow (unit test to the rescue!)  
Compiled language might be slower to get feedback. It can help to break dependency so code depends on an interface instead of a concrete class

## Chapter 8: How do I add a feature
TDD worflow
1. Write a failing test case.
2. Get it to compile.
3. Make it pass.
4. Remove duplication.
5. Repeat.

Programming by difference: is a popular technique for incremental object-oriented programming, whereby a new class is built from an existing class using the inheritance mechanism (be careful to not abuse inheritance!). There's an article <https://twasink.net/2005/02/07/programming-by-difference/> that discuss some related points

## Chapter 9: I can't get this class into a test harness
1. Objects of the class can’t be created easily.
2. The test harness won’t easily build with the class in it.
3. The constructor we need to use has bad side effects.
4. Significant work happens in the constructor, and we need to sense it.

The irritating parameter: Fake that parameter; Pass null;  
Hidden dependency: if constructor creates other objects, we can create a new overloading contructor that accept those objects to break the hidden dependency  
Construction blob: for java & C#, use Extract and Override Factory
Method (ie call another intialize method in constructor and let subclass override it). 
For C++, since we can not call subclass method in constructor, use Supersede Instance Variable, which is a setter to override a class variable after calling constructor  
Irritating global dependency: for global Singleton dependency, consider adding a setter setTestingInstance to overwrite the Singleton instance  
The horrible include dependencies: ??  
The onion parameter: when complex class require other complex classess in constructor, options can be Pass null or Extract Interface to break dependency  
Aliased parameter: (dont know why it's called aliased parameter) when the parameter is a subclass in the middle of hierachy tree, it's hard to Extract Interface, we can Subclass and Override that class in tests

## Chapter 10: I can't run this method in a test harness
Private method: make it public to test it. In language like Rust and Go, tests can be in the same module/package and has access to private method  
The method has parameters of class A that are non-trivial or impossible to construct: it requires some work. We can create an interface that mimic what we use from class A, and create our production class which wrap class A, and fake class for test. Our justifications is that if we depends on a library that is out of our control, it's already a problem, we should always wrap 3rd party library with our own class.  
Undetectatable side effect: the method call other objects' or global method, and cause side effect, for eg, creating a GUI window and change its title, there's no way to sense it.

## Chapter 11: I need to make a change, what method should I test?
Characterization tests: to capture the current behavior, but what method to test?
Reasoning about effects: drawing diagrams of which method/class depends on the current method
1. Return values that are used by a caller
2. Modification of objects passed as parameters that are used later. Pay attention to superclass and subclass that use instance variables.
3. Modification of static or global data that is used later
Effect analysis can help understanding a new codebase

## Chapter 12: I need to make many changes in one area. Do I have to break dependencies on all the class involved?
Interception point: a point in your program where you can detect the effects of a particular change. For eg, when analysing effect of a change, it can effect method of multiple class in the call stacks, and we should pick an interception point that close to the code change, and easy to bring to the test harness.  
A pinch point is a narrowing in an effect sketch, a place where tests against a couple
of methods can detect changes in many methods. If you can not find pinch point, maybe the change is too broad, so just try to test for individual changes as close as we can.  
When doing effect analysis, pinch point looks like the integration point or cluster of classes.  
Be careful to add too much tests for pinch points, it will look more like integration test than unit test.

## Chapter 13: I need to make a change but I dont know what test to write.
Characterization test: in chapter 11, we don't know what method to test. In this chapter, it's about what test to write for a method. When in doubt, write characterization test to capture current behavior.  
How to do: call method in test with assertion you know will fail, then change the test to document the right behavior.  
Sensing variable: remmeber when you print a variable to see its value, then delete the print when commit the code? You can have a sensing variable (a class variable that store that value), then assert that value in the test. The code will look awkward, but it's a chance to break down that method and create a new method to test that chunk of code you care about the value.  
Pay attention when writing characterization test: sometimes your test doesn't run the code path you want to change, for eg it only run the happy path and you change a corner case.

## Chapter 14: Dependencoes on libraries is killing me
Don't use direct library call all over place, wrap it in our own code instead.

## Chapter 15: My application is all API calls
It's important to indentify the functional core of a system: [Functional Core, Imperative Shell](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell)  
- Skin and wrap the API: if API is small, we just mock that API class with the same signature
- Responsibility-based extraction: create new class to store logic so it's easy to mock.
Can always use both techniques, Skin and wrap for testing, and Responsibility-based extraction for better public API of the application


## Chapter 16: I dont understand the code well enough to change it
spend enough time to understand the legacy code. It will pay dividend.
-  Draw dependency graph for the new codebase, it doesnt need to be comprehensive of all the classes, just enough to document our mental state.  
More read: <https://understandlegacycode.com/blog/safely-restructure-codebase-with-dependency-graphs/>  
- listing markup: print the code to a paper and mark it (! not sure if doable but idea is valid)
  - separate responsibilities: group multiple methods in the same class or multiple class that belong together
  - extract method: circle the code you want to extract
  - understand effect of changes: mark the variable andcode that affected by this change
- Scratch refactoring: just refactor, then throw away the patch afterward

## Chapter 17: My application has no architecture
Telling the story of the system: when you describe system in a simple way, you might realize it's more complicated than neccessary.  
Use naked CRC (class, responsibility, collaborator): just simple cards of class instances interacting with each others. Note: card is instance, not class.  
Converstation scrunity: be open to update the design for legacy code instead of sticking to existing structure. Pay attention when the concept in the discussion doesn't overlap with concept in code.

## Chapter 18: My test code is in the way.
Forget whether you are looking at test code or production code? Need naming convention.  
- class naming convention: for eg, test class has Test suffix DBEngineTest, fake class has Fake prefix
- test location: different between languages. Prefer to have tests next to the production code.

## Chapter 19: My project is not object oriented, how do I make code change?
This chapter focus on precedural languages. The question is how find a seam to break dependency?  
- link seam: link to a test library when testing. Use macros to switch between production and test code. It will be messy but we don't have much choice.
- adding new behavior: write new features in new methods so we can at least test new code. Change from global functions to use function pointer so we can swap it in tests.  
- taking advantage of object oriented: write C++ in C code (!)


## Chapter 20: This class is too big and i don’t want it to get any bigger
Single-Responsibility Principle (SRP): Every class should have a single responsibility: It should have a single purpose in the system, and there should be only one reason to change it.  
Techniques to find responsibilities of a class
- Group methods. Group class variables by responsiblity
- Find private methods. Too many private methods hint hidden responsibilities. Make new class with public methods also make it testable.
- Find relationship between internal variables and methods. Cluster of methods and variables can be a new class.
- Create few interfaces for a big class for different responsibilities, so client can use the interface they need instead of the class.
- When all else fails, do some scratch refactoring, ie write a patch and throw it.

## Chapter 21: I'm changing the same code all over the place.
It demonstrate a refactor of a few Command java class: move common functionality to base class, make subclass more configurable but still allow overriding.  
Open/Closed Principle: The idea behind it is that code should be open for extension but closed to modification. What does that mean? It means that when we have good design, we just don’t
have to change code much to add new features.

## Chapter 22: I need to change a monster method and I can't write test for it.
Different kinds of monster method:
- Bulleted method: the easier one, need to extract each block to a new method. Pay attention to shared variables.
- Snarled method: is a method dominated by a single large, indented section.
Tackling monster with automated refactoring method: if you use the tool, use it exclusively. After a
series of automated refactorings, you can often get tests in place that you can use to
verify any manual edits that you make. Don't mix automatic and manual edit.  
Manual refactoring techniques:
- introduce sensing variable: we add sensing variable to capture the desired behavior, then write tests before refactoring. We can remove the sensing variable afterward, but better if we convert sensing variable to be the output of some method.  
- Extract what you know: do small steps at a time.
- Gleaning dependencies: a big method may have lot of code path, try to preserve the important logics.
- Break out a method object: move the logic in method to a new class to refactor. You will need to construct the new class with method parameters and instance variable used.
- skeletonize method: extract condition and the body to different methods. Work better with snarled method
- find sequences: extract both condition and the body to the same method. Work better with bulleted method.
- extract to the current class first: sometimes it's temped to move logic to another class, don't do it, keep it in current class first.

## Chapter 23: How do I know if I'm not breaking anything
- Hyperaware editing: when typing, we need to know if we change behavior of the code, or not.
- Single goal editing: do one thing at a time, for eg don't refactor when adding feature.
- Preserve signature: when you try to make initial edit to bring some code into tests, there's no test yet (!), so do it with care and try to preserve existing behavior.
- Lean on compiler: good luck with python. The idea is to altering the declaration to cause errors, and fix them.
- Pair programming

## Chapter 24: We feel overwhelmed, it isn't going to get any better.
It's risky to rebuild code from scratch, refactoring legacy code can be fun, it's about team attitude.

# Part 3: Dependency breaking techniques
## Chapter 25: Dependency breaking techniques
All these techniques is to put code under tests, so try to do minimal editing without changing any behavior
### Adapt parameter 
It's adapter pattern: wrap a difficult-to-construct parameter in our own class, create a new interface for that production class so we can fake in test.  
Use Adapt Parameter when you can't use [Extract Interface](#Extract-Interface) on a parameter's class or when a parameter is difficult to fake. We also don't preserve signature, so be careful.

### Break out method object
Use to break a big method. Use the signature of current method for the constructor of new method object, if the method use class variable, add reference to original class as the first parameter  
It can be complex if the method use other class' variable or method. It can be even worse if the current class can not be put into test, in that case, consider create an interface for current class to test the methods that are used by the new method object.

### Definition completion
In C or C++, we can declare class/method in a header file and implementation in another file. We can take advantage to re-declare the definition in the test file, the compiler will happily use our definition. Use it as the last resource since duplicating definitions is not a good thing.

### Encapsulate global reference
Three choices to deal with globals: (1) make the globals act differently under test, (2) link to different globals, or (3) encapsulate the globals so that you can decouple things further.  
If several globals are always used or are modified near each other, they belong in the
same class. We can create a new class to encapsulate these global variables and methods, and provide a global instance to be used by client.  
In places where you want to use fakes, use Introduce Static Setter, Parameterize Constructor, Parameterize Method or Replace Global Reference with Getter

### Expose static method
If a method doesn't use instance variable, we can create a new static method with the same signature, change instance method to call static method. So we can bring it under test without constructing the object.

### Extract and override (E&O) call
When we have some dependency that is too hard to mock, we can extract the code that use the dependency to a new method, and override it in a subclass under test. Drawback is when we have to do this in a lot of place.

### Extract and override (E&O) factory method
You can’t do it in C++, C++ does not allow virtual function calls to resolve to functions in derived classes  
When there's code in constructor that you can't run under test, for eg create a hard-to-mock object, extract that code to a method and override in subclass

### Extract and override (E&O) getter
To complement E&O factory method with C++.  
Instead of create the object in constructor, we create it in a lazy getter the first time it's used. So we can override the getter in subclass  
If the object is only used in one method, prefer E&O call instead. However E&O getter is usefull when that object is used in multiple places.

### Extract implementer
It's the friend of Extract interface. With Extract Interface, you need to come up with a name, which should reflect the (subset) responsibity of the class.  
Use Extract implementer when the current class name is already good for interface. In that case, we make the current class an interface (or abstract object in C++), and create new class to implement this interface.  
Pay attention when the class is in a hierachy. Might be better using Extract interface instead.

### Extract interface
One of the safest dependency breaking technique, you can lean on the compiler.  
You need to create a new interface with a subset of methods from original class used in your code, then update your code to use the interface instead. In test, you can implement that interface to test your code.

### Introduce instance delegator
Static method is global. We can create new instance method and call static method inside, then in our code, use Parameterize Method to pass an instance instead

### Introduce static getter
Global data is annoying. To deal with Singleton pattern, consider adding a global public setter to set the singleton instance to a test instance.  
Usually constructor in Singleton pattern is private, we could make it protected so we can subclass it and pass the fake object to static getter.

### Link substitution
Useful for C. We can create function with the same signature and substitute at linking.  
Can also be used with java to change classpath

### Parameterize constructor
If you create object in constructor, you can create a new constructor with that object as a new parameter. Or if language support default parameter value, can just add to existing constructor  
Rust doesnt support constructor at all, and use factory method to create object.

### Parameterize method
Similar to Parameterize constructor, but apply for a method. We can also name the new method differently instead to make it clear, for eg change run to runWithResult

### Primitivize parameter
Sometimes a parameter is too hard to mock, however the method doesn't use much information from the parameter except some primitive attributes. In that case, we can create a new method that take primitive values and call the new method from current one. The new method should be easier to test.  
If the new method is not natural, it can make ugly code, so use it as the last resource.

### Pull up feature
Sometimes a class has dependency and you need to change a few methods that don't use a bad dependency. Instead of trying to put the class into test, you create a new base class and pull those methods up, then you can subclass it and test those methods.  
It's not great from design point of view because the logic is splitted to 2 classes, it's better to refactor the class and extract the dependency to another object instead. However it's pretty safe to have some tests.

### Push down dependency
Similar to Pull up feature

### Replace function with function pointer
Useful in procedural language like C.  
It's a tradeoff because this technique might not be safe, and have another indirection.  
In Rust, it's trait object vs generic.

### Replace global reference with getter
Same as Extract and override getter

### Subclass and override method
It's a base technique that used in other techniques. Simply you choose the method with dependency, subclass and override to nullify the effect of that method.

### Supersede instance variable
Useful for C++ when you can not override method called in constructor. The reason is safety.  
It's a dirty hack to set an instance variable after the object is constructed

### Template redefinition
Applied for C++. You create a template for an existing class, and use typedef to make sure the old name has the same behavior. In test, you can use the template with your mock class.  
Disadvantage is code of template is now in the header file, and client of template need to be recompiled if template changes

### Text redefinition
With language like Ruby, you can redefine any method at any time, so can do it in test to nullify some behavior without subclass and overriding.




## My Appendix 1: Mermaidjs
- [https://mermaid-js.github.io](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoiZ3JhcGggVERcbiAgICBBW0NocmlzdG1hc10gLS0-fEdldCBtb25leXwgQihHbyBzaG9wcGluZylcbiAgICBCIC0tPiBDe0xldCBtZSB0aGlua31cbiAgICBDIC0tPnxPbmV8IERbTGFwdG9wXVxuICAgIEMgLS0-fFR3b3wgRVtpUGhvbmVdXG4gICAgQyAtLT58VGhyZWV8IEZbZmE6ZmEtY2FyIENhcl0iLCJtZXJtYWlkIjp7InRoZW1lIjoiZGVmYXVsdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ) 
- [mermaid cheatsheet](https://jojozhuang.github.io/tutorial/mermaid-cheat-sheet)

Example:
```mermaid
graph LR
    S[Sender] %% when you see a component just note it here
    R[Receiver] %% note another component
    M[Mail Server]
    S -->|send to| R %% the moment you see relationship between them, note it too
    R --> M
    M --> S
    subgraph Data pipeline
        A --> B
    end
    B --> R
```