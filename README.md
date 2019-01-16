# Mechanic
Mechanic is the programming language for creating game content. Rather than use complex editor, Mechanic aims at giving game developers the tools needed to create the content using code and logic.

This document aims at explaining the language, giving examples of its use and writing down the basic syntax.

## Factories
The core of mechanic are factories, they are used to process external data, create objects and add behavior to those objects. Factories can be seen as multiple constructors calling each other in order to create a final product.

Each constructor - or method - inside the factory can have preconditions. These conditions set what the object being created must look like for the method to accept it. If the method can't accept that object, it simply isn't called. Methods chose which potential other methods can be called at the end of their processes and the factory makes sure preconditions are fulfilled.

The factory must also define the structure of the product it is going to create. This structure is used for typing and can include other structures from other factories.

For example, in pseudo code :

```
factory "Monster"
    | produce structure
    |   | name
    |   | weapon <- Structure from another factory
    |
    | main method(initial structure)
    |   | Generate a name for the monster
    |   | Generate a weapon type randomly, either a sword or a mace
    |   | Return to the factory, possible methods: Sword, Lance
    |
    | method Sword, should run only if weapon is of type "sword"(Structure from main)
    |   | Generate statistics for the sword
    |   | Return to the factory
    |
    | method Lance, should run only if weapon is of type "lance"(Structure from main)
    |   | Generate statistics for the lance
    |   | Return to the factory
```

In this factory, the main method (I.e, the first method called), generates a name and a weapon type for the monster. Once that is done, it returns the modified object and defines two possible methods that could run next, Sword and Lance. The factory would internally check each of these method's precondition and call the one that passes. For example, if the weapon type generated was a Sword, the next method to run would be Sword.

Factories should also be able to take in external data. For example, if a json file contains a list of possible monster names, the factory could be given that external data.

Factories should be able to build upon another. For example, if I created an "entity" factory, I should be able to extend that factory to create my "monster" factory. In this case, the methods are called in order of inheritances. So the "entity" factory would execute all it's method chain and then hand off it's result to the monster factory as the base structure. Structures produced by a child factory needs to be compatible with the parent structure.

## Type system
Since the language is not built with optimization in mind, the type system can be simplified and the engine running it can take care of any optimization it wants. To that end, I propose a set of primitive types.

* **string**, the string is the classic string of many programming languages. It does not have any utility methods associated with it. It's purely an array of chars. Defaults to "".
* **char**, a character available within the engine charset (How do we handle charsets?). defaults to ''.
* **int**, a integer number. It comes into the usual variety of **short**, **int** or **long**. Defaults to 0.
* **float**, a floating point number. Defaults to 0.

Primitives have a default value that is predictable to make creating content easier and prevent the need for null values management. When you create a variable of any of these primitive types without giving it a value, its value is the default value.

Each of these primitive types can be used to create lists, inspired by the golang slices. Slice are arrays of dynamic length (In golang, they're built using part of an underlying array). They're accessed using integer indexes and the data is not mutable, getting something out of a list means you get a "copy" of the data. Structures can also contain other structures to create a structural hierarchy. When not initialized, lists are set as empty.

Primitives can be used to create structures - like C or golang structs - that act like objects. A structures is mutable within its scope only, so passing a structures to a functions would not pass by reference by default. A structure's variable can be set as empty if it isn't initialized.

Any variable can be defined as a constant variable, which means it's value cannot be changed. In case of structures, it means that the structure is readonly.

A keyword should be available to use type inference, removing the need to specify the type name for every variable created if it's not empty.

### May also need
* Maps of some kind
* A cursor/iterator type 
* Methods in structures
* Error type that can be set to empty

## Modules
In a file, anything that is not identified as private is exported. Each file act as it's own module and the folder structure is the namespace. All imports starts at the project's root and are not relative to the current importing file's directory. I do not see the need to allow programmers to define custom namespaces as the folder structure is very often used as the standard for namespaces, better to just enforce the standard. Namespacing and modularization exists to allow the engine reading the language to only load in memory code that will need to be used.

## Functions
Functions exists in the language and can be used anywhere. A function should have the ability to be overloaded if the signature is different. A function can have multiple return values. If it does, it needs to return all the values, even if returning empty. They work the same as most languages. They can access anything from the global scope.

## Syntax
I personally prefer the syntax of languages like Visual Basic and Pascal for their readability, even if they're more verbose than other languages, so the language takes some cues from them. Here is quick overview of the language's syntax.

```
// This is a comment
/* This is a multiline comment */

global string GLOBAL_STRING = "global" // Global variable, can only be set at the root level
private global int GLOBAL_PRIVATE_INT = 0 // Global private variable, will not be exported.

begin function function1
    // Simple function without parameters or return value, can still access global variable
end function

begin private function privateFunction
    // Functions can also be private
end function

begin function function2(string parameter1, int parameter2) returns int
    // Functions with one return value can omit the parentheses
    return 0
end function

begin function function2(string parameter1, int parameter2, float parameter3) returns int
    // If using the same name, but different signature, function can be overloaded
    return 0
end function

begin function function3(string parameter1, int parameter2) returns (int, error)
    // Functions with multiple return value need to return everything. 
    return 0, empty // Empty is null, it's a keyword
end function

// Can create structures
begin structure External
    int test
end structure

begin function code()
    const string constant = "const" // Constant variable, cannot be changed
    char singleQuote = '' // Single quotes are used for char, double for strings
    var variable = 0 // Type can be inferred
    int[] slice = list() // An empty list of integers, omitting list() would make the value equal to empty
    int[] initalizedSlice = list([1, 2, 3, 4, 5]) // A list initialized with base values

    if (variable == 0) then
        // Simple if
    end if

    if (slice is empty) then
        // is can be used interchangeably with ==
        throw error("Empty slice") // Causes an error, would be handled by the engine unless caught somewhere in the code
    else if (slice[0] != 1) then
        // Else if case
    else
        // Else case
    end if
    
    for (int i < len(initalizedSlice)) loop
        // For act as the while and for loop, len gets the length of anything
         if (i == 2) then
            contine loop // go back to the loop
        end if
        if (i == 3) then
            end loop // Ends the loop early
        end if
    end for

    for (int element in initalizedSlice) loop
        // for each loop is a normal loop
    end for
    
    loop
        // Do while loop
    for (int i < len(initalizedSlice))
    
    select (constant) cases
        case "test" do
            // Adding the end case acts like a break
        end case
        case "foo" do
        case "bar" do
            // would run for both, could also use one case like this 'case "bar", "foo" do'
        end case
    end select
    
    for (int i < 5) loop: columns
        for (int j < 5) loop: rows
            if (j == 3) then
                // Loops can be labeled to act as fake gotos
                continue loop columns
            end if
        end for
    end for
end function

begin function structuresExample()
    External struct = External{
        test: 1,
    } // Creates an initialized structure of type External
    External structEmpty // creates another structure, but it is set to empty
    External structEmpty = External{} // Unset value are set to their default values
    
    // I can also create anonymous structures
    begin structure Internal
        int otherTest
    end structure
end function

// Base factory
begin factory MonsterFactory uses (string[] nameList)
    begin produce
        string name
        int health
        float xpGain
    end produce
    
    @main
    begin step main(EntityFactory.produce initial)
        // Main method of the factory, will always be the first called
        // Receives a base structure (Everything empty)
        // The external data is available in the self variable
        initial.name = self.nameList[randomize(0, len(self.nameList) - 1] // Lists are accessed with square brackets
        initial.health = randomize(1, 10) // Randomizes between 1 and 10
        initial.xpGain = randomize(0, 1) / 1 // Randomizes between 0 and 1, make it a float
        return initial to EntityFactory.powerfulEntity, EntityFactory.weakEntity
    end step
    
    @condition(xpGain >= 0.5)
    begin step powerfulEntity(EntityFactory.produce entity)
        // Will execute if xpGain from the previous step is high
        return entity
    end step
    
    @condition(xpGain < 0.5)
    begin step weakEntity(EntityFactory.produce entity)
        // Will execute if xpGain from the previous step is low
        return entity
    end step
end factory

begin function createStuff() returns MonsterFactory.produce
    // I can create an anonymous factory that extends another one
    begin factory GoblinFactory from MonsterFactory
        begin produce
            string name
            int health
            float xpGain // Needs to be compatible with previous factory
            int weaponType
        end produce
        
        @main
        begin step main(GoblinFactory.produce initial)
            // initial already has the stuff from the EntityFactory method chain
            return initial
        end step
    end factory

    // Goblins are also monsters
    return GoblinFactory.new(list(["Bob", "Beb"]) // I give it a list of names
end function
```

## Examples
* [Create a quest](/examples/create_a_quest.md)
