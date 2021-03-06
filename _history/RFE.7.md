
N - Add to pattern section: Stack, Tree, loop

N - Allowing functions in a struct is just like defining a function to create instances of the struct.
And initialising those lambda fields inside that function and calling it to create a new struct.
It is a useful simplification. No new concept is added. Everything is the same: Closure, function, ...

N - Can we import a module inside a struct definition?
Or import it "into" a struct definition?
The code inside the module has access to outer world's functions. So it will compile without an issue.
But the code outside will not have access to a module which is inside a struct. Because then it needs to prefix with type name or instance name.
This can be used to resolve name conflicts in a much easier way.
option 1: Import statement inside struct definition
option 2: Assign output of import to a new struct type
This will not be a struct you want to instantiate because everything is type-level. 
option 2 is simpler.
But what about types?
Nope. Having types inside a struct does not make sense at all.
But what if we import it there? So we can "use" their types and bindings but they will not be part of struct and they will not be available outside the struct.
```
Customer = 
{
    @["/core/st/Helper"]
    name: string,
    family: string,
    g: HelperType
}
```
But it doesn't make sense to use `g` in Customer without importing the module.

N - How can I import a sequence of modules? Because now, sequence is not built-in.

Y - Why we cannot assign a generic function to a lambda?
```
process = (T: type, x: T -> x.name)
```
then how can I have a pointer to process?
`g = process(_,_)` then: `g(int, 22)`?
But I also can call `g(string, "A")`?
what would be type of g then? `g: (T: type, x:T -> string)`
But similarly, I can call `process(int, 22)` which will throw a compiler error!
So compiler just needs to track what function does a lambda point to, so that when there is a call, it can generate the source for that function if needed.
If we keep that data during compilation, we can allow assigning generic functions to a lambda.
One less constraint

N - What new things can we do with instance and type level functions?

N - Recent changes:
+ vararg
+ generics
+ byte
+ ptr 
- seq
- map
- compile time sequence

N - Can user play with `ptr`? e.g. by having address of the first element of sequence, calculate address of the second item (x+4)?
we can do it via core functions.

N - Can we replace vararg with a linked list?
I don't think so. we can do it the other way around: build LL using vararg function.

N - How can I have recursive processing for variadic functions?
just use core functions.
`getVar(input, 3)` get a new variadic arg from 4th element afterwards

N - End a function with `...}` to imply it should be called forever?
so how can we exit?
and 99% of the time we want to recursively call self with a new argument set. so it is not good.

N - Should have an easier conflict resolution method?
for methods with the same name or types with same name
response: we cannot have methods with the same name.

N - Rather than channels, can we provide "tools" to build channels in the core?
we can build a channel using a mutex (what about immutability?).
and it is how channels are made.
Let's provide lowest level and let the code (core or std) implement higher levels.

N - And the problem of cache.
Most of solutions add some kind of opaqueness and confusion to the system.

N - Can we use this matching with messages for a union?
manually provide lambdas for each possible type.
`invoke(circle_or_square, (c: Circle -> ...), (t: Triangle -> ...))`
`invoke = (S,T: type, input: S|T, h1: (S->), h2:(T->))`
but what will be the body of the function?
receiving a `type` based on what is inside a union is dangerous.
maybe we can check for match types: `bool m1 = matchType(circle_or_square, h1)`
or: `tryInvoke(h1, circle_or_square, nothing)` try to invoke h1 with `circle_or_square`, if not possible return `nothing`.
`tryInvoke = (T: type, h: (T->), input: ?, default: x -> ?)`
`multiInvoke = (S,T: type, h1: (T->), input: ?, default: x -> ?)`
No. We have cast which returns a bool. That's enough.

N - If we can treat union like a struct, we can add member functions for it.
But can simply define a struct containing the union + member functions.

N - Constraints with `type`?
`type{name:string}` is a struct with string name field
`type{}` is a struct
`type{draw:(int->int)}` has this function
`process = (T: type, x: type{get:(->T)})`
It is not really needed.
`Data = [T: type -> {name: string, data: T}]`
`writeName = (x: Data[_] -> print(x.name))`
Suppose that I have a type which has a name:string and data:int.
can I call `writeName` with that type?
```
HasName = [T: type -> {name: string}]
Customer = {age:int, id: int, name:string}

writeName = (x: HasName[_] -> print(x.name))
#can I call this?
writeName(my_customer)
```
I don't think so. It is against function resolution rules.
What if we also have a writeName with Customer?
Statically it is allowed because they are two different functions.
But with above proposal, there will be confusion which one to call.
the good thing about go is that the set of functions you can call for a specific type are flexible.
You can easily add support for function f to type T. If f uses an interface and you provide appropriate functions for T it will work fine.
constraint for type can be checked at compile time. but for other types will be at runtime.
we can add this but I'm not sure about it's usefullness.
e.g. constraint to union -> only it's cases can be added (for candidateHandlers in polymorphism)
e.g. constraints to struct -> only for types with that members
```
Shape = Circle | Triangle | Square
ShapeType = type|Shape|
CandidateList = [T: ShapeType -> {X: ShapeType, handler: (T->), next: nothing|CandidateList[_]}]
```
and for struct:
```
QueryMessage = {key: string}
UpdateMessage = {key: string, data: int}
MessageType = type{key: string}
processMessage = (T: MessageType, msg: T -> msg.key)
...
processMessage(QueryMessage, q_msg)
```
Too much complication and not very much benefits.

N - Why do we need `ptr` type? Can't we just use `int`?
Or just define it as `ptr := int`
or `Ptr := int`
But it is not only a number. We need to keep track of size of the region too.
So let's just say it will be handled by core.

Y - Add to patterns: conditionals
How can we do this without sequence?
using indexed access to a memory region
add as a function in std
```
ifElse = (T: type, cond: bool, trueCase: T, falseCase:T -> get(T, int(cond), falseCase, trueCase)
get = (T: type, index: int, items: T... -> getVar(items, index))
```

N - Assigning to a binding which is not defined yet, make reading code difficult.
```
process = (->out:int) 
{
    out = f(out2)
    out2 = g(out3)
    out3 = ...
}
```
This was introduced to prevent declaring conditionals and early return
```
process = (x:int -> out:int)
{
    #if x is negative return early
    ???   
    #else
    out = innerProcess(x)
}
#we can write:
process = (x:int -> out:int)
{
    out = ifElse(x<0, 0, innerProcess(x))
}
```
The goal: make language simpler, more minimal and consistent and easy to read.

N - From go2 design: Error handling
```
x, err := strconv.Atoi(a)
	if err != nil {
		return err
	}
#becomes
handle err { return err }
x := check strconv.Atoi(a)
```
```
process = (x:int, y:int -> out:int)
{
    #if either of x or y are negative return 0
    #else return x+y
    out = ifElse(x<0, 0, out2)
    out2 = ifElse(y<0, 0, out3)
    out3 = x+y
}
```

Y - Can we move channels to core?
Then we will have `rchan[int]` and `wchan[int]`
They definitely cannot be in std.
This will simplify concurrency section.
If we provide some underlying structure, channels can be moved to std.
Buffered channel is like a linked list. It is not easy but can be done with some effort.
Let's just focus on a syncronous channel.
Basically it is a linked list with 0 or 1 elements.
If no element -> block
if one element -> delete and return.
But this can be subject to concurrency issues if we don't do it atomically.
Linux API support for concurrency:
- mutex: `mutex_lock` and unlock
- semapure: `down` and up
mutex is a binary semaphore
so we just need to have support for semaphore.
and semaphore is defined like this:
```
struct semaphore {
    raw_spinlock_t        lock;
    unsigned int        count;
    struct list_head    wait_list;
};
```
so internally, it uses a spinlock for sync. this lock is used to protect when counter is going to be changed.
spinlock is a lock/release data structure. if another thread has locked it, your lock will wait until it is released.
But all these locks are to protect data modification. We don't have any mutable data so what are we going to protect?
In Erlang, threads send messages to each other. 
Also there are other applications for channel: file, network, console, ...
But are they?
Can we make this simpler?
Maybe a single channel type? Or just a lock?
Lock is not enough because we also need data. 
Maybe if we can simplify channels, `select` can be simplified too.
another solution can be something like Erlang: Each thread (or green thread) has a channel (implicit). That you can query or send.
we can say: `x := process(10)` will create x as a messageBox.
inside process you can write: `data = receive()` to get data from the messagebox.
and you can write: `send(x, my_data)` to send.
x is like a channel. You can pass x to any other thread to send data to.
But how can we "receive" data from a thread?
Let's say `x := process(10)` will reutrn an int, which is code-id (cid) or thread-id.
Then you write `send(x, 100)` to send something to process.
priority? You can filter when receiving:
`data = receive(my_cid, {priority: high})`
and note that, send/receive can be done on any type. So these must be in core.
`x := process(10)`
`send(x, int, 100)`
`int_num = receive(my_cid, int)`
any code can get it's cid with a call to core. 
cid=0 is for the main app.
when sending, we just send a data to a cid.
when receiving, we specify type and filter. type is mandatory but filter is optional.
so this means, we won't need channels, ro, wo, `!` and `?`. only functions and `int`.
what about select? Select is used to see which channel has a data.
similarly, we have multiple cids, and we want to do a receive (or send) on them.
1. this can be a vararg function
q: select
q: buffered channel
state of a process should be embedded in the arguments of the function. state change = recursive call with new state
`data = receive(my_cid, PriorityMessage, {priority: high})`
idea: threads can join to a group. when send, you can send to a group. when receive you can receive from a group
a thread can be member of multiple groups.
send to a group? what does that mean? it is just multiple send.
send to a group is intuitive, it is basically multiple sends in one statement.
but receive from a group: it will act like a select: if you receive from a group of 10 workers, (let's call them workers rather than CID or thread or process or ...)
maybe 5 of them have something for you.
Who should decide worker's group? itself? or the caller?
I think worker should be dummy. They don't need to know their group. They just send and receive message and do their job.
```
x := process(10)
y := process(20)
z := process(30)
new_cid := {x, y, z}
send(new_cid, int, 10)
receive(new_cid, int)
```
The above receive call will act like this select:
```
select {
    x: ...
    y: ...
    z: ...
}
```
The first worker which has a data, will return and we are done.
If multiple -> only one is returned.
If none -> block?
In Docker, 90% of selects are for read/receive. Why complicate everything with a select that mixes read and write?
how can we handle variable number of workers with group receive?
```
x := process(10)
y := process(20)
z := process(30)
new_cid := {x, y, z}
send(new_cid, int, 10)
receive(new_cid, int)
```
q: How can we get result of `process` if called async using `:=`?
answer:
```
x := process(10)
g = receive(x, int, {type: RESULT})
```
q: buffered channel -> everything is unboundedly buffered, sync via core functions
q: how can we detect if a worker is finished? core function.
q: select
q: variable number of recipients?
we can have `receive` and `blockReceive` so we can say if nothing there, wait (or `peek` and `receive`)
buffered: workers have unbounded mailbox.
so send will never block. but receive may.
maybe we should also have `pid := int` and use it for worker id.
in Erlang, receive does not block unless queue is empty: "If the end of the queue is reached, the process blocks (stops execution) and waits until a new message is received and this procedure is repeated."
the problem is sync-async is easily solvable.
in other words, we only have worker communications: workers send data to each other and they call `receive` to get them. we do not receive from a specific process, we just receive from my own mailbox.  of course we can add "sender-cid" just like priority.
0 is for main app worker
1 for stdin
2 for stdout
3 for stderr
q: maybe we should make sender, a standard part of messages. this can affect select support too.
also we may want to send a message and wait until the message is received (like golang send to a non-buffered channel)
but we can implement this via messages too.
send and wait for a message from this cid with this type.
select = receive from multiple cids - like a normal receive block if no message matches with this
select = receive with multiple cids
```
x := process(10)
g = receive(int, x) #receive an integer from x
data, cid = receiveMulti(int, filter, x, y, z) #receive an integer from any of these channels
```
we can also have similar for send:
`sendMulti(data, x, y, z)`
but as send never blocks, this is same as multiple calls.
blocking send can be implemented via send and receive (waiting for ack).
blocking receive is already provided.
BUT we don't receive "from" a specific cid.
We just receive from current inbox.
so rather than receiveMulti we can simply use receive but with a more complicated filter: receive a message of this type, where sender_cid is any of these.
if none, it will block as it should be.
how can we provide "or" in the filter?
1. use a struct to provide multiple values
2. use lambda
```
x := process(10)
y := process(20)
z := process(30)
Message = [T: type -> {sender: cid, data: T}]
g = receive(Message[int], {sender: {x,y,z}}) #g will contain a sender field
originator = g.sender
data = g.data
```
lambda solution is too flexible and difficult to optimize
rather than having struct for field, we can have multiple filters.
```
x := process(10)
y := process(20)
z := process(30)
Message = [T: type -> {sender: cid, data: T}]
g = receive(Message[int], {sender: x}, {sender: y}, {sender: z}) #g will contain a sender field
originator = g.sender
data = g.data
```
But what is the type of `receive`?
`receive = (T: type, filter: {???}... -> T)`
`receive = (T: type, filter: T... -> T)`
So, can we say `{sender: x}` can be of type `Message[int]`?
Can we have this?
```
Point = {x:int, y:int}
f = Point{x: 10}
```
I don't think so. Every field must have a value here.
so, we specify all values for all fields:
`g = receive(Message[int], {sender: x, data: ?}, {sender: y, data: ?}, {sender: z, data: ?})`
q: filter is only for structs. right?
q: How does this whole thing work with union types?
`g = receive(Message[int], (m: Message[int] -> m.sender == x or m.sender == y or m.sender == z))`
then type of receive will be:
`receive = (T: type, lambda: (T->bool))`
Erlang does this via pattern.
Giving a message to a lambda which developer has written is a bit dangerous.
Basically, even the first argument of receive (type) is a filter.
`g = receive({type: Message[int], sender: {x,y,z}})`
what if we repeat sender?
`g = receive({type: Message[int], sender: x, sender: y, sender: z})`
we want to be able to receive from a dynamic number of senders. so this should be more flexible.
With lambda: 
`g = receive(Message[int], (m: Message[int] -> contains(senderList, m.sender))`
It will work fine with multiple dynamic number of senders too.
we can also use lambda for type!
`g = receive((m: Message[int] -> m.type == Message[int] and contains(senderList, m.sender))`
But as receive needs a generic type anyway, we don't need to repeat it:
`g = receive(Message[int], (m: Message[int] -> contains(senderList, m.sender))`
**proposal**
1. No channel notation, no channel send or receive
2. `:=` will return a worker id (wid) which can be used to send and receive data
3. Any two workers, can send message to each other: `send(my_data, wid1)`. which returns whether message was sent or not.
4. Any worker, can receive messages (block if nothing): `receive(int)`
5. You can also provide a filter for receive (only of this sender, ...)
6. Receive with multiple senders is like select
7. Send is not blocking, but if you need ack, you can receive and wait for "ack" response from recipient wid.
how can we say, receive anything? at least we have to know type.
other than that: a lambda that returns true for everything
How can we have a timeout for receive wait? use a special worker which returns what we want after a timeout.
send to a finished worker: it will return.
suggestion: we can add a bool return to send so we know whether message was delivered or not. e.g. is process is terminated, send will return false.
what about receive? What if I receive from a process which is already terminated?
To get result of a function (or exit code of a worker), we can use core: send me an update when this worker is finished
`coreRegister(wid1)` will send a message to current worker, when wid1 is finished.
receive from a terminated process will never work. It will never return.
How this works with file, console, network? You can get a wid any way you want.
```
std = 1  #in and out
send(std, "Hello")
send(std, SubscribeMessage)
```
using 0 for current wid is not good. Because it is not portable. What if I want to send my own wid to another worker?
How can I read from keyboard? with this new worker notation
With channels, you can specifically ask to receive data frmo a channel.
But here you receive from your own inbox.
`name = receive(Message[string], (x: Message[string] -> x.sender == std))`
This is not compatible with channels. With channel, you have a specific communication media so if the other side sends something, you (and only you) will get it.
But with this model, you cannot do it because socket does not know who is the listener.
Unless, we provide our wid when creating a reference to std or socket or ... .
```
socket_wid = createSocket(my_wid, ...)
send(socket_wid, "data")
receive(... sender == socket_wid)
```
Another advantage of this: We can host (migrate) a worker to another machine. It's just a wid.
the lambda for receive check is too much flexibility. can we limit it?
`g = receive(Message[int], (m: Message[int] -> contains(senderList, m.sender))`
user should be able to filter based on any field: sender, type, priority, ...
even if we only allow these fields in the filter, how are we going to handle arrays (dynamic number of values for sender).
`g = receive(Message[int], (m: Message[int] -> contains(senderList, m.sender))`
what if we use generics + lambda? any use of lambda will prevent compiler optomisation and allow room for misuse.
What is the absolute minimum we need here?
in Erlang pattern matching, the pattern must be calculated at compile time.
Anyway, how are we supposed to keep track of workers when their count is dynamic?
one solution: pre-specified range for wid and provide upper and lower bound for them when matching.
another solution: grouping. rather than relying on sender_id, rely on group_id and make it a fixed item.
```
x = process(10, "group1")
y = process(20, "group1")
z = process(30, "group1")
...
message = receive(Message, {sender_group: "group1"})
```
Ok. Now we only need to support a set of name/values for filter. Basically we need a sample 
instance of message type. any message whose fields are exactly same as the sample will be matched.
```
Message = {data: int, sender: wid, sender_group: string, priority: int}
...
msg = receive(Message, {sender_group: "AAA", priority: 1})
```
what about missing fields in the pattern? what is type of filter?
let's say: `nothing` is ignored in the matching process. so:
```
Message = {data: int, sender: wid, sender_group: string, priority: int}
Filter = [T: type
...
msg = receive(Message, {sender_group: "AAA", priority: 1})
```
But group is not always pre-defined or fixed.
I will still need `OR` support in the filter.
What about this: provide a lambda which extracts a field from message, and a value to match.
So filter is a lambda + target value. and we can any number of filters.
```
Filter = [S,T: type -> {lambda: (S->T), target: T}
Message = {sender: int, group: string, data: int}
receive(Message, Filter[Message, int]{lambda: (m: Message -> m.sender), target: s1_wid})
```
In this way we might be able to combine filters: AndFilter, OrFilter, ...
And receive will accept only one filter.
```
Message = {sender: int, group: string, data: int}
```
Scala pattern matching supports boolean predicates:
```
notification match {
    case Email(email, _, _) if importantPeopleInfo.contains(email) =>
      "You got an email from special someone!"
```
Basically, this is like polymorphism: a number of handlers for a number of cases (types: circle, triangle, ...) and an unknown
```
ReceiveHandler = [T: type -> {predicate: (T->bool), handler: (T->)}]
receive = (T: type, handlers: {predicate: (T->bool), handler: (T->)}...)
...
receive(Message, {predicate:(m: Message -> m.sender = 12), action:(m: Message -> processNormal(m))}, 
                 {predicate:(m: Message -> m.sender = 14), action:(m: Message -> processImp(m))})
```
Seems that predicate with lambda is the most practical solution. 
We can add a compiler warning if this predicate is not a simple expression.
1. No channel notation, no channel send or receive
2. `:=` will return a worker id (wid) which can be used to send and receive data
3. `was_sent_bool = send(my_data, receiver_wid1)`. 
4. `receive(int, handlers)`
5. Send is not blocking, but if you need ack, you can receive and wait for "ack" response from recipient wid.
6. Send to terminated/invalid wid returns false. receive from terminated/invalid wid will never return
We don't need a handler for receive. we just provide predicate.
```
Predicate = [T: type -> (T->bool)]
receive = (T: type, handler: predicate[T])
...
received_message = receive(Message, (m: Message -> m.sender == 13))
```
We can have a notation for send and receive:
```
was_sent = data>>wid
msg = <<(m: Message -> m.sender=1)
```
It is better for notation to be composable. So I can do something on the result immediately.
`x = <<(m: Message -> m.sender=12).source`
`was_sent = data<wid>`
`msg = {(m: Message -> m.sender=12)}`
`msg = ((m: Message -> m.sender=12))`
`msg = [(m: Message -> m.sender=12)]`
`was_sent = data->wid`
`was_sent = wid!data`
`msg = (m: Message -> m.sender=12)?`
```
was_sent = (wid<<data)
msg = <<(m: Message -> m.sender=1)
```
what about timeout in receive?
before receive we can initiate another micro-thread to send a message after x milliseconds 
```
#initiate a new thread which will send me a message after 300 ms
wid := timeout(my_wid, 300, {flag: false})
msg = <<(m: Message -> true) #wait for any message
{stop}>>wid
```
Let's make both notations the same:
```
was_sent = wid<<data
msg = <<(m: Message -> true)
```
immediate/eventual send
**Proposal**
1. `:=` will return a worker id (wid) which can be used to send and receive data
2. 
```
was_sent = wid<<data
msg = <<(m: Message -> true)
```
3. Send is not blocking, but if you need ack, you can receive and wait for "ack" response from recipient wid.
4. Send to terminated/invalid wid returns false. receive from terminated/invalid wid will never return
```
was_sent = wid<-data
msg = my_wid<-(m: Message -> true)
```
```
#it makes more sense to bring data first, when sending
was_sent = data::wid
msg = ::(m: Message -> true)
msg = ::lambda
```
Do we need a notation to represent current wid? I don't think so. put it in core: `currentWid()`
Example:
send and wait for ack:
`data::wid`
`::{sender:wid, type: Ack}`
Let's say after `::` you can put a binding (wait for exact message) or lambda (predicate)
`msg = ::(...)`
`createResponse(msg)::msg.sender`
**Proposal**
1. `:=` will return a worker id (wid) which can be used to send and receive data
2. 
```
was_sent = wid::data
msg = ::(m: Message -> true)
msg = ::(m: Message -> m == {source:1, type:2})
```
3. Send is not blocking, but if you need ack, you can receive and wait for "ack" response from recipient wid.
4. Send to terminated/invalid wid returns false. receive from terminated/invalid wid will never return
`:: msg1, msg2, msg3, lambda` wait for any of these -> no. too confusing.
`:: msg1` or
`:: (...)`
so: you cannot send a message which is a lambda. but this might be useful in some cases.
let's just have lambda. It is more general and you can put it inside a function to make call shorter.
- Let's address some of inefficiencies of actor model: implementing handshake model
- Limit mailbox
Suppose that a process wants to send a message, but wait until it is picked up.
We may want to implement a protocol, so wait after it is picked up for a reply.
```
was_sent = data=>wid
msg = (m: Message -> true)
msg = (m: Message -> m == {source:1, type:2})
```
Why should we have a special notation for this? Can't we rely on core functions?
```
was_sent = send(data, wid)
sendWait(data, wid) #send and wait for message to be picked up
msg = receive(Message, wid, (m: Message -> true)) #receive any message from this sender
msg = receive(Message, nothing, (m: Message -> m == {source:1, type:2})) #receive this specific message from any sender
```
let's put sender in predicate. So we can have a proper select.
```
was_sent = data>>wid
sendWait(data, wid) #send and wait for message to be picked up
msg = <<receive(Message, wid, (m: Message -> true)) #receive any message from this sender
msg = receive(Message, nothing, (m: Message -> m == {source:1, type:2})) #receive this specific message from any sender
```
Defining a notation for something complex like this, needs a lot of conventions which many people would find confusing.
Send, receive, send and wait, ...
Let's just stick to core functions.
```
was_sent = send(data, wid)
sendWait(data, wid) #send and wait for message to be picked up
msg = receive(Message, wid, (m: Message -> true)) #receive any message from this sender
msg = receive(Message, nothing, (m: Message -> m == {source:1, type:2})) #receive this specific message from any sender
```
What about socket, file, ...? which of these are inherently mutable?
file is not. 
```
f = open("a.txt")
data = read(f, int)
```
What if multiple threads read from this file?
```
f := open("a.txt")
send(f, "a.txt") #to write
receive(...)
```

Y - Can we have cache with message passing?
For cache at least we need a way to "receive" without removing item from queue.
But a cache can have a lot of subscribers. ok. They just need to have cache worker's wid.
Then they can receive. But cache is not supposed to send a message to every one of it's subscribers.
A cache can be a worker which accepts two types of messages: store and query
For store, it will update it's internal state for cache data
For query, it will send the result of cache query to the sender process.
With each store, it will call itself with a new state: the new cache which contains the newly added data
```
CacheState = Map[string, int]
cache = (cs: CacheState->)
{
    request = receive(Message[CacheStore])
    new_cache_state = update(cs, request)
    query = receive(Message[CacheQuery])
    result = lookup(new_cache_state, query)
    send(Message{my_wid, query.sender_wid, result})
    cache(new_cache_state)
}
```
This is closure, even inside cache function, we have access to itself.
Add to pattern section

N - Can we convert union to case class like in scala?
Will it make things simpler?
```
#Scala
abstract class Notification
case class Email(sender: String)
case class SMS(caller: String)
case class VoiceRecording(contactName: String)
...
def showNotification(notification: Notification): String = {
  notification match {
    case Email(email, title, _) =>
      s"You got an email from $email with title: $title"
    case SMS(number, message) =>
      s"You got an SMS from $number! Message: $message"
    case VoiceRecording(name, link) =>
      s"you received a Voice Recording from $name! Click the link to hear it: $link"
  }
}
```
maybe we can use a dot notation to make adding a new case to union more intuitive:
```
Shape.Circle = {r: float}
Shape.Square = {s: int}
...
Shape.Triangle = {x: int, y: int}
...
CandidateList = [T: Shape -> {T: type, handler: (T->), next: nothing|CandidateList[_]}]
shape_handlers = CandidateList[Square]{Square, drawSquare, shape_handlers}
shape_handlers = CandidateList[Triangle]{Triangle, drawTriangle, shape_handlers}

draw = (s: Shape, element: CandidateList[_] -> )
{
    
}
```
But what happens to simple unions like `int | string`?
this is named union so we have to assign a name:
```
Data.Int = int
Data.String = string
process = (x: Data ...
```
Another problem: we cannot enhance it with generics
`Data = [S,T: type -> S|T]`
We cannot use generics because the definition is not in one location. we can
```
Shape.Circle = [T: type -> {r: float, data: T}]
Shape.Square = [T: type -> {s: int, data: T}]
```
And `Shape` can be used to limit number of types for a generic type:
`CandidateList = [T: Shape -> {T: type, handler: (T->), next: nothing|CandidateList[_]}]`
proposal:
- No `|` for union notation. We use `A.B` naming where A is union name and B is subtype name.
- A binding of type A can hold any of the subtypes.
- You can cast a binding of type A to any of it's subtypes to check existence.
- Subtypes can be any type (simple, struct, ...)
advantage: extending an existing union is more intuitive (adding a new shape)
advantage: You can use it to constraint a generic type
q: should we have select for type matching?
q: Can subtype be another union?
```
MessageType.Voice = int
MessageType.SMS = string
QueryType.QType = ...
```
q: what about maybe/optional and DayOfWeek?
The dot notation is a bit confusing with struct. Can we use another notation? e.g. `Shape|Circle`
or think of it as a tag: `Circle@Shape = {...}` - in that case, can we have multiple tags for a type?
dot implies parent/child or container/contained relationship.
`|` implies or relationship.
we can have multiple tags: `Shape|Circle`, `Item|Circle`, ...
`Shape+Circle`
`Shape/Circle`
It should be a composable notation. 
Use regex? ShpCircle, ShpSquare, ... no too complicated
```
DayOfWeek.Saturday = 1 #this is a type which has only one valid value
DayOfWeek.Sunday = 2
...
```
```
Maybe[T: type].Nothing = nothing
Maybe[T: ty[e].Item = T
...
x: Maybe[int]
```
No. This is too complicated.
Can't we use current `Shape` union to restrict possible types?
`CandidateList = [T: Shape -> {T: type, handler: (T->), next: nothing|CandidateList[_]}]`
we can but it will make reading code more difficult.
Let's let it stay the same.
the problem with this union is that it is not composable with generics easily.
We can have generic at union type level and at subtypes level which makes it super confusing:
`Processor[T: type].Handler[S: type] = (S->T)`

Y - Can we make union with constant values more explicit/consistent?
`Shape = Circle | Square`
`DayOfWeek = SAT | SUN ...`
we can introduce fixed types where there is only one valid value for them, like `nothing`.
If right side of type is a constant (compile time calculatable), it defines a fixed type.
```
Sat = 1
Sun = 2
Mon = 3
WeekDay = Sat | Sun | ...
x: WeekDay = Sat
```

N - Can we combine generics?
`process = (T: ||, G: ||, data: G[T] ...`
This is not for language. Mostly for compiler.

Y - Having access to common parts of union types may be confusing at some times.
and hidden.
There should be a way to "declare" those common parts.
So if I write `Shape = Shape | Square` writer of Square knows what should be included.
So what are these common parts? Having specific fields of name + type.
```
Shape = Circle | Triangle { id: int, process: (?->int) }
```
This is applied where we have a Shape and don't know it's actual type.
Why not make it easier to cast?
```
#all shapes have id:int which we want to return here
process = (T: Shape, s: T -> int)
process = (T: Circle, s: T -> s.id) 
```
This can be done via specialisation. or you can cast.

N - If we go ahead with replacing generics notation with union, we can replace polymorphism with generic speciaisation:
`draw = (T: Shape, x: T -> int)`
`draw = (T: Circle, x: Circle -> int) ...`
`draw = (T: Square, x: Square -> int) ...`
`draw(myCircle)` will call draw for Circle
`draw(my_triangle)` will call generic draw (code generated for Triangle)
so, how can I address these in a lambda?
`myLambda = draw(_: Shape, _)`
`myLambda = draw(_:Circle, _:Circle)`
using `_` with lambda hides the fact that argument is a type or a binding.
If we do this, then we will not need `_` notation in generics.
So when using `_` notation to create lambda, we should specify type for some arguments so that compiler will know which one to pick.
Let's think about it like this: It is only one function. Lambda will point to the main one, and depending on the argument values, one of available functions will be used.
```
myPtr = draw(_,_)
myPtr2 = draw
myPtr3 = draw(Circle, _) #this will point to the function for draw with circle
```
What if we have more than one generic arg?
`Shape = Circle | Square`
`Color = Blue | Red`
`draw = (T: Shape, C: Color, shape: T, color: C -> ...`
`draw = (T: Circle, C: Color, shape: T, color: C -> ...`
`draw = (T: Shape, C: Blue, shape: T, color: C -> ...`
`draw(Circle, Blue, my_circle, my_blue_color)` which one will this call?
I think there should be a compiler error. Because third draw conflicts with the second draw.
Any specialisation, should be as specific as possible. Because it marks an entire tree of hierarchy for providing implementation for.
So `(Circle, Color, ...` means any call for a Circle and Red or Blue will be directed to this function.
so when third draw says `(Shape, Blue)` it has conflict because we can call it with `Cirtcle, Red` which conflicts with the second draw.
```

Shape, Color => Circle, Color => Circle, Red
				 Circle, Blue
		Square, Color => Square, Red
				 Square, Blue
```
If you specialise, it will cover every more specialised function. So `(Circle, Color)` will cover `Circle, Red` and `Circle, Blue`
So:
**Proposal**
1. You can specialise a generic function by providing concrete types and same name. They will all be merged as one function.
2. When creating a lambda, you cannot point it to any specific function. You can only provide concrete types when defining the lambda.
3. Any specialisation, will also handle for more specialised functions. This applied for generic functions with more than one type.
4. If there is conflict between specialisations, there will be compiler error.

N - If I allow for union-based generics can I specialise for values?
```
process = (x: int, flag: true|false) ...
process = (x: int, flag: true) ...
process = (x: int, flag: false) ...
```
It will be a bit confusing. If process is called with `true` which function should be called?
Someone would say, the most specific function. But in presence of multiple candidates it is difficult to know which one it will be.
In OOP, it is easy to know the most specific function: it is determined by the runtime type: `my_circle.draw()` will call draw in Circle not Shape.
Think about complexity that this will lead if we allow generics specialisation: multiple implementations under the same name, confusion about resolutions, ...
We are basically simulating inheritance in a little bit more explicit way.
Generics was introduced to solve a different problem: writing reusable code/data types for all types.

N - If we allow using generics with union, In this case, polymorphism can become generics specialisation. If we can do that:
`draw = (T: Shape, x: T -> int)`
`draw = (T: Circle, x: Circle -> int) ...`
`draw = (T: Square, x: Square -> int) ...`
So compiler will generate draw code for Triangle but not Circle and Square. We already have them.

N - We can implement protocol or type-class or interface or trait using unions.
we define functions that a type group should support.
This can provide polymorphism. And if implemented correctly, maybe it won't have confusion and ambiguity of generics specialisation solution.
```
Shape = Circle | Square { draw: (T->) }
Shape = [T: anytype -> {draw: (T->) }
ShapeX = Shape[Circle] | Shape[Square]
```
No. This won't give us polymorphism becase this functionality is not same across all union types: It is different because input of the draw is different.
We can, however, remove that variable type and assume draw will be a member function:
```
Circle = {... draw: (int->string) { ...} }
Square = {... draw: (int->string) {...} }
Shape = {draw:(->)}
Shape = Circle | Square
```
And it is not an active definition: We don't say this is a union of all types that have this
But we say, all types that want to join this union, you must have these fields (name and type).
One way: Define it like a struct but enclosed in `||`:
```
Shape = |draw: (int->string)| #Shape is a union. Any type that you want to add to this union, must have these fields
#currently there is no type in shape
Shape = Shape | Circle #We have already defined draw field inside Circle
Shape = Shape | Square
...
```
How does this affect expression problem? If we want to add a new operation (e.g. print) and we have followed above model, it will not be possible.
Because to add `print`, I will need to change all structs and Shape type and add this function.

N - Use tagged union.
Advantage: With untagged union we should put a rule that "no overlap between types". But with tagged this is allowed.
How will it affect current polymorphism?
Proposal: Drop proposal for using union for generics, use `type` + make unions tagged
How will we have dynamic compile time generics then? 
What is the advantage of amending a union and adding new types to it? `Shape = Shape | Circle`? The only one is to have a type to use for polymorphism.
```
#define Shape type using compile-time dynamic union
Shape = Circle | Triangle | Square
Shape = Shape | Square

#Type of a function candidate which does a specific operation for a specific type.
CandidateList = [T: type -> {T: type, handler: (T->), next: nothing|CandidateList[_]}]
#Define a linked list of handlers for different types.
shape_handlers = CandidateList[Circle]{Circle, drawCircle, nothing}

#amend the list of handlers
shape_handlers = CandidateList[Square]{Square, drawSquare, shape_handlers}
shape_handlers = CandidateList[Triangle]{Triangle, drawTriangle, shape_handlers}

#The draw function can be invoked on any shape
draw = (s: Shape, element: CandidateList[_] -> ) { #This is the only place where we need a Shape. and we want to make it extensible.
	if element.T is same as internal type of S, then call element.handler and return
	if element.next is nothing then return
	else recursively call with element.next
}
```
To implement polymorphism, the only place where we need a `Shape` is in draw. Can we eliminate it by using generics?
`draw = (T: type, s: T, element: CandidateList[_] -> )`
Yes, but how can I call draw with something I'm not sure about it's type. Remember the example where we read a shape from file/network/...
I think we cannot eliminate it.

N - We can say, union type when used as type of binding specifies range of possible values it can hold.
When used as a generic type specifier, shows possible types that generic type can hold.
```
process = (s: Shape) ... #s is a binding, so it can hold any shape
process = (T: Shape ... #T is a type, so it can be any Shape subtype
```
No change in syntax. No change in notation.
also we can say `Type = ||` means union of all possible types.
So no need for keyword `type`.
The difference is that values for `type` must be specified at compile time but for a binding of type `Type` their value can come at any time.
```
Stack = [T: || -> {data: T, next: Stack[T]}]
find = (T: ||, array: Seq[T], item: T -> int)
```
so:
**Proposal**: Replace generics notation with union types
1. When a union type is used for a binding, that binding may have value for any of it's types.
2. When a union type is used for a type specifier, it represents valid types for that type and that type can be used to specify type of other arguments.
3. `||` is a union type of all possible types.
If all types of a union share `name:string` then you can refer to it inside a generic function for that type.
Can we have a parameteric generic?
`process = (T: ||, X: ShapeHolder[T], ...`
So this also means I can define bindings of type `||`, which is same as `void*` or `interface{}`
`draw = (T: Shape, x: T`
We can prohibit using `||` for a binding but it is not according to universal rule of the language: orthogonality
**Con**: This makes things more complex. Right now we have `type` and that's it. But with unions, users can limit types, put constraints and maybe mis-use this by having access to common fields of a union, hence make it difficult to solve expression problem.

N - If we go with union for generics, we can add a new type: `anything` which is basically union of all types.
rather than `||`
It makes more sense than `||` or `type`.
Any binding of type `anything` must be assigned with a literal or another binding of the same type. So values are decided at compile time.
`process = (T: anything, x: T -> int)...`
`process(int, 10)`
`process(string, "A")`
anything is a bit too long and confusing. (maybe we should say anytype)
`nil`, `nothing`, `none`, `void`
`||`, `something`, `anything`, `any`
`int`, `float`, `char`, `byte`, `string`, `bool`, `nothing`, `type`, `ptr`
so we will remove `type`, and replace it with `||`?
If a binding is of type `anything` it can be anything.
If a type is, it can be any type.

N - How can we easily implement thread join?
```
wid = newProcess()
waitFor = (w: wid -> receive((m: Message -> m.sender = wid and m.type = DONE))
```

N - How to do complex logics or data validations?
`ifElse` and `::`

N - use `&` to get ptr for a binding or anything

N - Mayeb we should use `?` instead of `_` to show generic with some type.

N - Is there a way to call a function which accepts a Circle with a Shape type if we are sure args match?

N - Is there a way to get type id of a union?

N - (copied to next item because it got too long) Another alternative: Rather than providing compile-time union, let user implement an extensible union via a linked list
and an array of shapes via a linked list
```
Shape = {t: type, item: T, next: Shape}]
shape_type = Shape[Circle]{item: 
```
No.
We can have a linked list of "valid" types and prepend to this list in compile time:
```
AllowedTypes = {t: type, next: nothing|AllowedTypes}
shape_types = AllowedTypes{t: Circle, next: nothing}
shape_types = AllowedTypes{t: Square, next: shape_types}
shape_types = AllowedTypes{t: Triangle, next: shape_types}
```
Then we can define a safe seq:
```
SafeSeq = {data: Seq[{type, ptr}], allowed_types: AllowedTypes}
```
Then a method that reads shapes from a file, can return SafeSeq of shape_types.
Another way: use `+=`
```
Shape = Circle
Shape += Square
Shape += Triangle
data: Seq[Shape]...
```
This is same as what we already have.
Can we just have Haskell's typeclass?
```
Drawable = [T: type -> {draw: (T->int)}]
Circle = ...
Square = ...
Circle#draw = (Circle->int)...
Square#draw = (Square->int)...
process = (x: Drawable) ...int_var = draw(x)
```
Actually, in Haskell you use typeclass as a constraint for generic. But we want dynamic runtime polymorphism.
```
Drawable = [T: type -> {draw: (T->int)}]
Circle = ...
Square = ...
Circle#draw = (Circle->int)...
Square#draw = (Square->int)...
process = (x: Drawable) ...int_var = draw(x)
```
Another solution: Tagging a type. It is like union but extensible and each type can belong to multiple tags
```
Circle = ...
Square = ...
Circle@Shape
Square@Shape
draw = (x: Shape, ...)
```
Let's ask the question the other way around: Why do we need this?
Why not rely on a low-level solution like ptr+type?
When we keep a sequence of shapes, what is the type of what's inside the sequence? Is it a tag? => A new concept which makes language more complicated.
We prefer a solution which does not add a new concept, uses existing concepts, has an intuitive syntax and is expressive and easy to use.
If we can modify the ptr+type solution to be more static type it would be great. because it only uses existing features.
What about keeping original type in ptr (e.g. byte, Circle, ...) For a sequence this is ptr.
So, we enhance ptr to be more type checked: We provide a set of allowed types. But how? We don't have sequence.
variadic? Linked list? So internally, ptr is a number and a type which specifies size. at runtime we only need size but during compilation we need type too.
One advantage is people cannot mis-use ptr: A ptr to byte can be only used as byte. We cannot store an 80-bit double number in a byte ptr.
```
ShapeList = Seq[ptr of @Shape]
Circle@Shape
Square@Shape
```
But how can I draw shapes from a ShapeList?
Keeping a ptr with allowed types is what a union does.
Can we replace union with this? No. Union is simple and intuitive. ptr with type is not. it is not intuitive.
What is the current solution? Extensible union + linked list of functions
What is wrong with it? The problem is, no part of this needs `Shape` type except the final `draw` function. Why not drop this strange notation?
```
#define Shape type using compile-time dynamic union
Shape = Circle | Triangle
Shape = Shape | Square

#Type of a function candidate which does a specific operation for a specific type.
CandidateList = [T: type -> {T: type, handler: (T->), next: nothing|CandidateList[_]}]
#Define a linked list of handlers for different types.
shape_handlers = CandidateList[Circle]{Circle, drawCircle, nothing}

#amend the list of handlers
shape_handlers = CandidateList[Square]{Square, drawSquare, shape_handlers}
shape_handlers = CandidateList[Triangle]{Triangle, drawTriangle, shape_handlers}

#The draw function can be invoked on any shape
draw = (s: Shape, element: CandidateList[_] -> ) { #here is where we need ?
	if element.T is same as internal type of S, then call element.handler and return
	if element.next is nothing then return
	else recursively call with element.next
}
```
We can follow what we do for functions, for types. But it will be complicated.
Maybe we should provide some feature to make defining a linked list easier.
A shape type is a linked list of types of x or nothing. But this is not a type. This is a linked list with values.
Of course linked list has a type.
```
ShapeType = [T: type -> {value: T|nothing, next: ShapeType[_]}]
createCircle = (->ShapeType[Circle]) ...
createSquare = (->ShapeType[Square]) ...
draw = (s: ShapeType
```
Maybe we should put linked list inside the type:
```
ShapeType = {t: Circle, value: Circle|nothing, next: nothing} #this is the original definition
ShapeType = {t: Square, value: Square|nothing, next: ShapeType}
ShapeType = {t: Triangle, value: Triangle|nothing, next: ShapeType}
createCircle = (->ShapeType[Circle]) ...
createSquare = (->ShapeType[Square]) ...
draw = (s: ShapeType
```
Order of execution of above ShapeTypes does not matter. It can be Circle->Square->Triangle or Circle->Triangle->Square
No. This is wrong. Type of ShapeType changes so type of `next` changes.
Instead of `Shape = Circle | Square | Triangle` we can break it into 3 definitions:
```
Shape = Union of all types below
Shape[0] = Circle | nothing
Shape[1] = Square | nothing
Shape[2] = Triangle | nothing
```
Finding all types that are part of shape might be difficult: find `shape = shape |` in the code.
Maybe we can eliminate this by adding a notation to extend a union:
```
IntOrString = int | string
Shape = Circle | Square
...
Shape |= Triangle
```
The notation should be so obvious that it is easy to search by human or software.
Or, define an implicit protocol. Any type that has these, is implementing the protocol:
```
Shape = { name: string, draw: (->) }
Circle = {name: string, r: float, draw: (->)}
Square = {name: string, s: int, draw: (->)}

#The draw function can be invoked on any shape
draw = (s: @Shape, element: CandidateList[_] -> ) { #here is where we need ?
	s.draw()
}
```
So, `@T` creates a union type (we don't want to invent a new concept), containing all types that implement it.
We need access to common parts of a union types. Is it ok?
There should be a way to "declare" those common parts. And `Shape` is that way (Note that Shape is just a normal struct, but we make it a protocol by `@` prefix).
So, when you have an argument of type `@Shape`, you can access the items mentioned in Shape struct.
```
Shape = { name: string, draw: (->) }
Circle = {name: string, r: float, draw: (->)}
Square = {name: string, s: int, draw: (->)}

#The draw function can be invoked on any shape
draw = (s: @Shape -> ) { #here is where we need ?
	s.draw(s.name)
}
```
And still you can follow the linked list method for handlers.
BUT: this may add unwanted types to the union. `@Shape` may include customer too.
Unless we manually include types in it.
```
Shape = { name: string, draw: (->) }
Circle = {name: string, r: float, draw: (->)}

Square = {name: string, s: int, draw: (->)}
Square=>Shape

#The draw function can be invoked on any shape
draw = (s: %Shape -> ) { #here is where we need ?
	s.draw(s.name)
}
```
But this is too much code. If we define the protocl with care, it should be fine.
Btw now `%{}` will be a union of all structs. But we don't want to use it in generics.
```
Shape = { name: string, draw: (->) }
Circle = {name: string, r: float, draw: (->)}
Square = {name: string, s: int, draw: (->)}

#The draw function can be invoked on any shape
draw = (s: %Shape -> ) { #We are given a type which supports name and draw. And that's what we need.
	s.draw(s.name)
}
```
**Proposal**
1. No change in struct notation. But you can use a struct as a blueprint to gather a union of multiple types that include it's fields.
2. `%Shape` where `Shape={name:string}` will be union of all struct types that have name of string.
3. You can define bindings of this special union type.
4. This can be used to provide polymorphism.
```
Drawable = { draw: (->) }
Circle = {name: string, r: float, draw: (->)}
Square = {name: string, s: int, draw: (->)}

#The draw function can be invoked on any shape
draw = (s: %Drawable -> ) { #We are given a type which supports name and draw. And that's what we need.
	s.draw(s.name)
}
```
Making things centralised makes reading and understanding the code easier. But also makes it difficult to maintain and extend.
What if I want to add support for a protocol to an existing type?
e.g. I have triangle in an imported library -> I cannot change it and add `name:string`. So that's why it's better to rely on functions.
Even for fields, i can simply add a function which returns that field from struct.
But as we do not support specialisation, It's better not to use generic functions.
Go: interface is just a set of functions. You can implement it for any type you want by writing those functions.
Haskell: typeclass is a function map
```
Drawable = [T: type -> { draw: (T->) }]
Circle = {name: string, r: float}
Square = {name: string, s: int}
drawCircle [implement Drawable on Circle] = (c: Circle -> ...) #this is an implementation for drawable protocol on type Circle
drawSquare [implement Drawable on Square] = (s: Square -> ...)
#Here compiler knows that Circle and Square are two types that support Drawable
#The draw function can be invoked on any type for which we have implemented Drawable
draw = (s: Drawable -> ) { #We are given a type which supports name and draw. And that's what we need.
	s.draw()
}
```
Another example: Compare function. I need a function working on something which supports equality check.
We want to combine this with union. So 1. we have a new type which is union of all types that support this protocol.
2. we have the protocol which shows us the common functions across all those types.
I think when writing a function, we should indicate what protocol it is implementing for what type.
Because otherwise, we should specify this connection either at module level or as function argument.
If a type does not implement a protocol, we can simply add that support by writing appropriate functions. But this connection cannot be automatic because we cannot have functions with the same name. And without function name, there is no way for compiler to draw these connections.
Rust: `impl Animal for Sheep ` signals to the compiler that we are implementing trait X for type Y
Haskell: `instance Eq Praat where` Defines Praat type implements `Eq`
There are two ways to make connection (which function is implementing each function in the protocol):
1. naming convention `Drawable = [T: type -> { T##draw: (T->) }]`
2. explicit in the definition: `drawCircle [implement Drawable on Circle] = (c: Circle -> ...)`
The first one is simpler but not very flexible. So if a function `circleDraw` already exists, we have no way to add support.
The second one is better but needs more change.
I really don't want to change the notation to define a function. Because it will have a lot of implications.
```
Drawable = [T: type -> { draw: (T->) }]

Circle = {name: string, r: float}
Square = {name: string, s: int}

drawCircle = (c: Circle -> ...) 
drawSquare = (s: Square -> ...)

#here we register types with protocol
Drawable[Circle] = { draw: drawCircle }
Drawable[Square] = { draw: drawSquare }

#Now, the compiler knows that Circle and Square are two types that support Drawable
#The draw function can be invoked on any type for which we have implemented Drawable
draw = (T: type, item: T, s: Drawable[T] -> ) { #We are given a type which supports name and draw. And that's what we need.
	s.draw(item)
}
```
We can say, `Drawable[?]` means union of all types that are registered with Drawable.
Similarly, if we have a multi-type protocol, `Comparable[?, int]` is all types that can be compared with int.
```
draw = (s: Drawable[?] -> ) { 
	Drawable[s].draw(s)
}

draw = (T: type+Drawable, s: Drawable[T] -> ) { 
	Drawable[s].draw(s)
}
```
We know that generics can only be invoked with concrete hard coded types. But what if that type is a union?
I think I should be able to invoke generic with a union type and compiler will implement all cases (because it is limited).
Of course in this case, the generic function cannot have `T: type` because it means implement this function for every type.
The type should be limited via protocols.
```
draw = (T: Drawable, s: T -> ) { 
	Drawable[T].draw(s)
}
draw(type(my_shape), my_shape)
```
Why can't we simply use a function pointer?
```
xdraw = (T: type, item: T, draw: (T->) {
	draw(item)
}

xdraw(Circle, my_circle, drawCircle)
```
Before draw, we should think about the function that will return a shape. What will be it's output?
```
Drawable = [T: type -> { draw: (T->) }]

#here we register types with protocol
Drawable[Circle] = { draw: drawCircle }
Drawable[Square] = { draw: drawSquare }

getShape = (name: string -> Drawable[?]) {
	if name is Circle return Circle{1.5}
	if name is Square return Square{2}
}
#Drawable[?] is a type which can be any of types for which we have defined Drawable
#it can be used as a type for a generic function or as type of a binding
xdraw = (T: Drawable[?], item: T ->) {
	Drawable[T].draw(item)
}

my_circle = getShape("circle") #type of my_circle is Drawable[?]

#below two are the same
Drawable[type(my_circle)].draw(my_circle)
xdraw(type(my_circle), my_circle)
```
You can call a generic function with a normal hard-coded type or also you can call it with a union.
Because with union, number of cases is limited, so compiler can generate code and dynamic function invoke code.
But we are not interested in a union here. Because it requires dynamic union.
We have a protocol. But still we need union. At least for output of getShape.
But `Drawable[?]` is not a union. It is a struct with some fields. but it's type is dynamic.
Above code, seems too complicated: Special notation `?`, setting value for a type, generic with union, ...
What is the problem:
1. We need a mechanism to specify our expectation about `T` in a generic function.
2. We want to have a better polymorphism.
About 2, can't we just follow the protocol method? Define a polymorphic function and bind it for different types to different functions.
```
#basically following hashtable approach but for generic type
#kind of specialisation
DrawPoly = [T: type -> (T->)]
DrawPoly[Circle] = drawCircle
DrawPoly[Square] = drawSquare

Shape = Triangle|Square
Shape = Shape | Circle

createShape = (s: string -> Shape) ...
my_circle = createShape("Circle")
DrawPoly[type(my_circle)]()
```
No. It is not intuitive to assign to `DrawPoly[Circle]`. Instead of this, let's just focus on existing solution and make sure it is comprehensive, easy, expressiev and re-usable.
So we have one problem:
- To explicitly specify expectations in a generic function.
1. We need a mechanism to specify our expectation about `T` in a generic function.
If we replace generic type with union and add something to specify common things between union types, this can be solved.
Just make sure we still can solve expression problem.
```
#every type that wants to join Shape union, must have these
Shape = { getName: (->string) } 
process = (T: Shape, g: T -> ...) { x = g.getName() }
```
So joining union is not automatic, not every type that has `getName` will become member of `Shape` union. 
We have to join them manually.
but it will be difficult to extend existing types to support a union.
```
#every type that wants to join Shape union, must have these
Shape = [T: type -> { draw: (T->string) } ]
Shape[Circle] = { draw: drawCircle }
Shape[Square] = { draw: drawSquare }

Shape = Shape | Circle {getCircleName}
process = (T: Shape, g: T -> ...) { x = g.getName() }
```
Another easier way: Add compile time check to make sure some functions are defined.
```
#in process we expect some functions to be defined for type T
process = (T: type, g: T, ...) 
defined: getName: (T->string)
{ ... }
process = (T: type, g: T, getName: (T->string)) 
{ ... }
```
`process = (T: type, g: T, getName: (T->string))`
`process = (T: type, g: T, getName: (T->string))`
Rather than relying on some special notation, let's just add a function ptr to the argument list.
Caller can decide which implementation to use for this. And it is completely visible and explicit.
Can we use this to provide easier polymorphism? We want different `getName` implementations based on `T`.
`process = (T: type, g: T, invokeGetName: (T ->string))`
We said, it does not make sense to specialise generic functions but what about generic types?
This can also help with polymorphism.
```
ShapePainter = [T: type -> { draw: (T->string) } ]
ShapePainter[Circle] = { draw: drawCircle }
ShapePainter[Square] = { draw: drawSquare }
...
process = (T: type, shape: T, painter: (T->string) -> ...
...
process(type(my_shape), my_shape, ShapePainter[type(my_shape)].draw)
```
So, we will need these:
1. Specialisation for generic types
2. Calling generic function with union
But maybe we can do without calling generic with union.
```
ShapePainter = [T: type -> { draw: (T->string) } ]
ShapePainter[Circle] = { draw: drawCircle }
ShapePainter[Square] = { draw: drawSquare }
...
process = (my_shape: Shape -> ... ) {
	painter = ShapePainter[type(my_shape)].draw
	str = painter(unwrap(my_shape))
}
...
process(type(my_shape), my_shape, ShapePainter[type(my_shape)].draw)
```
Seems that with generics call with union it is more readable:
```
ShapePainter = [T: type -> { draw: (T->string) } ]
ShapePainter[Circle] = { draw: drawCircle }
ShapePainter[Square] = { draw: drawSquare }
...
process = (T: type, shape: T -> ) {
	painter = ShapePainter[T].draw
}
...
process(type(my_shape), my_shape)
```
**Proposal**:
1. Keep compile-time dynamic union
2. Support specialisation of generic types. The syntax forces you to specify concrete type for all parameters
3. Support calling a generic function with a union type.
q: Should we replace `type` with union? Will this help? 
I see more and more complexity. How can we do this with only supporting specialisation for genneric types?
```
ShapePainter = [T: type -> { draw: (T->) } ]
ShapePainter[Circle] = { draw: drawCircle }
ShapePainter[Square] = { draw: drawSquare }
...
process = (shape: Shape -> ) {
	hapePainter[type(shape)].draw(unwrap(shape))
}
...
process(my_shape)
```
So the proposal is:
1. Support specialisation of generic types. The syntax forces you to specify concrete type for all parameters.
2. We need a notation to get internal type of a union. This results in calling generics with a union binding. But it should be fine.
3. If we have `draw: (Circle->)` and a shape which contains a Circle, there should be a simple mechanism to do the call.
```
#you can store any type specific data inside this struct
ShapePainter = [T: type -> { draw: (T->) } ]
ShapePainter[Circle] = { draw: drawCircle }
ShapePainter[Square] = { draw: drawSquare }
Converter = [S,T: type -> { convert: (S->T) } ]
Converter[Customer] = ... #invalid
Converter[Customer, Int] = ... #valid
...
#To draw some shape
my_shape = getShape(...)
ShapePainter[type(my_shape)]
draw = (shape: Shape -> ) {
	shapePainter[type(shape)].draw(unwrap(shape))
}
...
draw(my_shape)
```
We want to have one thing, but distributed in several files. Now the problem is: How are we going to connect all of them together.
This is the way we define dynamic union, generic type specialisation and polymorphism.
The problem with above code: ShapePainter is a type, but we are assigning a value to it.
The notation of linked list at module level, makes some sense: We define a module-level struct. Then we update it during compile time.
And note that if we use above solution, we will still need dynamic union.
```
#define Shape type using compile-time dynamic union
Shape = Circle | Triangle
Shape = Shape | Square

#Type of a function candidate which does a specific operation for a specific type.
CandidateList = [T: type -> {T: type, handler: (T->), next: nothing|CandidateList[_]}]
#Define a linked list of handlers for different types.
shape_handlers = CandidateList[Circle]{Circle, drawCircle, nothing}

#amend the list of handlers
shape_handlers = CandidateList[Square]{Square, drawSquare, shape_handlers}
shape_handlers = CandidateList[Triangle]{Triangle, drawTriangle, shape_handlers}

#The draw function can be invoked on any shape
draw = (s: Shape, element: CandidateList[_] -> ) { #here is where we need ?
	if element.T is same as internal type of S, then call element.handler and return
	if element.next is nothing then return
	else recursively call with element.next
}
```
Maybe we can simplify the above notation and let it stay as a solution.
Maybe we don't really need polymorphism.
If we have `Shape = Circle | Square` and we add triangle, and we cannot modify existing code -> Then add a new type
`NewShape = Shape | Triangle` and forward other functions and ...
In this way we can use functions to cover all cases of a union.
We can say Haskell's type-class is same as generics.
And each instance is a specialisation of that generic.
read: https://koerbitz.me/posts/Solving-the-Expression-Problem-in-Haskell-and-Java.html
Another idea: Let union remain the same. Follow Haskell's approach.
To solve expression problem: Use typeclass, where you can implement a typeclass for any type.
And typeclass can be input of a function.
Rather than adding unintuitive notations, divide it into 3-4 small intuitive notations.
read: https://blog.codecentric.de/en/2017/02/ad-hoc-polymorphism-scala-mere-mortals/
Like what we did for concurrency: adding send and receive functions, instead of strange operators.
```
#you can store any type specific data inside this struct
ShapePainter = [T: type -> { draw: (T->) } ]
ShapePainter[Circle] = { draw: drawCircle }
ShapePainter[Square] = { draw: drawSquare }

...
getShape = (name:string -> ShapePainter[?]) {
	...
}

draw = (shape: Shape[?] -> ) {
	shape.draw()
}
...
#To draw some shape
my_shape = getShape(...) 
draw(my_shape)
```
But `Shape[Circle]` seems un-intuitive.
"Type systems with subtyping are dramatically more complicated than those without"
This means if we have this:
`coords :: [Coord]
coords = [a, b]`
We should be aware that a is of type CartesianCoord and also it is a Coord.
Implement both new type and new operation.
Do we really need to add support for expression problem?
Adding new cases to a type/operation can cause issues like fragile base class issue in OOP.
Is there a way to implement this without "ANY" change to the language?
We know in the operation side, this is possible using a linked list.
The only issue: extending union notation.
```
#you can store any type specific data inside this struct
Circle = {...}
Square = {...}

ShapePainter = [T: type -> (T->) ]
ShapePainter[Circle] = drawCircle
ShapePainter[Square] = drawSquare

Shape = ShapePainter[?]

...
getShape = (name:string -> ShapePainter[?]) {
	...
}

draw = (shape: Shape[?] -> ) {
	shape.draw()
}
...
#To draw some shape
my_shape = getShape(...) 
draw(my_shape)
```
No. This notation of generic type specialisation does not make any sense. Let's stick to the linked list solution for now.
```
#define Shape type using compile-time dynamic union
Shape = Circle | Triangle
Shape = Shape | Square

Shape = {}
Shape = {Circle, *Shape}
Shape = {Square, *Shape}
#now Shape is a struct of {Circle, Square}

#Type of a function candidate which does a specific operation for a specific type.
CandidateList = [T: type -> {T: type, handler: (T->), next: nothing|CandidateList[_]}]
#Define a linked list of handlers for different types.
shape_handlers = CandidateList[Circle]{Circle, drawCircle, nothing}

#amend the list of handlers
shape_handlers = CandidateList[Square]{Square, drawSquare, shape_handlers}
shape_handlers = CandidateList[Triangle]{Triangle, drawTriangle, shape_handlers}

createShape = (name: string -> Shape) {...}

#The draw function can be invoked on any shape
draw = (s: Shape, element: CandidateList[_] -> ) { #here is where we need ?
	if element.T is same as internal type of S, then call element.handler and return
	if element.next is nothing then return
	else recursively call with element.next
}
```
What if we define a ptr with a set of valid types for it?
```
Shape = {s: ptr, types: {}}
Shape = 
```
idea: OOP solves expression problem by visitor pattern which basically means adding an extension point inside classes, so we can easily add new operations.
Maybe we should follow the same. put an extension point in the functions so we can easily add new types.
Another idea: We define a master functio with type Shape and it's value is defined by specialisations on the master function.
But this will no longer be a type.
Doesn't this conflict with SOLID rule: A Type should be open to extension, but closed for edits.
Another solution based on visitor:
- createShape returns an `accept` lambda which is the internal lambda of Circle or Square or ...
- Caller will have a lambda `accept` which it can call for different operations.
- We don't even need to store `accept` inside the type. It can be a lambda inside createShape.
But if we do it inside createShape, we won't be able to extend types.
```
Circle = {...}
Square = {...}

getShapePainter = (string: name -> (Color->)) {
	if name is "Circle" c = read_circle, return (Color->draw_c)
	if name is "Square" ...
}

painter = getShapePainter(name)
painter(Black)
```
Suppose that we add another op: reverseShape 
How can we compose these? reverse and then paint the shape?
Another idea: getShape returns anything type, but the internal type has an `accept` function.
You can call accept with the hander LinkedList and it will find it's own type and execute the handler function.
```
#this is the definition of handler function on a Shape 
#which is also input of shape's accept functions
CandidateList = [T: type -> {T: type, handler: (T->), next: nothing|CandidateList[_]}]

#we don't know what will be the head of CandidateList so we use `_`
Circle = {... 
	accept = (CandidateList[_] -> find handler for Circle type and run it on me)
}
Square = {...
	accept = (CandidateList[_] -> find handler for Square type and run it on me)
}

#Define a linked list of handlers for different types.
shape_handlers = CandidateList[Circle]{Circle, drawCircle, next: nothing}

#amend the list of handlers
shape_handlers = CandidateList[Square]{Square, drawSquare, shape_handlers}
shape_handlers = CandidateList[Triangle]{Triangle, drawTriangle, shape_handlers}

getShape = (string: name -> (CandidateList->)) {
	if name is "Circle" c = read_circle {
		c = Circle{..., accept: (x: CandidateList[_] -> innerAccept(x, c)),
					innerAccept: (x: CandidateList[_], c: Circle -> ... }
		return Circle{...accept...}.accept
	}
	if name is "Square" ... return Square {...accept...}.accept
}

painter = getShape(name)
painter(shape_handlers)
```
q: Do we need a notation to refer to the containing struct? Can't we hide this from outside? I think we can.
And maybe we can write one universal `innerAccept` function.
```
#this is the definition of handler function on a Shape 
#which is also input of shape's accept functions
CandidateList = [T: type -> {T: type, handler: (T->), next: nothing|CandidateList[_]}]

#we don't know what will be the head of CandidateList so we use `_`
Circle = {... 
	accept = (CandidateList[_] -> find handler for Circle type and run it on me)
}
Square = {...
	accept = (CandidateList[_] -> find handler for Square type and run it on me)
}

#Define a linked list of handlers for different types.
shape_handlers = CandidateList[Circle]{Circle, drawCircle, next: nothing}

#amend the list of handlers
shape_handlers = CandidateList[Square]{Square, drawSquare, shape_handlers}

#adding a new shape
Triangle = { ...
	accept = (CandidateList[_] -> ...)
	...
}
shape_handlers = CandidateList[Triangle]{Triangle, drawTriangle, shape_handlers}

#adding a new operation:
save_handlers = CandidateList[Circle]{Circle, saveCircle, next: nothing}
shape_handlers = CandidateList[Square]{Square, drawSquare, shape_handlers}
shape_handlers = CandidateList[Triangle]{Triangle, drawTriangle, shape_handlers}


getShape = (string: name -> (CandidateList[_]->)) {
	if name is "Circle" c = read_circle {
		c = Circle{..., accept: (x: CandidateList[_] -> innerAccept(x, c)),
					innerAccept: (x: CandidateList[_], c: Circle -> ... }
		return Circle{...accept...}.accept
	}
	if name is "Square" ... return Square {...accept...}.accept
}

painter = getShape(name)
painter(shape_handlers)
```
To prevent using `_`, I can add a new dummy type which only points to shape handlers.
```
HandlerList = [T: type-> {t: type, handler: (T->), next: nothing|HandlerList[?]}

#we don't know what will be the head of CandidateList so we use `_`
Circle = {... 
	process = (HandlerList[?] -> find handler for Circle type and run it on me)
}
Square = {...
	process = (HandlerList -> find handler for Square type and run it on me)
}

#Define a linked list of handlers for different types.
shape_handlers = HandlerList[Circle]{t: Circle, handler: drawCircle, next: nothing}
shape_handlers = HandlerList[Square]{t: Square, handler: drawSquare, next: shape_handlers}

getShape = (string: name -> (HandlerList->)) {
	if name is "Circle" 
		c = Circle{...}
		lambda = (x: HandlerList[?] -> if x.t == Circle then run x.handler(c) else return lambda(x.next))
		return lambda
	}
	if name is "Square" ... 
}

myShapeProcessor = getShape("Circle")
myShapeProcessor(shape_handlers)
```
`CandidateList[?]` cannot be resolved at compile time.
It seems that we should give up type at some point in the chain.
Seems above is the most sensible and consistent solution. Now we should try to make it easy.
e.g. making sure `Handler[?]` can accept parameter x. we can use `func<data>` but it is a large change.
Why not use core? canInvoke: `canInvoke(handler, my_circle)::handler(my_circle)`
`canInvoke = (x: (?->?), data: ?)`
What if we add a core function for: look over these handlers and invoke the one which is suitable with these inputs.
What is we use a struct with `n*2` fields, for `n` handler functions?
The more code we write, the more control we can have over the dispatch.
```
#no LL, no next, no type, no generics, only one big anonymous struct
handlers = {Circle, drawCircle}
handlers = {*handlers, Square, drawSquare}

HandlerList = TypeOf(handlers)

Circle = {...}
Square = {...}

getShape = (string: name -> (HandlerList->)) {
	if name is "Circle" 
		c = Circle{...}
		lambda = (x: HandlerList, i:int -> if x.i == Circle then run x.(i+1)(c) else return lambda(x, i+2))
		return (x:HandlerList -> lambda(x,0))
	}
	if name is "Square" ... 
}

myShapeProcessor = getShape("Circle")
myShapeProcessor(handlers)
```
No. adressing a struct with `.i` makes it like an array.
Using generic specialisation will still give us the problem of creating a dynamic Shape type.
But maybe if we combine it with visitor, it helps.
No. no no 
we want to:
- Avoid strange cryptic notations
- Easy and convenient and intuitive code
- Extensible to support new operations and new shapes (types)
questions:
- How should we be storing operations? (e.g. draw code for different types)
- How should we write a method which given a string, returns some type?
We have to unify.
So the create method, will not return some type. It will return a lambda which is all the same regardless of the type: given a list of handlers (q1) it will find the correct one and execute.
Now about question 1:
Maybe genrics is not the solution, if we don't want to have strong static compile time type (which we cannot have because that string can be anything and we don't want to introduce compile time union).
Solution: `any` type -> `ptr`
```
HandlerList = {t: type, handler: (ptr->), next: nothing|HandlerList}

Circle = {...}
Square = {...}

drawCircle = (x: Circle -> int) {...}
drawSquare = (x: Square -> int) {...}

#Define a linked list of handlers for different types.
shape_handlers = {t: Circle, handler: drawCircle, next: nothing}
shape_handlers = {t: Square, handler: drawSquare, next: shape_handlers}

getShape = (string: name -> (HandlerList->)) {
	if name is "Circle" 
		c = Circle{...}
		lambda = (x: HandlerList -> if x.t == Circle then run x.handler(c) else return lambda(x.next))
		return lambda
	}
	if name is "Square" ... 
}

myShapeProcessor = getShape("Circle")
myShapeProcessor(shape_handlers)
```
How can we make this better?
- Attach a secondary type to the function so anyone wants to call checks that.
Idea: Define a protocol (a function with `any` input), and specialize/extend it for different types
no.
But if we want to use `any` why so much complexity? just return ptr and do the iteration outside: No. Because we don't know the original type.
Clojure's protocols is like this but in handlers, there is a special type `this`.
idea: store handlers as ptr and when needed cast them to appropriate type. Advantage: No need to type checking inside handler.
```
HandlerList = {t: type, handler: ptr, next: nothing|HandlerList}

Circle = {...}
Square = {...}

drawCircle = (x: Circle -> int) {...}
drawSquare = (x: Square -> int) {...}

#Define a linked list of handlers for different types.
shape_handlers = {t: Circle, handler: ptr(drawCircle), next: nothing}
shape_handlers = {t: Square, handler: ptr(drawSquare), next: shape_handlers}

getShape = (T: type, string: name -> (HandlerList->)) {
	if name is "Circle" 
		c = Circle{...}
		lambda = (x: HandlerList -> if x.t == Circle then run cast(x.handler as T)(c) else return lambda(x.next))
		return lambda
	}
	if name is "Square" ... 
}

myShapeProcessor = getShape("Circle")
myShapeProcessor(shape_handlers)
```
The dirty part is in lambda. So maybe we can put it in a core function: given a LL and type, returns a lambda to invoke appropriate function + two lambdas to get next and get handler.
`invoke = (T: type, I: type, O: type, list: T, getNext: (T->T), getType: (T->type), getHandler: (T->(I->O))`
`shape_handler` is a table (list of rows). Same as vtable in c++.
**Proposal**:
1. Add to pattern section above code to explain how polymorphism is done
2. Use `&` to cast anything to ptr.
3. Add a function to core to de-reference a ptr
4. No `_` in generics.
5. No compile time union. All unions are fixed without any change.

N - If we follow `$x` notation, can we have optional args?
```
process = (x: int, y:int|nothing)
process = (x:int, y:nothing) -> process(x, 100)
process = (x:int, y:int) -> ...
```

N - `type{}` for types that must be struct.
So when we have `... (InType: type, args: InType -> invokePtr(OutType, func_ptr, *{c, *args}))` we are sure that `InType` is a struct.

N - Shall we allow using union instead of `type` keyword? But no further syntax.
Only union types are allowed. That's the only constraint.
If we stop union extension, then it will be of no use. (?)
No. Because then we cannot emphasis type relationship.
`process = (T: type, x: T, y: T ->` says type of x and y must be the same. but
`process = (x: Shape, y: Shape ->` does not say that.

N - With generics, how can we define a lambda? Because a generic function's signature relies on name of the first argument.
And in lambda or function type, we just ignore names.
`process = (T: type, g: T -> string) ...`
`view = (T: type, processHandler(type, `
Maybe we should provide type when creating a lambda.
q: What is type of `process` above?
Maybe we can use `$1` and `$2`... to refer to generic arguments.
This is same in Java and there, you must specify types.
```
process = (T: type, x: T -> string) ...

myLambda = process(int, _)
```
I think we cannot have generic lambdas. Because lambda is a runtime concept and generic is compile time.
If we have a function which accepts a generic lambda, it can be called with different functions. 
So we don't know what to call if we have `myLambda(int, 10)`.

N - Make sure LL way for polymorphism is powerful and re-usable for other cases, types, multiple-types, ...

N - `t: Circle` is t of type Circle or it's value is Circle type?
Maybe we should use `=` for values. To make it explicitly different.

N - Review examples section

N - If it turns out we don't need dynamic union, can we put more effort on pattern matching?

N - Proposal for polymorphism
```
HandlerList = {t: type, handler: ptr, next: nothing|HandlerList}

Circle = {...}
Square = {...}

drawCircle = (x: Circle -> int) {...}
drawSquare = (x: Square -> int) {...}

#Define a linked list of handlers for different types.
shape_handlers = {t: Circle, handler: ptr(drawCircle), next: nothing}
shape_handlers = {t: Square, handler: ptr(drawSquare), next: shape_handlers}

getShape = (T: type, string: name -> (HandlerList->)) {
	if name is "Circle" 
		c = Circle{...}
		lambda = (x: HandlerList -> if x.t == Circle then run cast(x.handler as T)(c) else return lambda(x.next))
		return lambda
	}
	if name is "Square" ... 
}

myShapeProcessor = getShape("Circle")
myShapeProcessor(shape_handlers)
```
The dirty part is in lambda. So maybe we can put it in a core function: given a LL and type, returns a lambda to invoke appropriate function + two lambdas to get next and get handler.
`invoke = (T: type, I: type, O: type, list: T, getNext: (T->T), getType: (T->type), getHandler: (T->(I->O))`
`shape_handler` is a table (list of rows). Same as vtable in c++.
**Proposal**:
1. Add to pattern section above code to explain how polymorphism is done
2. Use `&` to cast anything to ptr.
3. Add a function to core to de-reference a ptr
4. No `_` in generics.
5. No compile time union. All unions are fixed without any change.
```
VTable = {t: type, handler: ptr, next: nothing|VTable}

Circle = {...}
Square = {...}

getShape = (string: name -> (VTable->(InType: type, OutType: type, args: InType -> OutType))) {
	if name is "Circle" {
		c = Circle{...}
		:: (x: VTable -> (InType: type, OutType: type, args: InType -> OutType))
		{
			func_ptr = findEntry(x, Circle);
			:: (InType: type, OutType: type, args: InType -> invokePtr(OutType, func_ptr, *{c, *args}))
		}
	}
	if name is "Square" ... 
}

drawCircle = (x: Circle, g: Canvas, scale: float -> int) {...}
drawSquare = (x: Square, g: Canvas, scale: float -> int) {...}

#Define a linked list of handlers for different types.
draw_handlers = VTable{t: Circle, handler: &drawCircle, next: nothing}
draw_handlers = VTable{t: Square, handler: &drawSquare, next: draw_handlers}

my_canvas = createCanvas()
int_result = getShape("Circle")(draw_handlers)({Canvas, float}, int, {canvas, 1.19})
```
ptr is like `void*`. We don't store it's internal type. 
So `T: type` means T is a full type we can use.
`T: type[type]` means T is a generic type and needs another full type to be useful. Otherwise it will be like `Stack` generic type.
Or we can use `type[?]` to indicate that.
By the second we introduce constraints, we should also have it for `?`, which makes things even worse.
Also we need a means to cast back ptr.
**Proposal**:
1. Add to pattern section above code to explain how polymorphism is done
2. Use `&` to cast anything to ptr.
3. Add a function to core to de-reference a ptr: `ref(int, my_ptr)`
4. No `_` in generics.
5. No compile time union. All unions are fixed without any change.
6. Add a core function `invokePtr` to invoke a ptr which has a function inside.
For de-reference, it is different from casting. In casting, we change type of some data we have.
But here we want to get a typed binding pointing to where this ptr points at.
`*x`? No we have to specify type.
How can we add extra arguments?
in OOP, this vtable is attached to the instance/object, but here it is separated.
idea: Add a function to core to invoke a `ptr` with a struct as it's parameters and specific output type: `invokePtr`
Problem is we want to solve expression problem, don't add anything fancy and still support polymorphism.
```
Circle = {...}
Square = {...}

drawCircle = (x: Circle, g: Canvas, scale: float -> int) {...}
drawSquare = (x: Square, g: Canvas, scale: float -> int) {...}

VTableRow = {t: type, handler: ptr, next: nothing|VTableRow}
draw_vtable = {
	handlers: draw_handlers, 
	#we may add many different v functions (paint, print, save, ...) but for each we will have this helper func
	draw = 

#Define a linked list of handlers for different types.
draw_handlers = VTableRow{t: Circle, handler: &drawCircle, next: nothing}
draw_handlers = VTableRow{t: Square, handler: &drawSquare, next: draw_handlers}

getShape = (string: name -> (VTable->(InType: type, OutType: type, args: InType -> OutType))) {
	if name is "Circle" {
		c = Circle{...}
		:: (x: VTable -> (InType: type, OutType: type, args: InType -> OutType))
		{
			func_ptr = findEntry(x, Circle);
			:: (InType: type, OutType: type, args: InType -> invokePtr(OutType, func_ptr, *{c, *args}))
		}
	}
	if name is "Square" ... 
}



my_canvas = createCanvas()
int_result = getShape("Circle")(draw_handlers)({Canvas, float}, int, {canvas, 1.19})
```
What if we keep dynamic union and enable specialisation for generic functions?
```
Shape = Circle | Square
Shape = Shape | Triangle

draw = (Shape, Canvas, float -> int) #no body, means this is an abstract function

draw = (c: Circle, s: Canvas, f: float -> int ) ...
draw = (e: Square, s: Canvas, f: float -> int ) ...

#adding new function
print = (Shape, Canvas, float -> string) 
paint = (c: Circle, g: Canvas, f: float -> string ) ...

#adding new type
Shape = Shape | Triangle

getShape = (name: String -> Shape) {
	if name is "Circle" return Circle{...}
	if "Square" ...
}

my_shape = getShape("Circle")
int_var = draw(my_shape, my_canvas, 1.2)
```
You can assign a lambda to `draw` as an abstract function using: `myLambda = draw(_: Shape, _, _)`
or to sub-funcs: `myLambda = draw(_:Circle, _,_)`
What if there are multiple generics?
`draw = (Shape, Color, Canvas -> int)`
and color, canvas are both generics: `Color = Red | Green | Blue`, `Canvas = Paper | Screen | ...`
now, can I write a draw for Circle, but generic color? what will be the point?
Shall we say: Any function that has at least one union arg, must be abstract? So if it is not abstract, it must have concrete values for all args.
This makes sense and can decrease complexity.
This is just like Haskell, where when you write a function on a sum type, you use pattern matching to specify behavior for each type
```
isZero :: Int -> Bool
isZero 0 = True
isZero _ = False

toBeOrNotToBe :: Bool -> String
toBeOrNotToBe True  = "Be"
toBeOrNotToBe False = "Not to be"
```
Maybe we should follow the same. But only for unions.
So for the first definition: `Shape = Circle | Circle` which forces shape to be a union (if we have only one type).
- One may add their type to Shape but not sure what functions they need to implement. That is fine, they can use compiler or IDE to find out.
Let's don't make things more complciated because of this.
```
Shape = Circle | Square

draw = (Shape, Canvas, float -> int) #no body, means this is an abstract function, if it was Draw then it would be a type

draw = (c: Circle, s: Canvas, f: float -> int ) ...
draw = (e: Square, s: Canvas, f: float -> int ) ...

#adding new function
print = (Shape, Canvas, float -> string) 
paint = (c: Circle, g: Canvas, f: float -> string ) ...
paint = (c: Square, g: Canvas, f: float -> string ) ...

#adding new type
Shape = Shape | Triangle
draw = (x: Triangle, s: Canvas, f: float -> int) ...
paint = ...

getShape = (name: String -> Shape) {
	if name is "Circle" return Circle{...}
	if "Square" ...
}

my_shape = getShape("Circle")
int_var = draw(my_shape, my_canvas, 1.2)
```
This makes sense, but how can we extend `getShape`? It we add a new type like Triangle? Can we make this one generic too?
```
getShape = (name: String -> Shape) #abstract

getShape = (name: "Circle" -> Circle)???
```
If an abstract fuction has generic inputs, it's fine. we can easily decide for implementation when someones calls them.
But what if output is generic? Making decision based on input value is very simplistic. We can have other criteria for this.
We can say, any function with a union inpu must be abstract.
And no two abstract functions can have the same name,
Can we make getShape extendable?
Like a dispatcher function that calls other functions, this can be simply done via a map.
But that map will not be extendable. We can always use the LL method, same as what we were doing for polymorphism.
```
Creator = {name: string, creator: (string->Shape), next: nothing|Creator}
shape_creators = Creator{"Circle", createCircle, nothing}
shape_creators = Creator{"Square", createSquare, shape_creators}
```
This makes sense. We don't need to do anything for generic types. 
This is only about unions.
How does this work with generics?
```
process = (T: type, data: T, s: Shape -> int) #abstract and generic
process = (T: type, data: T, s: Circle -> int) ...
process = (T: type, data: T, s: Square -> int) ...
```
If we use unions for generics, will this be simpler?
```
Shape = Circle | Square

draw = (T: Shape, item: T, Canvas, float -> int) #no body, means this is an abstract function, if it was Draw then it would be a type

draw = (T: Circle, item: Circle, s: Canvas, f: float -> int ) ...
draw = (T: Square, e: Square, s: Canvas, f: float -> int ) ...

#adding new function
print = (Shape, Canvas, float -> string) 
paint = (c: Circle, g: Canvas, f: float -> string ) ...
paint = (c: Square, g: Canvas, f: float -> string ) ...

#adding new type
Shape = Shape | Triangle
draw = (x: Triangle, s: Canvas, f: float -> int) ...
paint = ...

getShape = (name: String -> Shape) {
	if name is "Circle" return Circle{...}
	if "Square" ...
}

my_shape = getShape("Circle")
int_var = draw(my_shape, my_canvas, 1.2)
```
The rule of generics is that you have to call it with concrete type. Now, how can i call a generic draw with a concrete type when I only have a Shape?
No. Let's keep these separated.
**Proposal**
1. Keep dynamic compile time union
2. Define abstract function: Any function that has no body
3. For any abstract function, you can define implementations with the same name but non-union types.
4. If someone calls the abstract function, it will be redirected to appropriate function with correct type.
5. You cannot have more than one abstract function with the same name.
6. Fix patterns section
7. You cannot address implementations of an abstract function via lambda. You can only point to the abstract function and your call will be redirected.
What happens if I call impl function with `Circle|Triangle`?
In impl section, you can impl for a union, as long as there is no overlap.
When calling, if we have a function for static type, it will be called. else the one for it's dynamic type will be called.
What if we have multiple arguments?
`process = (x:Shape, c: Color -> int)`
if I call process with Circle and `Red|Blue`, and in impl I have `Circle|Square, Red`
and `Circle|Square, Blue` what will happen?
oh too much complexity. Let's just say no union type in impl functions.
They must be concrete types. And if you want same code for multiple types, repeat code or call another function.
But impl functions must all have concrete types, not union types.
And when calling? You can call with anything. If it is a union type, dynamic type will be used to dispatch. else static type.
If we have multiple unions, dynamic type of all of them will be used to match.
**Proposal**
1. Keep dynamic compile time union
2. abstract function: A function with union args and no body. You cannot have one of these two.
3. For any abstract function, you can define implementations with the same name but non-union types. impl function must not have union inputs.
4. If someone calls the abstract function, it will be redirected to appropriate function with correct type.
5. You cannot have more than one abstract function with the same name.
6. Fix patterns section
7. You cannot address implementations of an abstract function via lambda. You can only point to the abstract function and your call will be redirected.
8. If you call a function with union type, it's dynamic runtime type will be used to dispatch. Otherwise, static type.

N - `type{}` for types that must be struct.
So when we have `... (InType: type, args: InType -> invokePtr(OutType, func_ptr, *{c, *args}))` we are sure that `InType` is a struct.

N - Shall we allow using union instead of `type` keyword? But no further syntax.
Only union types are allowed. That's the only constraint.
If we stop union extension, then it will be of no use. (?)
No. Because then we cannot emphasis type relationship.
`process = (T: type, x: T, y: T ->` says type of x and y must be the same. but
`process = (x: Shape, y: Shape ->` does not say that.

N - With generics, how can we define a lambda? Because a generic function's signature relies on name of the first argument.
And in lambda or function type, we just ignore names.
`process = (T: type, g: T -> string) ...`
`view = (T: type, processHandler(type, `
Maybe we should provide type when creating a lambda.
q: What is type of `process` above?
Maybe we can use `$1` and `$2`... to refer to generic arguments.
This is same in Java and there, you must specify types.
```
process = (T: type, x: T -> string) ...

myLambda = process(int, _)
```
I think we cannot have generic lambdas. Because lambda is a runtime concept and generic is compile time.
If we have a function which accepts a generic lambda, it can be called with different functions. 
So we don't know what to call if we have `myLambda(int, 10)`.

N - Make sure LL way for polymorphism is powerful and re-usable for other cases, types, multiple-types, ...

N - `t: Circle` is t of type Circle or it's value is Circle type?
Maybe we should use `=` for values. To make it explicitly different.

N - Review examples section

N - If it turns out we don't need dynamic union, can we put more effort on pattern matching?

N - (moved to next item) q: How can we use this to implement hashCode function?
```
getHashCode = (data: ?  -> string)
```
or toString?
```
toString = (data: ?  -> string)
```
This is really really similar to generics. Maybe we can replace generics with this?
Suppose that we have a special union called `any` which is union of everything.
```
getHashCode = (data: any -> string)
getHashCode = (data: Customer -> string ) ...
getHashCode = (data: Circle -> string) ...
getHashCode = (data: Square -> string) ...

Stack = Seq[any]
push = (s: Stack, x: any)
push = (s: Stack, x: int) ...
push = (s: Stack, x: string) ...
```
What is the difference? getHashCode has to be different for each type.
But push doesn't need to.
So maybe we can just implement abstract function. But then we won't have type safety.
We can have data of different types on a Stack.
And what about when output type is union? How can we keep type safety?
`receive = (T: type, predicate: (T->bool) -> T)`
`receive = (predicate: (any->bool) -> any)`
And when we need a predicate which accepts any, we can give any implementation for that? `int->bool` for example?
But without proper generic we won't have sequence and map.
```
Map = [K: type, V: type -> 
{ 
	ref: ptr, 
	get = (index: K -> coreRead(K, V, ptr, index),
	create = (data: {K,V}... -> Map[T])
	{
		result = coreAlloc(T, data)
	}
}]
```
what if we keep generic data types but remove generic functions?
```
Stack = [T: type -> Seq[T]]
push = (s: Stack[any], x: any) #abstract
push = (s: Stack[$1], x: $1) ...
```
Generic data structures are much more limited.
**Proposal**:
1. Drop generic functions
2. Add `any`
3. Add `$x` notation to write implementations for an abstract functions.
Again, you cannot address impl function. only abstract one.
`$x` notation means compiler will generate a new code for each call/type.
q: How will this affect seq, map, receive, send, cache?
cache is just and receive.
```
send = (x: wid, data: any)...
send = (x: wid, data: $1) ... #using $ notation means type cannot be any union type
receive = (predicate: (any->bool) -> any)
receive = (predicate: ($1->bool) -> $1)
```
So the rule is, no actual function (a function with body, or impl function), can have union input type.
If I call the function with `int`, the code for `int` will be generated and called.
```
Seq = [T: type -> {
	Type: type = T, 
	ref: ptr, 
	length = (->coreLen(ref)),
	concat = (target: Seq[T] -> out: Seq[T])
	{
		data = alloc(len(ptr)+len(target.ptr))
		...
	},
	create = (data: T... -> Seq[T])
	{
		result = coreAlloc(T, data)
	},
	get = (index: int -> coreGet(ref, T, index))
}
]
#using a sequence
x = Seq[int].create(1,2,3,4)
len = x.length()
new = x.concat(my_int_sequence)
```
Putting functions inside struct will cause confusion. Because fn cannot be generic but a fn inside a generic type, is generic by nature.
```
Seq = [T: type -> {ref: ptr}]

createSequence = (data: any... -> Seq[any])
createSequence = (data: $1... -> Seq[$1]) ...
```
So: Proposal: Do not allow setting values in struct type.
What if I really want to call a function with a union?
e.g. create a sequence of int or string?
```
getHashCode = (data: any -> string)
getHashCode = (data: Customer -> string ) ...
getHashCode = (data: Circle -> string) ...
getHashCode = (data: Square -> string) ...
```
Does it make sense to write getHashCode for int_or_string type? I think it might make sense and as long as we only have one abs function it should be ok.
**Proposal**:
1. Drop generic functions
2. Add `any`
3. Add `$x` notation to write implementations for an abstract functions.
4. No value in struct types
How can we have a sequence of `int_or_string`? Or call function with union?
Union cannot be at the arg level.
e.g. get: `get = (Seq[any], index: int -> any`
what if seq is int or string?
`get = (Seq[$1], index: int -> $1)`
And without generics:
`get = (Seq[int], index: int -> int)`
`get = (Seq[string], index: int -> string)`
We can write:
`get = (Seq[int|string], index: int -> int|string)`
Note that function input is not union. Internal type of an argument is.
You cannot write a body for abstract function.
This concept helps us implement polymorphism.
Can `$x` notation be mixed?
`process = (data: $1|$2 -> $1|$2)`
- The fact is, we don't have abstract functions at all. Just write impl functions normally, and compiler will handle everything.
```
getHashCode = (data: Customer -> string ) ...
getHashCode = (data: Circle -> string) ...
getHashCode = (data: Square -> string) ...

send = (x: wid, data: any)...
send = (x: wid, data: ?) ... 
reverMap = (in: Map[K,V] -> Map[V,K])
```
Generic functions are like polymorphic functions but with inclusion of all types and same implementation.
```
Add[T] = (a: T, b: T -> T) 
{
	...
}
Add[int](10, 19) 
#or
add[?](10,19)
```
Can we unify these? Same notation for both?
```
my_shape = createShape("Circle")
draw[?](my_shape) #means invoke the draw version for ? type, which means you calculate at runtime.
getHashCode[T] = (data: T -> string)
getHashCode = (data: Customer -> string ) ...
getHashCode = (data: Circle -> string) ...
getHashCode = (data: Square -> string) ...
```
So, you can either write an implementation for a generic function (normal generics, like add numbers), or make it abstract.
If you make it abstract, you can write other functions with the same name but appropriate input types.
When calling either of these you can use `func[int]` to direct to a specific code, or `func[?]` to ask compiler to decide what type to use.
And no union input argument in any function.
Huge change, but solves a lot of issues about generics, seq and map, polymorphism, ...
How does it affect lambdas? can I define a generic lambda?
What will be the name of this type of functions? Apparently, we are unifying generics and polymorphism.
How can I have a function pointer to one of impl functions?
So we have a group of functions under the same name. these functions can be the same code (generics) or different codes (polymorphism).
`draw[T] = (obj: T -> int)`
`myDrawCircle = draw[Circle](_)`
`myFunc = draw(_)`. This does not make sense.
`myFunc = draw[?](_)`
`myFunc(my_circle)`
`myFunc(my_square)`
This again gives us a change to get rid of union type extension.
**Proposal**:
You can define a group of functions under the same name. If all group functions have the same code it is generics. If codes change it is polymorphism.
Function dispatch is based on union's runtime type.
Notation is `func[T] = (...)`
If it is generics, you provide body for this function. For polymorphism, you don't add a body here and write other functions for body:
Generics:
```
add[T] = (a: T, b: T -> T ) { :: a+b }
add[int](10,20)
```
Polymorphism:
```
draw[T] = (shape: T -> int)
draw[Circle] = (shape: Circle -> int) { ...}
draw[?](my_shape)
draw[Circle](my_circle)
draw[?](my_circle)
```
You can use `?` notation to use compiler to decide which function to call. This can be either via static or dynamic type.
`getShape = (name:string -> (`
What if output of a function is a group function?
`process[T] = (name:string -> [T](x:int->string))`
This is a change which makes inconsistent results. we should stick to normal naming.
For polymorphism, it is fine. We can easily drop the `[T]` notation and use a union.
```
#Generics:
add[T] = (a: T, b: T -> T ) { :: a+b }
add[int](10,20)

#Polymorphism:
draw = (shape: Shape -> int)
draw = (shape: Circle -> int) { ...}
draw(my_shape)
draw(my_circle)
```
But even for polymorphism, we may have some relations.
`compare = (s1: Shape, s2: Shape -> int)` s1 and s2 must have the same type.
Having abstract function helps user know he should add a draw function when adding a new type to Shape.
We can write generics as functions on any or a union.
The problem is relationship. So the general problem is: we want to have a relationship (checked at compile time), between function inputs and outputs.
This relationship can be equality (s1 and s2 must be same shape) or maybe another generic? `A: type, B: type, x: A[B]` but this is too complicated.
Let's stick to equality.
`process = (s: Shape, t: Shape -> Shape)` we want to say inputs have the same type and output will be the same.
We can say arguments with the same type name, should have the same type.
`draw = (s1: Shape, s2: Shape -> int)` s1 and s2 must have the same type
`Shape2 := Shape`
`draw = (s1: Shape, s2: Shape2 -> int)` s1 and s2 can have different types.
But how can I call draw with a Shape?
Suppose that I have two shapes of type Shape.
When calling `draw` I should write: `draw(my_circle, Shape2(my_square))`?
The point is `A := B` means A is just a label for data of type B. so you can easily convert in any direction.
But when we say `add = (x: int, y: int)` x and y must have the same type which makes sense.
`add = (a: any, b:any -> a+b)`
So will compiler generate code for add for each type we call it?
Let's say the rule is: Functions cannot have union input. If a function does, either developer or compiler should provide concrete implementations for basic types.
So `process(int|string)` is not allowed, unless you write two implementations for int and string.
How can we interpret this correctly and make it intuitive?
For polymorphism, no change is needed. Just write your functions and call them via a union type.
No new notation, no new syntax or interpretation.
For generics: We have two types of functions: normal functions with non-union input types, and generic functions with union input type.
For generic functions, the body of the function will be duplicated for all applicable types (compiler will optimize to only duplicate for types needed, but that is implementation dependant).
Generic functions have union inputs. And type names specify relationship between input and output types.
`process = (x: Shape -> y: Shape)` this means output will have the same type as input.
- Shall we use a special notation for generic functions?
e.g. their name should start with `%`? This can be helpful to make things more explicit and readable. Because these are super-functions.
- What about function with union output type? I think these are not generic.
Also note that we have `_` prefix for private functions.
`push%`, `push$`. Suffix is not beaufitul.
`$push`, `%push`, `$` is better because `%` is confusing with math operators.
`$push`, `[push]`. We have generic types that uses `[]`.
`push[]`, `pop[]` But empty brackets does not make sense. 
`push = (s: Stack[any], x: any -> Stack[any])`
Problem: This is not intuitive that three `any` types are generic types.
Let's just use current status: Type arguments
`push = (T: type, s: Stack[T], x: T -> Stack[T]) { ... }`
About lambda? `fp = push(_, _, _)`?
`fp(int, my_stack, data)` `fp = push(int, _, _)` `fp(my_stack, 10)`
ok. So our proposal only covers for polymorphism:
**Proposal**:
1. No `_` notation in generics
2. If you define multiple functions with the same name and number of arguments, the compiler will handle calling them based on dynamic type of unions.
3. No `_:Type` notation in lambda. You cannot discriminate a group of function with the same name. Just pass appropriate type and the corresponding function will be called.
4. Expression problem: Add new type using dynamic compile-time union, add new function using writing the function normally.

N - We say that argument name is not part of the type but for generics, it is.
You cannot send a generic lambda to another function as it is a compile time construct.
But you can remove the compile time part by specifying types for it.
`sort = (T: type, data: Seq[T] -> Seq[T])`
`sortAndProcess = (T: type, sorter: (T, Seq[T] -> Seq[T]) -> int`
or:
`process(Customer, customer_list, sort(Customer, _))`
`process = (T: type, list: Seq[T], sorter: (Seq[T]->Seq[T]))` You cannot have two levels of generics.
So: When assigning a generic function to a pointer, you must specify types for generic args.

Y - we need a type for wid.
so we can pass it to other functions.
`ref`?
I don't like acronyms like `pid` or `cid`. It is a keyword and should have a meaning.
So it is `thread`, `proc`, `codeid`, `ip`
`x := process()`
basically this is the mailbox. not the process itself.
but maybe later we want to add functions to get age of a process, get stats, ... so better to name it more general.
`pid`?
`actor`?
`process`?
`proc`
`thread`
**`task`**
`fibre`
This is done in 

Y - How can we implement complex logics?
```
if ( x ) return false
if ( y and z ) return false
if ( t and u ) return false
if ( y and r ) f=true
if ( f ) return false
return true
```
becomes:
```
out = ifElse(x, false, out2)?
```
Even if we add if/else keywords, it won't solve early return problem.
we can add this notation: `cond::retval` so if condition holds, we will have early return.
`::ret` to do normal return

N - Can we make `T: type` notation more elegant?
`process = (T: ?, ...`
to make this orth, we should allow it everywhere: even inside functions.

N - Again: How do we address a generic function using a lambda?
`serialise = (T: type, data: T -> string)`
If we apply above change, we won't have a generic function.
We don't address it directly. Either call function yourself or provide a type.
Because module level functions are compile-time but lambdas ar runtime (you can pass them around)
So: When assigning a generic function to a pointer, you must specify types for generic args.

N - Use case: How to implement a code which starts a helper thread to load some required data at the beginning
and later asks for that data?
```
task := process()
...
waitFor(task, FIN)
#or use core function, 
waitForFinish(task)
```

N - If we assume polymorphic functions using same name, can it replace generics?
we still have generic data types.
e.g. find something in a linked list.
`find = (ll: LinkedList[T], data: T)...`
`find = (ll: LinkedList[any], data: any)...`
We can use `any` but it won't capture relations.
Maybe we can have relationship with some compile time restrictions.
```
find = (ll: LinkedList[any], data: any -> boolean) type(ll) == type(data)
{
	...
}
reverseMap = (m1: Map[any, any] -> Map[any, any]) {
	...
}
```
Idea: we can tag union types to specify their relationship.
`any$1` tags type with `$1`.
```
find = (ll: LinkedList[any$1], data: any$1 -> boolean)
{
	...
}
reverseMap = (m1: Map[any$1, any$2] -> Map[any$2, any$1]) {
	...
}
```
But what if we really want a union here? `Map[int|string, float]`
What if I call draw with`Circle|Square`? It works on only one. So depending on the runtime type, one draw method will be called.
But in abstract data types like map, we sometimes need to do something e.g. search or sort, on a sequence of unions.
In other words, `any` type arguments will have only one type/
But `any` generic arguments for generic types, can be anything.
This is confusing. because we are re-using the union concept.
```
find = (T: type, ll: LinkedList[T], data: T -> boolean)
{
	...
}
reverseMap = (K: type, V: type, m1: Map[K, V] -> Map[V, K]) {
	...
}
```
Does the union issue exist for polymorphic functions?
e.g. we have a method called serialise which gets a sequence of shapes.
`save = (x: Seq[Shape]...)`
But, `Seq[Shape]` is not a union so it is fine. You can put any compliant binding inside `Seq[Shape]` and it will be fine to use it for save.
we didn't write: `Seq[Shape] | Seq[Drawable]`.
If an argument is of union type then polymorphism says, write separate functions and runtime will find appropriate impl for dynamic type.
So, back to generics, can we use this to solve initial problem?
```
find = (ll: LinkedList[any$1], data: any$1 -> boolean)
{
	...
}
reverseMap = (m1: Map[any$1, any$2] -> Map[any$2, any$1]) {
	...
}
```
any means the type is not important for us. if it is always coming with `$` can we eliminate it?
```
find = (ll: LinkedList[$1], data: $1 -> boolean)
{
	...
}
map = (arr: Seq[$1], pred: ($1->$2) -> Seq[$2]) {
	...
}
groupBy = (arr: Seq[$1], group: ($1->$2) -> Map[$2, Seq[$1]]) {
	...
}
```
Can we make this more limited? For 90% of the cases, only two generic type arguments are enough? `$` and `&`.
But we have a rule: zero, one or unlimited.
Can we do this via one type only? Maybe divide complex functions into multiple functions each with one type?
This notation is more limited that `T: type` notation because you cannot mix types (e.g. `$1[$2]`).
But we cannot re-assign a binding. We have many bindings with the same name `find`.
This is ok for polymorphism because we create a master union type. Can we create a master function for this case too?
We don't need to. Because we don't have a master union type.
```
for all types T:
get = (s: Seq[T], index: int -> T) {
	...
}
```
```
Stack = [T: type -> Seq[T]]
push = (T: type, data: T, stack: Stack[T] -> Stack[T]) { ... }
#alternative
StackProtocol = [T: type -> { push: (T -> Stack[T]), peek: (->T)}]
createStack = () {
	s = Stack[int]...
	return StackProtocol[int]{push = (int -> Stack[int]) ..., peek = (->int) ...}
	
}
```

N - For error handling we can use polymorphism concepts:
```
ErrorItem = A | B
ErrorItem = ErrorItem | MyFileError
Error = {errorItem: ErrorItem, cause: Nothing|Error}
process = (x:int -> nothing|Error)`
```
This way we can have nested and extensible errors.

N - We can use `Shape = Circle |` notation to have a union which for now has only one option.

N - (moved to next item) Polymorphism and generic:
**Proposal**:
1. No `_` notation in generics
2. If you define multiple functions with the same name and number of arguments, the compiler will handle calling them based on dynamic type of unions.
3. No `_:Type` notation in lambda. You cannot discriminate a group of function with the same name. Just pass appropriate type and the corresponding function will be called.
4. Expression problem: Add new type using dynamic compile-time union, add new function using writing the function normally.
Another idea: we have multiple methods with the same name but cannot call them with a shape.
But write a master method which accepts a shape and redirects. This way the process is less automated
```
draw = (c: Circle ...
draw = (s: Square..
masterDraw = (x: Shape ... ) {
	draw(unwrap(x), ...)
}
```
This will make less hidden parts and give the developer more control.
We can use `*` for unions to mean unwrap.
We want to make this process is flexible as we can but still have extensibility so people can add new types and functions.
```
draw = (c: Circle -> int) {...}
draw = (s: Square -> int) {...}
masterDraw = (x: Shape ... ) {
	draw(*x, ...)
}
```
We can even assign `*x` to a binding. It's type will be one of shape types but we don't know at compile time.
What is the difference between type of `my_shape` and `*my_shape`?
idea: Just like compile time dynamic union, we can have dynamic functions. These functions are all part of a single function.
```
Shape = Shape | Circle #here we extend shape function
drawCircle = (x: Circle -> int) { ... }
draw = draw | drawCircle
```
What if we don't have multiple functions with the same name? So lambda problem will be solved.
But how can I call a function without knowing its name? and without knowing it's exact type?
Idea: When adding a new type to a union, we specify it's identifier which is used to create it's polymorphic functions
```
Shape = Shape | Circle {circle}
drawCircle = (x: Circle -> int) ...
draw(my_shape)
```
```
#all shapes have id
Circle = {... id: "circle"}
Shape = Shape | Circle
drawCircle = (x: Circle -> int) ...
draw#my_shape.id(my_shape)
```
String concat is not very elegant.
what if we say we don't have polymorphism?
what happens?
Rather than adding some new concept (multiple functions with the same name), let's provide essentials to implement a vtable.
```
Circle = {...
	vtable: { draw = drawCircle }
}

Shape = Shape | Circle

process = (s: Shape -> s.vtable.draw)
```
But type of vtable is not the same across all shapes.
Another idea: All shapes keep track of their parent.
```
Circle = { s: Shape, ... }
```
Nope.
Another idea: traits, the function to read a shape, does not return a shape, returns a drawable.
```
ShapeTrait = { draw: (int->string), save: ... }
Circle = { ... draw = ..., save: ... }
getShape = (f: File -> ShapeTrait) {
	if type == "Circle":
		return ShapeTrait(Circle{...})
}
s = getShape(...)
s.draw(...)
s.save(...)
```
So ShapeTrait contains the common functionality of all shapes.
Pro: 
- No need to vtable 
- No need to dynamic union
- Is intuitive
- No new syntax.
- The only change: casting a struct to another struct
Con:
- How can I add a new type? Just implement it with appropriate functions so it can be casted to ShapeTrait
- How can I add a new operation?
we need to modify the shape trait.
`ShapeTrait = {*ShapeTrait, export: (...)}`
and we need to add export to all shape structs.
nope.
defining functions inside structs is nice but will be a huge problem when solving expression problem.
decl should be separate or else they won't be extensible: type decl and function decl
let's say we use trait method but using a notation to find functions of a specific type. This is not good because type is not everything.
How is polymorphism implemented internally? It is using vtable. So let's provide features that can be used to add a vtable.
But vtable is specific to one type. It is attached to an object in OOP.
```
my_shape.draw() #?
draw(my_shape) #?
```
I think the second version is better. To more separated things are, the easier to extend them.
```
draw_vtable.apply(my_shape)
```
What if we allow compile time sequences which can be modified at compile time?
Either we have to use `ptr` or very complex union types or unknown generics.
I think the third option is better.
```
getShape(my_file)(vtable)(name, age, input3)
```
The ideal solution: No change to union types, no functions with the same name
```
Circle = {...}
Square = {...}

drawCircle = (x: Circle, g: Canvas, scale: float -> int) {...}
drawSquare = (x: Square, g: Canvas, scale: float -> int) {...}

VTable = {t: type, handler: ptr, next: nothing|VTable}
DrawFunc = (Canvas, float -> int)

#Define a linked list of handlers for different types.
draw_handlers = VTable{t: Circle, handler: &drawCircle, next: nothing}
draw_handlers = VTable{t: Square, handler: &drawSquare, next: draw_handlers}

getShape = (string: name -> (VTable, FType: type ->FType)) {
	if name is "Circle" {
		c = Circle{...}
		:: (x: VTable, FType: type -> FType)
		{
			func_ptr = findEntry(x, Circle);
			:: coreInjectArg(FType, func_ptr, c)
		}
	}
	if name is "Square" ... 
}

my_canvas = createCanvas()
int_result = getShape(DrawFunc, "Circle")(draw_handlers)(my_canvas, 1.19) #we can keep drawfunc and drawfuncraw types inside vtable
```
I think we should have a notation that says, calling a function with insufficient args will create a lambda for remaining args.
This way, we can return a function with some number of args in the code without caring for each individual arg.
Generics: We have same function for all types
Polymorphism: We have different functions for each type
Idea: We can assign a linked list to a function. compiler will lookup appropriate element to call
```
draw = [drawCircle, drawSquare]
draw = draw & [drawTriangle]
```
But still we will have a problem about keeping track of a shape. How can we do that?
```
Circle = {...}
Square = {...}

drawCircle = (x: Circle, g: Canvas, scale: float -> int) {...}
drawSquare = (x: Square, g: Canvas, scale: float -> int) {...}

VTableRow = {t: type, handler: ptr, next: nothing|VTableRow}

#Define a linked list of handlers for different types.
draw_handlers = VTableRow{t: Circle, handler: &drawCircle, next: nothing}
draw_handlers = VTableRow{t: Square, handler: &drawSquare, next: draw_handlers}

VTable = [T: type -> { FType: T, rows: VTableRow}]
draw_vtable = VTable[(Canvas, float -> int)]{ rows: draw_handlers }

getShape = (string: name -> VFunc) {
	if name is "Circle" {
		c = Circle{...}
		:: stdCreateVFunc(Circle, c)
		:: (x: VTable -> VTable.FType)
		{
			func_ptr = findEntry(x, Circle);
			:: coreInjectArg(x.FType, func_ptr, c)
		}
	}
	if name is "Square" ... 
}

my_canvas = createCanvas()
int_result = getShape("Circle")(draw_handlers)(my_canvas, 1.19) #we can keep drawfunc and drawfuncraw types inside vtable
```
This works but we are relying too much on the developer. Mostly about `FType` being a correct type.
Can we embed FType inside vtable?
Why can't we use a generic type? Because we will still have the problem of storing output of getShape.
Another strategy: inclusion types: we can define a type which means "some type which includes a field with this name and type"
```
getShape = (name: string -> {Shape...})
```
No. too much change.
The good thing about current approach is that we don't need dynamic union.
But the con is that it's a bit far from static type (we store ptr and let developer pass us any type).
Now that we want to ask for compiler's help, why not store everything in one place? Something like a sequence but different?
But what will be it's type? We have to have a type for everything. We can store `ptr` in it. And compiler will handle type matching.
But how are we going to store output of getShape? That is also a question.
```
Circle = {...}
Square = {...}

drawCircle = (x: Circle, g: Canvas, scale: float -> int) {...}
drawSquare = (x: Square, g: Canvas, scale: float -> int) {...}

draw = [&drawCircle, &drawSquare]
```
This is too complex. 
**Proposal**:
1. No `_` notation in generics
2. If you define multiple functions with the same name and number of arguments, the compiler will handle calling them based on dynamic type of unions.
3. No `_:Type` notation in lambda. You cannot discriminate a group of function with the same name. Just pass appropriate type and the corresponding function will be called.
4. Expression problem: Add new type using dynamic compile-time union, add new function using writing the function normally.
Another idea: we have multiple methods with the same name but cannot call them with a shape.
But write a master method which accepts a shape and redirects. This way the process is less automated
```
draw = (c: Circle ...
draw = (s: Square..

draw(my_circle)
draw(my_square)
```
Simple and straight forward.
Only about lambda: When you say `x = draw(_)` if there is ambiguity, you should specify type.
if you want multi-method to be called: `x = (x: Shape -> draw(x))` but this does not work. we said function inputs cannot be union.
we may need sometimes to have a multimethod function pointer. a function that works on a shape.
"You cannot define a function with union input" is an exception which we don't want. Let's allow as much as possible.
Allow everythng and even for multimethod rely on developer defining it.
Of course there cannot be overlap between functions. This will be caught by the compiler.
so `(Shape)` and `(Circle)`? Is this overlap? Not really. If you call `func` with a circle it will call the second one.
If you call it with a Shape, it will call the first one.
In other words, the developer is responsible for redirection. Compiler and runtime will dispatch function calls based on static type (Shape).
How can I do the redirection?
```
xdraw = (x: Circle -> ...)
xdraw = (x: Square -> ...)
...
draw = (T: type, s: Shape -> xdraw(T(s)))
...
my_shape = getShape("Circle")
draw(type(my_shape), my_shape)
```
and `type` function only works on a union. in these cases, we generate all implementations for `draw` for all possible types of `my_shape`.
```
draw = (x: Circle -> ...)
draw = (x: Square -> ...)
...
my_shape = getShape("Circle")
draw(*my_shape)
hashCode = (x:int -> int)
hashCode = (x:string -> int)
hashCode = (x:Circle -> int)
hashCode = (x:Customer -> int)
sort = (T: type, data: Seq[T] -> ...) {
	f = hashCode(data.get(0))
}
```
Can this replace generic functions? Polymorphism is different code for different types. Generic is same code for different types.
```
Stack = [T: type -> ...]
push = (x: 

```
Generic also allows us to specify relationship between data.
If we call `sort` with `Seq[int|string]` it will call hashCode for `int|string` unless we use `*` but we don't know if T will be a union type.
Let's do this: functions can be defined with any type. call dispatch will be based on static type. 
```
draw = (x: Circle -> ...)
draw = (x: Square -> ...)
...
my_shape = getShape("Circle")
draw(my_shape)
xd = (x: Shape -> draw(x)) #dispatch based on static type?
xd(my_shape)
```
If we allow defining functions on union types, people can write a function for a large union and it will be diverging to generics.
If we don't, in some places it might make code more difficult to write.
Let's not allow defining functions with union type.
```
draw = (x: Circle -> ...)
draw = (x: Square -> ...)
...
my_shape = getShape("Circle")
draw(my_shape)
hashCode = (x:int -> int)
hashCode = (x:string -> int)
hashCode = (x:Circle -> int)
hashCode = (x:Customer -> int)
sort = (T: type, data: Seq[T] -> ...) {
	f = hashCode(data.get(0))
}
```
Here `hashCode` will be called with actual type of `data.get(0)`. It can be `int|string` in which case the underlying real type will be used.
**Proposal**:
1. No `_` notation in generics
2. If you define multiple functions with the same name and number of arguments, the compiler will handle calling them based on dynamic type of unions.
3. No function can have union argument.
4. No `_:Type` notation in lambda. You cannot discriminate a group of function with the same name. Just pass appropriate type and the corresponding function will be called.
5. Expression problem: Add new type using dynamic compile-time union, add new function using writing the function normally.
What about item 4? How are we going to have a lambda which covers all draw/hashCode functions?
What happens if I pass `hashCode(_)` to a function?
Maybe there should be a compiler warning: you should use `hashCode(_:int)` instead.
or, we can say we allow functions with union args as long as there is no conflict.
And compiler will decide which one to call.
```
draw = (x: Circle -> ...)
draw = (x: Square -> ...)
...
my_shape = getShape("Circle")
draw(my_shape) #no draw for Shape so use it's underlying type
hashCode = (x:int -> int)
hashCode = (x:string -> int)
hashCode = (x:Circle -> int)
hashCode = (x:Customer -> int)
sort = (T: type, data: Seq[T] -> ...) {
	f = hashCode(data.get(0))
}
```
But this will make things complicated in case of multiple union args. What if some have candidates but some don't.
Lets allow any type of functions and dispatch will be based on static type.
```
draw = (x: Circle -> ...)
draw = (x: Square -> ...)
...
xdraw = (s: Shape -> ...)
my_shape = getShape("Circle")
draw(my_shape) #not allowed
xdraw(my_shape) #allowed
hashCode = (x:int -> int)
hashCode = (x:string -> int)
hashCode = (x:Circle -> int)
hashCode = (x:Customer -> int)
superHashCode = (x: any -> ???)
sort = (T: type, data: Seq[T] -> ...) {
	f = hashCode(data.get(0)) #if T is a simple type this works fine. If it is int|string, we must have a hashCode for int|string.
}
myLambda = hashCode(_) #not allowed because it has ambiguity. You should specify the type of input
myLambda = hashCode(_:int)
```
Two questions:
1 - How do we redirect from xdraw for Shapes to draws for each shape?
2 - How can I have a lambda that points to all hashCode functions?
If we solve 1, we can solve 2.
`xdraw = (s: Shape -> draw(*s))`
`*x` will cast union binding x to it's underlying type. Obviously you cannot store it in a place where type is fixed because type can change.
Can we make this notation more elegant?
```
my_shape
Circle(my_shape).1::Circle(my_shape).0
Square(my_shape).1::Square(my_shape).0
...
```
We need this to have our important function call dispatch rule: dispatch only based on static type.
Currently we have two options:
1. Manual VTable implementation (std + some notations like &)
2. Multimethods (functions with same name with or without forwarding) + dynamic union
```
Circle = {...}
Square = {...}

drawCircle = (x: Circle, g: Canvas, scale: float -> int) {...}
drawSquare = (x: Square, g: Canvas, scale: float -> int) {...}

VTable = {t: type, handler: ptr, next: nothing|VTable}

#Define a linked list of handlers for different types.
draw_handlers = VTable{t: Circle, handler: &drawCircle, next: nothing}
draw_handlers = VTable{t: Square, handler: &drawSquare, next: draw_handlers}
DrawType = (Canvas, float -> int)

getShape = (string: name -> (type, VTable -> FType)) {
	if name is "Circle" {
		c = Circle{...}
		:: stdCreateVFunc(Circle, c)
		:: (FType: type, x: VTable -> FType)
		{
			func_ptr = findEntry(x, Circle);
			:: coreInjectArg(FType, func_ptr, c)
		}
	}
	if name is "Square" ... 
}

my_canvas = createCanvas()
int_result = getShape("Circle")(DrawType, draw_handlers)(my_canvas, 1.19)
```
This is not simple.
```
draw = (x: Circle -> ...)
draw = (x: Square -> ...)
...
xdraw = (s: Shape -> ...)
my_shape = getShape("Circle")
draw(my_shape) #not allowed
xdraw(my_shape) #allowed
hashCode = (x:int -> int)
hashCode = (x:string -> int)
hashCode = (x:Circle -> int)
hashCode = (x:Customer -> int)
superHashCode = (x: any -> ???)
sort = (T: type, data: Seq[T] -> ...) {
	f = hashCode(data.get(0)) #if T is a simple type this works fine. If it is int|string, we must have a hashCode for int|string.
}
myLambda = hashCode(_) #not allowed because it has ambiguity. You should specify the type of input
myLambda = hashCode(_:int)
```
What is the problem?
- We want to be able to solve expression problem.
- We want to have polymorphism.
What if we don't solve expression problem?
- (In terms of Shapes) We want to be able to add new shapes and new operations on them easily.
- We want to be able to define different implementations for different types.
I don't like extending types (`Shape = Shape | Square`) because it makes tracking type changes difficult.
We have two problems: on the operation side (how to add a new function for existing shapes), and on the type side (what is output of getShape).
Can we say: getShape return me something that I can feed into "draw" and we have multiple draw functions for different types.
```
draw = (s: Circle, Canvas, float -> int) {...}
draw = (s: Square, Canvas, float -> int) {...}
Drawable = draw(?,,)
...
getShape = (name: string -> Drawable) {
	if name is "Circle" return Circle{...}
	if name is "Square" ...
}

my_shape = getShape("Circle")
draw(my_shape, c1, 1.12)
```
A protocol is a type which is union of all types that can fill in it's blank.
Let's separate it from union. Because then we will need to change function call dispatch rules.
- How to have multiple functions in protocol?
- How this solves exp problem?
- What needs to change about function dispatch rules?
- What about generics?
```
draw = (s: Circle, Canvas, float -> int) {...}
draw = (s: Square, Canvas, float -> int) {...}
print = (s: Circle, int, int, float -> string) {..}
Shape = draw(?,Canvas,float) + print(?,int,int,float)
...
getShape = (name: string -> Shape) {
	if name is "Circle" return Circle{...} #this means we have draw(Circle, ...) defined in the code
	if name is "Square" ...
}

my_shape = getShape("Circle")
draw(my_shape, c1, 1.12)
print(my_shape, ...)

#Add a new shape
Triangle = {...}
draw = (Triangle ...)
print = (Triangle, ...)

#add a new op
getArea = (Circle ...)
getArea = (Square ...)
getArea = (Triangle ...)
#we also need to extend shape protocol
NewShape = Shape + getArea(?, int, float)
```
- Can we use protocols with generics?
Another way using closure:
```
draw = (s: Circle, Canvas, float -> int) {...}
draw = (s: Square, Canvas, float -> int) {...}

Drawable = {draw: (Canvas, float -> int)}
getShape = (name: String -> Drawable) {
	if name is "Circle" 
		c = Circle{...}
		return Drawable{draw = (n: Canvas, f: float -> draw(c, n, f))}
}
f = getShape("Circle")
f.draw(c, 1.12)
```
This needs absolutely no change and provides polymorphism.
Adding new shape: Add new draw and a new case in getShape
Adding new operation: Define a new struct e.g. Printable
```
draw = (s: Circle, Canvas, float -> int) {...}
draw = (s: Square, Canvas, float -> int) {...}

Draw = (Canvas, float -> int)
getShape = (name: String -> Draw) {
	if name is "Circle" 
		c = Circle{...}
		return (n: Canvas, f: float -> draw(c, n, f))
}
f = getShape("Circle")
f.draw(c, 1.12)
```
This basically simplified version of vtable approach. We can return multiple lambdas too, and enclose them in a struct.
```
draw = (s: Circle, Canvas, float -> int) {...}
draw = (s: Square, Canvas, float -> int) {...}

Shape = { draw: (Canvas, float -> int), paint: (Canvas, int -> string)}
getShape = (name: String -> Shape) {
	if name is "Circle" 
		c = Circle{...}
		return Shape{draw = (n: Canvas, f: float -> draw(c, n, f)), paint = (n: Canvas, x:int -> paint(c, n, x))}
}
f = getShape("Circle")
f.draw(c, 1.12)
```
Adding new shape: just implement appropriate functions
Adding a new operation: not so easily (unless we use more complex ways like VTable)
Suppose that we want to add area function:
```
draw = (s: Circle, Canvas, float -> int) {...}
draw = (s: Square, Canvas, float -> int) {...}
area = (s: Circle ->float)
area = (s: Square -> float)

Shape = { draw: (Canvas, float -> int), paint: (Canvas, int -> string)}
ShapeEx = { Shape, area: (->float) }

getShape = (name: String -> Shape) {...}

newGetShape = (name: string -> ShapeEx) {
	if name is "Circle" 
		c = Circle{...}
		return ShapeEx{area = (->area(c)), *getShape(name)}
}
f = getShape("Circle")
f.draw(c, 1.12)
```
- How can we reference to global `draw` in `return Shape{draw = (n: Canvas, f: float -> draw(c, n, f))`?
So it's not impossible to add a new operation. OTOH we have a simple language with no special syntax.
Everything is already there: closure, lambda, struct, ...
**Proposal**:
1. No `_` notation in generics. 
2. You can define multiple functions with the same name. Dispatch is based on static type.
3. Unions will not support dynamic extension.
4. You are free to have union args, but there should be no overlap.
5. When using function name to create lambda, if there are multiple candidates you have to use `_:int` notation to specify which one.
How can I return a lambda which can point to any draw? you can't. you can use `draw(_:Circle, _, _)` to create a lambda based on one function.
```
Hashable = {hashCode: (->int)}
```

N - Let developer use function calls to initialise module level bindings

N - Is it possible to have a generic logger function which accepts any function, logs something and calls the function?

N - how can someone implement BigInt? using a linked list of integers.

N - How can we initiate n threads and wait for all of them to finish? core func

N - anonym types
To keep the language small and uniform, all aggregate types in Zig are anonymous. To give a type a name, we assign it to a constant:
```
Node = struct {
    next: *Node,
    name: []u8,
};
```

N - Let developer use function calls to initialise module level bindings

N - Is it possible to have a generic logger function which accepts any function, logs something and calls the function?

N - how can someone implement BigInt? using a linked list of integers.

N - How can we initiate n threads and wait for all of them to finish? core func

Y - Poymorphism and exp problem
**Proposal**:
1. No `_` notation in generics. 
2. You can define multiple functions with the same name. Dispatch is based on static type.
3. Unions will not support dynamic extension.
4. You are free to have union args, but there should be no overlap.
5. When using function name to create lambda, if there are multiple candidates you have to use `_:int` notation to specify which one.
How can I return a lambda which can point to any draw? you can't. you can use `draw(_:Circle, _, _)` to create a lambda based on one function.
```
Hashable = {hashCode: (->int)}
```
```
draw = (s: Circle, Canvas, float -> int) {...}
draw = (s: Square, Canvas, float -> int) {...}

Shape = { draw: (Canvas, float -> int), paint: (Canvas, int -> string)}
getShape = (name: String -> Shape) {
	if name is "Circle" 
		c = Circle{...}
		return Shape{draw = (n: Canvas, f: float -> draw(c, n, f)), paint = (n: Canvas, x:int -> paint(c, n, x))}
}
f = getShape("Circle")
f.draw(c, 1.12)
```
**Proposal**:
1. No `_` notation in generics.
2. No functions with same name. 
3. You can define union or non-union for function input.
4. Functions can return a protocl: a lambda or a struct which has a set of lambdas.
5. Expression problem: Adding new types: just add new functions that support expected protocols, adding new operations (e.g. print): You need to write your own function to support this.
q: Can this be combined with generics to make it more powerful?

Y - How can we extend a struct?
e.g. we have `A={int, string}`
and we want to define B type as A plus a float.
Proposal: use `*` for this. makes sense

N - Can we implement smart slice without core?

N - dotLang and FP are ideal for data processing. Filter/map a set of data ...
Check some examples

N - Are we planning to support https://langserver.org/?
Language server
probably but this is not part of core or syntax.

N - Ability to import a module with only some functions or types.
Covered in another item

N - (covered elsewhere) If we allow types inside struct, it means we can import a whole module into a struct.
And because no functions have same names, it might actually be useful and won't cause a serious problem.
This can help with name conflicts.
And also importing only part of a module.

N - There is an argument about simplicity and map/seq types.
Should they be built-in or not?
Costs of having them outside core: 
	- We need ptr and byte type
	- We need core support for memory allocation and dereferencing
	- We won't have map/seq literals
Costs of having them inside core:
	- Extra notation
	- It will be difficult to implement some specialised structures (e.g. special map or treemap or set ...)
	- Confusion with generic types (argument: unless we use functions to generate types)
Advantage of having them outside core:
	- Less things in language spec (Argument: map/seq is not a very complicated subject and everybody needs them)
Advantage of having them in core:
	- more intuitive code
	

N - Idea: Use functions for generic types and return `[]` notation for map and sequence.
With generics we can allow for more customised hash
q: what about specialised types? e.g. TreeMap, HashSet,...

N - Think of a real-world example.
We have a set of identifiers `Set<String>`
Each identifier has some children. The children may have children too.
```
process = (ids: [string] -> Graph[int]) {
	result = map(ids, (s: string -> getChildren(s)))
	:: Graph[int]{result}...
}
```
We also need a very good notation for data processing.
Maybe better than map/reduce/filter functions.
If dotLang is going to be a choice for data processing systems (db, messaging, queue, cache, ...) which are also concurrent,
the features for data processing should be easy to use.
And I don't say strong/powerful. because those are going to be basic essential features.
More powerful tools will be created off them.

N - If `[]` will only be used for generics, maybe we should stop using it in import.
It will definitely be used either for generics or for map/seq.
But if we decide to import modules as structs, we may be able to use it as a function (or maybe even give name to it).

N - Do we need a defer?
something to run before end of method.
For res
`print("done")::`
But this makes conditional return confusing
`::100 <- dsdsds`?
Not needed

Y - Proposal about generics:
1. Allow functions to return types (generic data types)
2. Use `[]` for map and sequence and bring them back in syntax (with their literals)
q: what about specialised types? e.g. TreeMap, HashSet,...
`type`s are compile time so if a function returns a type, it must be callable at compile time.
```
x = [1,2,3]
y = ["A"':1, "B":2 ]
g = x[0]
g = y["A"] #returns int|nothing
```

Y - Proposal:
1. Allow defining types inside a struct.
2. Import a module into current module or as a struct using some special function
========
All types must be compile time decidable.
Struct defines a set of bindings that have the same purpose.
How does type fit in here?
```
Customer = {name: string, age:int, Case: [int]}
process = (x: Customer.Case -> int)
```
And we can have multi-level types:
`process = (x: Customer.Case.Element ...)`
But Customer type is explicit and we can easily know where it is.
If we allow this, then a module will virtually a struct.
Can this give us polymorphism? doesn't seem so.
What should be name of a function that returns type?
The naming rule becomes more complicated because things are becoming more and more similar.
We can have a lambda in a binding: `processCustomers = (int->int) {...}` not `process_customers`
But we don't name functions based on their output type. a function will return a binding but its not named like a binding.
```
createStruct = ...
or
CreateStruct = ...
```
We should ask: Is this a function or a type?
The goal is to have a function here. We will in future support custom code in these functions so for example we can implement generics specialisation.
How does this work when I have a map of date to list of pair of customers to a list of orders as input?
`process = (data: [Date:[Customer: [Order]]]` nothing to do with functions that return type.
What about when input is a stack of above?
`process = (data: Stack([Date:[Customer: [Order]]]), num: int -> [string]) ...`
- This can help us remove ptr and byte from code. But maybe we still need byte for some bit level processing (e.g. encryption)
`int, float, char, byte, bool, string, type, ptr, nothing`
We can say primitives are `int, float, char, byte, type, nothing`
bool and string are derived.
```
CacheState = [string:int]
cache = (cs: CacheState->)
{
    request = receive(Message(CacheStore))
    new_cache_state = update(cs, request)
    query = receive(Message[CacheQuery])
    result = lookup(new_cache_state, query)
    send(Message{my_wid, query.sender_wid, result})
    cache(new_cache_state)
}
```
if functions that returns a type should be named like a type.
then how should I name a lambda that points to these functions?
answer: Lambdas can never (?) return types. because lambdas are dynamic and can point to any code.
But types must be compile decidable.
But this can be combined so we can have: `Customer(int).Case(float)`
But we can have:
```
helpers = @("/core/std/utils")
helpers.io.write("Hello world")
#inside utils module
io = @("/core/utils/io")
```
There will be no problem with name conflicts.
We can use `*` to import into current: `_ = *@("/core/std/data")`
start does not mix with `@` beautifully.
`_ = *!("/core/std/data")`
`Helper = !("/core/std/data").DataUtils`
This also means that we can provide values for bindings inside a struct
```
Customer = {name: string, age:int, id = 12}
```
Can we define `name:string` within a module level decls?
```
name: string
printData = (->writeOut(name))
```
It does not make a lot of sense but should not be disallowed. We want orth and consistency.
What about comma separator?
I think we definitely need it when defining a real struct but for module we don't.
map/seq can be their own modules. we can import them as a struct.
If we assume a module is a struct, is it a type or a binding?
If it is a type (much similar to oop), then I can have a sequence module which I import and use it's type to define my sequences.
If it is a binding, then for sequence, the type must be defined within the module.
But I don't need to import anything for sequence. Let's say we have a set. then:
```
Set = !("/core/set")
process = (x: Set -> int) ...
#or
Set = !("/core/set").SetType
process = (x: Set -> int) ...
```
A module is a static definition of types and functions and it does not make a lot of sense to "instantiate" or "copy" that (?). Because of this, maybe it's better to have them as types.
So imported module will not be a value. So I cannot access its bindings unless I instantiate them.
Let's try to make things more simple and easy to understand.
What is the default behavior that someone expects?
Of course it would be easy if I can write: `Set = !("/core/set")` and then use Set directly as a type.
Doesn't this simulate OOP? Module is a type, it has private and public items. others can import it and use it but only public members.
in OOP (C#) when I import a package, I have access to it's defined classess and types -> types. But it is not a type itself.
If imported module is a type, I can easily have multiple types within it. Because a struct can contain other types too.
If imported module is a binding, I still can have multiple types in it.
when I have a binding, I expect data in it. or at least I expect I can set value for its fields.
Just like when I create a new customer: `c = Customer{name: "mahdi", age:10}`
Can I do the same with modules?
`c = !("/aaa/customer")?????` If it is a value, it already has everything and I cannot mutate it.
`c = !("/aaa/customer){name: "mahdi"}` if it is a type, I can set fields and use the type to create a new binding.
so I think it makes more sense to say imported modules are types. they are struct types. which contains all the types and bindings defined within the imported modules.
If it is value, to use it multiple times we need to import it multiple times.
can we gather all dependencies in one place then?
```
#root module
Set = !("/core/set")
Utils = !("/github/process/v1.5/master")
Helper = !("....")
#anybody who imports root (this module) will have a struct type which has 3 types in it: Set, Utils and Helper.

#other modules
Set = !("root").Set 
Utils = !("root").Utils #we don't care about version here
```
This can help with name conflicts.
And also importing only part of a module.
Types defined in a struct, cannot be overriden in its instances (?)
```
Customer = {name: string, Case = int}
g = Customer{name = string, Case = float}
```
Even if we allow this, using type from a binding is not possible. because types must be compile time decidable.
So referring to `my_customer.Case` may refer to any type. so it is not allowed
But `Customer.Case` is ok because at type level, Case is statically defined.
The whole idea of defining type inside struct was due to defining a linked list via a function, but actually we don't need it.
We should give a meaningful explanation about structs that have types and relation with instances.
But this idea simplifies import.


Y - What about adding operators for send/receive and string regex match `~`?
Don't forget about select/alt.
`send(my_task, data)`
`msg = receive((m: Message -> m == {source:1, type:2}))`
For send, we need a task and a message (anything) to send.
`data >> task`
`msgs = <<(m: Message -> m.sender = 1)`
They should be composable. Why not make it part of task type?
`task.send(x)`? no. makes no sense.
we also need sometimes to send and wait for message to be picked.
1. send a data to a task
2. receive data with a filter
3. send and wait for message to be picked up
4. receive with timeout
what if we give access to the mailbox? It's internal elements can have different types. How can we do this?
But if we do, send means append to mailbox.
receive means remove from my mailbox. but this conflicts with mutability.
receive with timeout can be simulated with a timeout process.
1. send a data to a task
2. receive data with a filter
3. send and wait for message to be picked up
I think 3 is also not very common. The whole point of tasks is being async.
1. send a data to a task
2. receive data with a filter
receive with a filter is actually a filter on a sequence but returns zero or one elements. so it is firstMatch.
```
was_sent = $.send(data, wid)
msg = $.firstMatch(Message, (m: Message -> m.sender = 12))
```
we can say that `$` is my mailbox. 
but what is its type? we have types for everything.
and what if I pass it to some other process?
And how does `firstMatch` imply item will be removed?
```
# you cannot send $ to any other function. It is just available eveywhere and points to the current process local mailbox.
was_sent = $.send(data, wid)
msg = $.firstMatch(Message, (m: Message -> m.sender = 12))
```
too much exceptions.
```
was_sent = data>>wid
msg = Message{sender=12}<<
```
Instead of lambda, we can write example to match. But what if we don't care about some fields?
The match has all the fields.
This is not receive. We have already received the message. 
This is a pickup operation. we just have to make sure it is not applicable to any other binding.
because it is mutating.
`data.(wid)`? not very intuitive.
Maybe its better to use functions.

Y - Built-in notation for map/reduce/filter:
Map/reduce/filter can be done on any type. In java it is a stream or iterable or collection.
So this should not be only limited to seq/map.
But 95% of use cases are for map/sequence. So why make things so complicated? Work for 95% and 5% will write their own code.
```
data = [1,2,3]
data2 = data.map((x:int -> x+1))
data3 = data.filter(x:int -> x> 0)
data4 = data.reduce((x:int, state:int -> x+state), 0)
tbl = ["A":1, "B":2] #[string:int]
tbl2 = tbl.map((k: string, v: int -> {k, v+1})
sum = tbl.reduce((k:string, v:int, state:int -> v+state), 0)
fltrd = tbl.filter((k:string, v:int -> v>0))
```
This is better than using strange notations for map/reduce/filter. and is easy to use.
Note that `map` function of a sequence is a real function that only accepts a lambda. It's owner is in its closure so we can use `data.map` as a lambda and pass it to others.
Similarly we can add other useful functions: `foreach`, `allmatch`, `anymatch`, ...
```
Set = !("/core/set").CreateType
process = (x: Set(int) -> 
```
**Proposal**
1. For map and sequence type, add methods for map, reduce, filter, anymatch, ...

Y - Add `task` as primitive type. But can we avoid it?
Why can't we use a struct? Defined in core but it's not a primitive type.
`task = {id: int, address: int, ...}`
If we do this, we can add appropriate functions for send/receive.
We can use a core function to get current task.
```
my_task = getCurrentTask()
my_task.mailbox.pick(Message, (m: Message -> m.sender = 12))
```
We can define two tasks in core: `CurrentTask` and `Task`
For current task I have `pick` function.
For Task, I have offer function to send a message.
```
getCurrentTask().pick(Message, (m:Message -> m.sender = 12))
xid := process(10)
#type of xid is Task
xid.accept(my_message)
```
we can say CurrentTask includes all fields of Task so we can send messages to it too.

Y - Now that no two functions can have the same name, why not force import into a struct?
Of course import to current ns should be allowed too.
```
_ = *@("/core/std/data")
DataModule = @("/core/data")
Type1, func2, binding3 = *@("/core/module1")
```
Import result is a struct so you can unpack it just like any other struct type.
We allow `*` for both struct values and type. so we can destruct result of an import
But does it make sense to use `*` on a struct outside another struct?
It makes sense inside a module. It defined module level bindings and types.
But not inside a function.
We don't define types/data inside a function. We can use `*` on a struct value though.

N - Do we need a defer?
something to run before end of method.
For res
`print("done")::`
But this makes conditional return confusing
`::100 <- dsdsds`?
Not needed

Y - Zig
https://andrewkelley.me/post/zig-programming-language-blurs-line-compile-time-run-time.html
The idea is like me.
Type are first class citizens. Functions can even return types.
```
fn max(comptime T: type, a: T, b: T) -> T {
    if (a > b) a else b
}

max(f32, a, b)
```
comptime means the value must be specified at compile time.
I think we can eliminate that and say types must be specified at compile time.
specialisation:
```
fn max(comptime T: type, a: T, b: T) -> T {
    if (T == bool) {
        return a or b;
    } else if (a > b) {
        return a;
    } else {
        return b;
    }
}
```
Using functions that return a type, we can have generic types.
```
fn List(comptime T: type) -> type {
    struct {
        items: []T,
        len: usize,
    }
}
```
In dot:
```
list = (T: type -> type) {
	:: {item: T, next: nothing|list(T)
}
```
So we can have:
`Customer = {name: string, age:int}`
or use a function to create a type.
`Customer = makeCustomerType(string)`
This will make this notation obsolete:
`Customer = [T: type -> {name:T, age: int}]`
And so we will have no application for `[]`.
Maybe we can use it for sequence notation at it's most raw form:
`x = [1,2,3]`
`process = (x: [int]...)`
we can use generic types like this:
`process = (c: makeCustomerType(string) ...)`
or
`process = (c: Customer ...)`
But we need to have a difference between types and values.
Functions that return a type should be PascalCased.
Bindings that store a type, should be PascalCased.
Other functions are camedCased.
Other bindings are under_line_separated.
So we have two kinds of binding: To store a type or to store a value.
type bindings are compile time decidable.
`Customer = MakeCustomerType(string)`
How to define a linked list?
```
LinkedList = (T: type -> type) 
{
	Node = {
		data: T,
		next: Node
	}
	:: Node
}
```
We can return a struct of two types and use `*` to use both of them.
```
LinkedList = (T: type -> {type, type}) 
{
	T1 = {
			data: T,
			next: T1|nothing
		}
	:: T1
}
f: LinkedList(int)
```
Does this mean we can define a type inside a struct?
If we think of types as first class values, it makes sense.
We still have generic functions as usual: with type inputs. 
But output of a generic function is not a type.

N - Review primitive types
consider cryptography use cases and see what can be removed.

Y - What about `&` to concat?

N - What if we define types inside anonymous struct?
That won't be accessible. It must have a name.
```
data = {x = 12, Case: float}
```

Y - Do we still need vararg functions?
no.

Y - What if I send a CurrentTask to another task using send?
That might cause problems because the whole purpose of task is being synchronized.
Now for `Task` it should be fine because we can only send or `save` messages.
But for CurrentTask, if you send it to multiple concurrent codes, they might start consuming messages and it can cause problems.
The underlying assumption is that mailbox is for a single thread and won't be shared.
Another solution: Don't give anything like `CurrentTask` to the developer.
Provide a notation to pick a message.
```
my_task = getCurrentTask() #type of my_task is CurrentTask
msg = my_task.pick(Message, (m: Message -> m.sender = 12))

task := process(10) #type of xid is Task
accepted = task.save(Message, my_message)
picked_up = task.saveAndWait(Message, my_message)
```
getCurrentTask will return a Task which can be used to send a message.
To receive a message:
`msg = $(m: Message -> m.sender = 12)`
If we use a notation to receive, why not use a notation to send?
`accepted = task$my_message`
A process may decide to only accept certain types of messages and drop everything else. 
The possible options are numerous (send, sendwait, pick, pickwithdefault, ...) so let's stick to functions.
The limitation of: you cannot pass currenttask to another function makes sense and is not a huge deal.
```
msg = getCurrentTask().pick(Message, (m: Message -> m.sender = 12))

task := process(10) #type of xid is Task
accepted = getCurrentTask().send(Message, my_message, task)
picked_up = getCurrentTask().sendAndWait(Message, my_message, task)
```

N - Use string for task id
Why do we need details about another task? Why not make it just a number/identifier?
If the task is located on a remote machine, still the task_id will be an index into a reference table.
But we can send this task to any function. So it should be possible to find out about all the info we need to send messages.
If we use int, it won't be possible. But string can contain anything. a simple number, a hostname + port + number, an IP address, ...
Anyway, this is just a matter of `Task := string` definition. No big deal.

Y - How can we read data from a file if it is a task?
We need to send a command "read" then wait for a message, which is not ideal.
```
file = open("a.txt")
new_file, line = *read(file, string)
```
what about network?
we can do the same
```
socket = net("192.168.1.1")
socket2 = send(socket, "A")
```
console:
```
io = getConsole()
new_io = write(io, "Hello world")
new_io2, data = *read(new_io, string)
```
for socket, we can use task concept. but for console and file, there needs to be a signal that says "read input"
because they do not have an active party on the other side. We decide when to read. 
- For console IO there is no mutation. You can simply use standard functions.
- For file: 1) we can say file open gives an integer (just like C) which is index of an open file in OS table.

Y - Using `$` in place of `getCurrentTask()`.
`$.pick(...)`
`$.send(...)`

Y - We can have both `List.insert` for List module and `List.insert` for List struct.
Does it make sense?
```
List = @("/core/List")
my_list = List.insert(int, 1)
vs.
List = @("/core/ListUtils").ListType
my_list = List.insert(int, 1)
insertOp = @("/core/ListUtils").ListType.insert
x = insertOp(int, 1)
```
I think both are possible because a module is a type and it is a struct type.
Add to pattern section

N - Add to pattern:
```
Point = {x:int, y:int, 
	mult = (p: Point, p2: Point -> p.x * p2.x + p.y * p2.y)
}
```

Y - This is how you init a struct in Zig:
```
const p = Point {
    .x = 0.12,
    .y = 0.34,
};
```

Y - We can even init a struct imported 
```
my_customer = @("/data/customer").Customer{.name = "mahdi", .id = 112}
```

Y - Shall we return notation to set type for bindings?

N - With recent change, should we say Modules should be named like types?
`Set = @("/core/set")`
`Set = @("/core/Set")`
We import modules into types so the file name is no longer important.

N - Is this ok?
`point4 = Point{point3, .y = 101} #update a struct`

Y - Can we import from a sequence?

Y - Extra comma at the end?

Y - What if I use `Type{}` notation when there are fields in type that need initialisation?
`Point = {x: int, y:int, size:int = 2}`
`p = Point{}`?
Is this ok?
I think it should not be allowed. Because 1) compiler is not supposed to do thing on behalf of the developer
2) the code/data inside Point may rely on appropriate values for those fields.

N - `map` on a sequence has access to the owner struct. Does that make sense?
```
Customer = { name: string, print = (->console.writeLine(name))}
```
yes this is closure and is compatible with import and module concept.

N - Add to pattern
DB code reading with sql
```
Customer = {name: string, age: int}
saveCustomer = (c: Customer -> ...) {
	sql = string.format("insert into table values (%s, %s)", c.name, c.age)
}
```
It is similar to parsing json.

N - format/printf/...
Is there a good/elegant way of converting things to string with format?
e.g. rounded, date with ymd format, number with 6 digits, with padding zeros, ...
we can define them in separate functions and use string in our string.format
```
Customer = {name: string, age: int}
saveCustomer = (c: Customer -> ...) {
	sql = string.format("insert into table values (%, %)", format.toLower(c.name), format.prefixed(c.age))
}
```

Y - Can we have const definitions inside a struct?
```
Numeric = { PI = 3.14 }
```
But PI is not a type
```
Numeric = { pi = 3.14 }
```
You can have this at module level and use it inside module.
But same question happens if you import that module.
`pi_number = Numeric.pi`
Maybe we should have a separate notation for consts? 
Because `pi` might be `pi: float`
But this is obvious.
But someone can instantiate Numberic with a different value for pi!
```
n: Numeric = Numeric{pi = 1.2}
```
Idea: You cannot override value for bindings inside struct which already have a value.
This is like module level bindings that are constant.
Proposal: When instantiating a struct (which can also be a module), you cannot override values for bindings which already have values.
Proposal: You can access bindings at struct type level if they are initialized in struct decl.
If I can access a binding at type level, what happens to closure?
If I cannot, what happens to module level constants?
I think it makes complete sense to have closure at struct decl. and its useful (seq.map).
Let's say you can only access non-function bindings at type level.
So:
Struct type -> You can access types and value bindings
struct value -> You can access value and function bindings.
Let's also allow access to struct inner types. They have values (must have) and you cannot override them.
q: If I don't have access to a struct's internal function with its type, how can I do it with a module?
what if I say, importing a module gives you a struct value not a struct type?
Then you have closure, you can initialise values upon instantiation and you can call functions normally.
About constants: Still you have access to them.
Proposal:
- Import gives you a struct binding not a struct type.
- With a struct type you have access to inner types
- With a struct binding, you have access to everythin.
- For constants, just define a struct type + a struct literal
```
MathConst = { .pi = 3.14 }
mathConst = MathConst{}
```
You have access to module level bindings inside a module level function because it is closure.
If import gives me a binding, how can I define type of that binding?
Maybe we can use the name that comes left?
```
SetType = @("/core/set")
my_set = @("/core/set")
```
with SetType, I have the type so I can instantiate from it and access its internal types
With `my_set` I have a struct binding, closure, access to internal everything
so `@("")` can give either type or binding.
q: Why not make it one thing: type and you can instantiate from it? 
q: Even with type, I don't have access to decls like `MathConst.pi`?
with type I can have bindings. 
If import gives me a type, I can easily use it to instantiate multiple bindings. But if it gives me binding, I cannot have its type.
And it will make things simpler.
Also you can have bindings easily: `my_set = @("/core/set"]){}`
But with a pure import, you have a type so you only have access to types defined inside the module.
For anything else, you need to instantiate. 
so for example, if you have some functions defined inside a module, you don't have access to them with a simple import.
The only problem: For constants (pi, dayOfWeek, ...) you have to instantiate. which is not end of the world.
`math = @("/core/math").MathConstants{}`
Proposal:
1. Import gives you a struct type.
2. With struct type, you only have access to inner types
3. With struct bindings, you have access to everything including defined types.
Nothing is wrong with getting a binding with import. With a type I can create a binding.
The point is, we want to make things simple. So instead of defining two functionalities for one notation, we just have one.
q: What if that module has bindings without value? When someone wants to instantiate, they must provide value.
And if I import that module into current module, they will be my arguments too.
So, if I want to have access to functions of another module, I cannot simply use output of import or its type.
I have to create an instance. `helpers = @("helper"){}`
The output of import is not an issue. I can easily convert type to binding.
And it makes sense that import does not give me binding because module might have some inputs.
Point is about access, closure, types, ...
Proposal:
1. Import notation gives a struct type.
2. To instantiate a struct, you must define values for types or bindings that don't have values:
`x: int`
`T: type`
`process: (int ->int)`
3. Inside a module or struct, you have a closure which is access to bindings defined at outside scope.
4. If you have a struct type (which can also be a module), you only have access to it's inner types. Not the ones without value e.g. `T: type`
5. If you have a struct binding, you have access to everything defined inside it (not private of course)
So note that then you cannot write: `process = @("/core/Helper").process`
because `@` will give you a type, so you will have to write: `process = @("/core/Helper"){}.process`
And you cannot access any of functions (most important functionality of a module) without instantiating.
Maybe we should instantiate with `@`. We have access to types inside the module anyway, even with a binding.
If I get a binding, what happens to constants like pi?
`pi = @("/core/math").pi`
inside math module: `pi: float = 3.1415`
or if I define a type:
`MathConst = { .pi = 3.14 }`
I will need to instantiate it: `math_const = @("/core/math").MathConst{}`
Proposal:
1. Import notation gives a struct value.
2. To instantiate a struct, you must define values for types or bindings that don't have values:
`x: int`
`T: type`
`process: (int ->int)`
3. Inside a module or struct, you have a closure which is access to bindings defined at outside scope.
4. If you have a struct type (which can also be a module), you only have access to it's inner types. Not the ones without value e.g. `T: type`
5. If you have a struct binding, you have access to everything defined inside it (not private of course)
The problem with struct value is that we cannot come to type from it. So e.g. if I want to pass it to another function, type is not known.
Con of getting type: To access functions or bindings, I have to instantiate.
Con of getting binding: There is no certain way to have type in case I want to pass it, you cannot embed a module inside a struct definition
We can have polymorphism with modules, I think.
```
Persister = { save: (string->int) }
#file_persister
save = (string -> int) ...

#console_persister
save = (string -> int) ...

#main
p = $("/core/file_persister"){}
q = $("/core/console_persister"){}
```
But this can be easily achieved using function pointers. The main use for polym is for having different types.
If import gives us types, shall we change naming rules for modules? not necessary. Because you should be free to choose any name for result of import.
`Set = @("/core/set")`
`SetUtils = @("/core/set")`
`@("/core/set").Helpers{}.format`
Proposal:
1. Import notation gives a struct type.
2. To instantiate a struct, you must define values for types or bindings that don't have values:
`x: int`
`T: type`
`process: (int ->int)`
3. Inside a module or struct, you have a closure which is access to bindings defined at outside scope.
4. If you have a struct type (which can also be a module), you only have access to it's inner types. Not the ones without value e.g. `T: type`. B
5. If you have a struct binding, you have access to everything defined inside it (not private of course)


N - The border between type and binding becomes more and more blurred.
We can define a struct type with some functions.
Or we can define anonymous struct and store it in a binding with some functions.
```
Helpers = { format = ... }
helpers = { format = ... }
```

N - You are not allowed to have types without value:
`DataType: type`
Because this can give developer ability to instantiate the struct with some random type.
But maybe it is a good thing.
Maybe this can give us another way for generics.
We can say any type must be specified at compile time.
so:
`Stack = {T: type, data: [T], push = (x: T) ...}`
`s = Stack{.T=int, ...}`
What's wrong with this?
We can say even types can get their value at the time of defining structs. but definitely compile time.
We can say, closure also covers types. I can have access to `T` when defining `push`.
But of course, I cannot have a lambda pointing to `Stack of int.push` because it's a generic function.
So we can say, when calling a function or instntiating a struct, you have to specify type for types.
But we were supposed to support generic data types by using functions, not structs.
It is orth and consistent to support both approaches.


N - Can we mix import with struct decl? import inside struct def?
```
Customer = { name: string,
	age: int,
	*@("/std/customer_info")
}
```
If import gives us a binding, you cannot use it when defining a type like above.
But if import gives a type, you can prefix it with `*` to embed inside another struct.
and it makes sense and should not be forbidden.
If import gives binding, I can embed it inside a struct binding:
`my_customer = {name: "A", age: 12, @("/dasdsad"){}}`

N - Formalise `T: type` as a type parameter vs. named type or type alias

N - How can this be used to ease interop with other languages e.g. C or Java or C++ or Go?

Y - When I write `my_task = process(10)` there should be a way to get result of `process`.
Like:
```
isReady(my_task)::getResult(int, my_task)
```
So task structure needs to be generic.

N - Casting sometimes returns two items and sometimes one.
clarify
cast to union returns two items
anything else one. because source and destination are fixed.

Y - If a struct has a field with a literal value, can I access it without instantiating the struct?
I think it shouldn't be allowed.
But what if I import a module which has some module-level bindings with constant values?
Just instantiate it.
```
Customer = {id = 12, name: string}
c = Customer{.name = "mahdi"}
id_12 = Customer.id
new_id_12 = c.id
...
Numeric = { pi = 3.14 }
pi_number = Numeric.pi
```
Let's allow it.
So with a struct type, you have access to types and constants.
for anything else, you need to instantiate.

N - Can we make code more readable?
Normally code (in real world) is polluted with two things: error checking and logging.
So it will make it difficult to read the actual code.
```
log("running process with args: " & 1)
data = process(1)
log("checking result")
data == nothing :: nothing
log("running process2 with args: " & 2)
data2 = process2(2, data) 
log("checking result 2")
data2 == nothing :: {100}
...
```
In Go2 one proposal is for collect (run multiple commands and as long as no error continue):
```
collect err {
		_! := SomeErrorProneFunction()
		_, _! = AnotherFunc()
		// ...

		i, _! = LastFunc()
	}
	if err != nil {
		fmt.Println("Error in SomeBigFunction:", err)
		return 0, err
	}
```
another proposal:
`x, checked err := someCall()`
meaning if err was not null, return it.
can we write this?
`data2 = process2(2, data) // :: 100`
as a shortcut for:
`data2 = process2(2, data)`
`data2 == nothing :: 100`

Y - We say you cannot re-use function names. So what about casting?
`int(x)` x can be float or string or ...
We have 3 applications for cast:
1. To NamedType: `x = MyInt(int_var)`
2. To primitive types `y = int(my_age_str)`
3. Union `int_value, is_valid = int(int_or_float)`
option 1 : We can say these are not functions. They just look like functions. Problem: orth, what if I want to send a lambda of int function?
option 2: use generics
1. To NamedType: `x = MyInt(int, int_var)`
2. To primitive types `y = int(string, my_age_str)`
3. Union `int_value, is_valid = int(int|float, int_or_float)`
option 3: As members of a struct
1. To NamedType: `x = int.parse(MyInt, int_var)`
2. To primitive types `y = int.parse(string, my_age_str)`
3. Union `int_value, is_valid = int.parse(int|float, int_or_float)`
opt 3 can be congfusing and not possible for named types.
1. To NamedType: `x = MyInt(int, int_var)`
2. To primitive types `y = int(string, my_age_str)`
3. Union `int_value, is_valid = int(int|float, int_or_float)`
or we can have a special notation for casting: e.g. `$Type`
So it is different from normal function call.
for casting string to int and similar, we can use normal functions: `strToInt(my_age_str)`, or `floatToInt(pi)`
For named type, a syntax similar to struct? `x = MyInt{int_var}`
or we can say, `Type{...}` is used to cast what we have inside `{}` to type `Type`.
this can be used for other examples too.
`x = MyInt{int_var}`
`y = int{my_age_str}`
`int_value, is_valid = int{int_or_float}`
But this last one does not make sense.
`int_or_nothing_value = int|nothing{int_or_float}`
We can use a core function: `hasType(int, int_or_float)` will return true if given binding has int.
So, the casting happens only to named types and for primitives.

N - Can we remove type spec in bindign decl and use casting notation instead?
`x: int = 12` vs `x = int{12}`
So, casting a literal means type spec.
You can also use cast to "stress" the type:
`int_result = int{getIntVar(10)}`
The only problem: what about complex types?
`g = [int]{1,2,3}`
`h = [string:int]["A":1, "B":2]`
We can either use `[]` eveywhere (what about structs then?)
or use `{}` everywhere -> what about seq and map?
First one is better:
`Type[data]` to seq, map.
`Type{fields}` to cast struct
Or we can allow `name: type ` notation to prevent all this confusion. To cast `Type{}` and thats it.
For sequence and map, you can still cast but for a literal, you just need to define type of the left side of `=` if you like.

Y - Is this notation elegant?
`Queue, Stack, Headp = *@("/core/std/queue, stack, heap") #import multiple modules from the same path`
It is confusing. You can simply take out the prefix and re-use it.
`p = "/core/std/"`
`Queue, Stack, Headp = *@(p & "queue") #import multiple modules from the same path`
`Queue, Stack, Headp = *@(p & "stack") #import multiple modules from the same path`
`Queue, Stack, Headp = *@("/core/std/queue, stack, heap") #import multiple modules from the same path`

N - Can we have a better notation than `*`?
Goal is to expand/destruct a struct.
`}{` no. too confusing.
`f,g = *struct1`
`Circle = {r: float, *Shape}`
No its ok.

N - From reddit (I just found the topic tonight):
1. I do like files implicitly being modules, but there's so much value to declaring modules freely, and having to make a new file for each is friction.
2. The comparison table is a bit odd ... why don't compare it to other functional languages like Haskell
3. Less keywords is not really being achieved with weird symbol usage.
4. It seems pretty terrible to reuse = and := for such radically different things.
Notations that might be replacable with keyword:
`*`, `$`, `_`, `::`, `@`, `:=`
`$` -> `task`
`::` -> `return`
`@` -> `import`
`_` for lambda
`_` for assignment: `x,_ = *getData()`
Using `:=` to define new type is confusing.
`MyInt = int` type alias
`MyNewInt := int` named type
Lets use a notation after `=` rather than using `:=`
`MyNewInt = clone(int)`
`MyType = %int`
or:
`MyAlias: int` alias
`MyType = int` new named type

Y - Use `:` for type alias 

N - Can we replace `{}` with new?
what about modify on struct?
`location = Point{.x=10, .y=20, .data=1.19}`
`point4 = Point{point3, .y = 101}`
`point4 = Point{point3, .y = 101}`
becomes:
`location = new Point{.x=10, .y=20}`
`point4 = modify point3{.y=101}`? 

Y - Can we also stop using `:=` for concurrency?
`x = process(100)`
`task1 = \\process(100)`
We can use `&` for concurrency. and `+` for concat.
`x = &process(10)` 
**Proposal**:
1. Use `+` for concat
2. Use `&` to start parallel processing
3. Remove `:=` for concurrency
But what if I don't use `&` prefix? It won't run in parallel.
Also, you can have a normal function which returns a task_id.
what about socket/...?
`socket_task = net("192.168.1.1")`
Don't prefix `&`. It will give you a task always.

Y - can we make the notation for destruct, better?
`x,y = *struct1`
this is taken form python
Note that we can also use it for type in addition to value. which might be confusing.
`Circle = {*Shape, r: float}`
we can use `_` for this: `_: *Shape`? no. still confusing.
This is basically a notation for compound data structures.
It is confusing because it mixes type and value (binding). When I write `*Shape` what will sit there? Some `x: type`s? or it will also have values like `draw = ...`?
we can write: `{a,b} = getData()` basically, enclose left side in `{}`
the negative point: You cannot use this inside a function call.
`processXY(*getPoint())` -> `{x,y} = getPoint()` and `processXY(x,y)`
but maybe it is better. Less powerful but more readable and understandable.
can we do the same for type?
`Shape = {name: String}`
`Circle = {r: float, _: Shape}`
we are using `_` to create a lambda and for assignment to ignore result of destruction.
`_` in `{x,_} = getPoint()` means we know something's there but we don't care where and under what identifier it is stored in memory.
why do we even need this for types? maybe not. originally, I think it was needed for polymorphism but no longer.
**Proposal**:
1. Remove `*`
2. To destruct a struct: `{x,y,_} = getStruct()`
3. No destruct for types

N - Do we still have slices? with re-use features?
This should be in core.

N - Extend comparison table

Y - To simplify map query, we say `map[key1]` will return `Value|nothing`. So you are not supposed to store nothgin as value or else your data will be lost.
Shall we do the same for array?
from SO: `The very reason we have Haskell is to avoid such runtime errors!`
Same for casting for unions.
But for union, it is fairly common to have `int|nothing`. So how can I say if `int{my_value}` gives me nothing, then I don't have int inside my_value.
`nothing{my_value}` will always give me nothing. because if it has a nothing, I will get nothing. If it doesn't, I will get nothing to indicate cast failed.
In these cases, you can use `hasType` from core or check for other types.

Y - For struct modify, can we use `+`?
`{10} + {20}` will give you a struct with two int fields.
`{.x=10} + {.y=20}`
`point2 = point1 + Point{.y=100}` take x from point1. No does not make sense. How is compiler going to know which field to keep and which one to take from point1?
```
{x,y, _} = point1
point2 = {.x=x, .y=y, .data="A"}
point2 = {.x=point1.x, .y=point1.y, .data="A"}
point2 = point1{.data="A"}
```

N - How can I import into current ns? e.g. I don't want to prefix core functions.
Maybe we should not allow that.

Y - Fix import section, remove `*` and multiple impotrs 

N - If we use `import` keyword, can we import a struct defined locally?

Y - Replace range operator with core function

N - Reddit feedback:
The main ones I object to are `&, @ and ::` and function declaration without any keyword (nothing bad about () and {}, but I wish there were more to indicate what kind of block each is rather than -> inside the () turning the whole thing into a function).
`fn`?
`process = fn (x:int -> int)`
Rust: 
```
fn is_divisible_by(lhs: u32, rhs: u32) -> bool {...
}
Lambda: 
let x = |i: i32| -> i32 { i + 1 };
```
Swift:
```
func greet(person: String) -> String {
... return x
}
```
Scala:
```
def addInt( a:Int, b:Int ) : Int = {
	...
      return sum
   }
Lambda:
val even =  ( x :Int ) => x % 2 == 0
```
Go:
```
func add(x int, y int) int {
	return x + y
}
var baz Stringy = func()string{
    ...
};
```
Can we act like Rust and use last expression in the block as return? And make early return easier by using nested blocks?
Proposal for function decl:
```
process = fn(x:int->int) { :: x+1 }
process2 = fn(x:int -> x+1)
```
Or we can use a symbol:
`process = ~(x:int -> x+1)`
`process : fn(int->int) = fn(x:int -> int) :: x+1`
`process : fn(int->int) = fn(x:int -> int) { :: x+1 }`
Golang does not allow shortcut syntax for functions.
`process : fn(int->int) = fn(x:int -> int) { :: x+1 }`
`process = fn(x:int, y: fn(int->int) -> fn(string->int))`
Or we can use `[]` for decl and `()` for call.
`process : fn[int->int] = fn[x:int -> int] { :: x+1 }`
`process = fn[x:int, y: fn[int->int] -> fn[string->int]] { ... }`
`[]` is also used for decl of seq and map and their literals.
So:
**Proposal**:
- `fn[]` is used for function type and declaration `fn[x:int -> int] { :: x+1 }`
- `()` is used to call function
q: Can we make it similar to a hash?
`process = [string:int] ...`
`process = [string:int] { ... }`
But then we need to use `[]` to invoke too.
Let's use `()`
- `fn()` is used for function type and declaration `fn(x:int -> int) { :: x+1 }`
`process = fn(x:int, y: fn(int->int) -> fn(string->int)) { ... }`
Summary in later item.

Y - Reddit feedback:
Five out of the 16 are not well known or really used that that way in any other language I can remember:
```
& Concurrency

$ Access current task

// Nothing-check operator

@ Import

:: Return
```
That's a third of the operators being invented then and there. What's worse is I can think of popular and renowned languages that use these operators for completely different purpose, so saying they're well known is not really right: all the symbols on the keyboard are well known but using them with new semantics changes everything.
Reply:
`//` I think `//` is fine because it is also used in Perl so not completely invention.
`&` Concurrency: Go uses `go`. Maybe I should use a core function? 
`$` access current task. 
`@` import: We can use `import` function from core. But this is not a function. It gives you a completely new type which didn't exist before.
`::` return. This either has to be a statement or a symbol. Because it does not give you an expression.
We can follow Perl way:
`return 100`
`return 200 if x < 0`
If we use `if` in this context, nothing should stop people to use it in other contexts.
Maybe we can use `@import`, `@task`, `@run` ...
Issues with: return, current task, concurrent run, import, function decl.
`$` -> `currentTask()`
`&` -> `currentTask().spawn(int, ( -> process(10)))`
`//` keep it
`@` 
`::`
Maybe we can only use `return` but introduce `match` 
if I can only keep two of these, they will be `$` and `::`.
Then what about import?
`$` -> keep it
`&` -> `$.spawn`
`//` keep it
`@` 
`::` keep it?
`customer = import "/core/a/data"`
if I use `()` it will be like a function.
The responsibility of import is to create a type based on definitions in another file. You can use this type to instantiate or ...
using `::` to access types inside another type makes more sense (similar to C++).
But then, what are we going to do about return?
`return 100`
`return 200 if X`
`->100` return 100
`(x>0)->100` return if
**Proposal**:
- Keep `::` for return and `@` for import and `//`
- use `getCurrentTask` instead of `$` and use it for `&` too.
But it will make things more difficult to read for concurrency, because everything must be converted to a lambda.
`x = $.spawn(int, (->process(10)))`
vs
`x = &process(10)`
Let's revert back to `:=`. Because we use `:` for type alias and `=` for named type.
`:=` gives us task ids?
What if on the right side we have a function that gives you int.
`x := getCustomerAge(a)`
having int on the right but task on the left seems a bit unusual.
what about using case?
`x = task[int](getCustomerAge(a))` but here I will need to execute the getCustomer first.
This is a different concept and semantic.
What if a function already returns a task? Then you don't need to use `:=`.

N - Getting type as a result of import is a bit odd.
Suggestion: Import gives a struct instance and you can use the import notation to refer to other types inside the struct.

Y - Can we avoid mixing type and struct?
It is confusing.
We can use a different notation to access types. e.g. `Mod..Type1`
Let's say import gives us a binding with everything that the module has.
We can then use `.x` to access inside bindings or `..X` to access inside types.
`Module1..X..Y` means type Y defined inside struct type X defined in some module.
Proposal:
- Ban type inside struct
- Import gives you the decl inside the module but not types
- For types use a different notation.
If we ban types inside a struct, then what is the output of an import?
Maybe it should be something new, not a struct.
So, we won't be able to pass it around.
Suppose that we have customer module. It has a Customer type + CRUD functions for customer.
```
Customer = {name: string, age: int}

saveCustomer = (x: Customer -> ... )
loadCustomer = (id: int -> Customer) ...
deletecustomer = (id: int -> )
upadteCustomer = (id: int, new: Customer ) ...
```
Then when we import this module, in addition to the functions, we also need access to the customer type.
We have two options:
1) Import everything in one import and access using `.` or `..`
2) Import types separately.
`chelper = @("customer")`
`CustomerType = @(`
a module with types is a struct with `type` fields.
```
Customer: type = {name: string, age: int}
```
So, if we look at it like this, a struct binding would be enough. We have access to bindings (functions, values and types).
But, if the module has some variable bindings (e.g. `x:int`) we have to specify values for it.
`customer_module = @("customer"){.x=100}`
We can simply ban bindings without value inside a module. But it will be a new exception.
What's wrong with having type? Because if import notation gives us value, we then will have to add a new rule (ban).
Can we embed `..` inside import?
`A..B` means access to type B defined inside sturct/module A. Now we can use `@` as the root type.
`CustomerType = @..("Customer")`
We are accessing type "Customer" module from root type. No. too confusing.
`CM = @("customer")`
`Customer: CM..Customer`
Let's say:
- Types are accessible using struct type and `..` prefix.
- Types in another module are accessible using import's output type and `..`
- Import gives you type.

Y - Use core fn for casting
- When you see ? it is a type cast: `data = cast(string, int, some_string)`
We can use core functions to reduce usage of notations or using them for multiple purposes. Casting is one of the areas that we can use.
We use casting for union, named type and primitives.
`{age, is_valid} = tryCast(int|string, int, my_int_or_string)`
`int_val = cast(MyInt, int, my_int_val)`
`int_val = cast(string, int, age_str)`

Y - Can we make `:=` notation more natural?
e.g. `{result, task} := process(10)`
most minimal way: `result := process(10)` where result is the output of the call.
But how can we get a reference to the new task?
one way: make another call
```
result := process(10)
task_1 = getLastTask()
```
or
```
result := process(10)
task_1 = getTask(result)
```
But regarding the second option, it will be difficult to keep track of result because it can be sent over to other functions.
In this interpretation, `:=` is exactly like `=` but the output will not be ready immediately.
```
result := process(10)
task1 = task{result}
```
But there is room for mis-use. People can cast garbage to task and cause exception.
```
result := process(10)
task1 = getCurrentTask().getChildren().last()
```
Y - Should we be able to define type alias inside a function?

N - use `fn` for functions
- `fn()` is used for function type and declaration `fn(x:int -> int) { :: x+1 }`
`process = fn(x:int, y: fn(int->int) -> fn(string->int)) { ... }`

N - Idea: treat array and map as a function. Good re orth and generality but bad re performance.
If I get `[int]` in a function how am I supposed to compile `x[0]` or `x(0)`?
I think this might be possible. I can save accessor code (which can be a function invoke or memory ref)
I can keep track of this accessor and inline it if it is small.
So, sequence and map are also functions. Everywhere that you need a map/sequence, the caller can also send a function instead.
```
process = [int->string] { ... }
data = [int->string] {1->"A", 2->"B", 3-> "C"}
a_data = data(1)
data2 = [string] { "A", "B", "C" }
func2 = [a:int, b:float->string] { string{a+b} }
func3 = func2(9,_)
```
So `[]` will be used for function decl and `()` for function call.
We also use `[]` for generics? No. We use functions to define generics.
So `[]` will solely be used to declare a function type or literal.
`()` only for function call (which also is for generic types)
Ultimate unification: `[]` for map and sequence and function declaration
`{}` for map and sequence and function literal
`()` for map and sequence and function get/invoke
We can say, a sequence or map definition is actually a function which you can invoke using the key.
Hence we should be able to define a map with multiple keys:
`map2 = [int,float->string] {1->2->"A", 3->4->"B"}`
`f2 = [x:int, y:float->string] { :: string{x+y} }`
We also use `{}` for struct decl and literal and casting.
`[string]` is a shortcut for `[int->string]`
But we are using same symbol for many different usages. `[]` is ok because it is only function decl.
`()` is also ok because it is only for function call.
But `{}` is for A) struct type decl B) struct literal C) code block and D) casting
We can either use another notation or use some keyword prefix.
```
Customer = {name:string, age:int}
my_customer = Customer{.name="A", .age=12}
new_customer = old_customer{.name="B"}
process = [x:int->string] { ... }
data = int{other_data}
```
For casting and struct literals, we have a prefix but still it can be confusing.
We need to have simple rules, this can make reading code and writing it and also writing the parser and compiler easy
- When you see `fn {}` it means this is a code block. that's it.
- When you see `struct {` it is a struct type decl
- When you see ? it is a struct literal `Type{`
- When you see ? it is a type cast: `data = cast(string, int, some_string)`
We can use core functions to reduce usage of notations or using them for multiple purposes. Casting is one of the areas that we can use.
We use casting for union, named type and primitives.
`{age, is_valid} = castUnion(int|string, int, my_int_or_string)`
`int_val = cast(MyInt, int, my_int_val)`
`int_val = cast(string, int, age_str)`
So we have:
```
Customer = struct {name:string, age:int}
my_customer = Customer{.name="A", .age=12}
new_customer = old_customer{.name="B"}
process = [x:int->string] { ... }
```
We can say `[]` prefix means it is a struct type, but it will be super confusing.
Can we say struct is a combination of multiple functions?
```
Customer = {name:string, age:int}
Customer = ["name"->string, "age"->int]
```
Too much change and this is not intuitive and lacks a lot of properties that a struct should have.
`{}` is for A) struct type decl B) struct literal C) code block D) Sequence literal E) map literal
C,D,E are same or similar. But what about A and B?
```
Customer = {name:string, age:int}
my_customer = Customer{.name="A", .age=12}
new_customer = old_customer{.name="B"}
process = [x:int->string] { ... }
```
If we do this, what happens to 2-d sequence? 
`matrix = [[int]]` 
`matrix = [int->[int->int]]`
We can say either:
1) Everything is a map (sequence and function are maps)
2) Everything is a function (Sequence and map are functions)
second option is more flexible.
`age = nameToAgeMap("A")`
`lambda1 = my_map(_, "A")`
Sequence is a special function:
`data2 = [string] { "A", "B", "C" }`
This can give us some sort of polymorphism.
Caller can send a real function when an array is needed. or a hash-map.
But, we have some things like map or length on array. How are they going to be defined for a function?
it will be confusing and sometimes undoable.
what should `.length` return if sequene is a function? We don't know. Because sequence is a function not a struct!
So let's ignore this.
Back to the original problem: Use of notations, especially function decl is a bit confusing,
`()` function call
`[]` sequence and map
`{}` code block and struct
Now `(x:int->string)` as a function decl. It is a bit confusing maybe, because until we see `->` we don't know if this is a function decl.
But it's not totally confusing. What else can it be? struct? No. it has to start with `{`.
binding decl? It appears on the right side of `=`.
`x: (int->int)` x is a function that given an integer will return an integer.

Y - Instead of making it forbidden to send "current task" to other functions, design core functions so that user can only "query" for its internal data.
`getCurrentTaskChildren`, `getCurrentTaskId`, ...

Y - If we see this:
`process = (x:int -> {` 
we still don't know if we `{` specifies output type or output expression.
Only type will be allowed there.

N - How are we going to handle toString for different types?
suppose that I want to print something to stdout or a file or ...
`int_val = cast(string, int, age_str)`
why not use normal functions?
`str = customerToString(my_customer)+" "+locationtoString(my_lcoation)`

N - Everything is a function even seq or map.
`age = nameToAgeMap("A")`
`lambda1 = my_map(_, "A")`
Sequence is a special function:
`data2 = [string] { "A", "B", "C" }`
This can give us some sort of polymorphism.
Caller can send a real function when an array is needed. or a hash-map.
But, we have some things like map or length on array. How are they going to be defined for a function?
it will be confusing and sometimes undoable.
what should `.length` return if sequene is a function? We don't know. Because sequence is a function not a struct!
Sequence must have a fixed length. 
So does map. But maybe not. All we care is when we want to go over a sequence and do a map.
if we do this, map becomes function combination:
`x = [int] { 1,2,3 }`
`y = (index:int -> x[index]+1)` y is map of x
This is too confusing.

N - Select construct.
This enforces checks for input so for example if input can only be 1 or 2, we can have:
```
select x {
1=> ...
2=> ...
else=> throw_exception
}
```
Helps check for all possible cases and make code robust.
This can be used with a any type like int, string, or a union type. 
But how can we unify value and type selects?
If select is on a non-union, you must use values.
If select is on a union, you must use types.
This should be an expression. So, for union you must specify all possible types. and for values, you must have a default case.
```
result = $exp0
{
	exp_cond_1 => exp1
	exp_cond_2 => exp2
	_ => exp_default
}
```
- Think about how this will work when nested?
```
result = $exp0
{
	exp_cond_1 => $other_value {
							
							  }
	exp_cond_2 => exp2
	_ => exp_default
}
```
But value based select is possible by using a function and probably a sequence.
Same can be done for union by having `hasType` function in core.

N - Empty sequence is `[]`, empty map is `[:]`

N - Review channel ops and casting notation

N - Review notations and if they can be replaced with keywords.
We use `cast(A,B, value)`, but isn't it a bit too verbose?
At least don't have to use source type. only target type.
Unusual notations:
12. `@`   Import - A candidate for using core `import` function.
13. `::`  Return - This returns an expression and also can be conditioned.
14. `:=`  Parallel execution
15. `..`  Access type within a module/struct
07. `->`  Function declaration - Used in Swift, Rust
06. `|`   Union data type - used in F# and Haskell
08. `//`  Nothing-check operator, used in Perl
11. `_`   Place-holder (lambda creator and assignment), used in Scala
What can I replace `::` with?
`:: 10`
`cond :: 10`
`ret 10`
`cond ret 10` this does not look good
`(cond) ret 10`
`cond ret process(x)` the function will only be called if condition is satisfied.
That is starting point of ambiguity.

Y - Replace `@` with import keyword.
because memorizing `@` is difficult and unintuitive. This notation for this purpose is never used anywhere else.


Y - source of ambiguity: in conditional return `cond :: exp` will expression be evaluated all the time? even if condition does not hold?
```
ifElse = (T: type, cond: bool, true_case: T, true_case:T -> T) 
{
	cond::true_case
	:: false_case
}
```
we cannot say: You HAVE to use a literal or identifier in return
consider this example:
```
process = func(x:int -> string)
{
	x>0 :: saveLargeFileToDB("SDSDASDA")
	process2(x)
	:: saveSmallFileToDB("ddsadsadas")
}
```
How do we know that saveLargeFileToDB is not called all the time?
in other words will above function be like this?
```
process = func(x:int -> string)
{
	temp = saveLargeFileToDB("SDSDASDA")
	x>0 :: temp
	:: saveSmallFileToDB("ddsadsadas")
}
```
If yes, then it is not good because we don't want to save the large file if `x<=0`
we can do this via lambda or other functions:
```
process = func(x:int -> string)
{
	temp = [x>0 : fn(->string) { saveLargeFileToDB("SDSDASDA") }, 
		x<=0: fn(x:int->string) { innerProcess(x) },  ]

	:: temp[true]()
}
innerProcess = fn(x:int -> string) {
	process2(x)
	:: saveSmallFileToDB("ddsadsadas")
}
```
What about a notation that return if not nothing?
`c = ifOrNothing(x>0, fn (->int) { saveLargeFileToDB("DSADAS") })`
`return c // fn(->int) { ... }()`
Essentially it is still the old way of using lambda to represent rest of the function.
Proposal:
- Use `return` instead of `::` and remove conditional return
```
process = func(x:int -> string)
{
	return ifCond(x>0, fn{ return saveLargeFileToDB("SDSDASDA") },
		fn { return saveSmallFileToDB(">>>>") }
}
```

Y - Use a shortcut to create lambda based on a code block
instead of 
`fn (->int) { saveLargeFileToDB("DSADAS") }`
we write:
`fn{ saveLargeFileToDB("DSADAS") }`
If we decide to ditch conditional return, it will help. Also with conditionals in general.
Because we will have a lot of code blocks which have no input but some output.
for example:
```
process = func(x:int -> string)
{
	temp = [x>0 : fn{ return saveLargeFileToDB("SDSDASDA") }, 
		x<=0: fn(x:int->string) { return innerProcess(x) },  ]

	:: temp[true]()
}
```
and it will be a general rule: Any function without input can be shown by `fn { ... }` notation.

Y - Can we define a type inside a function?
we should be able to do that. this is specially useful in generic function types.

Y - Marker to differentiate type vs literal
It makes code more readable to differentiate between struct type and value.
We can say the same for seq/map
```
process = (x:int -> {name: String})
process = (x:int -> {x})
process = (x:int, y:int -> Point{.x=x, .y=y)
process = (x:int, y:int -> {x,y)
process = (X: Type -> {X})
numbers = [1, 2, 3]
IntArr = [int]
populations = ["CA": 45, "NY": 22]
Pop = [string:int]
```
option1: use keywords: `struct, map, seq`
```
Pop = map[string:int]
IntArr = seq[int]
process = (X: Type -> struct{X})
```
option 2: use a notation like `$`.
But then we will have to use same for all 3 cases (which is confusing) or use 3 different notations for 3 cases which is also too much.
If we use keywords, we can also then use `func` keyword for function types.
But what about literals?
```
process = (x:int -> {name: String})
process = (x:int -> {x})
process = (x:int, y:int -> Point{.x=x, .y=y)
process = (x:int, y:int -> {x,y})
process = (X: Type -> struct {X})
numbers = [1, 2, 3]
IntArr = seq[int]
populations = ["CA": 45, "NY": 22]
Pop = map[string:int]
```
I think we don't need this for seq and map.
note that combining these with a union will make reading it difficult: `Data = struct {int,int} | seq[int] | map[string:int]`
we have:
- function: `(x:int -> int)` value: `process = (x:int -> int ) { :: x+1 }`
- struct `{name:string, age:int}` value: `x = Point{.x=10, .y=20}`
- sequence `[int]` value: `x = [1,2,3]`
- map `[string:int]` value: `x=["A":1, "B":2]`
option 1:
- function: `fn (x:int -> int)` value: `process = fn (x:int -> int ) { :: x+1 }`
- struct `struct {name:string, age:int}` value: `x = struct Point{.x=10, .y=20}`
- sequence `seq[int]` value: `x = seq[1,2,3]`
- map `map[string:int]` value: `x = map["A":1, "B":2]`
It will make code ugly.
For struct literals, we always use type before `{` unless it is anonymous. In that case we can use `_` as type.
- function: `(x:int -> int)` value: `process = (x:int -> int ) { :: x+1 }`
- struct `{name:string, age:int}` value: `x = Point{.x=10, .y=20}` or `x = _{10,20}`
- sequence `[int]` value: `x = [1,2,3]`
- map `[string:int]` value: `x=["A":1, "B":2]`
So:
If we have a function definition before `{` then this is a code block.
If we have a type name or `_` before `{` then it is a struct literal
If we have none of them, this is a struct type.
what about a fn that returns another fn?
`process = (x:int -> (y:int -> y+x))`
rules: minimal, consistent, orthogonal, simple
Proposal:
- `fn` for function type and literal as prefix
- `struct` for struct type
- `_` for untyped struct literals and mandatory type for typed struct literals
```
process = fn (x:int -> {name: String})
process = fn (x:int -> {x})
process = fn (x:int -> (y:int -> y+x))
process = fn (x:int, y:int -> Point{.x=x, .y=y)
process = fn (x:int, y:int -> _{x,y})
process = fn (X: Type -> struct {X})
```
- function: `T = fn (int -> int)` value: `process = fn (x:int -> int ) { :: x+1 }`
- struct `S = struct {name:string, age:int}` value: `x = Point{.x=10, .y=20}`
- unnamed struct `S = struct {string, int}` value: `x = S{"A", 10}`
- untyped struct `x = _{"A", 10}`
Can I use type when defining a function literal? Just like struct.
```
Adder = (int,int->int)
myAdder = Adder(x,y->int) { ... }
```
Not beautiful. but otherwise it will be different from struct which means more exceptions.
for struct type FollowUp
This is how you define a lambda in Scala: `var inc = (x:Int) => x+1`
Swift: 
```
x = () -> Int {
        runningTotal += amount
        return runningTotal
}
```
Rust: `let plus_one = |x: i32| x + 1;`
They don't have any identifier for a lambda.
So, what if we just focus on structs?
- struct `S = ${name:string, age:int}` value: `x = Point{.x=10, .y=20}`
- unnamed struct `S = ${string, int}` value: `x = S{"A", 10}`
- untyped struct `x = _{"A", 10}`
But it feels odd, why do we need a prefix for struct and nothing else? If we want to have prefixes, we have to have them for all types.
when compiler sees a `{`:
- If there is a fn declaration before it -> this is a code block
- If no fn decl before, and no prefix -> struct type
- If no fn decl before and some prefix (`_` or a type name or a binding name) -> struct value
`process = (x:int -> {int,int})`
is output of this function a type? or a struct with two integers? we (and compiler) have to look further to see if there is a code block or no.
If there is a code block then output might or might not be a type
If no code block ?
No. even if there is a code block we don't know if output will be `{1,2}` struct or a struct type.
No. output cannot be atype.
You have to write it like this:
`process = (x:int -> type) ...`
`{int,int}` means output will be a struct value with two int numbers.
But what if we want to use the shortcut mode: like `process = (x:int -> x+1)` where we write output on the right side of `->`
We may write similarly: `process = (x:int -> {int,int})`
we have map, sequence, struct and function. to be consistent, either we have to use prefixes for all of them or none.
option: force right side of `->` to be a type and not an expression. it make writing more difficult but reading easier and also parsing easier.
`process = (x:int -> int ) { :: x+1 }`
so what comes before `{...}` is always type of the function
`process : (int->int) = (x:int -> int ) { :: x+1 }`
Example:
```
data = (x:int->int) { :: x+1 } (10)
```
now, when you see `(x` you don't know what is this. 
Proposal:
- use `fn` prefix for function type and literal
- right side of `->` must be type.
If we have a deterministic way to differentiate struct value and type, maybe this can be solved.
and `fn` prefix for fn types? It makes reading code easier: `(x:int->int)` vs `fn(x:int->int)`
What about this?
`data = ::(x:int->int) { :: x+1 } (10)`
We are already using name/keywords for types: `int, string, bool, char, ...`. 
Why not use the same for `function, sequence, map, struct`?
`x:int`
`x:seq[int]`
`x:map[int:string]`
`t:map[int:seq[string]]`
`r:struct{x:int, y:string}`
`T = struct{x:int, y:string}`
`r = func (x:int -> bool) { ... }`
Now we are sure that `{` without struct before it, is a code block.
```
MapType = map[int:string]
x: map[int:string]
SeqType = seq[int]
x:seq[int]
FType = func(int,string->bool)
fvar = func(int, string->bool) { ... }
fvar = func(x:int, y:int -> x+y) #not a type
fvar = func(x:int, y:int -> int) { :: x+y }
```
So, proposal:
- In function decl, after `->` it must be a type
- We use `seq`, `map`, `fn` and `struct` keywords before types
- No prefix for sequence and map literals.
- Struct values must be prefixed with type or `_` for untyped structs.
- Struct type must be prefixed by `struct` keyword
So, how can we define a lambda? What comes on the right side of `=` is a lambda.
`process = fn(s: fn(int->int) -> bool)`
`process = fn(x:int -> fn(int->int)) {...}`
How will this work with generic type functions?
`ValueKeeper = (T: type -> {data: T})`
and lambda or anything based on them? maybe we should use `type`.
But then, we can have two function with same name that return different things, both denoted via `type`.
which is not a good idea. Functions should not have same name (and cannot).
But, point is, function decl will not indicate the type of output. because we decided to have only type on the right side of `->`.
But generic type IS a type, but it might be complicated, e.g. combining two types ...
Makes sense to use `type` for return type. Also note we cannot pass these functions around or play with them very much because anything related to type must be compile time calculatable.
Do we really need `seq` and `map` prefix for those types? no.
it is not big source of confusion. Of course compiler/parser needs some info before it can say if `[` is type or value but that should be fine.

N - Review module import notation and using type/values from imported module
```
Std = import("/core/std")

x:int = Std{}.x_value
print = Std{}.printHelper
Customer = Std..CustomerType
```

Y - What if I want to import from project root?
not DOTPATH and not current module's path.
Let's say DOTPATH is set to where you run compiler from.
But we may actually want to import a sibling module.
`import("./module")` import from relative path
`import("../dir1/module")` import from relative path
`import("/dir2/module")` import from project root
If code imports something from std, first time compiler will download from github and put it in project's modules path
So to import std:
`StdMapper = import("git/dotLang/std/helpers/map")`
If compiler will download above, then we will never have to import from `/`?
We can put above statements in startup file so compiler will download them and put in modules root path.
Then everywhere else, we just write:
`StdMapper = import(/git/dotLang/std/helpers/map")`
We may have some local modules we want to use. If so, you need to copy them to project dir and use `/` to import them.
But for remote modules, we just use github/... url.
what about versioning?
`StdMapper = import(/git/dotLang/std/helpers/map")`
This should be put in branch name.
`StdMapper = import(/git/dotLang/std/v1.5.1/helpers/map")`
`StdMapper = import(/git/dotLang/std/v1.5.*/helpers/map")` import v1.5 with any revision number
`StdMapper = import(/git/dotLang/std/v1.5.4+/helpers/map")` import 1.5.4 and newer in `1.5.*`
`StdMapper = import(/git/dotLang/std/[84cfe230]/helpers/map")` specific commit
`StdMapper = import(/git/dotLang/std/{1.9.0-RC}/helpers/map")` specific release
Above is just a simple convention and naming.
Assume we only have one version or we don't care about version.
How are we going to handle importing a module on multiple places and module is on GH or local?
Let's say we dont allow local copy of files. but this is against gen.
But not really. Compiler will clean `deps` directory.
Rust uses `deps` and golang uses `vendor`.
We will use something like deps.
So, question is: when compiler sees `import(git/...` it will download and save to deps
when compiler sees `import(/..` it will look into deps directory.
But we may have other modules locally and not on github (for example they are sensitive).
Compiler should support using those local modules.
this will be project structure:
```
root ---- deps
	- src
	- build
	- ...
```
You can create an additional dir in above structure and put everything you want there.
and do `import(/myDeps/...`.
so this should be fine.
But when compiler sees `import(git/...` it will download and save to deps?
Or maybe not. why deps?
we can write `import(/git/dotLang/std/...)` and if the dir is not there, compiler knows what the URL is.
my point is, we may need to import some specific branch of a module from github in multiple locations in the code.
how can we prevent repeating the same string multiple times?
Why not use module level bindings?
define a main module and there:
```
#main
std = "/http/github.com/dotLang/std/v1.9.5/MapHelper"
```
and everywhere else:
`refs = import("/src/main"){}`
`X = import(refs.std)`

Y - Example of a small app which generates a random number between 1-100, asks for a number from input and outputs if these two are equal/greater/smaller.
```
stdin = import("/http/github.com/dotLang/std/stdin"){}
stdout = import("/http/github.com/dotLang/std/stdout"){}
rand = import("/http/github.com/dotLang/std/random"){}

main = fn
{
	secret = rand(1,100)
	stdout.write("Please enter a number: ");
	guessRaw: int|nothing = tryParseString(int, stdin.readLine())
		
	ifElse(guessRaw == nothing, 
	fn{
		stdout.write("Invalid number")
	},
	fn {
		guess = cast(int, guessRaw)
		actions = [guess<secret: fn{ stdout.write("Too small!") },
				   guess>secret: fn{ stdout.write("Too large!") },
				   guess==secret: fn{ stdout.write("Well done!") }
				  ][true]()
	})
}
```
compare above ifElse with what we would write in C:
```
if ( guessRaw == nothing) {
	printf("Invalid number");
} else {
	if ( guess<secret ) printf("Too small!");
	else if ( guess>= secret) printf("Too large!");
	else printf("Well done!");
}
```
Not a huge difference in terms of number of characters.

N - We can say compiler will look in two places for imports.
`src` directory, and
`deps`?
But it will be a limitation.
All external dependencies have http or https prefix.
No. It is better to have minimum number of rules.

N - We should make it easy for people to download and run dotlang project from web (e.g. GH)
So, user sees a GH project. how can he run?
first `git clone` to have a local copy
then `dot compile` without any input, it will look for `main.dot` at project root.
You can also provide path: `dot compile src/a/b/c/d.dot`
same for `dot run`.
But these are secondary after language, syntax, semantics and core.

N - Do we need traits?
For example in Rust:
```
fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}
```
We can use above to tell what types we accept in generic function.
No. this is a big change and does not provide much value.

N - When a module is main module vs. when it is dependency of another module, the role of `/` will change in imports.
Suppose I am compiling ModuleA which needs ModuleB which needs ModuleC.
will this be dir structure?
```
/ ---- http/www/github.com
			---- ModuleB
			---- ModuleC
	- src
		--- ModuleA
	- build
	- ...
```
this works fine with remote dependencies.
But when I use `import("/src/ModuleX")` inside source code for ModuleC, where should ModuleX be looked?
It is not remote and not external dependency.
One solution: Compiler will first try to find `/src/ModuleX` by mounting `/` to `/http/www/github.com/ModuleC/` directory.
If not found, it will look in higher levels.
So, when compiler is compiling dependencies of the main module, it will first try to find referenced modules within module source. If not found, it will check higher level. If not found will download to main module's structure.
But anyway, compiler will NOT look into main module's source for dependencies of sub modules.

Y - Can we remove `return` keyword?
Like Rust, say the last expression evaluated in the function will be returned

Y - Remove the rule that braces must be on their own line

N - How can we define a map with custom hasher?
easy solution: Core function
`x = createMap(int, string, fn(x:int->int) { x })`

Y - Idea: Provide hlper methods for array/map to return `x|nothing` type, where `a[100]` can exit process.

N - Pattern matching
For values, this is not very useful because we can easily use boolean with array or map.
can't we extend `//`?
In rust we can write:
```
let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
```
So, user can easily rely on union/sum types in the code.
what about us?
```
f = fopen("...") 
#now type of f is File|Identifier|nothing for example
number_inside_the_file = [
	File:		fn{freadInt(f)}, 
	Identifier: 	fn{convert(int, Identifier)}, 
	nothing: 	fn{0}]
[getType(f)]()
```
I think this is a good enough pattern matching.


N - Do we need some kind of static if in generic functions to check input type?
I think compiler can check them if a core function is called.

N - Trait as type
In rust you can define trait (similar to interface) and then say this function accepts any type which satisfies this trait (or for its return value).
Can we simulate this with generic types?
A specific generic type which has these functions.
For example, suppose that we have different animals (e.g. Sheep and Horse). And we want to have function `talk` for them.
```
trait Talker = { talk = fn(string->nothing) }
implement Talker for Horse as { ... }
implement Talker for Sheep as { ... }

process = fn(x:Talker ... )
```
In dotLang:
```
Talker = fn(T: type -> type) 
{
	Result = struct {
		data: T,
		talk: fn(string->nothing)
	}
	return Result
}
Sheep = struct {...}
Horse = struct {...}

process = (x: Talker(?) ...)

x = process(Talker(Sheep){.data = my_sheep, .talk = ...})
```
Or to make it easier:
```
Talker = fn(string->nothing)

Sheep = struct {...}
Horse = struct {...}

process = (x: Talker ...)

sheepTalker = fn(s: string -> nothing) { return talkSheep(my_sheep, s) }
x = process(Talker(Sheep){.data = my_sheep, .talk = ...})
```
I think we can do it via function pointers. 

N - Is the closure scope for in-struct functions ok?
Doesn't it cause confusion?
Go already has this:
```
type Rectangle struct {
    length, width int
}

func (r Rectangle) Area() int {
    return r.length * r.width
}
```
But in Go it is a method. Here we define the function within the struct. There is no self or this and we access struct members via closure.
Also D has this:
```
struct Foo
    {
        int a = 7;
        int bar() { return a; }
    }
    
    Foo f;
    writeln(f.bar());
```

Y - underscored identifiers are private in their context: function, struct or module.
these identifiers inside function, will not be in closure.
in module, will only be accessible to functions inside the module.
in struct will only be accessible to code inside struct, and so cannot be overwritten.
what if I want to have a struct member as public but not changeable?
This mostly applies for functions.

N - Early return
Scala does not have it.
In favor of no return: https://tpolecat.github.io/2014/05/09/return.html

N - We have a lot of different collections in Java. If we say map/reduce is a member of seq, how can we implement them efficiently?
Examples:
- treeset
- Hashset
- Linkedhashset
- Vector
- Stack
- PriorityQueue
We can define them as generic types that embed a seq/map and expose a map/filter function that calls them.

N - Scala like Perl, allows you to write lambda separate from function call parens.
```
def whileLoop(condition: => Boolean)(body: => Unit): Unit =
  if (condition) {
    body
    whileLoop(condition)(body)
  }

var i = 2

whileLoop (i > 0) {
  println(i)
  i -= 1
}  // prints 2 1
```
This can enable us to write if/for but in fact calling a normal function.
No this is confusing and we are not interested in tricky ways to write short code.

N - Should there be an easy way to import from other module into current module without needing to use dot?
e.g.
```
math = import("math"){}
PI = math.PI
#use PI as if it was defined internally, no need for dot.
```
Can we make above easier? Not needed.

N - An easy notation for number range can be useful for loops:
`forLoop([1..10], fn{ ... })`
Can be easily done via functions in core.

Y -  A better notation for casting?
Scala: `val recognizer = cm.lookup("recognizer").asInstanceOf[Recognizer]`
Go: `var z uint = uint(f)`
D: `MyType castedValue = cast(MyType) someValue;`
Rust: `let size: f64 = len(values) as f64;`
Swift: `case 0 as Int:`
`int_or_nothing_value = int{int_or_float}`
Usage:
- Primitive types: int to float
- Named types
- Union
`x = type(y)`
So, we can treat type name as a function to do the casting.
But this is not a real function. is it?
What is function input type? can I use it as a lambda?
we should not treat it as a function.
`x = type{value}` is good but a bit confusing. we are already using `{}` for code block, struct type and struct value.
why not use cast function?
it is actually a function. But to be so, we need to have 3 inputs:
`float_var = cast(int, float, int_var)`
This makes sense but is too much. Maybe we can fina an `auto-infer type` notation for compiler. but anyway, this is the right way to do it.
And cast will be a real function. But a generic function.

N - Can we have by-name parameters now?
now that there cannot be two functions with the same name?
Scala: `printName(first = "John", last = "Smith")`
q: What if arg name changes? Should client also change?
When should one use named arguments, and when not? Easily can lead to needless style-guide wars.
What about using untyped struct?
```
process = fn(x:int, y: struct{id: int,age: int} -> int) ...
...
process(10, _{.id=10, .age=20})
```
No. too much confusion and it can be achieved using other ways.

N - For generic functions like cast or map or filter, we have assumed they don't need type. Compiler will deduce that.
Can't we do this for all functions?
`x = map(int, int_array, fn(x:int -> int) { return x+1 })`
instead we can write:
`x = map(int_array, fn...)`
can't we use a special notation to say: Hey compiler! infer the needed generic type to use here.
and if compiler cannot, it will throw compile time error.
Easiest: Caller can ignore type arguments completely. Compiler will infer them.
This can be confusing. 
We can define map/fiter as member of seq or map so there will be no need for type args.

N - Can we have default parameter value?
Can't we use `|nothing`?
`process = (x:int, y:int|nothing -> ...) { actual_y = y //100 ... }`
when we can have it via existing constructs, why add something new?
the only draw back is that caller won't know what the actual value will be but this is also some kind of encapsulation.
This argument is optional so if you don't have a value for it, forget about it and don't care.

Y - Can untyped struct's fields have name?
`process = fn(x:int, y: struct{id: int,age: int} -> int) ...`
struct type can provide field type and possibly field name.
untyped struct must have field type but optional field name.

N - What about variadic functions?
can't we have them via sequence?

N - Can user create a function pointer using a built-in function?
What important built-in function do we have?
import -> not allowed obviously.
cast?
getType
getCurrentTask
set/get
It should be fine. We don't have any magical function.

Y - Import multiple modules at once
`{Socket, Stack, Queue} = import("/core/st/{socket, stack, queue}")`
We can use some core function to generate a string sequence
`{Socket, Stack, Queue} = import(multi("/core/st/", ["socket", "stack", "queue"]))`
But this is supposed to be compile time.
But above notation is wrong. You cannot destruct types. 
But we can have types inside a struct. and we can destruct a struct, 
What will happen when we destruct a struct? All its bindings and types will be exploded.
I think that makes sense.

Y - If a module writes `import("/src/ref")` and there are two ref modules in the directories under src, which one should compiler pick?
when using relative paths, there is no issue.
when path is absolute, compiler will try module's root path. Even if module is a dependency inside another module.
But what if the dependency is already downloaded by the parent module?
So, if path is absolute, compiler will try current module's root, if not found, it will try parent module's root and ...

Y - If a struct has bindings and types and they have no name, using `.0, .1` will give us both bindings and typs?
```
g = _{int, int, 10}
int_var = g.2
MyType = g.0
```
I think this makes sense. 

N - Do we need a notation that says "A gneric type of any type"?
For example: a function that accepts a Stack of any type.
But can't that be defined as a generic argument?
```
process = fn(x: Stack(?) ...
process = fn(T: type, x: Stack(T), ...
```

N - 
We can define a type/binding/lambda inside another lambda (function)
We can define a type/binding/lambda inside another struct
We can define a type/binding/lambda inside another module

Y - You can import at anywhere: at module level or at function level.
even in struct because struct is like a module

Y - Suppose that we want to have `seq` function to create a sequence of integers.
```
x = seq(10)
x = seq(0,10)
```
We want to have two definitions. But cannot because two functions cannot have the same name.
We can say: nothing arguments at the end of argument list which are omitted, will be passed as nothing.
```
seq = fn(start:int, end:int|nothing -> ...)
```
But this can change semantics of `start` argument. If end is present, it is start, if not present, it is length.
or for substring: `seq1.slice(10)` returns 10 first elements
`seq1.slice(1,5)` returns elements 1 to 5
```
seq = fn(start_or_length:int, end:int|nothing -> ...)
slice = fn(start: int, end: int|nothing -> ...)
```
Proposal:
- If function argument is `|nothing`, caller can omit it if it is last argument.

Y - If we see `T:int` in a module, how do we know if it is a type alias or a type argument which needs to get a value?
`T = struct { x:int }`
In above case, x is a field and must be initialised when creating an instance of type T/
`T = struct { X: type }`
If we see `T:int` it is a type alias because right hand side is a type identifier.
But `T: type` means it is a type argument
It is a bit confusing and difficult to read.
We should provide another notation for type alias so that it is fully different from module-level or struct-level arguments.
`T = int` is for named type
`T := int` no. is used for parallel computation.
What about using `T := int` for named type and `T = int` for type alias?
If we have a struct or module level argument, it cannot have value, so `T=int` does not cause confusion.
`Capital=Something` is a type alias
`Capital: type` is a type argument
`non_capital: int` is a binding argument
`non_capital = something` is a normal binding with value
`non_capital: Type = something` same as above, but with explicit type

N - If app usese A v1.0 and A uses B v1.0.
Later if B is updated to 1.1 but A is not.
How can main app force A to use newer version of B? It cannot and shouldn't.
This will work only if A was not strict when defining B dependency and said "I need any B version which is 1.*"

N - Allow selective destruction of struct elements.
Can be used to import some definitions from a module.
```
{a,b,c} = point.{data1, data2, data3}
```
`point.{data1, data2, data3}` is a new struct which only contains 3 fields from point. You can use it as is, or destruct it.
So:
`f1, f2, f3 = import("...."){}.{function1, function2, function3}`
`.{}` notation will give you bindings and can be applied on a struct binding
`..{}` notation will give you types and can be applied on a type
`{x,y} = point1`
But, this is only a shortcut, a syntax sugar. Not a necessary thing.
We can do it without this notation too.
I think this is too much new notation. We can live without it, and although it makes code smaller and simpler but the new notation will also increase complexity.
New things: `.{}`, `..{}` 

Y - A notation to ask compiler to infer type arg values for generic functions.
```
add = fn(T: type, x:T , y: T -> T) { ... }
result = add(int, int1_var, int2_var)
int_v = cast(float, int, float_v)
```
```
add = fn(T: type, x:T , y: T -> T) { ... }
result = add(!, int1_var, int2_var)
int_v = cast(!, !, float_v)

result = add($, int1_var, int2_var)
int_v = cast($, $, float_v)

result = add(::, int1_var, int2_var)
int_v = cast(::, ::, float_v)

result = add(?, int1_var, int2_var)
int_v = cast(?, ?, float_v)

result = add(auto, int1_var, int2_var)
int_v = cast(auto, auto, float_v)
```
We can use this notation and really treat import as a function. We import a module but output type is now known and compiler should infer that.
Leaving the place blank works too but is confuging:
```
result = add(, int1_var, int2_var)
int_v = cast(,, float_v)

result = add(__, int1_var, int2_var)
int_v = cast(__, __, float_v)
g = pop(?, my_int_stack)
g = pop(:, my_int_stack)
```
Or maybe an empty space:
```
g = pop( , my_int_stack)
result = add( , int1_var, int2_var)
int_v = cast( , , float_v)
```
As long as it is one or more whitespaces, it is fine.
It is easy to write, but makes reading a bit difficult.
```
g = pop(., my_int_stack)
result = add(.,.int1_var, int2_var)
int_v = cast(.,., float_v)
int_array = map(., int1, fn(x:int->int){x+1})
```
using dot is more readable.
So when calling a function, you can use dot in place of type arguments. And compiler will try to find a value for that argument.
another candidate: `;`
But still none of them are intuitive. dot is used elsewhere, blank is confusing, semicolo is odd.
Or maybe we can use a notation which can also replace `typeof` and `hasType`?
Like `$(x)` which returns type of x. 
```
g = pop($my_int_stack, my_int_stack)
result = add($int1_var,$int2_var, int1_var, int2_var)
int_v = cast($float_v,int, float_v)
int_array = map($[int], int1, fn(x:int->int){x+1})
```
Java uses `<>` diamond notation to say compiler should infer.
Candidates: `?, ::, @, $, !, *`
`*` is not good because it is also used for multiplication.
`@` is going to be used in import.
S we have `!, $, ?, ::`
`::` is confusing and difficult to type
`?` is also not relevant and counter intuitive
So we have `$` and `!`.
`!` means not so will be confusing.
So let's use `$`.
Proposal:
- When calling a generic function, you can use `$` for type arguments to indicate compiler should infer the type to be used.

Y - writing `import("/http/github.com/dotLang/stdlib/...")` all the time is difficult.
Maybe we can make it easier?
Maybe add a rule. just like http, absolute paths starting with std will be treated differently. 
But what if we later change path/url for std?
Even http rule is a bit unintuitive.
Write down a full app with 3-4 levels of dependency and figure out how dependency resolution will work.
In Go, when you import `/http/github.com/...` compiler does not download it. It just looks for it in local directory.
You have to download it first via `go get`. But that is a PITA, why not do it when compiling in a smart way?
We do not differentiate between module, package and library. We import a single module.
The problem is, a module can embed its dependencies in itself and make things complicated. And we cannot ban that because it will add an exception to the language and 
make it more difficult to learn/read.
When we import from absolute path, we should specify the border between library and directories. So that compiler can later find out root path for that dependency.
Lets just forget about web and http. Assume that we have some local modules we want to use. 
They are in `/usr/lib`. so no download from web.
But I cannot use them unless they are inside project dir.
So if project is in `/home/docker/project1` they should be copied or symlinked to project dir.
`ln -s /usr/lib/lib1 /home/docker/project1/vendor/lib1`
So now I will have access to `/vendor/lib1`.
One guiding principle is everything should be as dumb and simple as possible.
Simplification 1: No commit id, only `@tag_or_branch` notation, with support for `*` and `+` for semver. First we look for tag. If not found we will search for branch.
Simplification 2: Compiler knows root path for every module including master module (top most)
Rule: Every import with absolute path, if not found, will be downloaded into master module's root. So we won't have to download a dependency twice in two different places.
Should it be possible to have multiple modules (and versions) in a single GH repo?
We can save branch/tag name as a directory under main package dir.
In go, transitive dependencies will be saved under the main module.
q: What if we have a local server with zipped archives of dependencies?
q: What if we have a local directory with zipped archived of dependencies?
So, proposal:
1. When compiler tries to compile a dot file, it will keep track of the master root path which is root path of the top most project.
2. Compiler will also keep track of the current module being compiled (its root path).
3. For any import with absolute path, current module's path will be searched, if not found top most project will be searched.
4. If still not found, compiler will try to get from external source (based on the url prefix). If it points to a compressed file, compiler will decompress.
5. The dependency downloaded will always be saved into root path of top most project.
6. `*` and `+` and `@` have special meanings in url. `1+` means 1 and newer. `*` means any version. `@` is prefix for version or branch name.
7. If no `@` is used, compiler will assume `@master` and will just do a `git clone`
8. Examples of import:
`T = import("/https/github.com/uber/web/@v1.9+.*/request/parser")`
`T = import("/https/github.com/uber/web/@new_branch/request/parser")`
`T = import("/https/server.com/web/@v1.9+.*.zip/request/parser")`
`ref = import("/refs"){}`
`T = import(ref.uber_web + "/request/parser")`
Idea: we can create our own type which is based on string and add an import function to it.
```
ref = import("/refs"){}
ref.uber.import(
```
No. Import should be 100% compile time. This is confusing.

N - Can we use `$` also to get type of a union?
`t = tryCast($, int, int_or_float)`

Y - The `{}` notation in import is a bit counter intuitive.
Why not replace it with seq?
instead of:
`X = import("/dev/{a,b,c}")`
write:
`X = import("/dev/", ["a", "b", "c"])`
or `X = import("/dev", ["a", "b", "c"])`

N - it is possible to have multiple modules in one repo.
Just use master branch for the repo (or that can even be versioned).
Use dirs for modules and subdirs for versions.
`X = import("/http/github.com/lib/module1/@1.4+/dir/stack")`
`X = import("/http/github.com/lib/module2/@1.1+/dir2/helper")`
So `lib` is the repo and `module1` is one of modules inside that repo.
We then need to implement versions using directories there. Because git cannot tag per directory.
`git tag b/v0.1.1 ` You can use this to apply tag to a module withint repo.
maybe use `import("...@pack1/1.5+.*")`
This is already provided.

N - How to simplify import?
`X = import("/http/github.com/lib/module1/@1.4+/dir/stack")`
vs
```
ref = import("/refs"){}
X = import(ref.stack)
```

N - What is the reason Rust or Go keep track of extra files like Cargo.toml?
Basically, `refs.dot` can act just like Cargo.toml or gopkgs file.
They need `Cargo.lock` to pinpoint to a specific version (commit id) so that if later someone
adds/removes something from that branch or tag, no one has a problem.
But if we specify a tag, that's what we want. Latest information under that tag.
Also if you want to have specific version, just keep it locally so compiler won't download it.
So we replace `Cargo.toml` with import code in code files (+ optional use of refs file)
And we replace `cargo.lock` with ability to include source code inside the main code.
So no need 

N - Make import structure more formal.
Instead of a simple string, use a struct that has separate fields for protocol, version, tag, ...
```
stack_ref = Dependency
{
	protocol: "git",
    path: "https://github.com/a/b"
    version: "v.1.5+"
}
queue_ref = Dependency
{
	protocol: "zip",
    path: "https://server.local/path/a.zip"
    version: "v.1.5+"
}
```
But, what is the advantage? If we can include everything inside a string (protocol, path ,version, tag, ...)
why do we need to do this?
It definitely makes the code more readable, but also more difficult to import. Because
we will need to write more things.

N - Syntax sugars: 
- Nothing arguments at the end
- `//`

Y - Should we replace `getCurrentTask` with a keyword or notation?
Erlang `pid!message`
`msg = getCurrentTask().pick(Message, fn(m: Message -> bool) { m.sender == 12 })`
`int_result := process(10)`
`task_id = getCurrentTaskChildren().last()`
We can use `this`
`msg = this.pick(...)`
`task_id = this.newestChild()`
In continue of our efforts to make things simple, should we allow for so much flexibility here, when receiving a message?
`pick(SocketMessage, fn(m: SocketMessage -> bool) { m.sender = "192.168.1.1" })`
Btw, we don't even need a notation to refer to current process. We will use core functions for all of this.
`send(msg, target_id)`
`pick(...)`
Erlang uses pattern matching. But we have eliminated that. We have map and it can be used for union type matching.
we can rely on message type only:
`pick()` pick any message
`pick(SocketMessage)` pick any message of this type
`pick(SocketMessage, task_id)` pick any message of this type from this sender
For other types of logics, just do them inside the processing code. If the message is not to be processed you can put it back, or throw it away.
or keep it somewhere else.
Another idea: each process can setup a mailbox processor. This will process any incoming message. It can be used to tag messages.
In receive part, we can specify key value for tags to match. So we receive by filtering tags.
One more thing: We promote composition for threads. Many of common patterns can be implemented via a mediator process.
For example throttling or limiting size of mailbox, or load balancer.
Any addition to sender and message type will open a can of worm.
We can use a tag instead of sender. So it will give process more flexibility. 
They can use tag to show sender id, or any other type of data, or any combination.
Or maybe we can make tag a sequence of strings?
What if tag is "sender_id+priority" and I want to get all messages from a specific sender regardless of priority?
We can achieve this using regex. But it will mean complexity. How often do we need that?
What are big examples of concurrent systems?
- Web server: master + n-slaves
It would be helpful to allow process to have full control on what to send. 
For example we may want to keep sender pid when forwarding a message. If runtime automatically sets this, then it won't be possible.
So, if runtime does not set that, then on the receiver side, we cannot support filtering based on sender at language level.
If message is not what I wanted, I can pass it to a delay process to send it back to me after x seconds.
Assume a complex scenario like a distributed database: Processes can send files, lines, records, ...
What if we really need to know sender? For example, we listen to n processes, but one of them is high priority.
and it doesn't know. So they all send a normal `Message` structure.
The order is important. We should process messages in the incoming order.
What if replace message with a simple map?
Then we can even remove message type from receive function.
But what will be type of values in the map?
We may need some king of tag to track things.
Suppose that we only have two processes P1 and P2. P1 sends a lot of messages all of the same type to P2.
P2 will process and reply back. How can P1 know which response is for which request?
This does not seem to need support from language. You can add id field to message and support it from both P1 and P2.
so, proposal:
- In concurrency part, support only message type when receiving a message.

N - Can we use `type` in union type?
I don't think so.
`T = type | int`

N - Maybe we should still use some more keywords just like the way we used `fn` and `import` and ...

N - Assume `[nothing:...]` can we know if this is a map type or literal?
We can use the left side of `=`. If it starts with capital it is a type otherwise a value.

N - Write a sample code for reading something from a file.
Is file pointer mutable?
```
f = fopen("a.txt")
line = fReadLine(f)
fClose(f)
```
or
```
f := fopen("a.txt")
line = fReadLine(f)
```

Y - Are we giving enough tools to developer to implement below patterns easily?
- VTable for polymorphism
what is a vtable?
it is a list of functions with the same signature, each bound to a specific type.
When we invoke vtable, we pass arguments common for all functions. and we get the result.
So basically this is a map of type to function pointer.
But those function pointers, are almost the same except for one argument type.
e.g. `draw` type is Circle, we have `Circle, Color, Canvas`
but if type is `Square` we have `Square, Color, Canvas`
Now we can make this super flexible: any type of function, custom logic to decide dispatch, ...
But how much flexible this should be?
If we put a restriction on this (one argument type should differ or ...) it is against orth.
If we allow maximum flexibility, again it might be difficult to use in practice. 
The usual example: We have Circle, Square and Triangle types.
we have draw for all of them.
we want to have a single draw function for all of them which delegates to specialised draw functions (drawCircle, drawSquare, ...)
```
Circle = struct {...}
Square = struct {...}
Triangle = struct {...}
drawCircle, drawSquare, drawTriangle ... = fn(item: Circle/Triangle/Square, c: Color, n: Canvas -> ...)

vtable = [Circle: drawCircle, triangle: drawTriangle, square: drawSquare]
draw = fn(T: type, x:T, c: Color, n: Canvas -> nothing) { vtable[typeof(T)](x, c, n) }

```
or another way:
```
drawCircle = fn(s: Circle, Canvas, float -> int) {...}
drawSquare = fn(s: Square, Canvas, float -> int) {...}

getDraw = fn(T: type, x: T -> fn(Canvas, float -> int)) 
{
	vtable = [Circle : fn{drawCircle(x, _, _)},
                Square: fn{drawSquare(x, _, _)}]
                
    vtable[T]()
}
f = getDraw(Circle, my_circle)()
```

N - Can we make it simpler?
Most complicated notations in terms of learning the language or reading the code:
- `$`
- `..`
- `:=`

N - This example is not correct
```
drawCircle = fn(s: Circle, Canvas, float -> int) {...}
drawSquare = fn(s: Square, Canvas, float -> int) {...}

vtable = {.T = Circle, .func = drawCircle, next: {.T = Square, .func=drawSquare}}

getDraw = fn(T: type, x: T -> fn(Canvas, float -> int)) 
{
	
	vtable = [Circle : drawCircle, Square: drawSquare]
                
    vtable[T](x, _, _)
}
f = getDraw(Circle, my_circle)()
```
One way to solve this is to have a core function which guarantees no compilation for non-matching cases.
So if we call above function with a Circle, the `Square: fn{drawSquare(x,_,_)}` will not even compile.
```
getDraw = fn(T: type, x: T -> fn(Canvas, float -> int)) 
{
	
	vtable = [Circle : drawCircle, Square: drawSquare]
                
    vtable[T](x, _, _)
}
f = getDraw(Circle, my_circle)()
```
This is much better. The only issue: `vtable[T]` will give you a union of two fns. Because that is the type of map values.
Maybe we can cast.
```
getDraw = fn(T: type, x: T -> fn(Canvas, float -> int)) 
{
	
	vtable = [Circle : drawCircle, Square: drawSquare]
                
    cast(fn(Circle, Canvas, float)|fn(Square, Canvasm float), fn(T, Canvas, float), vtable[T])(x, _, _)
}
f = getDraw(Circle, my_circle)()
```
Or simpler:
```
getDraw = fn(T: type, x: T -> fn(Canvas, float -> int)) 
{
	
	vtable = [Circle : drawCircle, Square: drawSquare]
                
    cast($, fn(T, Canvas, float), vtable[T])(x, _, _)
}
f = getDraw(Circle, my_circle)(my_canvas, 1.52)
```


N - Can we call a generic function with a union type?
if we have `fn(int->int)` we can call it with only int.
But if it is `fn(int|string->int)` we can call it with int or string or `int|string`.
but if it is: `process = (T: type, x: T->int)`
can I call it with `int|string`?
`int_var = process(type(int_or_string_var), int_or_string_var)`
or:
`getDraw(type(my_shape), my_shape)`
Suppose that we have:
`process = (T: type, x: T -> ...) ...`
and we have this union type:
`Shape = A | B | C | D | E`
and `my_shape` is of type `Shape`.
Clearly, process is generic and can accept any type.
Technically, I can call process with a series of if/switch commands (implemented via functions):
if type is A then call `process(A, cast(Shape, A, my_shape))`
if type is B then call `process(B, cast(Shape, B, my_shape))` 
etc.
But can this be simplified?
In any case, compiler will have to generate code for all possible case types for `Shape`.
Maybe we can add a syntax sugar so that: `process(typeof(my_shape), my_shape)` can then be automatically handled via compiler.
or even simpler: `process($, my_shape)`.
But what if we really want to call process with `Shape` type and not `A, B, C, ...` types?
This can be handy many times but makes the language inflexible in some cases.
Because compiler is making an assumption and doing some stuff behind the scene, that can sometimes be wrong.
Also we are trying to avoid "behind the scene" things.
so if I cann `process($, my_shape)` the method will be called with `Shape` type.
If I want to call with detailed inner type, I 

N - Now that we use `struct` for struct, can we use `union` keyword for union?
```
Payload = union {
    Int: i64,
    Float: f64,
    Bool: bool,
}
var payload = Payload{ .Int = 1234 };
payload.Float = 12.34;
```
This may also help with doing stuff like C unions where you map type A and B so that you have access to internals of A.
q: How does this affect Polymorphism? switch? lambda definition?
q: readability of an expression?
q: union switych?
q: enums?
q: can we have multiple fields of the same type but different names? is this good? This will be a suprtset of current unions.
q: how does it affect function/binding resolution?
q: can an imported module be a union instead of struct?
```
MyType = union { x:int, y:int, z: float }
DayOfWeek = union { sat, sun, mon, tue, wed, thu, fri }
t: MyType = MyType{.x = 100} #only x has a value, init same as struct
flt = t.z #you can read anything from a union -> no rule/exception here
#to see which field was set, you can use this notation
h = t.z? #returns true if z is set
h = t..z
h = t::z
```
This will give us both flexibility of C union and an enum feature and sum type of Haskell.
It would be good if we can mix this with a map which can be used in place of switch.
So that we do not need to add a new notation/keyword.
q: can we have lambdas in a union? How will it be treated?
q: if we allow access to any of union fields, what if union has lambdas. if I set lambda1 and access lambda2 and call it, what will happen?
q: can we say if type is not specified, it is `nothing`? everywhere.
maybe we can add a core function to return index of the field which is set.
q: If I define a struct, inside which there is a lambda and a union, how will that lambda access union fields? Is it still closure? what if union is unnamed?
q: Can I define unnamed struct/union inside each other?
q: How does this affect `//`? Can we make use of it for unions?
maybe we can say, each field inside a union is `T|nothing`. Then we can use `//` to write expressions for each field.
```
MyType = unin {int_var: int, int_var2: int, float_var: float}
data = my_type.int_var // my_type.int_var2 // toInt(my_type.float_var)
```
Allowing fields to overlay in memory is not a very good idea. and not very useful. and can be confusing.
but this will be a rule/exception on itself: You can NOT read from all fields in the union except the field which is initialised.
q: How can we destruct a union?
q: can union destruction be used for matching?
Examples:
```rust
enum OptionalTuple {
    Value(i32, i32, i32),
    Missing,
}

let x = OptionalTuple::Value(5, -2, 3);

match x {
    OptionalTuple::Value(..) => println!("Got a tuple!"),
    OptionalTuple::Missing => println!("No such luck."),
}
```
```swift
directionToHead = .south
switch directionToHead {
case .north:
    print("Lots of planets have a north")
```
```kotlin
val directoryType = UnixFileType.D
 
    val objectType = when (directoryType) {
        UnixFileType.D -> "d"
        UnixFileType.HYPHEN_MINUS -> "-"
        UnixFileType.L -> "l"
    }
```
```scala
notification: Notification
notification match {
    case Email(email, title, _) =>
      s"You got an email from $email with title: $title"
    case SMS(number, message) =>
      s"You got an SMS from $number! Message: $message"
    case VoiceRecording(name, link) =>
      s"you received a Voice Recording from $name! Click the link to hear it: $link"
  }
```
Problem is, I don't want to add a new nested level.
another option: `union1.unwrap()` will give you internal type. But this is not good in a statically typed language.
q: Isn't allowing users to write to any or all fields of a union, allowing them to mis-use the language? shouldn't we limit this? But how exactly can they mis-use it?
```
U = union { i: int, f: float }
u = U{.i=100}
data = ifElse(u::i, u.i, nothing) // toInt(u.f) #no. not scalable to many types in union
```
How can we support reading both data? suppose that the union has int and string. if we write to string, reading from int, will give us what?
the first 8 bytes? 
```
U = union { i: int, f: float }
u = U{.i=100}
str_var = match(u) {
    fn(i:int -> string) { "A" }
    fn(f:float -> string) { B" }
}
```
less readable but:
```
U = union { i: int, f: float }
u = U{.i=100}
str_var = index(u) [ 0: fn{"i is set"}, 1: fn{"f is set"} ]()
}
```
Also we can delegate this to the developer:
```
S = struct { index: int, union { i: int, f: float } }
my_s: S = S{.index=0, .i = 100}
```
Not a good idea. dotLang should be simple.
we do not want to indent. So no lambdas.
no match/when/case/switch keyword.
q: Do rules about function input/output in presense of union types still hold?
    if a function accepts int and works fine, later if it is changed to `union{int, string}` does caller need to change anything?
Maybe its not a good idea after all.
1. It makes notation more complicated. Matching will be needed and either we have to use something like `unwrap` or `getIndex` or a new indentation.
2. Makes union like a new type notation while it is not a new notation. It is simple combination of other types with "OR".
3. If I define a fn that takes int and call it via `f(5)` then later change it to `union {x:int, y:int}` then which one is set? will caller need to change their code? this is not good. even if I use `union{x:int, f:float}` the call syntax should change or else I should have two completely different ways to initialise a union.
can we provide a better notation for the current union?
`has_int = hasType(int, int_or_float), int_or_nothing_value = tryCast(int|float, int, int_or_float)`
we can have core function `typeof` which works only on unions and will return type id.
we can mix it with map.
But we said, anything related to type must be compile time.


N - Optional types.
If we have `process` inside function with 2 args int and string and a process in module level with the same name but int, string, float or nothing input
and we call process with an int and string, which one will be called?
Of course the local one. because it is in a nearer scope and this is in line with resolution that we have.

N - The fact that everything is inside a struct, doesn't it make us OOP? or less functional?

Y - Should we be providing more for unions?
We can provide a core function `match`
```c
x: int|string|float = ...
match(int|string|float, string, x, 
    fn(i:int -> string) { ...}, 
    fn(f: float -> string) { ...}, 
    fn(s:string->string) { ...}
)
match($, $, x, 
    fn(i:int -> string) { ...}, 
    fn(f: float -> string) { ...}, 
    fn(s:string->string) { ...}
)
```
No additional indentation but instead we have a function call.
But match is a special function. for example, you cannot pass a pointer to match.
this is because match is generic. you cannot pass a ptr to any generic because they need type.
You can however pass ptr to match with specific types.
So it will be:
`match = fn(U: type, Target: type, x: U, fn(?->Target) ...`
similar to this but not completely. Compiler will also help.
So match is a syntax sugar? or a real function?
why not use `//`?
```
x: int|string|float = ...
result = check($,$,$, x, fn(i:int -> string) { ... }) //
         check($,$,$, x, fn(s: string -> string) {...}) //
         check($,$,$, x, fn(f:float->string){...})
```
so check is:
`check = fn(Type: type, Remain: type, Target: type, x: Type|Remain, logic: fn(Type->Target) -> Target|nothing) {...}`
Proposal: Indicate that we have check function that can be used with unions for type matching.

Y - It is a bit weird to write `$` in function call.
In java we write `<>` and it looks good because java separates generic type args from normal args.
canwe handle this with optional arguments? mark it as `type|nothing` and in fn body write: `Type1=type//core_infere` and compiler will handle it properly
but this means type should be at the end.
```
check = fn(x: Type|Remain, logic: fn(Type->Target),
		Type: type|nothing, Remain: type|nothing, Target: type|nothing -> Target|nothing) 
{
    Type2 =  Type // $
    Remain2 = Remain // $
    Target2 = Target // $
}
```
But optional argument means if caller did not provide it, the function will.
But what happens to function title/prototype?
we can introduce optional type arguments which appear at the end of arg list and are `type|$` so if caller did not include it, compiler will provide value.
But this is not a real union type. 
So if we want to introduce a new notation, why ruin/change meaning of an existing one?
we can write `T: type//$` to tell compiler that if T was not provided, infer it. should they be at the end in this case?
`check = fn(Type: type//$, Remain: type//$, Target: type//$, x: Type|Remain, logic: fn(Type->Target) -> Target|nothing) {...}`
question is: can we replace `$` with something else? for example int.
We should be able to do that. But if the caller sends strings and ignores type, then compiler will complain.
This should be fine because no functions can have same name. So just by writing function name, compiler can deduce all these.
can this be source of ambiguity? 
`check = fn(T: type//$, S: type, f: int|nothing)`
what about this?
```
check = fn(x: Type//$|Remain//$, logic: fn(Type//$->Target//$), 
    Type: type|nothing, Remain: type|nothing, Target: type|nothing -> Target|nothing) {...}
```
It is compatible with `|nothing` notation. We `//` with inferred type within function decl. so no confusion about when this should happen.
we are re-using union type concept and optional argument concept. so more orth.
But OTOH, if we use the new `//$` notation:
A. `check = fn(data: T, S: type, f: int|nothing, T: type//$)`
B. `check = fn(data: T//$, S: type, f:int|nothing, T: type|nothing)`
`//`is not applicable to types but we are using it. 
B. `//` is allowed for types too but with same semantic: if it is nothing, then use right side of `//`
What if we really want to have generic with nothing type? then we pass other parameters accordingly so that compiler will infer nothing.
so `T//$` will become `T//nothing` which will become `nothing//nothing` which will become `nothing`.
But, what is the difference between `type//$` and `T//$`?
the first one, `type` is not nothing. It is completely different thing. And this can be source of confusion.
what we mean by `type//$` is if type is missing then infer. But missing type is only acceptable if argument is optional which is `|nothing`.
So I think B is more compatible with other rules and concepts that we have.
`check = fn(data: T//$, S: type, f:int|nothing, T: type|nothing)`
`push = fn(data: T//$, stack: Stack(T//$), T: type|nothing -> Stack(T//$)){...}`
We have to use `T//$` everywhere and when calling push: 
`result = push(10, int_stack)`
can we simplify and eliminate need to use `T//$` multiple times?
`push = fn(data: S, stack: Stack(S), S: T//$|nothing, T: type|nothing -> Stack(T//$)){...}`
it will be a bit confusing and counter intuitive.
and maybe wrong: What does `S: T//$|nothing` mean? is T an argument? If so, how can caller provide a value for it?
It is an argument of type `T`. If S is missing value, it will be inferred. no. too confusing.
`check = fn(data: T//$, S: type, f:int|nothing, T: type|nothing)`
`push = fn(data: T//$, stack: Stack(T//$), T: type|nothing -> Stack(T//$)){...}`
So, proposal:
1. When defining a function, you can use `type|nothing` to indicate an optional generic type.
2. When using an optional generic type, you can say if that is missing, you want compiler to infer appropriate type.
3. So you should use `T//$` to indicate if T (an optional generic type) was missing, then compiler should infer type.
can we then remove `$` notation alltogether? why not use convention?
optional generic type arguments will be implied by compiler.
`push = fn(data: T, stack: Stack(T), T: type|nothing -> Stack(T)){...}`
Now if we explicitly pass `nothing` as T it is fine. But if we omit that, compiler will infer.
We don't need to write `S = T // $`.
Just add a simple rule:
- Optional generic type arguments will be infered using compiler if not specified.
So the concept of optional argumetns was one arrow for two targets:
- Make it easier to have optional arguments
- Provide generic type inference

N - How are we going to do console I/O?
print something or read something?
For core functions, you do not need prefix.
```
write("Hello")
result = readLine()
```
For network:
```
socket_task = net("192.168.1.1")
send(socket_task, "A")
receive(SocketMessage)
```
But how are we going to receive data from a socket?
We can subscribe.
```
socket_task = net("192.168.1.1", fn(data: SocketMessage->nothing) { ...} )
```
Or we can simply tell the socket our own task id to it will send anything received.
```
socket_task = net("192.168.1.1", my_task_id)
receive(SocketMessage)
```
But I think having sockets as process/task is too much load. Some systems can have up to 1M concurrent connections.
If we save them all as task, it will be difficult to manage and slow.
No problem. We can use what we use with map/sequence update.
Actually maybe we can treat socket like a linked list. We read and traverse until we reach the end.
Same for file. Actually socket is just a file where we read and write.
They are the same concept (ipc, socket, file, device, directory, ...).
So we have a handle. We send data or we get data.
we simply use: `write(socket1, "A")` or `read(socket1, int)`
but, what happens to the original socket?
and what if I have a sequence of 10K sockets and send on one/some of them?
```
s = createSocketAndBind("0.0.0.0", 8080, IPV4, 2, 
	fn(c: Connection -> ) { c.write("Hello") }
    
)
conn = accept(s)
conn.send("Hello")
#client side
s = socketAndConnect("172.16.1.1", 8080, ...)
int_var = s.read(int)
s.write("Hello")
```
We can have one task for multiple connections/sockets.
A server usually has one (or a small number of) socket and that socket has a lot of connection.
Here (https://medium.freecodecamp.org/million-websockets-and-go-cc58418460bb) also they say they use goroutines for connections.
We can have a special task which has a linked list of sockets/connections.
```
socket_task = newTask...
send(socket_task, NewSocket{my_id, ...})
socket_id = receive(SocketCreated)
socket_task.send(socket_id, "Hello")
socket_task.wait(socket_id, ...)
socket_task.bind(socket_id, ...)
socket_task.listen(socket_id, ...)
```
All options are possible:
- Core functions
- Tasks: one task per socket
- Tasks: one task per multiple socket
Let's deal with this later.

N - Do we need to/can we implement stream concept of Java here?
We can do it via a task. 


N - import creates a binding, `Import` creates a type?
I think it is too much. We can easily have binding with type. 

N - links to read
idea: cmu fox project
lambda th eultimate: why do we need modules at all
5 mistaked in PL desig

Y - remove xor keyword

N - How can I easily import multiple bindings/types from a module?
Support `.{}` notation to create a subset struct:
```
Point = {x: int, y:int, z:int, f: float}
p = Point{...}
x = p.{x,y}
x = _{p.x, p.y}
{a.b} = p.{x,y}
```
Advantage: can be used in import
```
assert = import("std"){}.debug.{assert}
```
But is it really needed? Can't we just use normal assignment?
```
T = import("std")
t = import("std"}{}
assert = t.assert
trace = t.trace
MyType = T..MyType1
```
But it is a big too much. we will be having these statements a lot and there should be a way to simplify/minimalise them.
```
t = import("std"){}.{assert, trace}
```
We need a short notation to refer to bindings/types inside a struct type or value.
If we do that, we will have multiple values or types. How can we use them?
The only existing way is destruction.
`{x,y} = point1`
`{name1, name2, name3} = import("A"){}.{n1, n2, n3}`
`{Type1, Type2, Type3} = import("A")..{T1, T2, T3}`
**Proposal:**
1. A new notation `A.{B}` where A is a struct binding and B is a list of field names withing A
2. `A..{B}` where A is a struct type and B is a list of types within A
Above two will give you a new struct binding/type that you can use directly or destruct.
`{assert, trace} = import("std"){}.{assert, trace}`
Why do we use `..` to refer to types?
Initially we wanted to avoid confusion. because we had everything inside structs.
Reminder:
- With type we only have access to inner types
- With value we only have access to inner values
**Proposal**
1. Introduce a new notation `A.{B}` where A is a struct binding and B is a list of field or type names inside A.
2. Remove `..` notation.

Y - Why import gives us type?
- When using struct type, you have access to inner types and bindings that have literal value. 
- When using a struct value, you have access to everything defined inside the struct (types, bindings, ...).
why do we need type when importing?
maybe we have a fn with input of type T where T is a module.
we can have a notation (compile time) which gives type of a binding.
it is easy with generics and optional generic type:
```
getType = (x:T , T: type|nothing -> type) { T }
process = (data: getType(imported_module))
```
this works fine and is consistent with all semantics that we have.
I think it is exceptional that we need type of an imported module but that is possible anyway.
So, proposal:
- import gives you instantiated type of a module.
- If module needs input (has bindings without value like `x:int`) you can use `{}` notation to set their values.
but above is conflicting.
what about banning bindings without value at module level?
they are not really needed. The only use case: have functions at module level with a common argument at module level initialised from outside.
But you can easily add them to fn args.
**Proposal**:
1. import gives you struct value
2. module level bindings must have value.
Ask zig website: Can we init a struct without setting value for a field? Can we have types inside module?
How to access a type defined inside imported module?
q: Suppose that I have a struct type. How can I access its types? use dot

N - We can have `this` or `self` by using optional arguments in struct methods
```
Customer = struct {
	age: int,
    print = fn(this: Customer|nothing) {
        
    }
}
```
But what is use of this, if we allow closure?
Dlang has this type of closure.
Now we can scrap closure alltogether and use this. 
but what about normal closure inside a function?
https://michaelfeathers.silvrback.com/variable-capture-considered-harmful
Closure makes reading code difficult because a function uses a binding but we don't know where it is. 
We have to look at the parent fn then parent struct then parent module, ... to find its source
"Locality is one of the most important conditions for understandability in code."
When do we need closure?
- Lambda defined inside a fn (lambda)
- Lambda defined inside a struct
- Lambda defined inside a module
Also we may need to access parent module while inside a struct
and this can have many/multiple levels.
module -> struct -> struct -> function -> function: needs access X defined at module level
one way: define keywords to access current struct, fn, module. But this won't help because we may have multiple struct/fn in context
another way: manually implement this or any other pointer we need to a parent closure
In zig:
```
fn List(comptime T: type) type {
    return struct {
        const Self = @This();

        items: []T,

        fn length(self: Self) usize {
            return self.items.len;
        }
    };
}
```
we can use a keyword like `this` to point to the "current" closure (fn, struct, module, ...) depending on where it is defined.
is it a function? binding? type? keyword?
```
T = struct { x:int, t: this, process = fn(->){t.x}
```
When do we need closure?
1 - Lambda defined inside a fn (lambda): just pass the data it needs and set other args to `_`
2 - Lambda defined inside a struct. We need to access parent struct fields
3 - Lambda defined inside a module
```
Record = struct 
{ 
    x:int, 
    process = fn(holder: Record|nothing->int)
    {
        self = holder // this
        self.x+1
    }
}
my_t.process()
```
But what about calling methods in the current module? Do we still need to use some prefix?
e.g. `this.module`
And if we allow `this` keyword, people will stop using `holder:Record|nothing` and directly use this.
Then it will be more and more OOP.
Anyway, `this` is exactly same as closure. The only difference is notation and being explicit/clear.
And the fact that there are "TWO" ways to call these functions is also confusing. So better avoid it.
Let optional arguments be there when they are really optional.
And the only exception for now is generics with type inference.
We can say:
- Lambda does not have access to parent functions' bindings.
- Lambda (or module level or struct level functions) have access to their parent data structure (struct/module) bindings/types

Y - Do we need to simplify structs?
https://github.com/ziglang/zig/issues/1250
If we ban value setting inside a struct, we should do the same inside a module.
or else, module cannot be struct.
no confusion about where to define functions.
Do we really need encapsulation and `_`? If not, then anything can access any binding inside a struct.
we can remove the `_` rule and just use it as a convention. fields or methods starting with `-` are private so do not rely on them.
what are structs?
- C structs
- namespace
- C++ class
I prefer first option: structs be C structs.
so that we move away from OOP as much as possible.
So, the only place to define a lambda is inside a module or inside another lambda.
But:
q: what will a module be? it will be a new thing. Not a type, not a struct, not a binding.
q: what is output of import function? A module.
q: How can we access another module's types and bindings? Shall we use dot notation?
Treating module like a struct has its own advantages but is a bit confusing. Everything can be defined everywhere!
Let's say: module/ns can have bindings with compile-time decideable values and types.
struct: can have binding definitions without any value (No types).
we want to avoid confusion because it leads to ambiguity and different ways of doing one thing which makes code difficult to read and maintain.
Now we can have functions defined inside struct or at the module level `->` two ways to define functionality.
If we only allow lambda at module level, this will be reduces.
q: What about aliasing? no aliasing
q: So it will not be allowed to define a type inside a struct? no,
q: Can we still define a type inside a function?
```
#dlang
import std.stdio; 
//rename import
import io = std.stdio;
std.stdio.writeln("hello!");
io.writeln("hello!");
//selective import
import std.stdio : writeln, foo = write;
there is also scoped import where import only works inside a fn
#kotlin
import foo.Bar
import foo.* 
import bar.Bar as bBar 
#rust
mod my_mod {...}
my_mod::function();
extern crate deeply;
use deeply::nested::{
    my_first_function,
    my_second_function,
    AndATraitType
};
my_first_function();
use deeply::nested::function as other_function;
#swift
import moduleName
import func Pentathlon.swim
import func Darwin.fputs
import var Darwin.stderr
#c++
MyNamespace::MyClass* pClass = new MyNamespace::MyClass();
using namespace MyNamespace;
#zig
const warn = @import("std").debug.warn;

```
Namespace is too generic and gives us impression of being nested.
Why not use module?
```
stack_module = import("/core/std/Stack")
T = stack_module::Type1
f1 = stack_module::processFunction
import("/core/std/Stack") #import into current module
selective_module = import("/core/std/Stack")::{Type1, func1, binding3}
selective_module::Type1
selective_module::func1
selective_module::binding3
```
Look at the questions people have asked in SO for import in Rust, Swift, ... and try to prevent those type of questions
- Allow import inside a function
- We can import into current module, named import and selective named import
Why do we need selective import? Because in case we want to import inside current module it can prevent name collission.
But then, do not import into current module. Just import with name.
Can we simplify `::` notation? maybe use `..`
so, lets remove selective and rename import. Always import into a name. 
and you can import anywhere there are braces: module level, fn level, struct level and it will be valid inside braces or module level
**Proposal:**
- import: `T = import("/core/std/Stack")`, no rename, no selective, no nested modules, name of import result using PascalCase
- use: `MyType = T.Type1` to access identifiers inside a module
- import can be in a module or inside a function
- struct: only a list of bindings without value assignment
- You can not import inside a struct and cannot define a type (struct is just a dumb list of fields)
- Naming: Like a type 
What happens if I import inside a struct and use a type from imported module? Then anyone using that struct should have access to that module too.
Can we expose our imports? For example: `T = import("A")` then `f1 = T..m1..m2` yes why not? we can expose elements inside imported module but not module itself
No this will be confusing because we will have to study inside m1 to see what is m2 and then study inside another module.
q: What should be naming of a module? should it have a prefix? it is definitely not a fn. type or binding? it will be used a lot so lets go with binding name.
q: can we import multiple modules? `a,b = import("/core/{a,b}")`
Another option is to use dot for modules too.
Java does that: `java.util.LinkedList`
dlang does that: `import D; D.foo()`
golang does that: `import "fmt"; "fmt.Println...`
But it kind of looks not ok. Now that structs have only fields, if we see dot, before it we definitely have a binding.
We can differentiate that by using PascalCase for imported modules.
So, if I se `MyMy.` I know this is a module.
but `my_my.` is a binding of struct type.

N - idea: for generic type: inference dont use nothing but use a new keyword

N - idea: fn inside struct, can we allow it to have access to parent struct?
no. because we assume functions and structs are totally different. so giving struct access to a function makes things difficult.

N - You cannot define a type inside a function or struct. only at module level.

N - What happens for `x= int_nothing // float_nothing // string_int_nothing`? 
what will be output type?

Y - Remove semantics from `_`. Make it a convention

N - What is we have nested structs? then we need dot notation for them.
```
Customer = struct { name: string, age: int, address: struct {postcode: string, street: string}}
c = Customer{...}
c.address.postcode
```
No. still you cannot define named type inside a struct.
So, you won't need to access the nested struct.

N - Can we make sure grammar is LL(1)?

N - Each element/concept must have one simple well defined mission.
e.g. struct/module/function/type/...
maybe in this way, functions that create types is not a very good concept
if this is not the case, people will be confused, ask questions, implement in multiple different/wrong methods, ...
```
LinkedList = fn(T: type -> type) 
{
	Node = struct {
		data: T,
		next: Node
	}
	Node
}
LinkedListNode = struct {
	data: T,
	next: LinkedListNode|nothing
}
LinkedList = LinkedListNode | nothing
```
Having types that accept a generic argument is more a change than having functions that accept a generic type argument.

N - How else can we use optionals to make language more expressive?
It is generally not good for one concept to have multiple meanings.

Y - generic type arguments are all optional if they are at the end, no need for `nothing`.
why would writer of generic function want force caller to specify generic type?
This should be like module level bindings: type is optional.
Same for generic type arguments: type is always optional (if at the end).
So no need to write `type|nothing`.
what if it is mixed with optional args?
it should be fine. `process = (x:T, y:float|nothing, T: type)` 
then you call `process(10)` it will be called as `process(10, nothing, int)`
**Proposal**
1. Generic type arguments are inferred by compiler if they are at tht ene and omitted

Y -  things that look different should work differently
About `=` and `:=`?
Now that we have removed module level bindings without value, I think we can restore `:` notation for type alias
`T : int` to define type alias
`T = int` to define named type
idea: `=` for binding, `:=` for concurrency
`<-` for type alias
`<=` for named type
`X <- int` type alias
`X <= int` named type
now: `=` binding and named type, `:=` concurrency, `:` type alias and fn arg and struct field
I think `:` and `=` make enough sense
**Proposal**
1. Use `:` for type alias and `=` for named type

N - How can we make lambda out of a function which has no input?
just use function name or assign it
`process = fn(->int) { 5 }`
`...x = process`
because functions ARE lambdas


N - Can we move away from using lots of braces?
`ValueKeeper = fn(T: type -> type) { struct {data: T} }`
idea: `fn` and `endfn`
`ValueKeeper = fn(T: type -> type) struct {data: T} endfn`
idea: use binding name in the output
`ValueKeeper = fn(T: type -> R: type) R = struct {data: T}`
function ends when that binding is assigned a value
idea: julia:
```
function f(x,y)
           x + y
       end
```
option1: end
`ValueKeeper = fn(T: type -> type) struct {data: T} end`
option2: assign output
`ValueKeeper = fn(T: type -> R: type) R = struct {data: T}`
con of option2: it makes it difficult to write `fn{...}` notation.
```
temp = [x>0 : fn{ saveLargeFileToDB("SDSDASDA") }, 
	x<=0: fn(x:int->string) { innerProcess(x) },  ]

	temp[true]()
```

N - can we make closure more explicit?
In C++ you have to explicitly specify which vars to capture in lamnbda.
Proposal for Julia: https://github.com/JuliaLang/julia/issues/14959
```
process = fn(x:int->int) {
	y=f1(x)
    z = fn{
        g = f2(x) #x?
        g
    }
}
```
C++ uses this notation:
`[&] (const string& addr) { return addr.find( name ) != string::npos; }`
Let it be free like other languages. Adding more notation will be more confusing.

N - Do we support specifying name for arguments?
no. What if arg name changes? Should client also change?

N - Make notation to invalidate a binding more explicit
`dispose(name)` is not explicit enough

N - Do we need to support a group of modules. either importing multiple modules or having a package concepts?
In java you have to import them separately. Also we provide for group import.
Idea: Allow to provide multiple strings to import so we can import base part from Refs module and provide the rest.
e.g.
```
Refs = import("refs")
stack = import(Refs.base_import, "/utils/Stack")
```
What about string concat?
**Proposal**
1. State that `+` can be used to concatenate string or sequences

N - A fn that accepts any fn and logs before/after calling it. can we have this?
We can say our logger accepts a function with no input so caller can lambda it
```
log = fn(f: fn(->T), msg: string, T: type) {
    write(msg)
    result = f()
    write("done")
    result
}
```
But what if F has inputs? they will be passed by caller
```
z_result = log(fn{processData(x,y,z)}, "Hello")
```

N - We use `_` for multiple purposes:
1. type for untyped structs: `x = _{10,20}`
2. create lambda: `process = processFunc(10,_,_)`
3. destruc to ignore items in assignments `{x,_} = my_data` or `_ := process()`
`_` should mean, I don't know or I don't care.
so for lambda creation maybe we should pick another symbol or keyword.
`x = process(10,?`
`x = fn(x:int ->`
I think using `_` for assignment and lambda is ok.
But for struct, we should find something else.
scala: `val p = scala.math.pow(_, _)`
this is divided into two points

N - Can we eliminate `_` when creating a lambda?
`process = f(int, int, int -> string) ...`
`p = fn(x:int->string) { process(10, x, 20) }`
We need a shortcut for p, right now we have `p = process(10, _, 20)`
`process(10, _, 20)` vs `fn(x:int->string) { process(10, x, 20) }`
obviously this is a syntax sugar because right side is completely possible to do.
can we do this via generics?
`lambda = fn(f: fn(T->O),`
or maybe we can extend `fn{}` notation:
`fn(x){process(10, x, 20)}` so that compiler will infer type for x from process call.
`fn(x,y){process(10, x, 20, y)}`
won't this be difficult to read?
and what if inside `{}` is a big block?
`fn(x,y){x=process(10);h=save(x,y,10); ...}`
this can be mis-used because we have braces.
Let's keep this.

N - Why it must be a type on the right side of `->`?
ambiguity with structs

N - Will it have a conflict between generic functions that generate generic types with casting notation?
`x = Stack(int){...}` create a struct of type `Stack(int)` 
`x = Stack(int)(data)` cast

N - How does type inference work with generic struct types when defining bindings?
suppose that `Stack(t)` is a struct
`x = Stack(){...}` now what if I want to infer type of Stack input?
type inference works fine when I call a generic function. I ignore the type and compiler will infer that.
But what about generic types?
Generic types are functions that generate types for us. So just ignore their input!
`x = Stack(){...}` 
can we simplify above to: `x = Stack{...}`?

Y - Casting for union or primitives is runtime. But for named types it is compile time.
Can we use a separate notation for named types?
`int_val = cast(MyInt, int, my_int_val)`
idea: Use named type as a function
`int_val = MyInt(my_int_val)`
can we use the same for union?
```
MyType = int | float | string
x: MyType = 12
y = int(MyType) # this will give us int|nothing
```
**Proposal**
1. Allow treating type names as function which accept a union or named type to cast.
2. For named types, output is type name (e.g. `int(my_int_value)`)
3. For unions, output is type or nothing
q: what about primitives? can we use the same notation?
we did not use type name as function because it was re-using a fn name. But with generics and type inference that is fine.
q: can developer write a function with name of a type to provide custom casting? No. This is compiler service and function naming should not 
be the same as type naming.

N - Maybe we can use a new notation like `.[]` for casting.
or `.()`

Y - How can we get rid of `_{....}` notation for structs?
scala: `val t = (1, "hello", Console)`
above is syntax sugar for `val t = new Tuple3(1, "hello", Console)`
we have generics and we have type inference.
so why not replace `_` with a std type?
`x = _{1,2,3}`
`x = Tuple3(){1,2,3}`
`x = Tuple3(int,int,int){1,2,3}`
**Proposal**:
1. No use of `_` for untyped structs.
2. No untyped structs. You have to set type.
3. You can use standard generic `Tuple1`, `Tuple2`, ... types for your struct.
Why do we need untyped struct? Because we just want to return some stuff and dont want to add a new type for that.
idea: use golang's approach to allow return multiple outputs and ban untyped struct.
```
process = fn(x:int, y:int -> int, string, float) {
	1, "A", 2.9
}
```
Seems like a big change.
But nothing prevents us to write:
`process = fn(x:int -> struct{int, float, string})...`
And how can I return something?
`result = struct{int, float, string}{1,1.2,"A"}`
It is long but it is based on rules. If you don't want that, you can write a type for it, like Tuple2.
**Proposal**:
1. No use of `_` for untyped structs.

Y - We can also get rid of `.1` notation and use destruction.

Y - Don't use `{}` for destruction
`name, age, weight = my_data` 
but it fails for a struct with just one member
`name = my_name_struct`
`name,_ = my_name_struct`
This seems good. Add a dummy `_` at the end which will cover 0 fields.

Y - q: So now when compiler sees something like `identifier(...)` what is that?
it can be function call `camelCase`
or struct `TypeName`
or struct modification `binding_name`
or casting
can we get rid of struct modification? but that is useful. when we want to change only one field.
`new_customer = old_customer(name: "A")`
but the more robust way is to move `old_customer` inside `()`
`new_customer = Customer(old_customer, name: "A")` but again, confusing.
idea: use `+`: `new_c = old_c + Customer(name:"A")`
we can use `+` to join structs. but `Customer(name:"A")` is missing id which is not good.
lets do it difficult way: `new_c = Customer(name:"A", id: old_c.id)`
**Proposal**
1. No `name(...)` notation to copy-modify struct. use normal notation: `new_c = Customer(name:"A", id: old_c.id)`

N - q: can we use struct to provide function args?
e.g. `process = (int,int->int)`
`Customer=struct(int,int)`
then call process with a binding of type Customer?
this can be used with generics to call any function.
but is that really useful?

Y - Can we eliminate too much use of `{}`
usages:
- code block in function
- struct type
- struct literal
- destruction
scala: `val(name, age, weight) = ("Al", 42, 200.0)`
one way is:
`name, age, weight = my_data` 
but it fails for a struct with just one member
`name = my_name_struct`
`name,_ = my_name_struct`
This seems good. Add a dummy `_` at the end which will cover 0 fields.
```
Customer = struct { name: string, id: int }
Customer = struct(string, int)
Customer = struct(
```
can we treat struct as a function?
`Customer = struct(name: string, id: int)`
and to create a customer call struct function:
`x = struct("A", 10)`
or even simpler:
`Customer = fn(name: string, id: int -> struct)`
`my_customer = Customer("A", 10)`
`my_customer.name`
`my_customer.id`
basically, above is constructor of the struct type.
but, how can we specify field names? This makes code more readable.
If we allow that, we effectively allow calling functions by arg name.
`process = fn(id: int, name: string -> int) { ... }`
`x = process(id: 10, name: "A")`
The reason against calling fn by arg name is what if arg name changes.
Same as what if struct field name changes.
both have same chances of change, for struct the chance is even higher.
IDE can provide names for the developer so it is not a burden for them.
About go: The reason for this is because the names are not really important for someone calling a method or a function. What matters is the types of the parameters and their order.
pro: we can re-arrange arguments as we like
q: What happens to optional args? they can appear anywhere.
q: what about generic types? They also can appear anywhere. Just ignore them if you want compiler to infer them.
**Proposal**:
1. `{}` will be only used for code blocks.
2. struct type is a function. you call it with values and it gives you a struct.
3. you should also call fn with arg name same as struct.
`Customer = fn(name: string, id: int -> struct)`
`my_customer = Customer(name:"A", id:10)`
`process(data: a, names: [1,2,3])`
4. you can specify args in any order you want
5. optional arguments including types and `|nothing` can be omitted and don't have to be at the end.
q: shall we make it optional to specify arg name?
struct is just like a generic: `Stack = fn(x: T, T: type -> type)`
but it does not have a body.
or maybe we can drop the fn notation altogether.
`x = Customer(name: "A", id: 10)`
`x = (name: "A", id: 10)` and x is a struct
but how can we refer to its type?
`(name:"A", id:10)` is bad because ignored type and dotLang is supposed to be strongly typed.
but can we write a body for Customer?
```
Customer = fn(name: string, id: int -> struct) {
	(name: name, id: id)
}
```
so we accept that output of above is a struct.
so we accept we can have unnamed/anonymous struct
how can we show that? `struct ("A", 1)` is a value
`struct(name: string, id:int)` is type.
So why do we need functions?
The reason we can use this is that now struct is a bare list of fields. No value, no import, no types, ...
**Proposal**:
1. `{}` will be only used for code blocks.
2. define struct type: `Customer = $(name: string, id: int)`
3. define struct value: `c = Customer(name: "A", id: 10)`
4. anonymous struct: `process = fn(->struct(int, string))`
5. anonymous struct value: `x=struct(int,int)(10, 20)`
q: so, shall we name it tuple now? or maybe something shorter?
q: what happens to call fn by arg name?
maybe use `$` prefix?
But in `Customer(name: "A", id: 10)` we are really treating struct type like a function.
so, just like generic types, struct types are functions.
```
Customer = fn(name: string, id: int -> struct) {}
```
But what should be the return value? If we specify anything here, it will be the "actual" struct type.
so some magic by compiler should happen here.
basically, the body is defined in the argument list: name and types, 
so compiler will write the return for us. we can write nothing or `{}` or `{struct}`
I think writing nothing makes most sense.
So:
to define a struct type, you must define a function with appropriate args.
q: but what happens to anonymous structs?
q: can struct be type of an input? it can be. because it is type of an output.
so if something is of type struct, it can be any struct. but it makes no sense.
if it makes no sense, why use the keyword `struct`? so that we explicitly state this is a struct type.
`Customer = fn(name: string, id: int -> )` this is too confusing.
```
Customer = fn(name: string, id: int -> type)
```
struct is a generic type. so we define `Customer` just like a Stack, but without body. this makes sense but is definitely confusing.
```
Stack = fn(T: type -> type) { ... }
Customer = fn(name: string, id: int -> type) 
```
so how can I define a generic LinkedList?
```
LinkedList = fn(T: type -> type) {
    Node = fn(data: T, next: Node(
    Result = fn(value: T, next: ?)
    Result
    
}
```
About anonymous struct we can define std functions to create tuples:
`Tuple2 = (T: type, U: type, field1: T, field2: U -> struct)`
Main issue is combining generic fn with struct fn is confusing.
```
LinkedList = fn(T: type -> type) {
    Node = Tuple2(T, Node)
    Node
}
```
We can define core fn to define recursive data types. These are really edge cases and exceptions.
so, what is the explicit sign to differentiate struct from a generic function?
struct is a generic function without body.
`Customer = fn(name: string, id: int -> type)`
`LinkedList = fn(T: type -> type) { SelfPtr(T) }`
`my_customer = Customer(name:"A", id:10)`
`my_ll = LinkedList(int)`
No. These two are totally different: struct function returns a value
generic function returns a type.
Both are types, but generic fn is a type that generates types so `-> type`
But struct fn is a type that generates values so `-> struct`
**Proposal**:
1. `{}` will be only used for code blocks.
2. define struct type: `Customer = fn(name: string, id: int -> struct)`
3. define struct value: `c = Customer(name: "A", id: 10)`
4. anonymous struct: `process = fn(->Tuple2(int, string))`
5. anonymous struct value: `x=struct(int,int)(10, 20)`
Tuple is a generic type that generates structs.
`Tuple2 = (T: type, U: type, field1: T, field2: U -> struct)`
so `-> struct`: does it mean output is a struct type or a struct value?
it is not struct type.
`Tuple2 = (T: type, U: type, field1: T, field2: U -> type)`
I think this whole thing is super confusing. We should be ok with adding new keyword notation at the cost of making language easier to understand.
Original goal: remove usages of `{}`
to define struct type: `Customer = struct(name: string, id: int)` field name is optional
struct value: `x = Customer(name: "A", id: 10)`
anonym struct: `process = (x:int -> struct(string, int))`
an. struct value: `f = struct(string, int)("A", 10)`
```
LinkedList = fn(T: type -> type) {
    Node = struct(data: T, next: Node)
    Node
}
```
**Proposal**:
1. `Customer = struct(name: string, id: int)`
2. `my_cust = Customer("A", 10)` or `my_cust = Customer(name:"A", id: 10)`
3. `process = fn(->struct(int, string)) { struct(int, string)(10, "A") }`
This is same as existing notation except for replacing `{}` with `()`
so, this is same as casting? what differentiates them?
`x = Customs(1)`
maybe casting notation should be similar to `:type`
`x = int_var:float`
but we want to make it composable.

N - q: Can a generic function call with all arguments inferred, omit `()`?
but, if I see `x = MyType{...}` I won't be able to know if `MyType` is generic or not.
and then can I use the same in function decl?
`process = fn(x: MyType, ...` no because I need the type of MyType. unless I don't care.
`process = fn(x: Stack, y: ...` this means `x` can be stack of any type.
no. wrong. confusing.
`process = fn(x: Stack(T), T: type, ...`
you can omit types when calling process.
when in argument list, you cannot omit `()` because you must specify inputs. there is no inference.
when calling function, you can infer and if all inferred, you can omit `()`.
because `Stack(){...}` is also misleading.
what if we use `Type[T]` notation for generic data types? will it be easier to read and understand? No.
proposal: If the function only has generic type argument and all of them are going to be inferred, you can omit `()` too (Example 7).
Any specific example which depicts above problem?
This is not relevant in fn definition. 
also in fn call, we already have all the data.
e.g. set of T
`process = fn(s: Set(T), T: type...`
`x=process(my_int_set)` works fine
`x=process(Set(int){4,[1,2,3,4]})`
This only applies for generic functions that return a struct. Because how else can compiler deduce generic types?
`data = Customer(int)(10)`
`data = Customer(10)`
`data = ValueHolder(10)` short for `data = ValueHolder(int)(10)`
but won't it be confusing? what if the struct HAS types.
`data = ValueHolder(int)`, `data = ValueHolder(type)(int)`
this is for a generic type which is internally a struct.
shall we say this is forbidden? When calling a generic type, you must provide value for type arguments.
but when calling a generic fn, you can omit them and let compiler deduce.

N - How can I cast to a type alias? if we use core fn it will be easy.


Y - Casting notation can be confusing with new struct notation.
maybe casting notation should be similar to `:type`
`x = int_var:float`
but we want to make it composable.
`result = Type(data)`
`result = Type?(data)`
or maybe we cal altogether get rid of castings.
why do we need casting?
- union: cast to `T|nothing`
- named type: 
- primitives
- type alias
`x = (int|nothing).(value)`
Ada, Fortran and golang use `type(value)` notation.
But we need something to assert this is a type casting, not a function call.
we want to have a notation which is easy to read and remember and different (so that it is not ambiguous)
`value as Type`
`(type).(value)`
maybe using cast function is the best choice.
it will be a core fn so compiler will handle it.
you can use `tryCast` for unions.
```
int_val = cast(my_int_val, int)
possibly_int = tryCast(int_or_float, int) #gives int|nothing
int_var = cast(3.14, int)
```

N - The new notation for struct, will it be confused with:
- type casting: no more
- generic types
`x = Customer(...)` a struct
`Type = LinkedList(...)` a generic type
if result is used as a type, it is a generic type
if result is used as a value, it is a struct
shall we make them differ more?
struct/generic types/function call all have `a(b)` notation.
`x = process(x:10, y:20)` call fn with arg name
`X = Stack(int)`
`x = Pointer(x:10, y:20)` create struct. Capital letter at the beginning
idea: we can use `!` for generic types. but it is used for inequality check.
so no confusion with fn call.
The confusion is between struct creation and generic type.
But it should be easy to say if we need a type or a value at some point.
So if we need a value, then this is a struct.
If we need a type, then this is a generic data type function.

N - We treat functions as type for generics.
We treat types as functions for casting. no more
is that not confusing?

N - Review concepts. Are there any case where it is not a simple clear one with a basic well-defined mission?
Sometimes unification makes things more complicated: for example module is a struct. So we don't need a new "Module" concept. 
But now the "struct" concept is having a more complicated meaning which makes things more difficult.
1. union/enum
2. function/generic types

Y - Another thing that is acting like two concepts: unions
is used for sum types and enums
Also concrete types
Purpose of union: A type that can only accept a subset of allowed values
e.g. int can accept any int that can fit in 8 bytes
but Color is a special type of int that can only accept a limited values
Proposal: Treat it like union but with bindings (as name for values)
```
a = 1
b = 2
MyUnion = a | b
```
So a sum type of types is a union 
but sum type of bindings is an enum
**Proposal**
1. Union must have types as alternatives
2. Enum is a sum type with bindings as their alternatives
3. Bindigns of enum type can only accept specified values
Technically we can define bindings of different types for an enum (by mixing union and enum)
so: `MyType = int1 | int2 | str1 | str2` should be allowed.
Also technically we can mix values with types because:
```
MyType = int1 | int2
MyUnion = MyType | float
MyUnion2 = int1 | int2 | float
```
then what happens? how can I use check function? 
if I check for type, then it can either have int or int1 -> this is not possible because union cannot have repeated types.
With MyUnion I check for type first (float or MyType?) then if it is MyType, I can cast and then check for int1 or int2.
alternatively: use `enum` keyword: `MyType = enum { int1, int2 }`
in this way it will not be possible to mix it with union and cause confusion.
In Haxe these two are mixed:
```
enum Color2 {
      red;
      green;
      blue;
      grey( v : Int );
      rgb( r : Int, g : Int, b : Int );
  }
 static function toInt( c : Color2 ) : Int {
        return switch( c ) {
            case red: 0xFF0000;
            case green: 0x00FF00;
            case blue: 0x0000FF;
            case grey(v): (v << 16) | (v << 8) | v;
            case rgb(r,g,b): (r << 16) | (g << 8) | b;
        }
    }
```
But the power that comes with above comes with a cost
Can we show enum with a map?
map: values must have the same type, you can have int key and use binding names for key
the only missing piece is banning values other than keys defined in the map.
```
saturday = 0
sunday = 1
DayOfWeekMap: [int, string] = [saturday: "Saturday", sunday: "Sunday", ...]
DayOfWeekSeq: [int] = [saturday, sunday, ...]
myDayOfWeek: int???
DayOfWeek = int@[satuday, sunday, ...]
DayOfWeek = @DayOfWeekMap
```
Best case: Leave it to the developer, write a map where key is labels you want to use, value is actual values you want
goal: I want to have a type which can only accept a limited number of possible values (for example DayOfWeek).
idea: allow to append a sequence after type name to allow list of allowed values for it.
what is the absolute minimum the language can provide?
I want to make sure bindings of type X, can only have these types.
I want to make it easy to assign those values.
I want to make sure when I read value of a type X binding, I check for all possible options.
idea: Specify the only way to create an instance of type X is by calling function F.
idea: `Enum[T] = [T: nothing]`
This can be compile time checked. If the map is compile time.
```
DayOfWeek = enum { saturday, ... }
DayOfWeek = int[0,1,2,3]
saturday=0 ...
DayOfWeek = int[saturday, sunday, ...]
DayOfWeek = T[T]
Enum[T] = T[T]
```
So this will be a new notation: `type[sequence of values]` means enum.
```
DayOfWeek = int[0,1,2,3,4,5,6]
DayOfWeek = enum { 0, 1, 2, 3, 4, 5, 6 }
CustomerTypes = int[valid, invalid]
x: CustomerTypes = valid
CustomerTypes = int[valid=0, invalid=1]
```
But this notation is difficult to read and memorize. `int[...]` who knows this is an enum.
everybody knows enum so why not use it?
```
CustomerType = enum:int{valid=0, invalid=1}
x: CustomerType = CustomerTypes.valid
x: CustomerType = 0 #invalid, but you can cast CustomerType to int
```
This `enum:int` notation is not ideal.
We should not limit enum to integers. Any type can be enum.
being enum means having a limited available allowed values.
`CustomerType = int{valid=0, invalid=1}`
`Data=string{ok="OK", not_ok="NOK"}`
`str: string = string(Data.ok)`
`x: Data = Data.ok`
So `Data.ok` is a string and bindings of type `Data` can only have one of two above values.
`TypeName = TypeName { Binding_Name=Literal, ... }`
it should be very simple and minimal but general and orth and concise
**Proposal**
1. Union must have types as alternatives
2. You can make any type enum by using below notation:
`NewTypeName = TypeName { Identifier=Literal, ... }`
q: Can we do above for `type`?
`MyType = type { Int=int, Float=float, String=string }`
then use it in generics?
`process = fn(x: T, T: MyType, ...`
This makes sense. Anything of type `MyType` can either be int or float or string.
So enum can also be used for limiting allowed types in generics.
q: can we omit name? and only provide literals? That is possible but you must then pass literal when that type is needed.
q: how can I convert a string to StrEnum value? write a function.
or if it is literal, you can cast. 
`Data=string{ok="OK", not_ok="NOK"}`
`x = Data("OK")` but in this case it makes more sense to write `Data.ok`
in general, with a runtime string, you need to write your own function
**Proposal**
1. Union must have types as alternatives
2. You can make any type enum by using below notation:
`NewTypeName = TypeName { Identifier=Literal, ... }`
Literal must be compile time and of type `TypeName`
3. Note that `TypeName` can be `type` too.
`NumericType = type { int, float }`
4. Identifier is optional.
we don't really need TypeName. It can be deduced from literals.
`NewTypeName = enum { Identifier=Literal, Identifier2=Literal2, ... }`
`DayOfWeek = enum { Saturday=0, Sunday=1, ... }`
`x: DayOfWeek = DayOfWeek.saturday`
we want to make it minimal. so why assign values here?
```
saturday=0
sunday=1
...
DayOfWeek = enum{saturday, sunday, ...}
x: DayOfWeek = saturday
```
q: Can I use named types in enum?
q: can I just write values in enum? No. there should not be two ways to do this. only one way.
So:
`Type = enum {list of already defined bindings of the same type}`
so `Type` is a new type and any binding of this type can only be assigned one of given bindings.
why `{}`?
`DayOfWeek = [saturday, sunday, ...]`
enum is a list of bindings of the same type. so this is a sequence.
but sequence can also have values.
`array1 = [0,1,2]`
`array2 = [saturday, 1, 2]`
`MyType = [1,2,3]`
MyType is a new type that can only accept one of the given values.
So enum is just a compile time sequence which we treat as a type.
**Proposal**
1. Union must have types as alternatives
2. You can assign any compile time sequence to a type and it will be an enum type.
`NewTypeName = enum [sequence literal]`
3. Note that sequence can have types too.
`NumericType = enum [int, float]` and this can be used in generics.
4. variables of enum type must accept values of exactly what is specified inside sequence, nothing else, even if they have same value.
No. difficult to read. lets add `enum` keyword

Y - In line with providing needed features, should we provide a switch statement for union and enums?
What is wrong with core `check` function? We can always add something which is missing but removing something we not really need?
But almost all languages have the pattern matching/switch concept.
We can use maps for enum and union. and compiler can check for completeness of the map
```
x = [1: "A", 2: "B", ...][my_int_enum]
x = [int: fn(x:int|float->string)..., float: fn(x:int|float->string)...][type(int_or_float)](int_or_float)
```
Add to enum and union section.

N - Can we call fn by arg name?
fn arg name can change just like struct members names can change.
`data = fnName(a:1, b:2, c:3)`
it will be more like structs but because of the naming it won't be confused.
even if we allow, it should be optional. And I don't like optional things. Either mandatory or banned.
what if we think of function input as a struct?

N - If we call fn with arg names, maybe we can use something else instead of `_` to create lambda.
`p = process(x: 10, y: 20, z: ?)`
but what if we make it optional.

Y - `typeOf` function in core will return type of a binding.
For union, its output is an enum of possible types of that enum
```
IF = int|float
IFType = enum {int, float}
typeOf = (x: int|float -> enum {int, float})
```

N - What can we remove from language?

N - We are using `()` for 4 purposes:
1. fn def:  `process = fn(int,int,int)...`
2. fn call `process(1,2,3)`
3. struct decl `S = struct(x:int, y:float)`
4. struct binding instantiation `s = S(x:10, y:1.23)`
3 is different because it has `struct` prefix
4 is different because it has type name prefix
1 is different because it has `fn` prefix

N - Is there an easy way to document for developer and compiler, out expectation of a generic type?

Y - We use functions for two purposes: function and generic types.
Should we make them differentiate?
What about this: for generic types, we write them as function but with `[]`
```
Stack = fn[T: type -> type] { ... }
...
x: Stack[int]
```
Won't it be confused with seq/map access? No because type name starts with Capital letter.
**Proposal**
1. Generic types are defined just like functions but instead of `()` we use `[]` to declare and invoke them.

N - Can we have type inference with generic types?
when I call push function, its generic type can be inferred from other inputs.
but when I call Stack generic type, how can its type be inferred?
in java I can write `Map<Int, String> x = new HashMap<>()` and on the right side type is inferred but from the left side.
But now there is no left side and it is also optional.

Y - Naming: module files should be named like bindings

N - Can we have module level bindings which do not have compile time values?
like `stdio` at module level which is a struct of all required functions.
```
Stdio = struct(write: fn..., read: fn...)
stdio = createStdio()
createStdio = fn{ Stdio(write: writeToOutput, read: readFromOutput) }
```
In another module:
```
io = import("/core/std/io")
io.stdio.write(...)
```
Not needed. You can simply assign value to stdio:
```
Stdio = struct(write: fn..., read: fn...)
stdio = Stdio(write: ..., read: ...)
```

Y - We use `.` for two purposes: struct and module
rust usese `::`
Idea: use `..`
```
std = import("/core/std/stack")
```
on one hand we want to have a notation different from existing ones.
on the other hand, we want this to be easy to write because devs will need to write lots of this.
idea: use a completely new and different notation, BUT allow developer to import into current ns
```
std = import("/core/std/stack")
g = std::g_data
import("/core/std/stack")
process(g_data) #g_data comes from stack module
```
`g = std::g_data`
`g = std..g_data`
`:: ..`
both `.` and `:` are already used.
`::` is more intuitive.

N - How to have iterators?
e.g. reading a very long xml element by element
or multiple file reader/writers
it will be a combination of struct, generics, function.

N - How to have fn redirect?
e.g. getNext can accept `int|nothing` so we want to redirect to two other fns based on whether input is int or nothing
can we do it with above notation?
```
intn, floatn = int_or_float
invoke(intn, int_func) // invoke(floatn, float_func)
```
but can it be done at a higher level? so that compiler can optimize?
even if compiler does this, it'll be similar to above.

N - we can allow string interpolation
and use `{base}/helper` as module path to import where base is taken from another module
```
x = import("{refs.a}/helper")
x = import(refs.a+"/helper")
```

Y - using `::` is not very good.
use another notation
option1: use `.`
option2: use `..`
option3: import at identifier level not module level.
```
myFunc = import("/core/std/helper", myFunc)
MyType = import("/core/std/helper", MyType)
my_binding = import("/core/std/helper", my_binding)
a,b,c = import("/core/std/helper/{a,b,c}")
_ = import("/core/std/helper/{a,b,c}")
d,e,f = import("/core/std/helper/{a,b,c}")
_ = import("/core/std/helper")
```
if we decide to import at id level:
1. it might be difficult to import many things
2. how can I import everything?
3. how can I rename?
4. should identifiers be part of module path string?
but import at module level: no need for alias or selective import
but in this case we need to decide notation for accessing inside a module.
```
m = import("...")
MyType = m..MyCustomType
MyType = m
```
lets go with `..`

Y - what about `import("...",["a","b"...])` notation?
dropped

Y - for union use destruction syntax and remove all type checking functions from core
another wy, that can be used is to treat a union binding as a struct
`int_or_float.int`, `int_or_float.float`, ...
this forces developer to use named types for a union
also it can be used in expressions.
`f = int_or_float.int // cast(int, int_or_float.float)`
`int_iorfloat.int` is of type `int|nothing`.
so if `T = int|nothing`, a binding of type T will have `.int`? yes and it will be itself.
and `my_t.nothing`? what will it be? it is not orth.
also `x.Type` notation will be confusing.
**Proposal**:
1. You can destruct a union to its types (except nothing)
`intv, floatv = int_or_nothing`
each element on the left will be of type `T|nothing`.

Y - can we say fn with `[]` are compile time functions?
con: it will be confusing with bindings
`data[...]` is it a seq or a compile time fn call?

Y - can we remove casting altogether?
usage:
primitive: can use custom purpose fns
union: can use destruct
named type: can we use compile time fn?
```
MyInt = int
toInt = fn[x: MyInt -> int] { x }
h: MyInt = ...
g:int = toInt[h]
```

N - what other applications can we have for compile time fns?
- implement language level if? no.
these are not inline functions. they are compile time evaluated
zig has this
Dlang has them as ctfe
also c++ has them
can they act like macros? because this is how generics work.
```
process = fn[cond: boolean, ifPart: fn(->T), elsePart: fn(->T), T: type -> T] {
	[ifPart, elsePart][cond]()
}
```
there is no use for having macros with ctfe. compiler can inline normal fns and that is enough.
- we can do assertion
what about this? if fn inputs are compile time, then compiler will process it at compile time.
other wise runtime
but then what happens to casting of named types? the `[]` notation does not help them.
```
MyInt = int
toInt = fn(x: MyInt -> int) { x }
h: MyInt = ...
g:int = toInt[h]
```
Y - cancel `[]` notation for compile time fns.
just use normal notation but compiler will optimise.

N - can we provide monads?

Y - can we destruct a module? to only have some of its symbols?
```
Set, process, my_data = import("/core/set")..{SetType, processFunc, my_data}
```

N - Need for clarification: 
what if I write `x,y=union_var` but right side is union of 5 types?
option1: this is not allowed
option2: x will be the first type and y will be union of the rest
but option 2 is confusing, and the notation will not be same as struct.
`x,y=int_or_float_or_string_union`
x will be `int|nothing`
y will be `float|string|nothing`
this can be used in a generic function to have things like `hasType`
```
hasType = fn(x: T|U, T: type, U: type -> bool) {
	a,b = x
	a!=nothing
}

has_int = hasType(int_or_float_or_string, string) #T will be string, U will be int|float|nothing
#so: a,b = x means a will be int|nothing, b (or _) will be float|string|nothing
```
But result of destruction is based on the union types, not T and U in the generic function.
so destruction, does not work based on the type I need. It works based on the type that the binding is defined as.
maybe we can do a recursive call for this purpose?
```
hasType = fn(x: T|U, T: type, U: type -> bool) {
	if U is nothing then return this:
        if x is nothing return false
        if x is not nothing return true
    otherwise, if U is not nothing:
        a,b = x
        if a is of type T|nothing and is not nothing then return true
}
```
No this is confusing.
Maybe we can use identity function? no. 
```
getType = fn(x: T|U, T,U: type -> T|nothing) {
    x
}
```
no this is also confusing.
so how can we use destruction to provide `hasType` function?
what about this?
`a:string|nothing,_ = int_or_float_or_string_var`
confusing. when having a single identifier on the left `:` notation is fine but with multiple identifiers it becomes confusing.
what if we write fns for all numbers of types?
```
hasType = fn(x: T|U, T: type, U: type -> bool) {
    a,b = x
    if a's type is T and is not nothing return true
    if b's type is T and is not nothing return true
    else return hasType(x)
}
```
our purpose is to be able to get `int|nothing` from a union which has int in its types (at any location).
```
getData = (x: T|U, T,U: type -> T|nothing) {
    #return nothing if x does not have type T
    #othertiwse if x's internal value is of type T, return it
}
```
Let's leave it. developer can use destruction whenever needed to decide.

Y - q: if we have `int|string|float` can it be sent to a generic function of type `T|U`?
it should be fine. Because, anyway `U` can be a union type itself.
`T|U` says input should be a union of at least two types.
clarify

N - Refering to an index outside sequence bounds, why would it return nothing?

N - Why allow defining named type inside a function?
what if fn returns a binding of that named type? How are others supposed to access it?
For function section: State you can only have bindings and a return 
also say there is no function overriding.

N - How can I write a binding of type of named type?
```
MyInt = int
x = 10 #this is int type
x = ? #how to define MyInt type
```
`x = MyInt(10)`
This will not be confused with function.
But it will be confused with struct.
`x = Point(10)` is this a named type or a struct type?
But even struct, is a named type in above case.
So, `Point(10)` will create a new binding of named type `Point` with value `10`.
I think this is consistent with struct decl.

N - How can I convert named type to original type?
identity function

N - For generic section
## Type argument

These are binding of type `type`. You can use these bindings anywhere you need (inside function arguments, part of a struct, ...) but their value must be specified at compile time.
More in "Generics" section.

N - In type section mention about generic types.

N - Literals can and should be castable to named types.
Because:
`test = fn(x:int -> PlusFunc) { fn(y:int -> int) { y + x } }`
