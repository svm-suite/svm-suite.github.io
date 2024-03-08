# SVM Suite

```
// work in progress //

project status:
    [ ] alpha conception
        [x] initiating the idea
        [ ] theorizing
        [ ] implementing
    [ ] beta testing and revising code
    [ ] gamma release
```

#### TOC

- [ ] [SVM Suite instructions](#svm-suite-instructions)
    - [ ] [A. local installation](#a-local-installation)
    - [x] [B. router.svm](#b-routersvm)
        - [x] [B.1. starting and stopping services](#b1-starting-and-stopping-services)
        - [x] [B.2. passing messages between services](#b2-passing-messages-between-services)
        - [x] [B.3. modularity](#b3-modularity)
        - [x] [B.4. conclusion](#b4-conclusion)
    - [x] [C. console.svm](#c-consolesvm)
    - [x] [D. compute-stateful.svm](#d-compute-statefulsvm)
        - [x] [D.1. theoretical background](#d1-theoretical-background)
            - [x] [D.1.1. syntax](#d11-syntax)
            - [x] [D.1.2. semantics](#d12-semantics)
                - [x] [D.1.2.1 temporary and permanent data storage](#d121-temporary-and-permanent-data-storage)
                - [x] [D.1.2.2. pattern matching](#d122-pattern-matching)
                - [x] [D.1.2.3. variables](#d123-variables)
                - [x] [D.1.2.4. modularity](#d124-modularity)
        - [ ] [D.2. examples](#d2-examples)
        - [x] [D.3. conclusion](#d3-conclusion)
    - [ ] [E. compute-stateless.svm](#e-compute-statelesssvm)

# SVM Suite instructions

Name *SVM Suite* stands for Service Virtual Machine Suite, and it represents a programming framework intended for human-computer interaction and automated reasoning. *SVM Suite* includes a set of service virtual machines built on principles of service oriented programming ([SOP](https://en.wikipedia.org/wiki/Service-oriented_programming)) paradigm. SOP paradigm clearly distincts between services that perform given tasks. Such services communicate between each other by passing messages. Benefits of this approach to programming is high modularity needed for code reuse, high agility in process of code development, and granular independence between services that can be invoked parallelly in a multitasking environment.

Some examples that may be represented as services are RAM, permanent storage, keyboard or mouse. Other, more abstract examples may embed higher or lower level computing platforms. By composing these kinds of services together, depending on what the services are, we may build a whole system in the role of programming library, computer application, operating system, or maybe even enthusiastic self-controlling hardware system driven by artificial intelligence empowered form of existential being.

Minimal viable product of *SVM Suite* includes four services called service virtual machines. Central, *router.svm* mediates between *console.svm*, *compute-stateful.svm*, and *compute-stateless.svm*. These virtual service machines should be just about enough to establish meaningful communication between human and computer, guided by your software inspiration. It is also not excluded that *SVM Suite* would be enriched by sound, vision, or other similar services in the future.

> Note that all the code and exchanged data in *SVM Suite* services is written in [s-expression](https://en.wikipedia.org/wiki/S-expression) form borrowed from [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)) family of programming languages. Ingenious s-expression form is chosen because of its very convenient properties relating to code related tasks.

## A. local installation

```
// work in progress //
```

## B. router.svm

*Router.svm* is intended to be a router between arbitrary pluggable services. It is conceived as a minimalistic, yet powerful service integrator. As a central hub for exchanging messages between different services, the importance of *router.svm* may be considerable in spite of the simplicity of code that may represent a specific router instance.

The following code pattern would give an insight to what a router code would look like:

```
(
    ROUTER
    ...
    (
        DIRECT
        (LISTEN (SOURCE ...) (DATA ...))
        (INVOKE (TARGET ...) (DATA ...))
    )
    ...
)
```

In this manner, the router directs messages from one service to another. Each `SOURCE` and `TARGET` section hold a service name, while each `DATA` section holds a s-expression.

### B.1. starting and stopping services

Each service has its lifetime during which it sends or receives messages. Services begin their lifetimes using always active built-in service `start`, while they end using also always active built-in service `stop`. When services start or stop, they emit related messages that can be caught by the router. Given that the router service is already running, in such a way, we can intercept a starting or stopping service named `this` denoting the current router. On interception of such messages, we can start or stop other services, which would be the root point of our service management.

There are several key points in starting a service, which we will analyze in the following example:

```
...
(
    DIRECT
    (LISTEN (SOURCE start) (DATA this))
    (
        INVOKE
        (TARGET start)
        (
            DATA
            (
                service
                (type console.svm)
                (name cns1)
                (params (prompt "input>"))
            )
        )
    )
)
...
```

First, we listen to the start of `this` service. Thus, when the router starts, we invoke the start of a service of type `console.svm`, named `cns1`, providing some additional parameters. When the console service starts, in further message passing, we will refer to the console by its assigned name `cns1`. Note that we can start any number of instances of the same service type using only different service names. Although this example does not use it, particular services require an additional `src` parameter. Lastly, service type `console.svm` requires a `params` parameter which defines what prompt is displayed in the console when it waits for an input from a user during its lifetime. In short, important key points of starting a service are:

- mandatory `service` where we state that we are starting a service
- mandatory `type` where we denote a type of the service
- mandatory `name` where we denote an arbitrary name of the service we will refer to in further message passing
- optional `src` where we specify source code file if it is required for starting the service
- optional `params` where we pass arbitrary parameters to the service

Stopping services is somewhat simpler, as we only need the service name to end its lifetime. The following example:

```
...
(
    DIRECT
    (LISTEN (SOURCE stop) (DATA this))
    (INVOKE (TARGET stop) (DATA cns1))
)
...
```

stops service named `cns1` when `this` is being stopped.

To resume, if the above two examples are coded in the same router, they start `cns1` service when the router lifetime begins, and stop it when the router lifetime ends.

### B.2. passing messages between services

When services are up and running, they receive and emit their input and output messages. To direct and pass around these messages, we use the same pattern from the previous examples. For example, we may have started two services of a type `console.svm` named `cns1` and `cns2`. If we wanted to make, say, a chat system out of them, these two services may be paired in a router to output what the other service inputs while users are typing to their interfaces. To begin with the creation of such a system, let's start from a simple indicator of whether the users input a letter `A` into consoles. Such system would contain the following code:

```
...
(
    DIRECT
    (LISTEN (SOURCE cns1) (DATA (input "A")))
    (INVOKE (TARGET cns2) (DATA (output "the other user typed 'A'")))
)
(
    DIRECT
    (LISTEN (SOURCE cns2) (DATA (input "A")))
    (INVOKE (TARGET cns1) (DATA (output "the other user typed 'A'")))
)
...
```

Note how we use data parameters `input` and `output` specific to console instances. Now the both consoles are able to input any text from users, indicating the other user only on typing a letter `A`. Of course, this code can hardly be of any use, but we will introduce some new constructs to our router code, namely sections `MATCH` and `VAR`. Now, we may find the router capable enough not only to program our chat system, but also for many other purposes. This is how we could write the code for our chat system: 

```
...
(
    MATCH
    (VAR <X>)
    (
        DIRECT
        (LISTEN (SOURCE cns1) (DATA (input <X>)))
        (INVOKE (TARGET cns2) (DATA (output <X>)))
    )
)
(
    MATCH
    (VAR <X>)
    (
        DIRECT
        (LISTEN (SOURCE cns2) (DATA (input <X>)))
        (INVOKE (TARGET cns1) (DATA (output <X>)))
    )
)
...
```

All that is now left to do is to insert some code for starting and stopping services we learned from the previous section, and we have a fully functioning chat system between two instances of consoles.

Of course, we may use any number of variables in the `DATA` section, and we can name them however we want. Additionally, we may also use the variables in `SOURCE` and `TARGET` sections if we want to parameterize service names we operate on.

### B.3. modularity

To achieve modularity of a router, we may also start other router services from the root one, stacking them in the hierarchical structure. Such modularity may be proven to be useful in more complicated systems where we may want to isolate passing messages of the same sort.

To start a new router, we may write:

```
...
(
    DIRECT
    (LISTEN (SOURCE start) (DATA this))
    (
        INVOKE
        (TARGET start)
        (
            DATA
            (
                service
                (type router.svm)
                (name rt1)
                (src "rt1.svm")
            )
        )
    )
)
...
```

Note the use of `src` parameter. Using this code, we start a router coded in file `rt1.svm`. Similarly, if some other service we start (like *compute-stateful.svm* or *compute-stateless.svm*) depends on a particular code, we use the same `src` parameter to specify the code file.

In a direction of modularity, we introduce two more built-in services, source `input` and target `output`. Using them, we make *router.svm* comply with essential service input/output definition, and we are finally able to pass messages between different routers from a parent node, thus making use of their modularity.

### B.4 conclusion

In this section, we presented a simple router service for directing messages between other services. Being a service glue element, *router.svm* takes a central role among all the services in *SVM Suite*. Provided with simple modularity, one may find it easy to imagine a specific system of routers coordinating between *console.svm*, *compute-stateful.svm* and *compute-stateless.svm* to perform different tasks of interest.

## C. console.svm

*Console.svm* is a simple service virtual machine providing textual console input/output. It consists of a text box with a prompt where the user inputs text. On input, after pressing the <enter> key, the service emits an `input` message with relevant data. Output to the console is managed by sending an `output` message with relevant data to the service. Console service doesn't require any underlying code specific to a particular instance for its functionality.

To see initial examples of using consoles, please refer to the [2. router.svm](#2-routersvm) section.

During a console service runtime, it is possible to change its prompt label from router service by sending it a `prompt` message like in the following example:

```
...
(
    DIRECT
    (LISTEN (SOURCE start) (DATA cns1))
    (
        INVOKE
        (TARGET cns1)
        (DATA (prompt "user>"))
    )
)
...
```

This code sets the prompt of the console to `user>` label when the console service starts.

*Console.svm* is intended to be a default input/output interface to *SVM Suite* based applications. In the case of creating a chatbot, beside the basic conversational interface, consoles can be used for monitoring intermediate thought processes, as we may output a stream of thought during the data computing process.

## D. compute-stateful.svm

In computing theory, there are two mainstream branches of programming systems: declarative and imperative. Declarative branches typically abstract from states, providing optimized structures for other forms of computations. However, sometimes, when we may find dealing with states the best thing to do, imperative branches may take a place because dealing with states is what they do the best.

*Compute-stateful.svm* is a service virtual machine providing computing data using state operations. Programs written in this service resemble a sort of finite state machine. Programs are composed of a series of only one kind of statement which performs a discrete compute step in a process of computation. Each statement quotes the current step of the program execution, and points to the next step in execution, thus the order of statements is not relevant. Also, each step may input from or output to some memory cell, thus carrying on the computation. 

Finite state machines which *compute-stateful.svm* appearance is based on, can perform various tasks like driving vending machines, elevators, traffic lights, combination locks, and many others. The  limitations of finite state machines to perform only a subset of all possible tasks is surpassed by introducing reading from and writing to arbitrary memory cells during program execution. This places *compute-stateful.svm* model of computation side by side with Turing machines model which is known to be the most expressive model of computation.

### D.1. theoretical background

*Compute-stateful.svm* service combines a kind of deterministic [finite state machine](https://en.wikipedia.org/wiki/Finite-state_machine) model with memory cell assignment and pattern matching. A program running by this service can be in exactly one of a finite number of states at any given time. The program begins with the `start` state, changes its state during execution, and finally ends with the `stop` state. As the state changes, the program inputs from and outputs to given temporary or permanent memory cells, thus computing new memory data when the current memory data matches the given pattern. It is possible to draw a directed graph by connecting nodes representing the states, while lines between nodes associate to arbitrary memory cell input and output. Specific nodes may also connect in a cyclic manner, thus forming loops controlled by memory cell inputs. As a measure of its completeness, this computing model is [Turing complete](https://en.wikipedia.org/wiki/Turing_completeness), meaning that it can carry on any kind of computations known to us.

#### D.1.1. syntax

*Compute-stateful.svm* code itself resembles a type of s-expression. S-expressions consist of lists of atoms or other s-expressions where lists are surrounded by parenthesis. In the code, the first list element to the left determines a type of a list. There are a few predefined list types used for data computing depicted by the following relaxed kind of [Backus-Naur form](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form) rules:

```
    <start> := (STATEFUL <statement>+)
<statement> := (COMPUTE <current> <next>)
  <current> := (CURR (STATE start) (INPUT (DATA <S-EXPR>))?)
             | (CURR (STATE <ATOM>) (INPUT (<cell> <ATOM>) (DATA <S-EXPR>))?)
     <next> := (NEXT (STATE stop) (OUTPUT (DATA <S-EXPR>))?)
             | (NEXT (STATE <ATOM>) (OUTPUT (<cell> <ATOM>) (DATA <S-EXPR>))?)
     <cell> := TEMP
             | PERM
```

Each `STATE` section holds a name of the state, each `<cell>` section holds a name of the memory cell, while each `DATA` section holds a single s-expression.

The above grammar rules define the syntax of the code. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one more more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols are considered as parts of the code.

In addition to the above grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//`, and reach to the end of line. Multiline comments begin with `/*` and end with `*/`, so that everything in between is considered as a comment.

#### D.1.2. semantics

Word "semantics" is another word for the word "meaning". In this section, we are dealing with an intuitive semantics of *compute-stateful.svm* code. Semantics of the code will be explained using various simplistic examples, describing how the states and memory cells change and what inputs and outputs the whole examples accept and generate.

##### D.1.2.1 temporary and permanent data storage

**temporary data storage**

Data in *compute-stateful.svm* code is computed using memory cells. Temporary memory cells are noted by `TEMP` sections, and they live from the moment they are created until reaching the `stop` state, or until their data is set to `NIL`. Computing with temporary cells is much faster than computing with permanent cells since temporary cells are typically stored in RAM.

The next example:

```
(
    STATEFUL
    (
        COMPUTE
        (CURRENT (STATE start))
        (
            NEXT
            (STATE 1)
            (OUTPUT (TEMP X) (DATA fst))
        )
    )
    (
        COMPUTE
        (CURRENT (STATE 1))
        (
            NEXT
            (STATE 2)
            (OUTPUT (TEMP Y) (DATA snd))
        )
    )
    (
        COMPUTE
        (CURRENT (STATE 2))
        (
            NEXT
            (STATE 3)
            (OUTPUT (TEMP Z) (DATA ((temp X) (temp Y))))
        )
    )
    (
        COMPUTE
        (CURRENT (STATE 3))
        (
            NEXT
            (STATE stop)
            (OUTPUT (DATA (temp Z)))
        )
    )
)
```

takes no start parameters because of the absence of `INPUT` section in `start` state. Moving through states from `start` to `stop`, the following computations apply:

- after reaching state `start`, we store the atom `fst` in memory cell `X`
- after reaching state `1`, we store the atom `snd` in memory cell `Y`
- after reaching state `2`, we store the expression `(fst snd)` in memory cell `Z`. This is done by using `temp` sections to refer to memory cells named `X` and `Y`
- after reaching state `3`, we `stop` the service outputting the contents of memory cell `Z` 

Computing processes using only temporary storage are [referentially transparent](https://en.wikipedia.org/wiki/Referential_transparency) from the world outside of the processes.

**permanent data storage**

Permanent memory cells are noted by `PERM` sections, and they live forever, or until their data is set to `NIL`. Computing with permanent cells is much slower than computing with temporary cells since permanent cells are typically stored on disk drives.

The next example:

```
(
    STATEFUL
    (
        COMPUTE
        (CURRENT (STATE start))
        (
            NEXT
            (STATE 1)
            (OUTPUT (PERM "dirX/fileX") (DATA "fst"))
        )
    )
    (
        COMPUTE
        (CURRENT (STATE 1))
        (
            NEXT
            (STATE 2)
            (OUTPUT (PERM "dirY/fileY") (DATA "snd"))
        )
    )
    (
        COMPUTE
        (CURRENT (STATE 2))
        (
            NEXT
            (STATE 3)
            (OUTPUT (PERM "dirZ/fileZ") (DATA ((perm "dirX/fileX") (perm "dirY/fileY"))))
        )
    )
    (
        COMPUTE
        (CURRENT (STATE 3))
        (
            NEXT
            (STATE stop)
            (OUTPUT (DATA (perm "dirZ/fileZ")))
        )
    )
)
```

is analogous to the one in [4.1. temporary data storage](#41-temporary-data-storage). The only difference is that we use permanent storage cells. Such use stores data in the file system while `PERM` sections hold paths to specific files. Relatedly, `perm` sections refer to those files. Permanent data storage introduces referential opaqueness from the outside world of the process.

##### D.1.2.2. pattern matching

We can also define multiple `COMPUTE` steps with the same `CURRENT` state which may introduce possible branching choices. When the next computing step may land at multiple steps, *compute-stateful.svm* service chooses to select the first available step with the same name, from the top of the code, whose `INPUT` matches against the memory cell contents. Thus, the example:

```
(
    STATEFUL
    (
        COMPUTE
        (
            CURRENT
            (STATE start)
        )
        (
            NEXT
            (STATE mediate)
            (OUTPUT (TEMP X) (DATA snd))
        )
    )
    (
        COMPUTE
        (
            CURRENT
            (STATE mediate)
            (INPUT (TEMP X) (DATA fst))
        )
        (
            NEXT
            (STATE stop)
            (OUTPUT (DATA "choice one"))
        )
    )
    (
        COMPUTE
        (
            CURRENT
            (STATE mediate)
            (INPUT (TEMP X) (DATA snd))
        )
        (
            NEXT
            (STATE stop)
            (OUTPUT (DATA "choice two"))
        )
    )
```

outputs the atom `choice two`. Constructs like this may also be useful when we implement loops by employing circular references. In such cases, branching choices may save us from infinite loops.

##### D.1.2.3. variables

Similarly to *router.svm* services, we may want to make use of variables. In a similar manner, variables are specified using `MATCH` and `VAR` sections:

```
(
    STATEFUL
    (
        MATCH
        (
            (VAR <X>)
            (
                COMPUTE
                (
                    CURRENT
                    (STATE start)
                    (INPUT (DATA (<X> <Y>)))
                )
                (
                    NEXT
                    (STATE stop)
                    (OUTPUT (DATA (<Y> <X>)))
                )
            )
        )
    )
)
```

The above example inputs the s-expression of two elements, and outputs them in a swapped position. To reuse it in the following section, let's declare this example as saved in a file `swap.svm`.

##### D.1.2.4. modularity

Defining computing streams may become very complex. To tackle this problem, each *compute-stateful.svm* service may scatter its code in many files and directories. We can then invoke such files by previously declaring them in `IMPORT` sections:

```
(
    STATEFUL
    (IMPORT (NAME swap) (SRC "swap.svm"))
    (
        MATCH
        (
            (VAR <X> <Y>)
            (
                COMPUTE
                (
                    CURRENT
                    (STATE start)
                    (INPUT (DATA (<X> <Y>)))
                )
                (
                    NEXT
                    (STATE stop)
                    (OUTPUT (DATA (swap (<X> <Y>))))
                )
            )
        )
    )
)
```

The above example also outputs swapped input s-expression of two elements, but now using the `swap` section in output to invoke the `swap.svm` code. Let's just mention that in a similar way, we can also safely import and invoke *compute-stateless.svm* services from the *compute-stateful.svm* services.

### D.2. examples

```
// work in progress //
```

### D.3. conclusion

In this section we exposed basic constructs of *compute-stateful.svm* services. Although there are many cases where dealing with states may pose a stumbling stone in writing programs, there are some cases where being able to express states is very desirable. And that is the purpose of *compute-stateful.svm* service virtual machine, to easily express operations with states. Here, we only scratched the surface of what *compute-stateful.svm* services are capable of. Since we are dealing with a Turing complete system, we may expect that we are not bound in any way considering the domain of supported computations.

## E. compute-stateless.svm

```
// work in progress //
```

*Compute-stateless.svm* is planned to be a service virtual machine providing stateless computing operations. Such operations, using this service, may be suitable to perform automated reasoning tasks.

```
(
    STATELESS
    (
        RULE
        (
            READ
            
            (EXP (goes <name> <voice>))
            
            (RULE (READ (EXP <name> )) (WRITE (EXP Milo)))
            (RULE (READ (EXP <name> )) (WRITE (EXP Nora)))
            (RULE (READ (EXP <voice>)) (WRITE (EXP bark)))
            (RULE (READ (EXP <voice>)) (WRITE (EXP meow)))
        )
        (
            CHAIN
            
            (
                MATCH
                (VAR <X>)
                (RULE (READ (EXP (goes <X> meow))) (WRITE (EXP (isA <X> cat))))
            )
            
            (
                MATCH
                (VAR <X>)
                (RULE (READ (EXP (goes <X> bark))) (WRITE (EXP (isA <X> dog))))
            )
        )
        (
            WRITE
            
            (RULE (READ (EXP Milo)) (WRITE (EXP <name>  )))
            (RULE (READ (EXP Nora)) (WRITE (EXP <name>  )))
            (RULE (READ (EXP cat )) (WRITE (EXP <living>)))
            (RULE (READ (EXP dog )) (WRITE (EXP <living>)))
            
            (EXP (isA <name> <living>))
        )
    )
)
```

```
// work in progress //
```
