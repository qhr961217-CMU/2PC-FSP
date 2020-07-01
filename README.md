# Two-Phase Commit protocol analysis with FSP

## What is <em>Two-Phase Commit</em> protocol ? 

Two-Phase Commit protocol is a atomic distributed commitment protocol aims to coordinates all the processes that participate for extending transaction concept across multiple nodes.

## What is <em>FSP</em> and why using <em>LTSA</em> ?

The Two-Phase Commit protocol can be described using state machines, known as Labelled Transition Systems (LTS). These are described textually as finite state processes (FSP) and displayed and analysed by the Labeled Transition Systems Analyzer (LTSA) tool.

LTSA download: https://www.doc.ic.ac.uk/ltsa/

## Model Architecture
![Two-Phase Commit](documentation/Two-Phase%20Commit%20Diagram.png)

## Model Assumptions

1. In the two phase commit protocol, whenever the server start a new commit process, it will send requests for all the involved users. In real world, the involved users might be only a part of all potential users. In this model, I assume all users in models are involved users, which means once the server start a new commit, all users in our model will receive the requests.

2. Messages among server and user nodes are delivered by networks. I assume network in the model as a separate process with a series of actions that provides network services.

3. As for server and users actions should be deterministic, even when the users reply their voting mes- sage(i.e. Anytime the same commit message will result in the same voting response from each user). So, in this model, all the packet loss only existing in the network layer. (i.e. the network layer process have the non-deterministic choice to model packet loss feature).

4. The network communication is not a perfectly service that packet loss could happen because of errors in data transmission or network congestion. I assume that all packets delivered in the model might be lost with some possibility. There are four cases that the packets would be lost:

    a. *Commit message*: when the commit message sent from server to user is lost, the server does nothing but wait until timeout. Once timeout happens, the server aborts the commit process.

    b. *Voting message*: when the voting message sent from user to server is lost. After timeout, the server aborts the commit process.

    c. *Decision message*: once server sends the decision message to users, it will wait for acknowledge messages from users. After timeout, the server would resend the decision message to unacknowl- edged users until it receives acknowledge message from all users.

    d. *Decision acknowledge message*: Same as decision message lost, the server will resend the message when timeout happens until it receives acknowledge message from all users.

5. The commitment decision result will be made if and only if the server has received all the voting messages even if the user message timeout.


6. Both server and users might be failed and failure recovery mechanism should deal with commit process recovery. In the model, I assume that all completed actions would be recorded in log files atomically. Therefore, both server and users could be restored by their records in log files.


## *Safety* & *Liveness* Properties

### *Safety*

0. The system never deadlocks.
   
1. If all the users voting `"Yes"` on this commit without packet loss in *PHASE 1*, the server will never return `"abort"` decision result.

2. If at least one user voting `"No"` on this commit, the server will never return `"commit"` decision result.

3. If the server does not get response (include `"timeout"`) from any one of the users, the server will never make decision result.

4. If the server does not get all the `"ACKs"` from users, this commit process will never `finish`.

5. If the server make the decision as `"commit"`, all of the users will never received the decision `"abort"`.

6. If the server make the decision as `"abort"`, all of the users will never received the decision `"commit"`.

7. If any packet loss happens in *PHASE 1* (prepare commit), the decision result will never be `"commit"`.



### *Liveness* 

1. If all the users voting `"Yes"` on this commit without packet loss in *PHASE 1*, the server will eventually return `"commit"` decision result.

2. If at least one user voting `"No"` on this commit, the server will eventually return `"abort"` decision result.

3. If the server has already get all the response (include `"timeout"`) from all of the server, the server will eventually make decision result.

4. If the server has already get all the `"ACKs"` from users, this commit process will eventually `finish`.

5. If the server make the decision as `"commit"`, all of the users will eventually received the same decision `"commit"`.

6. If the server make the decision as `"abort"`, all of the users will eventually received the same decision `"abort"`.

7. If any packet loss happens in *PHASE 1* (prepare commit), the decision result will eventually be `"abort"`.

8. The commit process will eventually `finish` even when any packet loss happens in *PHASE 1* (prepare commit).

9.  The commit process will eventually `finish` even when any packet loss happens in *PHASE 2* (distribute decision).

## System Components

### *UserNode*
User node are involved in response voting message and response acknowledgement to decision results. In this model, we will start with 2 user nodes.

1. Voting Message: The user node makes a decision to the new-received prepare commit message. Once decision has been made, the user node saves the decision result and send the decision message to the server.

2. Ack to results: Once the user node receives the decision results message from the server, it would accept the "commit" or "abort" the decision. The user will send the acknowledge message back to the server.


### *Server*
Server node takes the responsibility to initialize the prepare commit process, sending it to each user node, waiting for user’s voting message and distribute the decision result to each users.

1. Initializing commit process: The server node will create a new commit process and send it to all users and wait for their voting messages.

2. Gathering voting messages and make decision: Once the server receive all responses from users, it will make the final decision. If all users say ”yes”, the final decision is ”commit”. If one user says ”no”, the final decision is ”abort”. 

3. Gathering ACKs: The server sends the final decision to all users and waits for acknowledgements.


### *Network*
Network helps server and users communicate in a stable or unstable approach. The network gets packet to the server and sends it to all users. Similarly, the network gets packets from users and sends it to server.