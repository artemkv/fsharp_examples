# map

```fsharp
// Hint: think what map does to the List<T>
// (map calls function ('a -> 'a) on every item of the list and produces a new list)
// List<T>.map: (f (x : 'a) : 'b) 'a list -> 'b list

module Message =
    type Message<'a> = Message of 'a

    let map f args =
        match args with
        | Message x -> Message (f x)

    let returnMessage x =
        Message x

open Message
module MessageTest =
    let shout (msg : string) = 
        msg.ToUpper()

    let exclaim (msg : string) =
        sprintf "%s!" msg

    let (Message result) = 
        Message.returnMessage "hello world"
        |> Message.map shout
        |> Message.map (exclaim >> exclaim >> exclaim)

    // "Result is HELLO WORLD!!!"
    printfn "Result is %s" result
```

# lift2

```fsharp
module Message =
    type Message<'a> = Message of 'a

    let lift2 f arg1 arg2 =
        match arg1, arg2 with
        | Message x, Message y -> Message (f x y)

open Message
module MessageTest =
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
