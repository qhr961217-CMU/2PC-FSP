# Two-Phase Commit protocol analysis with FSP

## What is <em>Two-Phase Commit</em> protocol ? 

Two-Phase Commit protocol is a atomic distributed commitment protocol aims to coordinates all the processes that participate for extending transaction concept across multiple nodes.

## What is <em>FSP</em> and why using <em>LTSA</em> ?

The Two-Phase Commit protocol can be described using state machines, known as Labelled Transition Systems (LTS). These are described textually as finite state processes (FSP) and displayed and analysed by the Labeled Transition Systems Analyzer (LTSA) tool.

## Model Assumptions

1. In the two phase commit protocol, whenever the server start a new commit process, it will send requests for all the involved users. In real world, the involved users might be only a part of all potential users. In this model, I assume all users in models are involved users, which means once the server start a new commit, all users in our model will receive the requests.

2. Messages among server and user nodes are delivered by networks. I assume network in the model as a separate process with a series of actions that provides network services.

3. As for server and users actions should be deterministic, even when the users reply their voting mes- sage(i.e. Anytime the same commit message will result in the same voting response from each user). So, in this model, all the packet loss only existing in the network layer. (i.e. the network layer process have the non-deterministic choice to model packet loss feature).

4. The network communication is not a perfectly service that packet loss could happen because of errors in data transmission or network congestion. I assume that all packets delivered in the model might be lost with some possibility. There are four cases that the packets would be lost:

    a. *Commit message*: when the commit message sent from server to user is lost, the server does nothing but wait until timeout. Once timeout happens, the server aborts the commit process.

    b. *Voting message*: when the voting message sent from user to server is lost. After timeout, the server aborts the commit process.

    c. *Decision message*: once server sends the decision message to users, it will wait for acknowledge messages from users. After timeout, the server would resend the decision message to unacknowl- edged users until it receives acknowledge message from all users.

    d. *Decision acknowledge message*: Same as decision message lost, the server will resend the message when timeout happens until it receives acknowledge message from all users.

5. Both server and users might be failed and failure recovery mechanism should deal with commit process recovery. In the model, I assume that all completed actions would be recorded in log files atomically. Therefore, both server and users could be restored by their records in log files.

