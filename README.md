# My attempt on shortest and simplest code examples that demonstrate map, lift2, bind and apply

# map

```fsharp
// Hint: think what map does to 'a list
// (map calls function ('a -> 'b) on every item of the list and produces a new list)
// So if you defined List.map it would be something like:
//     let map (f (x : 'a) : 'b) 'a list : 'b list = ...
//
// Now forget about the list and focus only on the type signatures.
// Specifically, FORGET THAT LIST IS A COLLECTION, and focus only on the fact that it combines 2 things:
//     1. value(s) of type 'a
//     2. the box that holds the value(s) of type 'a
// Think of 'a list as a specific case of a generic type 'a something (e.g. 'a option).
// Thinking this way, what does map do?
//     - knows how to open the box and get values of type 'a
//     - calls the transformer function on 'a to get 'b
//     - puts the resulting items in the box again

type 'a message = Message of 'a

let map f args =
    match args with
    | Message x -> Message (f x)

let shout (msg : string) = 
    msg.ToUpper()

let exclaim (msg : string) =
    sprintf "%s!" msg

let (Message result) = 
    Message "hello world"
    |> map shout
    |> map (exclaim >> exclaim >> exclaim)

// "Result is HELLO WORLD!!!"
printfn "Result is %s" result
```

# lift2

```fsharp
// Hint: map is simply lift1
// Now take map and add an extra argument - you get lift2

type 'a message = Message of 'a

let lift2 f arg1 arg2 =
    match arg1, arg2 with
    | Message x, Message y -> Message (f x y)

let concat suffix prefix =
    sprintf "%s%s" prefix suffix

let (Message result) = 
    Message "Hello"
    |> lift2 concat (Message ", ")
    |> lift2 concat (Message "world")
    |> lift2 concat (Message "!")
        
// "Result is Hello, world!"
printfn "Result is %s" result
```

# bind

```fsharp
// Hint: think flatMap
// Think of times when you try to use map on the list, 
// and realize your mapping function also returns list.
// Now you end up with list of lists.
// Instead you want all returned lists to be flattened in a single final list.
// So if you defined List.flatMap it would be something like:
//     let flatMap: (f (x : 'a) : 'b list) 'a list : 'b list = ...
//
// Now, the same as with map, forget about the list is a collection and focus on the signatures.

let parseInt str =
    match System.Int32.TryParse str with
    | (true, x) -> Some x
    | _ -> None

let bind f args  =
    match args with
    | Some x -> f x
    | None -> None

// with map it would produce int option option
// with bind it produces int option
Some "75"
|> bind parseInt 
|> Option.iter (printfn "Result is %d")
```

# apply

```fsharp
let getCustomerFirstName id =
    match id with
    | 123 -> Some "John"
    | _ -> None

let getCustomerLastName id =
    match id with
    | 123 -> Some "Smith"
    | _ -> None

let getFullName firstName lastName =
    sprintf "%s %s" firstName lastName

let map f argsopt =
    match argsopt with
    | Some args -> Some (f args)
    | None -> None

let apply fOpt argsOpt =
    match fOpt, argsOpt with
    | Some f, Some args -> Some (f args)
    | Some f, None -> None
    | None, Some args -> None
    | None, None -> None

let partiallyApplied =
    getCustomerFirstName 123 |> map getFullName

let finalResult = 
    getCustomerLastName 123 |> apply partiallyApplied

// This is just a syntactic sugar
let (<!>) = map
let (<*>) = apply

// The same thing in a more concise way
let result =
    getFullName <!> getCustomerFirstName 123 <*> getCustomerLastName 123

// "Result is John Smith"
result 
|> Option.iter (printfn "Result is %s")
```
