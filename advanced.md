# Advanced/Proposal content
This file contains ideas for advanced functionalities in the language. They are still far to WIP to be on the readme, but will likely be soon in one form or another.

## Behavior on structs
Structs could use some behavrio code. For example, if a factory produces a Goblin, I would like to be able to run a method to reduce its HP or get its damage output. Allowing a coder to design these processes could help the language and would prevent hiding any behavioral code in the game engine.

The proposal is to allow defining the signature of method on structs without implementation. When defined this way, all methods will be equal to `empty` by default and must be implemented when the struct is created. In the external language reading mechanic code, these are nothing more than function pointers.

Implementing a method in a created struct is as simple as assigining it an existing function, an annonymous function or creating one inline.

### Example of code
```
begin structure Example
    string name
    int age
    float xpMultiplier
    method()(float) calculateXP // First parentheses are the parameters, names are omitted. Second are return values
end structure

// The method keyword could be used to make the function clearly a "method function". Is it useful?
// We need a way to tell this method that it should expect an Example struct as it's "self". This doesn't look right
begin method Example.calculateXP() returns float
    return self.age * self.xpMultiplier
end method

Example test = Example{
    name: "test",
    age: 3,
    xpMultiplier: 1.5,
}

test.calculateXP = Example.calculateXP
// Inline methods, should we also allow creating full methods like this?
test.calculateXP = method Example.calculateXP() returns float; return self.age * self.xpMultiplier
```

## Private methods in factories
Factories should be able to have private methods that can only be called internally to simplify and shorten the code.

To achieve this, a coder could simply create a function with the method keyword and it would be accessible anywhere within the factory's self. 

### Example of code
```
begin factory ExampleFactory
    begin produce
        string name
        int age
        float xpMultiplier
    end produce
    
    @main
    begin step main(ExampleFactory.produce initial)
        self.calculateXP()
    end step
    
    // Method keyword is used for clarity, is it even necessary?
    begin method calculateXP() returns float
        // Do something
    end method
end factory
```
