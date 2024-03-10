```
. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
.                                                                           .
.   #####   ##     ##  ##     ##        #####           ##    ##            .
.  ##   ##  ##     ##  ###   ###       ##   ##                ##            .
.  ##        ##   ##   ## ### ##       ##       ##  ##  ##  ######   ####   .
.   #####    ##   ##   ##  #  ##        #####   ##  ##  ##    ##    ##  ##  .
.       ##    ## ##    ##     ##            ##  ##  ##  ##    ##    ######  .
.  ##   ##     ###     ##     ##       ##   ##  ##  ##  ##    ##    ##      .
.   #####       #      ##     ##        #####    ####    ###   ###   ####   .
.                                                                           .
.              A set of service virtual machines intended for               .
.            human-computer interaction and automated reasoning             .
.                                                                           .
.                            (PREVIEW DOCUMENT)                             .
. . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . .
```

## table of contents

- [SVM Suite preview document](#svm-suite-preview-document)
    - [A. LOCAL INSTALLATION (WIP)](#a-local-installation)
    - [B. ROUTER.SVM](#b-routersvm)
        - [1. starting and stopping services](#1-starting-and-stopping-services)
        - [2. passing messages between services](#2-passing-messages-between-services)
        - [3. modularity](#3-modularity)
        - [4. afterword](#4-afterword)
    - [C. CONSOLE.SVM](#c-consolesvm)
    - [D. COMPUTE-STATEFUL.SVM](#d-compute-statefulsvm)
        - [1. theoretical background](#1-theoretical-background)
            - [1.1. syntax](#11-syntax)
            - [1.2. semantics](#12-semantics)
                - [1.2.1. temporary and permanent data storage](#121-temporary-and-permanent-data-storage)
                - [1.2.2. pattern matching](#122-pattern-matching)
                - [1.2.3. variables](#123-variables)
                - [1.2.4. modularity](#124-modularity)
        - [2. examples (WIP)](#2-examples)
        - [3. afterword](#3-afterword)
    - [E. COMPUTE-STATELESS.SVM](#e-compute-statelesssvm)
        - [1. theoretical background](#1-theoretical-background-1)
            - [1.1. syntax](#11-syntax-1)
            - [1.2. semantics](#12-semantics-1)
                - [1.2.1. expressions](#121-expressions)
                - [1.2.2. rules](#122-rules)
                - [1.2.3. rule systems](#123-rule-systems)
                - [1.2.4. meta-rules and typing](#124-meta-rules-and-typing)
        - [2. examples (WIP)](#2-examples-1)
        - [3. afterword](#3-afterword-1)

# SVM Suite preview document

Name *SVM Suite* stands for Service Virtual Machine Suite, and it represents a programming framework intended for human-computer interaction and automated reasoning. *SVM Suite* includes a set of service virtual machines built on principles of service oriented programming ([SOP](https://en.wikipedia.org/wiki/Service-oriented_programming)) paradigm. SOP paradigm clearly distincts between different services dedicated to perform given tasks. Such services communicate between each other by passing messages. Benefits of this approach to programming is high modularity needed for code reuse, high agility in process of code development, and granular independence between services that can be invoked parallelly in a multitasking environment.

Some examples that may be represented as services are RAM, permanent storage, keyboard or mouse. Other, more abstract examples may embed higher or lower level computing platforms. By composing these kinds of services together, depending on what the services are, we may build a whole system in the role of programming library, computer application, a sort of operating system, or maybe even an advanced self-controlling system driven by artificial intelligence.

Minimal viable product of *SVM Suite* includes four service virtual machines. Central, *Router.svm* mediates between *Console.svm*, *Compute-stateful.svm*, and *Compute-stateless.svm*. It is not excluded that *SVM Suite* would be enriched by sound, vision, or other similar service virtual machines in the future. These service virtual machines should be just about enough to establish meaningful communication between human and computer, guided by your software inspiration.

> Note that all the code and exchanged data in *SVM Suite* services is written in [s-expression](https://en.wikipedia.org/wiki/S-expression) form borrowed from [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)) family of programming languages. The ingenious form of s-expression was chosen for its very convenient properties related to code-related tasks.

## A. LOCAL INSTALLATION

```
// work in progress //
```

## B. ROUTER.SVM

*Router.svm* is intended to be a router between arbitrary pluggable services. It is conceived as a minimalistic, yet powerful service integrator. As a central hub for exchanging messages between different services, the importance of *Router.svm* may be considerable in spite of the simplicity of code that may represent a specific router instance.

The following code pattern would give an insight to what a router code would look like:

```
(
    ROUTER
    ...
    (
        DIRECT
        (LISTEN (SOURCE ...) (EXP ...))
        (INVOKE (TARGET ...) (EXP ...))
    )
    ...
)
```

In this manner, the router directs messages from one service to another. Each `SOURCE` and `TARGET` section hold a service name, while each `EXP` section holds a s-expression.

### 1. starting and stopping services

Each service has its lifetime during which it sends or receives messages. Services begin their lifetimes using always active built-in service `start`, while they end using also always active built-in service `stop`. When services start or stop, they emit related messages that can be caught by the router. Given that the router service is already running, in such a way, we can intercept a starting or stopping service named `this` denoting the current router. On interception of such messages, we can start or stop other services, which would be the root point of our service management.

There are several key points in starting a service, which we will analyze in the following example:

```
...
(
    DIRECT
    (LISTEN (SOURCE start) (EXP this))
    (
        INVOKE
        (TARGET start)
        (
            EXP
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

First, we listen to the start of `this` service. Thus, when the router starts, we invoke the start of a service of type `console.svm`, named `cns1`, providing some additional parameters. When the console service starts, in further message passing, we will refer to the console by its assigned name `cns1`. Note that we can start any number of instances of the same service type using only different service names. Although this example does not use it, particular services require an additional `src` parameter. Lastly, service type `console.svm` requires a `params` parameter which defines what prompt is displayed in the console when it waits for an input from a user during its lifetime. The important key points of starting a service are:

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
    (LISTEN (SOURCE stop) (EXP this))
    (INVOKE (TARGET stop) (EXP cns1))
)
...
```

stops service named `cns1` when `this` is being stopped.

To resume, if the above two examples are coded in the same router, they start `cns1` service when the router lifetime begins, and stop it when the router lifetime ends.

### 2. passing messages between services

When services are up and running, they receive and emit their input and output messages. To direct and pass around these messages, we use the same pattern from the previous examples. For example, we may have started two services of a type `console.svm` named `cns1` and `cns2`. If we wanted to make, say, a chat system out of them, these two services may be paired in a router to output what the other service inputs while users are typing to their interfaces. To begin with the creation of such a system, let's start from a simple indicator of whether the users input a letter `A` into consoles. Such system would contain the following code:

```
...
(
    DIRECT
    (LISTEN (SOURCE cns1) (EXP (input "A")))
    (INVOKE (TARGET cns2) (EXP (output "the other user typed 'A'")))
)
(
    DIRECT
    (LISTEN (SOURCE cns2) (EXP (input "A")))
    (INVOKE (TARGET cns1) (EXP (output "the other user typed 'A'")))
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
        (LISTEN (SOURCE cns1) (EXP (input <X>)))
        (INVOKE (TARGET cns2) (EXP (output <X>)))
    )
)
(
    MATCH
    (VAR <X>)
    (
        DIRECT
        (LISTEN (SOURCE cns2) (EXP (input <X>)))
        (INVOKE (TARGET cns1) (EXP (output <X>)))
    )
)
...
```

All that is now left to do is to insert some code for starting and stopping services we learned from the previous section, and we have a fully functional chat system between two instances of consoles.

Of course, we may use any number of variables in the `EXP` section, and we can name them however we want. Additionally, we may also use the variables in `SOURCE` and `TARGET` sections if we want to parameterize service names we operate on.

### 3. modularity

To achieve modularity of a router, we may also start other router services from the root one, stacking them in the hierarchical structure. Such modularity may be proven to be useful in more complicated systems where we may want to isolate passing messages of the same sort.

To start a new router, we may write:

```
...
(
    DIRECT
    (LISTEN (SOURCE start) (EXP this))
    (
        INVOKE
        (TARGET start)
        (
            EXP
            (
                service
                (type router.svm)
                (name rt1)
                (src "rt1.srv")
            )
        )
    )
)
...
```

Note the use of `src` parameter. Using this code, we start a router coded in file `rt1.srv`. Similarly, if some other service we start (like *Compute-stateful.svm* or *Compute-stateless.svm*) depends on a particular code, we use the same `src` parameter to specify the code file.

In a direction of modularity, we introduce two more built-in services, source `input` and target `output`. Using them, we make *Router.svm* comply with essential service input/output definition, and we are finally able to pass messages between different routers from a parent node, thus making use of their modularity.

### 4. afterword

In this section, we presented a simple router service vitual machine for directing messages between services. Being a service glue element, *Router.svm* takes a central role among all the service virtual machines in *SVM Suite*. Provided with simple modularity, one may find it easy to imagine a specific system of routers coordinating between *Console.svm*, *Compute-stateful.svm* and *Compute-stateless.svm* services to perform different tasks of interest.

## C. CONSOLE.SVM

*Console.svm* is a simple service virtual machine providing textual console input/output. It consists of a text box with a prompt where the user inputs text. On input, after pressing the <enter> key, the service emits an `input` message with relevant data. Output to the console is managed by sending an `output` message with relevant data to the service. Console service doesn't require any underlying code specific to a particular instance for its functionality.

To see initial examples of using consoles, please refer to the [B. ROUTER.SVM] section.

During a console service runtime, it is possible to change its prompt label from router service by sending it a `prompt` message like in the following example:

```
...
(
    DIRECT
    (LISTEN (SOURCE start) (EXP cns1))
    (
        INVOKE
        (TARGET cns1)
        (EXP (prompt "user>"))
    )
)
...
```

This code sets the prompt of the console to `user>` label when the console service starts.

*Console.svm* is intended to be a default input/output interface to *SVM Suite* based applications. In the case of creating a chatbot, beside the basic conversational interface, consoles can be used for monitoring intermediate thought processes, as we may output a stream of thought during the data computing process.

## D. COMPUTE-STATEFUL.SVM

In computing theory, there are two mainstream branches of programming paradigms: declarative and imperative. Declarative programming typically abstracts from states, providing optimized structures for other forms of computations. However, sometimes, when dealing with states seems the most natural thing to do, imperative programming may be a very good fit because dealing with states is what it does the best. *Compute-stateful.svm* belongs into the category of imperative programming paradigm.

*Compute-stateful.svm* is a service virtual machine providing computing data using state operations. Programs written in this service virtual machine resemble a sort of finite state machine. Programs are composed of a series of only one kind of statements which perform a discrete compute step in a process of computation. Each statement quotes the current step of the program execution, and points to the next step in execution, thus the order of statements is not relevant. Also, each step may input from or output to some memory cell, thus carrying on the computation. 

Finite state machines which *Compute-stateful.svm* appearance is based on, can perform various tasks like driving vending machines, elevators, traffic lights, combination locks, and many others. The limitations of finite state machines to perform only a subset of all possible tasks is surpassed by introducing reading from and writing to arbitrary memory cells during program execution. This places *Compute-stateful.svm* model of computation side by side with [Turing machines model](https://en.wikipedia.org/wiki/Turing_machine) which is known to be the most expressive model of computation.

### 1. theoretical background

*Compute-stateful.svm* service virtual machine combines a kind of deterministic [finite state machine](https://en.wikipedia.org/wiki/Finite-state_machine) model with memory cell assignment and pattern matching. A program running in this service virtual machine can be in exactly one of a finite number of states at any given time. The program begins with a starting state, changes its state in controlled manner during the execution, and finally ends with a stopping state. As the state changes, the program inputs from and outputs to given temporary or permanent memory cells, thus computing new memory data when the current memory data matches the given pattern.

It is possible to draw a directed graph by connecting nodes representing the states, while lines between nodes associate to arbitrary memory cell input and output. Specific nodes may also connect in a cyclic manner, thus forming loops controlled by variable memory cells.

#### 1.1. syntax

[Syntax](https://en.wikipedia.org/wiki/Syntax) is the set of rules that defines the combinations of symbols that are considered to be correctly structured statements or expressions. *Compute-stateful.svm* code itself resembles a type of s-expression. S-expressions consist of lists of atoms or other s-expressions where lists are surrounded by parenthesis. In the code, the first list element to the left determines a type of a list. There are a few predefined list types used for data computing depicted by the following relaxed kind of [Backus-Naur form](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form) rules:

```
    <start> := (STATEFUL <statement>+)
    
<statement> := (COMPUTE <current> <next>)

  <current> := (CURR (STATE <ATOM>) <input>?)
  
    <input> := (INPUT (EXP <S-EXPR>))
             | (INPUT (<cell> <ATOM>) (EXP <S-EXPR>))

     <next> := (NEXT (STATE <ATOM>) <output>?)

   <output> := (OUTPUT (EXP <S-EXPR>))
             | (OUTPUT (<cell> <ATOM>) (EXP <S-EXPR>))

     <cell> := TEMP
             | PERM
```

The above grammar rules define the syntax of a *Compute-stateful.svm* service code. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one re more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols are considered as parts of the code.

**Remember** that starting states are always indicated by including `INPUT` section and excluding `<cell>` section within, while stopping states are always indicated by including `OUTPUT` section and excluding `<cell>` section within.

In addition to the above grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//`, and reach to the end of line. Multiline comments begin with `/*` and end with `*/`, so that everything in between is considered as a comment.

#### 1.2. semantics

Word "[semantics](https://en.wikipedia.org/wiki/Semantics)" is another word for "meaning". In this section, we are dealing with an intuitive semantics of *Compute-stateful.svm* code. Semantics of the code will be explained using various simplistic examples, describing how enter and exit the computation, how the states and memory cells change, and what inputs and outputs the whole examples accept and generate.

##### 1.2.1. temporary and permanent data storage

Data in *Compute-stateful.svm* is stored in so-called memory cells. Memory cells hold arbitrary complex s-expressions, and can be temporary or permanent. Computing with temporary cells is much faster than computing with permanent cells since temporary cells are typically stored in RAM. Computing with permanent cells is much slower than computing with temporary cells since permanent cells are typically stored on disk drives.

###### temporary data storage

Temporary memory cells are noted by `TEMP` sections, and they live from the moment they are created until reaching the stopping state, or until their data is set to `NIL`.

Let's consider the next example:

```
(
    STATEFUL
    (
        COMPUTE
        (CURR (STATE start) (INPUT (EXP NIL))
        (NEXT (STATE 1))
    )
    (
        COMPUTE
        (CURR (STATE 1))
        (
            NEXT
            (STATE 2)
            (OUTPUT (TEMP X) (EXP fst))
        )
    )
    (
        COMPUTE
        (CURR (STATE 2))
        (
            NEXT
            (STATE 3)
            (OUTPUT (TEMP Y) (EXP snd))
        )
    )
    (
        COMPUTE
        (CURR (STATE 3))
        (
            NEXT
            (STATE 4)
            (OUTPUT (TEMP Z) (EXP ((temp X) (temp Y))))
        )
    )
    (
        COMPUTE
        (CURR (STATE 4))
        (NEXT (STATE stop) (OUTPUT (EXP NIL)))
    )
)
```

Starting and stopping states are set to `start` and `stop` states. The service takes no input nor returns an output because they are set to `NIL`. Moving through states from `start` to `stop`, the following computations apply:

- we enter the service at state `start`, and we move to state `1`
- after reaching state `1`, we store the atom `fst` in memory cell `X`
- after reaching state `2`, we store the atom `snd` in memory cell `Y`
- after reaching state `3`, we store the expression `(fst snd)` in memory cell `Z`. This is done by using `temp` sections to refer to memory cells named `X` and `Y`
- after reaching state `4`, we exit the service

Computing processes using only temporary storage are [referentially transparent](https://en.wikipedia.org/wiki/Referential_transparency) from the world outside of the processes.

###### permanent data storage

Permanent memory cells are noted by `PERM` sections, and they live from the moment they are created, until their data is set to `NIL`, even between starting and stopping particular *Compute-stateful.svm* services.

The next example:

```
(
    STATEFUL
    (
        COMPUTE
        (CURR (STATE start) (INPUT (EXP NIL)))
        (NEXT (STATE 1))
    )
    (
        COMPUTE
        (CURR (STATE 1))
        (
            NEXT
            (STATE 2)
            (OUTPUT (PERM "dirX/fileX") (EXP "fst"))
        )
    )
    (
        COMPUTE
        (CURR (STATE 2))
        (
            NEXT
            (STATE 3)
            (OUTPUT (PERM "dirY/fileY") (EXP "snd"))
        )
    )
    (
        COMPUTE
        (CURR (STATE 3))
        (
            NEXT
            (STATE 4)
            (OUTPUT (PERM "dirZ/fileZ") (EXP ((perm "dirX/fileX") (perm "dirY/fileY"))))
        )
    )
    (
        COMPUTE
        (CURR (STATE 4))
        (NEXT (STATE stop) (OUTPUT (EXP NIL)))
    )
)
```

is completely analogous to the one in [temporary data storage] section. The only difference is that we use permanent storage cells instead of temporary ones. Such use stores data in the file system, while `PERM` sections hold paths to specific files. Relatedly, `perm` sections refer to those files. Permanent data storage introduces referential opaqueness from the outside world of the process.

##### 1.2.2. pattern matching

We can also define multiple `COMPUTE` steps with the same `CURR` state which may introduce possible branching choices. When the next computing step may land at multiple steps, the service chooses to select the first available step with the same name from the top of the code, whose `INPUT` matches against the memory cell contents. Thus, the example:

```
(
    STATEFUL
    (
        COMPUTE
        (CURR (STATE start) (INPUT (EXP NIL))
        (
            NEXT
            (STATE 1)
            (OUTPUT (TEMP X) (EXP snd))
        )
    )
    (
        COMPUTE
        (
            CURR
            (STATE 1)
            (INPUT (TEMP X) (EXP fst))
        )
        (
            NEXT
            (STATE 2)
            (OUTPUT (TEMP Y) (EXP "choice one"))
        )
    )
    (
        COMPUTE
        (
            CURR
            (STATE 1)
            (INPUT (TEMP X) (EXP snd))
        )
        (
            NEXT
            (STATE 2)
            (OUTPUT (TEMP Y) (EXP "choice two"))
        )
    )
    (
        COMPUTE
        (CURR (STATE 2))
        (NEXT (STATE stop) (EXP NIL))
    )
```

After storing the atom `snd` in the cell `X`, the above example chooses to stores the atom `choice two` in the cell `Y`. Constructs like this may also be useful when we implement loops by employing circular references. In such cases, branching choices may save us from infinite loops.

##### 1.2.3. variables

Similarly to *Router.svm* services, we may want to make use of variables. In a similar manner, variables are specified using `MATCH` and `VAR` sections:

```
(
    STATEFUL
    (
        MATCH
        (
            (VAR <X> <Y>)
            (
                COMPUTE
                (
                    CURR
                    (STATE start)
                    (INPUT (EXP (<X> <Y>)))
                )
                (
                    NEXT
                    (STATE stop)
                    (OUTPUT (EXP (<Y> <X>)))
                )
            )
        )
    )
)
```

This time we are actually making the service takes an input and gives an output. The above example inputs the s-expression of two elements, and outputs them in a swapped position. To reuse it in the following section, let's declare this example as saved in a file `swap.srv`.

##### 1.2.4. modularity

Defining computing streams may become very complex. To tackle this problem, each *Compute-stateful.svm* service may scatter its code in many files and directories. We can then invoke such files by previously declaring them in `IMPORT` sections:

```
(
    STATEFUL
    (IMPORT (NAME swap) (SRC "swap.srv"))
    (
        MATCH
        (
            (VAR <X> <Y>)
            (
                COMPUTE
                (
                    CURR
                    (STATE start)
                    (INPUT (EXP (<X> <Y>)))
                )
                (
                    NEXT
                    (STATE stop)
                    (OUTPUT (EXP (swap (<X> <Y>))))
                )
            )
        )
    )
)
```

This example also inputs a pair of elements and outputs them swapped up, only now using the `swap` section in output to invoke the `swap.srv` code. Let's just mention that, in a similar way, we can also safely import and invoke any *Compute-stateless.svm* service from the *Compute-stateful.svm* services.

### 2. examples

```
// work in progress //
```

### 3. afterword

In this section we exposed basic constructs of *Compute-stateful.svm* services. Although there are many cases where dealing with states may pose a stumbling stone in writing programs, there are some cases where being able to express states is very desirable. And that is the purpose of *Compute-stateful.svm* service virtual machine, to easily express operations with states. Here, we only scratched the surface of what *Compute-stateful.svm* services are capable of. Since we are dealing with a [Turing complete](https://en.wikipedia.org/wiki/Turing_completeness) system, we may expect that we are not bound in any way considering the domain of supported computations.

## E. COMPUTE-STATELESS.SVM

*Compute-stateless.svm* is planned to be a service virtual machine providing stateless computing operations. Such operations, using this service virtual machine, may be suitable to perform automated reasoning tasks.

Characteristic of nowadays widespread [imperative programming](https://en.wikipedia.org/wiki/Imperative_programming) is manual managing of state dynamics by program instructions to produce wanted states. In contrast to imperative, [declarative programming](https://en.wikipedia.org/wiki/Declarative_programming) abstracts from states using descriptions usually represented by rules repeatedly applied to parameters in a goal of producing wanted results. *Compute-stateless.svm* belongs into the category of declarative programming paradigm.

Two most prominent types of declarative programming are [functional](https://en.wikipedia.org/wiki/Functional_programming) and [logic programming](https://en.wikipedia.org/wiki/Logic_programming). Presenting a novel [algebraic](https://en.wikipedia.org/wiki/Algebraic_data_type) [term graph](https://en.wikipedia.org/wiki/Term_graph) [rewriting](https://en.wikipedia.org/wiki/Rewriting) approach, *Compute-stateless.svm* exhibits properties of both functional and logic programming worlds without a special treatment of either paradigm. This is made possible by using rewriting rules in a form of positive [sequents](https://en.wikipedia.org/wiki/Sequent).

Strictly speaking, *Compute-stateless.svm* programming expressions span on a level below functional and logic programming, representing a [rule-based system](https://en.wikipedia.org/wiki/Rule-based_system). We may consider *Compute-stateless.svm* as an "assembler" for declarative programming. Behavior of rewriting rules in *Compute-stateless.svm* is very similar to behavior of [production rules](https://en.wikipedia.org/wiki/Production_(computer_science)) in [parsing](https://en.wikipedia.org/wiki/Parsing) expressions, additionally allowing non deterministic expression matching to take a place during the parsing process. Non deterministic matching is something that naturally arises from using an extended version of production rules involving [conjunctions](https://en.wikipedia.org/wiki/Logical_conjunction) on the left and [disjunctions](https://en.wikipedia.org/wiki/Logical_disjunction) on the right side of the rules, which are exactly qualities belonging to sequents.

The determination of input and output types makes *Compute-stateless.svm* a typed language. Using typed rules, *Compute-stateless.svm* term graph rewriting algebra is based on implicative and its dual, co-implicative rewriting. Implication and co-implication show perfectly symmetrical behavior when applying them on the same set of rules in processes of forward and backward chaining. Thus, both processes may be implemented by the same algorithm, changing only the direction of applying rules. Dual reasoning in *Compute-stateless.svm* flows by rules from two sides between input and output typing rules, connecting the two referent end points during rules application. Typing in *Compute-stateless.svm* is made possible by observing each rule as a function from its input to its output. The input side of a rule may be considered as a set of accepting values (input type), while the output side may be considered as a set of producing values (output type). In between the input and output end points, we may place a set of chaining rules (the function body) that maps different values of the input type to different values of the output type, turning the rule into a typed function. Because both input and output types may be represented by a set of embedded rewriting rules, we finally get an uniform appearance of all three notions: input, chain, and output, each consisted of their own set of rules, altogether recursively forming a single composite rule in a role of a typed function.

Finally, sequents in *Compute-stateless.svm* operate on [s-expression](https://en.wikipedia.org/wiki/S-expression) data. S-expressions are a valuable heritage of the [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)) family of programming languages. Being a simple, but powerful data definition format, s-expressions make *Compute-stateless.svm* suitable for symbolic data analysis and synthesis in the surrounding functional-logic environment.

### 1. theoretical background

As a declarative programming language, *Compute-stateless.svm* implements a term graph rewriting system to be a blend of functional and logical inference engine.

Term graph rewriting is a method of reconstructing one form of data from another form of data. In this reconstruction, a new data may be introduced, or existing data may be eliminated or reshaped to suit our requirements. To be able to do this, *Compute-stateless.svm* uses a set of user definable rules of a form similar to formulas in mathematics, with the difference that *Compute-stateless.svm* rules may transform not only math expressions, but also any kind of data in a form of s-expressions.

Logical inference in *Compute-stateless.svm* implicitly relates existing rules or their parts by proper logical connectives. Thus, each rule in *Compute-stateless.svm* becomes logical implication, while their mutual interrelation simplifies logical reasoning about different forms of data they operate on. This logical reasoning corresponds to a kind of logic that naturally emerges from the algebraic aspect of rules, which is conveniently captured by [sequent](https://en.wikipedia.org/wiki/Sequent) like rules.

To show a clear correspondence between *Compute-stateless.svm* rules and sequents, we will use an analogy in which *Compute-stateless.svm* may be seen as a form of a [deductive system](https://en.wikipedia.org/wiki/Formal_system#Deductive_system) specifically adjusted to operate on s-expressions. Let's shortly overview the most used deductive systems in area of logical inference. [Hilbert style deduction](https://en.wikipedia.org/wiki/Hilbert_system), [natural deduction](https://en.wikipedia.org/wiki/Natural_deduction), and [sequent calculus](https://en.wikipedia.org/wiki/Sequent_calculus) all belong to a class of deductive systems. They are characterized by respectively increasing number of primitive logical operators. Hilbert style deduction incorporates only [implication](https://en.wikipedia.org/wiki/Material_conditional) where all injected rules take a form of `A -> B`. Natural deduction adds conjunction to the left implication side, so that rules take a form of `A1 /\ A2 /\ ... /\ An -> B`. Sequent calculus further extends the basic language by including right side disjunction, like in `A1 /\ A2 /\ ... /\ An -> B1 \/ B2 \/ ... \/ Bn`.

The price paid for the simple syntax of Hilbert-style deduction is that complete formal proofs tend to get extremely long. In contrast, more complex syntax like in natural deduction or sequent calculus leads to shorter formal proofs. This difference in proof lengths exists because often, on higher levels of abstraction, we would want to use the benefits of conjunction and disjunction constructs. Since Hilbert-style deduction doesn't provide these constructs as primitive operators, we would have to bring their explicit definitions into the implicational proof system, which could be avoided in a case of natural deduction to a certain extent, or sequent calculus to even greater extent.

In contrast to Hilbert style deduction and natural deduction, sequent calculus comes in a package with a full set of mappings from basic [logical connectives](https://en.wikipedia.org/wiki/Logical_connective) to uniform sequents. Logic operators appear natural enough to be fluently used in performing inference, while *sequents* appear simple enough to be reasoned about, which are both important qualities for choosing a base for underlying inference algorithm. One may say that sequent calculus characteristic mappings from logical connectives to sequents may seem imbued with elegant symmetry. In cases of Hilbert style deduction and natural deduction, lack of these mappings is compensated by non-primitive definitions of logical connectives, but we take a stand that those definitions do not reflect the elegance found in sequent calculus mappings.

Although sequent calculus, compared to Hilbert style deduction and natural deduction, may not seem like the simplest solution at first glance, we find it reasonable to base *Compute-stateless.svm* exactly on sequent calculus because, in the long run, benefits may seem to be worth the effort. After all, the simplistic duality elegance of sequent calculus transformations seem too valuable to be left aside in favor of simpler systems. We are taking a stand that the mentioned duality deserves a special treatment which sequent calculus provides us with by its definition. Thus, we choose sequent calculus as a foundation basis for performing inference in *Compute-stateless.svm*.

By the definition, *Compute-stateless.svm* borrows *sequents* from sequent calculus, and extends them by a notion of variables. Although *Compute-stateless.svm* is sharing some primitive foundations with sequent calculus, beyond borrowed sequents, it employs its own proving method during logical reasoning process, namely making use of [constructive proofs](https://en.wikipedia.org/wiki/Constructive_proof). This allows us to generate a meaningful s-expression output upon providing a computational rule system and s-expression input.

#### 1.1. syntax

In computer science, the [syntax](https://en.wikipedia.org/wiki/Syntax) of a computer language is the set of rules that defines the combinations of symbols that are considered to be correctly structured statements or expressions in that language. *Compute-stateless.svm* language itself resembles a kind of s-expression. S-expressions consist of lists of atoms or other s-expressions where lists are surrounded by parenthesis. In *Compute-stateless.svm*, the first list element to the left determines a type of a list. There are a few predefined list types used for data transformation depicted by the following relaxed kind of [Backus-Naur form](https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form) rules:

```
       <start> := (STATELESS <expression>)

  <expression> := (EXP <S-EXPR>)
                | (RULE (READ <expression>+) (CHAIN <expression>+)? (WRITE <expression>+))
                | (MATCH (VAR <ATOM>+) <expression>)
```

The above grammar rules define the syntax of *Compute-stateless.svm*. To interpret these grammar rules, we use special symbols: `<...>` for noting identifiers, `... := ...` for expressing assignment, `...+` for one or more occurrences, `...?` for optional appearance, and `... | ...` for alternation between expressions. All other symbols are considered as parts of the *Compute-stateless.svm* language.

In addition to the above grammar, user comments have no meaning to the system, but may be descriptive to readers, and may be placed wherever a whitespace is expected. Single line comments begin with `//`, and reach to the end of line. Multiline comments begin with `/*` and end with `*/`, so that everything in between is considered as a comment.

In *Compute-stateless.svm* language, there is no additional type checking, meaning that every expression valid in the above grammar is also valid in *Compute-stateless.svm*. However, we may form *Compute-stateless.svm* valid constructions that regardless of input may never return a successful response. Some of such cases indicate inconsistency in rules and always cause an input or output error.

Other such cases are not indicated by *Compute-stateless.svm* because they may already fall into the category of theoretically undecidable proof constructions. One strategy to deal with these cases is to restrict the rule recursion depth, suppressing further computation if the recursion reaches the posed limit. For more information about this undecidability, interested readers are invited to examine [Gödel's incompleteness theorems](https://en.wikipedia.org/wiki/G%C3%B6del%27s_incompleteness_theorems) and [halting problem](https://en.wikipedia.org/wiki/Halting_problem).

#### 1.2. semantics

[Semantics](https://en.wikipedia.org/wiki/Semantics) is the study of meaning, reference, or truth. In our understanding, semantics is tightly bound to interpretation of syntactically correct expressions. To know what an expression means, it is enough to know how it translates to a form that is already understood by a target environment. In this section, we are dealing with the intuitive semantics of *Compute-stateless.svm*. Semantics of *Compute-stateless.svm* will be explained using various simplistic examples and defining what inputs and outputs the examples accept and generate.

##### 1.2.1. expressions

*Compute-stateless.svm* operates on s-expressions written in `EXP` sections. The simplest example would be outputting an s-expression:

```
/*
    hi there example
    
     input: NIL
    output: `(hi there)'
*/

(
    STATELESS
    (EXP (hi there))
)
```

This example inputs `NIL` atom and outputs `(hi there)` s-expression.

##### 1.2.2. rules

Rules are analogous to implications. They have their input `READ` side and output `WRITE` side, each containing a set of `EXP` sections as conjunction on the `READ` side and a set of `EXP` sections as disjunction on the `WRITE` side.

###### constants

Constants are just that, constant s-expressions without variables:

```
/*
    hello world example
    
     input: `(hello machine)`
    output: `(hello world)`
*/

(
    STATELESS
    (RULE (READ (EXP (hello machine))) (WRITE (EXP (hello world))))
)
```

This example inputs s-expression `(hello machine)` and outputs s-expression `(hello world)`.

###### variables

Variables stand for unknown s-expressions that are yet to be specified. The use of variables is noted using `MATCH` and `VAR` sections:

```
/*
    hello entity example
    
     input: `(greet <name>)`
    output: `(hello <name>)`
*/

(
    STATELESS
    (
        MATCH
        (VAR <X>)
        (RULE (READ (EXP (greet <X>))) (WRITE (EXP (hello <X>))))
    )
)
```

Within a rule, there can be any number of variables, and we can name them however we want. Thus, if we pass an input `(greet John)` to the above example, we get an output `(ḣello John)`.

##### 1.2.3. rule systems

Rule systems are sets of rules working together to produce some results. In this section we are dealing with untyped rule systems using composite rules. Because types in composite rules always have to be specified, to simulate untyped behavior, we make use of variables in surrounding `READ` and `WRITE` sections, noting generic expressions in a form of `(MATCH (VAR <X>) (EXP <X>))`. These expressions stand for any kind of s-expressions.

###### constants

Let's consider a rule system of constant rules:

```
/*
    toy making decision
    
     input: `(isGood girl/boy)`
    output: `(makeToy doll/car)`
*/

(
    STATELESS
    (
        RULE
        (
            READ
            (MATCH (VAR <I>) (EXP <I>))
        )
        (
            CHAIN
            (RULE (READ (EXP (isGood girl))) (WRITE (EXP (makeToy doll))))
            (RULE (READ (EXP (isGood boy) )) (WRITE (EXP (makeToy car) )))
        )
        (
            WRITE
            (MATCH (VAR <O>) (EXP <O>))
        )
    )
)
```

The root rule has a `CHAIN` section composed of two other rules. What rule will be applied to an input, is determined by pattern matching of the rules `READ` side against the input. Thus, inputting `(isGood girl)` to the above example outputs `(makeToy doll)`, while inputting `(isGood boy)` outputs `(makeToy car)`.

###### variables

And here is another rule system, this time using variables:

```
/*
    job title decision
    
     input: `(isDoing <Name> drivingRocket/healingPeople)`
    output: `(isTitled <Name> astronaut/doctor)`
*/

(
    STATELESS
    (
        RULE
        (
            READ
            (MATCH (VAR <I>) (EXP <I>))
        )
        (
            CHAIN
            (
                MATCH
                (VAR <Name>)
                (
                    RULE
                    (READ (EXP (isDoing <Name> drivingRocket)))
                    (WRITE (EXP (isTitled <Name> astronaut)))
                )
            )
            (
                MATCH
                (VAR <Name>)
                (
                    RULE
                    (READ (EXP (isDoing <Name> healingPeople)))
                    (WRITE (EXP (isTitled <Name> doctor)))
                )
            )
        )
        (
            WRITE
            (MATCH (VAR <O>) (EXP <O>))
        )
    )
)
```

If we, for example, input the expression `(isDoing Jane healingPeople)`, we get the output `(isTitled Jane doctor)`.


###### chaining rules together

Rule systems may be composed of rules that chain their `WRITE` side to other rules' `READ` side in producing results. Thus, the two rules `(RULE (READ (EXP a)) (WRITE (EXP b)))` and `(RULE (READ (EXP b)) (WRITE (EXP c)))` may chain into a new rule `(RULE (READ (EXP a)) (WRITE (EXP c)))` in a process of intermediate pattern matching against the expression `b`.

To show in example:

```
/*
    job title decision
    
     input: `thereIsADoctor`
    output: `peopleAreHealthy`
*/

(
    STATELESS
    (
        RULE
        (
            READ
            (MATCH (VAR <I>) (EXP <I>))
        )
        (
            CHAIN
            (
                RULE
                (READ (EXP thereIsADoctor))
                (WRITE (EXP peopleAreHealed))
            )
            (
                RULE
                (READ (EXP peopleAreHealed))
                (WRITE (EXP peopleAreHealthy))
            )
        )
        (
            WRITE
            (MATCH (VAR <O>) (EXP <O>))
        )
    )
)
```

passing input `thereIsADoctor` chains the two rules, finally producing the output `peopleAreHealthy`.

###### non deterministic rules

Rules stand for implications with conjunctions on the `READ` sides and disjunctions on the `WRITE` sides. Using this feature, we can compose rules in a [non deterministic](https://en.wikipedia.org/wiki/Nondeterministic_programming) manner.

Having multiple elements on the `WRITE` sides in one rule, each of those elements may be pattern matched against all the elements of any other rule `READ` side. If such a match is met, and the `WRITE` sides of the later rules are all equal, we may produce the `WRITE` sides of the later rules as a result of the chaining process.

This behavior follows from properties of logic implications in proof construction process:

```
 A -> B \/ C,  B -> D,  C -> D
-------------------------------
            A -> D
```

To show it in example:

```
/*
    student decision
    
     input: `(isBeingEducated <Name>)`
    output: `(isAStudent <Name>)`
*/

(
    STATELESS
    (
        RULE
        (
            READ
            (MATCH (VAR <I>) (EXP <I>))
        )
        (
            CHAIN
            (
                MATCH
                (VAR <Name>)
                (
                    RULE
                    (READ (EXP (isBeingEducated <Name>)))
                    (
                        WRITE
                        (EXP (attendsSchool <Name>))
                        (EXP (attendsCollege <Name>))
                    )
                )
            )
            (
                MATCH
                (VAR <Name>)
                (
                    RULE
                    (READ (EXP (attendsSchool <Name>)))
                    (WRITE (EXP (isAStudent <Name>)))
                )
            )
            (
                MATCH
                (VAR <Name>)
                (
                    RULE
                    (READ (EXP (attendsCollege <Name>)))
                    (WRITE (EXP (isAStudent <Name>)))
                )
            )
        )
        (
            WRITE
            (MATCH (VAR <O>) (EXP <O>))
        )
    )
)
```

The first rule `WRITE` side holds a disjunction where each element matches against `READ` sides of the second and the third rule. Having the same `WRITE` side in the second and the third rule, the chaining may be performed. Thus, inputting `(isBeingEducated Jane)` in this example results with `(isAStudent Jane)` output.

Similar analogy holds in the symmetric case. Having multiple elements on the `READ` sides in one rule, each of those elements may be pattern matched against all the elements of any other rule `WRITE` side. If the such match is met, and the `READ` sides of the later rules are all equal, we may produce the `WRITE` sides of the former rule as a result of the rule chaining process.

This behavior follows from properties of logic implications in proof construction process:

```
 A -> B,  A -> C,  B /\ C -> D
-------------------------------
            A -> D
```

To show it in example:

```
/*
    computer expert decision
    
     input: `(buildsARobot <Name>)`
    output: `(isAComputerExpert <Name>)`
*/

(
    STATELESS
    (
        RULE
        (
            READ
            (MATCH (VAR <I>) (EXP <I>))
        )
        (
            CHAIN
            (
                MATCH
                (VAR <Name>)
                (
                    RULE
                    (READ (EXP (buildsARobot <Name>)))
                    (WRITE (EXP (mastersHardware <Name>)))
                )
            )
            (
                MATCH
                (VAR <Name>)
                (
                    RULE
                    (READ (EXP (buildsARobot <Name>)))
                    (WRITE (EXP (mastersSoftware <Name>)))
                )
            )
            (
                MATCH
                (VAR <Name>)
                (
                    RULE
                    (
                        READ
                        (EXP (mastersSoftware <Name>))
                        (EXP (mastersHardware <Name>))
                    )
                    (WRITE (EXP (isAComputerExpert <Name>)))
                )
            )
        )
        (
            WRITE
            (MATCH (VAR <O>) (EXP <O>))
        )
    )
)
```

The third rule `READ` side holds a conjunction where each element matches against `WRITE` sides of the first and second rule. Having the same `READ` side of the first and second rule, the chaining may be performed. Thus, inputting `(buildsARobot John)` in this example results with `(isAComputerExpert John)` output.

##### 1.2.4. meta-rules and typing

After exposing all of the examples above, we can finally introduce meta-rules and typing in *Compute-stateless.svm*. By recursively embedding rules within `READ` and `WRITE` sections, we enter the world of seamless rules treatment. That way we can combine rules to the arbitrary complexity measure.

###### meta-rules

Meta-rules are rules whose `READ` or `WRITE` sections contain other rules. In `READ` sections, starting from mandatory standalone `EXP` sections, pattern matching is performed by applying chaining rules towards an input expression. This process is analogous to applying production rules in a process of code parsing. If the input expression can be derived in this process, we proceed to the `WRITE` section to produce the output. In the `WRITE` section, output is produced again starting from mandatory standalone `EXP` sections, but this time we chain the rules backwards, from right to left. If there is more than one production available in constructing the output, the first one is chosen after there are no more applicable rules.

Backward chaining in `WRITE` sections is analogous to forward chaining in `READ` sections. While forward chaining is based on logic [implications](https://en.wikipedia.org/wiki/Material_conditional) with conjunctions and disjunctions, backward chaining is based on their dual, co-implications with disjunctions and conjunctions, respectively. Both processes are perfectly symmetrical from the standpoint of an implementation algorithm, only that in forward chaining we move from left to right, while in backward chaining we move from right to left.

In logic, co-implication operator is not studied as well as the more common implication operator. Co-implication is an expression that we get by negating an implication expression. Logic inference on such expressions is diametrically opposite to traditional logic inference, meaning that for all the inference rules that we can apply within implications, we can apply their inverse within co-implications. This fact follows from the simple way of treating co-implications: when we negate a co-implication expression, we can apply ordinary inference rules on the negation, and when we afterwards negate it back, what we get are results of the inference over the original co-implication expression.

The following example depicts a metarule:

```
/*
    world spinning decision
    
     input: `(peopleAre happy/sad)`
    output: `(stillTurns world)`
*/

(
    STATELESS
    (
        RULE
        (
            READ
            (EXP (peopleAre <mood>))
            (RULE (READ (EXP <mood>)) (WRITE (EXP happy)))
            (RULE (READ (EXP <mood>)) (WRITE (EXP sad  )))
        )
        (
            WRITE
            (RULE (READ (EXP world)) (WRITE (EXP <object>)))
            (EXP (stillTurns <object>))
        )
    )
)
```

This example inputs either `(peopleAre happy)` or `(peopleAre sad)` and outputs `(stillTurns world)` regardless of the passed input.

###### typed rules

Typed rules are rules that include `CHAIN` section in their body, along with `READ` and `WRITE` sections. `READ` and `WRITE` sections in typed rules behave exactly like those in meta-rules, only that now they stand for input and output types ready to be checked against input and output expressions. Actual computation is being performed within the `CHAIN` section containing chaining rules that we already encountered in previous examples.

Thus, the example:

```
/*
    weighting decision
    
     input: `(orbitsAround Sun/Earth/Moon Sun/Earth/Moon)`
    output: `(weightsMoreThan Sun/Earth/Moon Sun/Earth/Moon)`
*/

(
    STATELESS
    (
        RULE
        (
            READ
            (EXP (orbitsAround <object> <object>))
            (RULE (READ (EXP <object>)) (WRITE (EXP Sun  )))
            (RULE (READ (EXP <object>)) (WRITE (EXP Earth)))
            (RULE (READ (EXP <object>)) (WRITE (EXP Moon )))
        )
        (
            CHAIN
            (
                MATCH
                (VAR <O1> <O2>)
                (
                    RULE
                    (READ (EXP (orbitsAround <O1> <O2>)))
                    (WRITE (EXP (attractsMoreThan <O2> <O1>)))
                )
            )
            (
                MATCH
                (VAR <O1> <O2>)
                (
                    RULE
                    (READ (EXP (attractsMoreThan <O1> <O2>)))
                    (WRITE (EXP (weightsMoreThan <O1> <O2>)))
                )
            )
        )
        (
            WRITE
            (RULE (READ (EXP Sun  )) (WRITE (EXP <object>)))
            (RULE (READ (EXP Earth)) (WRITE (EXP <object>)))
            (RULE (READ (EXP Moon )) (WRITE (EXP <object>)))
            (EXP (weightsMoreThan <object> <object>))
        )
    )
)
```

expects an input in a form of `(orbitsAround <X> <Y>)` where `<X>` and `<Y>` stand for `Sun`, `Earth`, or `Moon`. The input is pattern matched against rules in the `READ` section. If the input is correct, the rule chaining is performed to compute the actual result. After computation, the result is pattern matched against rules in the `WRITE` section. As the matching passes, finally, the output in the form of `(weightsMoreThan <Y> <X>)` is produced.

### 2. examples

```
// work in progress //
```

### 3. afterword

If properly performed, there could be numerous kinds of uses of the *Compute-stateless.svm* inference mechanism. One use may be in editing input in sessions that produce some mathematical, logical, or other kinds of computations, while looping back to editing sessions until we are satisfied with the output. Some other, maybe industrial use may involve compiling a program source code to some assembly target code. In other situations, it is also included that we could form a personal, classical business, or even scientific knowledge base with relational algebra rules, so we can navigate, search, and extract wanted information. Ultimately, data from the knowledge base could mutually interact using on-demand learned inference rules, thus developing the entire logical reasoning system ready to draw complex decisions on general system behavior. And this partial sketch of possible uses is just a tip of the iceberg because with a kind of system like *Compute-stateless.svm*, we are entering a nonexhaustive area of general knowledge computing where only our imagination could be a limit.
