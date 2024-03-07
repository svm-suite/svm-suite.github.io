```
    // under construction //
```

```
project status:
    [ ] alpha conception
        [x] theorizing
        [ ] implementing
    [ ] beta testing and revising code
    [ ] gamma release
```


# SVM Suite instructions

Name *SVM Suite* stands for Service Virtual Machine Suite, and it represents a programming framework intended for automated reasoning and human-computer interaction. *SVM Suite* includes a set of service virtual machines (SVM) built on principles of service oriented programming ([SOP](https://en.wikipedia.org/wiki/Service-oriented_programming)) paradigm. SOP paradigm clearly distincts between services that perform given tasks. Such services communicate between each other by passing messages. Benefits of this approach to programming is high modularity needed for code reuse, high agility in process of code development, and granular independence between services that can be invoked parallelly in a multitasking environment.

Some examples that may be represented as services are RAM, permanent storage, keyboard or mouse. Other, more abstract examples may embed higher or lower level computing platforms. By composing these kinds of services together, depending on what the services are, we may build a whole system in the role of programming library, computer application, operating system, or maybe even enthusiastic self-controlling hardware system driven by artificial intelligence empowered form of existential being.

Minimal viable product of *SVM Suite* includes four services called service virtual machines. Central, *router.svm* mediates between *console.svm*, *compute-stateful.svm*, and *compute-stateless.svm*. These virtual service machines should be just about enough to establish meaningful communication between human and computer, guided by your software inspiration. It is also not excluded that *SVM Suite* would be enriched by sound, vision, or other similar services in the future.

> *Note that all the code and exchanged data in SVM Suite services is written in [s-expression](https://en.wikipedia.org/wiki/S-expression) form borrowed from [Lisp](https://en.wikipedia.org/wiki/Lisp_(programming_language)) family of programming languages. Ingenious s-expression form is chosen because of its very convenient properties relating to code related tasks.*

## 1. router.svm

*Router.svm* is intended to be a router between arbitrary pluggable SVMs. It is conceived as a minimalistic, yet powerful service integrator. As a central hub for exchanging messages between different SVMs, the importance of *router.svm* may be considerable in spite of the simplicity of code that may represent a specific router instance.

The following code pattern would give an insight to what a router code would look like:

    (
        ROUTER
        (
            DIRECT
            (LISTEN (SOURCE <service-name>) (DATA <data>))
            (INVOKE (TARGET <service-name>) (DATA <data>))
        )
        (
            DIRECT
            ...
        )
        ...
    )

In this manner, the router directs messages (contained within `<data>` parts) from one service to another (noted within `<service-name>` parts).

### 1.1. starting and stopping SVMs

Each service has its lifetime during which it sends or receives messages. Services begin their lifetimes using always active built-in service `start`, while they end using also always active built-in service `stop`. When services start or stop, they emit related messages that can be caught by the router. Given that the router service is already running, in such a way, we can intercept a starting or stopping service named `this` denoting the current router. On interception of such messages, we can start or stop other services, which would be the root point of our service management.

There are several key points in starting a service, which we will analyze in the following example:

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
                    SERVICE
                    (TYPE console.svm)
                    (NAME cns1)
                    (PARAMS (prompt "input>"))
                )
            )
        )
    )
    ...

First, we listen to the start of `this` service. Thus, when the router starts, we invoke the start of a service of type `console.svm`, named `cns1`, providing some additional parameters. When the console service starts, in further message passing, we will refer to the console by its assigned name `cns1`. Note that we can start any number of instances of the same service type using only different service names. Lastly, by the parameters passed to `cns1`, related to the type `console.svm`, `cns1` displays the prompt `input>` when it requires input from a user during its lifetime. In short, important key points of starting a service are:

* `SERVICE` where we state that we are starting a service
* `TYPE` where we denote a type of the service to start
* `NAME` where we denote an arbitrary name of the service we will refer to in further message passing
* `PARAMS` where we pass some parameters needed to start the service

Stopping services is somewhat simpler, as we only need the service name to end its lifetime. The following example:

    ...
    (
        DIRECT
        (LISTEN (SOURCE stop) (DATA this))
        (INVOKE (TARGET stop) (DATA cns1))
    )
    ...

stops service named `cns1` when `this` is being stopped.

To resume, if the above two examples are coded in the same router, they start `cns1` service when the router lifetime begins, and stop it when the router lifetime ends.

### 1.2. passing messages between SVMs

When services are up and running, they receive and emit their input and output messages. To direct and pass around these messages, we use the same pattern from the previous examples. For example, we may have started two services of a type `console.svm` named `cns1` and `cns2`. If we wanted to make, say, a chat system out of them, these two services may be paired in a router to output what the other service inputs while end users are typing to their interfaces. To begin with the creation of such a system, let's start from a simple indicator of whether the users input a letter `A` into consoles. Such system would contain the following code:

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

Note how we use data parameters `input` and `output` specific to console instances. Now the both consoles are able to input any text from users, indicating the other user on typing a letter `A`. Of course, this code can hardly be of any use, but we will introduce some new constructs to our router code, namely keywords `MATCH` and `VAR`. Now, we may find the router capable enough not only to program our chat system, but also for many other purposes. This is how we could write the code for our chat system: 

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

All that is now left to do is to insert some code for starting and stopping services we learned from the previous section, and we have a fully functioning chat system between two instances of consoles.

Of course, we may use any number of variables in the `DATA` section, and we can name them however we want. Additionally, we may also use the variables in `SOURCE` and `TARGET` sections if we want to parameterize service names we operate on.

### 1.3. modularity

To achieve modularity of a router, we may also start other router SVMs from the root one, stacking them in the hierarchical structure. Such modularity may be proven to be useful in more complicated systems where we may want to isolate passing messages of the same sort.

To start a new router, we may write:

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
                    SERVICE
                    (TYPE router.svm)
                    (NAME rt1)
                    (PARAMS (file "rt1.svm"))
                )
            )
        )
    )
    ...

Using this code, we start a router coded in file `rt1.svm`.

In a direction of modularity, we introduce two more built-in services, source `input` and target `output`. Using them, we make *router.svm* comply with essential service input/output definition, and we are finally able to pass messages between different routers from a root node router, thus making use of their modularity.

### 1.4 conclusion

In this section, we presented a simple router SVM for directing messages between other SVMs. Being a SVMs glue element, *router.svm* takes a central role among all the SVMs in *SVM Suite*. Provided with simple modularity, one may find it easy to imagine a specific system of routers coordinating between *console.svm*, *compute-stateful.svm* and *compute-stateless.svm* to perform different tasks of interest.

## 2. console.svm

*console.svm* is a simple service virtual machine providing textual console input/output. It consists of a text box with a prompt where the user inputs text. On input, after pressing the <enter> key, an `input` message with relevant data is emitted. Output to the console is managed by sending an `output` message with relevant data from the parent router. To see initial examples of using consoles, please refer to the [router.svm](#1-router-svm) section.

When a console service is running, it is possible to change its prompt label like in the following example:

    ...
    (
        DIRECT
        (LISTEN (SOURCE start) (DATA rt1))
        (
            INVOKE
            (TARGET rt1)
            (DATA (prompt "user>"))
        )
    )
    ...

This code sets the prompt of the console to `user>` label when the console service starts.

## 3. stateful.svm

*stateful.svm* is planned to be a service virtual machine providing state operations and computing data. Data computed using states may be stored in temporary or permanent data storage in computer memory.

    (
        COMPUTE
        (
            CURRENT
            (STATE ...)
            (INPUT (PATH ...) (DATA ...))
        )
        (
            NEXT
            (STATE ...)
            (OUTPUT (PATH ...) (DATA ...))
        )
    )

## 4. stateless.svm

*stateless.svm* is planned to be a service virtual machine providing stateless computing operations. Such operations may be suitable to perform automated reasoning tasks.

    (
        RULE
        (
            READ
            
            (RULE (READ) (WRITE (goes <name> <voice>)))
            
            (RULE (READ <name> ) (WRITE Milo))
            (RULE (READ <name> ) (WRITE Nora))
            (RULE (READ <voice>) (WRITE bark))
            (RULE (READ <voice>) (WRITE meow))
        )
        (
            CHAIN
            
            (
                MATCH
                (VAR <X>)
                (RULE (READ (goes <X> meow)) (WRITE (isA <X> cat)))
            )
            
            (
                MATCH
                (VAR <X>)
                (RULE (READ (goes <X> bark)) (WRITE (isA <X> dog)))
            )
        )
        (
            WRITE
            
            (RULE (READ Milo) (WRITE <name>  ))
            (RULE (READ Nora) (WRITE <name>  ))
            (RULE (READ cat ) (WRITE <living>))
            (RULE (READ dog ) (WRITE <living>))
            
            (RULE (READ (isA <name> <living>)) (WRITE))
        )
    )

```
    // under construction //
```
