# My attempt on shortest and simplest code examples that demonstrate map, lift2, bind and apply

# map (lift1)

```fsharp
// Hint: think what map does to 'a list
// (map calls function ('a -> 'b) on every item of the list and produces a new list)

let rec map f alist =
    match alist with
    | [] -> []
    | head :: tail -> f head :: map f tail

let square (x : int) =
    x * x

let numbers = [ 1; 2; 3 ]

// Produces [ 1; 4; 9 ]
let squares =
    numbers |> map square
```

```fsharp
// Now forget about the list and focus only on the type signatures
// Specifically, FORGET THAT LIST IS A COLLECTION, 
// and focus only on the fact that it combines 2 things:
//     1. value(s) of type 'a
//     2. the box that holds the value(s) of type 'a
// Think of 'a list as a specific case of a more generic case,
// when you have type 'a something (e.g. 'a option).
// Thinking this way, what does map do?
//     - knows how to open the box and get values of type 'a
//     - calls the transformer function on values of 'a to get values of 'b
//     - puts the resulting items in the [same kind of] box again

type 'a message = Message of 'a

let map f args =
    match args with
    | Message x -> Message (f x)

let shout (msg : string) = 
    msg.ToUpper()

let exclaim (msg : string) =
    sprintf "%s!" msg

// Now we can chain trasformations of the internal value
// Using the functions that know nothing about the "wrapping" type.
let msg = 
    Message "hello world"
    |> map shout
    |> map (exclaim >> exclaim >> exclaim)

let (Message result) = msg

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

// We can still happily chain methods.
// Notice that if the second argument wasn't "Message x" but simply "x",
// we could just partially apply concat and use map.
let msg =
    Message "Hello"
    |> lift2 concat (Message ", ")
    |> lift2 concat (Message "world")
    |> lift2 concat (Message "!")
    
let (Message result) = msg
        
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

let rec bind f alist =
    match alist with
    | [] -> []
    | head :: tail -> f head @ bind f tail

let dup (x : int) =
    [ x; x ]

let numbers = [ 1; 2; 3 ]

// Produces [1; 1; 2; 2; 3; 3]
// With map it would have been [[1; 1]; [2; 2]; [3; 3]]
let duplicated =
    numbers |> bind dup
```

```fsharp
// Now, the same as with map, forget about the list is a collection 
// and focus on the signatures.

let bind f args  =
    match args with
    | Some x -> f x
    | None -> None

let parseInt str =
    match System.Int32.TryParse str with
    | (true, x) -> Some x
    | _ -> None

// with map it would produce int option option
// with bind it produces int option
Some "75"
|> bind parseInt 
|> Option.iter (printfn "Result is %d")
```

```fsharp
// Speaking of practical application,
// bind has a very useful property: it can chain functions that return
// wrapped type, the same way map can chain functions that return 
// unwrapped type.
// This allows "Railway Oriented Programming" with Option and Result.
// Basically that means that if any function in the chain returns None/Failure,
// The whole chain is short-circuited and the whole expression returns None/Failure.

let bind f args  =
    match args with
    | Some x -> f x
    | None -> None

type Customer = { FirstName : string; LastName : string }

let parseInt str =
    match System.Int32.TryParse str with
    | (true, x) -> Some x
    | _ -> None

let getCustomer id =
    match id with
    | 123 -> Some { FirstName = "John"; LastName = "Smith" }
    | _ -> None

let resultGood = 
    Some "123"
    |> bind parseInt
    |> bind getCustomer

let resultBad = 
    Some "aaa"
    |> bind parseInt
    |> bind getCustomer
```

```fsharp
// Maybe builder uses bind internally to re-write the previous expressions
// in a way that looks like "a normal program". Well, kind of

type MaybeBuilder() =
    member this.Bind(x, f) = bind f x
    member this.Return(x) = Some x

let maybe = MaybeBuilder()

// Imagine this is a controller method that takes idStr from url,
// so sometimes it is present, sometimes it is not
let get idStr =
    maybe {
        let! id = parseInt idStr
        let! cust = getCustomer id
        return cust
    }

let resultGood1 = get "123"
let resultBad1 = get "aaa"
```

# apply

```fsharp
// Apply allows another technique.
// I could not come up with any explanation in plain English yet,
// But here's example:

let map f argsopt =
    match argsopt with
    | Some args -> Some (f args)
    | None -> None

let apply fOpt argsOpt =
    match fOpt, argsOpt with
    | Some f, Some args -> Some (f args)
    | Some _, None -> None
    | None, Some _ -> None
    | None, None -> None

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

let partiallyApplied =
    getCustomerFirstName 123 |> map getFullName

let finalResult = 
    getCustomerLastName 123 |> apply partiallyApplied

// This is just a syntactic sugar
let (<!>) = map
let (<*>) = apply

// The same result in one line
let result =
    getFullName <!> getCustomerFirstName 123 <*> getCustomerLastName 123

// "Result is John Smith"
result 
|> Option.iter (printfn "Result is %s")
```
