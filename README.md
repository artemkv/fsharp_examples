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
//     - transforms values from 'a to 'b
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
// Think of times when you try to use map on the list, an realize your mapping function returns list.
// Now you end up with list of lists. Instead you want all returned lists to be flattened in a single list.
// List.flatMap: (f (x : 'a) : 'b list) 'a list -> 'b list
//
// Now, the same as with map, forget about the list and focus on the signature.
// Think of 'a list as a specific case of a generic type 'a something (e.g. 'a option).

let parseInt str =
    match System.Int32.TryParse str with
    | (true, x) -> Some x
    | _ -> None

let bind f args  =
    match args with
    | Some x -> f x
    | None -> None

"45"
|> parseInt 
|> Option.iter (printfn "Result is %d")

Some "75"
|> bind parseInt // That is all bind does
|> Option.iter (printfn "Result is %d")
```

# apply

```fsharp
let exec action person =
    sprintf "%s %s!" action person

let actions = [ "kick"; "kiss" ]
let people = [ "Bill"; "Mary"; "Samantha" ]

// apply [f; g] [x; y] produces [ f x; f y; g x; g y ]
let rec apply flist xlist = 
    match flist with
    | [] -> []
    | head :: tail -> (xlist |> List.map head) @ (apply tail xlist)

// Convenient way allows infix notation
let (<*>) = apply
let (<!>) = List.map

// We have to lift function (put in a list) before we can apply
[exec] <*> actions <*> people
    |> List.iter (printfn "%s")

printfn "****"

// We notice that if we first map, 
// then we end up with the list, so no need to lift anymore
actions |> List.map exec <*> people
    |> List.iter (printfn "%s")

printfn "****"

// And now final version 
// that justifies the whole pain of going through this example
exec <!> actions <*> people
    |> List.iter (printfn "%s")

```
