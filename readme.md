# Interprocess communication using Message queue

Message queues are one of the inter-process communication mechanism which allows processes to exchange data in the form of messages between two processes. It allows processes to communicate asynchronously by sending messages to each other where the messages are stored in a queue, waiting to be processed, and are deleted after being processed.

IPC using message queue involves following steps:

- Defining a Message Structure
- Creating the Message Queue
- Sending the Message to the Process A through the Message Queue
- Receiving the Message from the Process A through the Message Queue

<Image>

Each message is given an identification or "type" so that processes can select the appropriate message and process must share a common "key" in order to gain access to the queue in the first place.

A process can send or receive a message from the queue, the queue must be initialized through *msgget()* operations to send and receive messages are performed by the *msgsnd()* and *msgrcv()* functions respectively.

### msgget() - creating/using a queue

```C
int msgget ( key_t key , int msgflg )
```
- returns (creates) a message queue identifier associated with
the value of the key argument.
- A new message queue is created, if key has the value
IPC_PRIVATE.
- If key isn’t IPC_PRIVATE and no message queue with the
given key exists, the msgflg must be specified to IPC_CREAT
(to create the queue).
- If a queue with key key exists and both IPC_CREAT and
IPC_EXCL are specified in *msgflg*, then *msgget* fails with
errno set to EEXIST.

- IPC_EXCL is used with IPC_CREAT to ensure failure if the segment already exists.
- Upon creation, the least significant bits of *msgflg* define the
permissions of the message queue.
- These permission bits have the same format and semantics as
the permissions specified for the mode argument of *open()*.

### msgsnd() - sending a message to a queue

```C
int msgsnd ( int msqid , const void * msgp , size_t msgsz , int msgflg );
```
- send *msgp* (pointer to a record – see below) to message queue with id *msqid*.
```C 
// msgp structure
struct msgbuf {
long mtype ; /* msg type - must be >0 */
char mtext [ MSGSZ ]; /* msg data */
};
```
- sender must have write-access permission on the message queue to send a message

### msgrcv() – fetching a message from a queue

```C
ssize_t msgrcv (int msqid , void * msgp , size_t msgsz , long msgtyp , int msgflg );
```
- receive a message *msgp* from a message queue with id *msqid*
- *msgtyp* is an integer value.
- if *msgtyp* is zero, the first message is retrieved regardless its
type.
- This value can be used by the receiving process for designating
message selection).
- *mesgsz* specifies the size.
- By and large, *msgflg* is set to 0.

*msgtyp* specifies the type of message requested as follows:
- if *msgtyp = 0* then the first message in the queue is read.
- if *msgtyp > 0* then the first message in the queue of type
*msgtyp* is read.
- if *msgtyp < 0* then the first message in the queue with the
lowest type value is read

### msgctl() - controlling a queue

```C
int msgctl ( int msqid , int cmd , struct msqid_ds * buf )
```

- performs the control operation specified by *cmd* on the
message queue with identifier *msqid*
- The *msqid_ds* structure is defined in *<sys/msg.h>* as:
```C
struct msqid_ds {
    struct ipc_perm msg_perm ; /* Ownership and permissions */
    time_t msg_stime ; /* Time of last msgsnd (2) */
    time_t msg_rtime ; /* Time of last msgrcv (2) */
    time_t msg_ctime ; /* Time of last change */
    unsigned long __msg_cbytes ; /* Current number of bytes
    in queue (non - standard )*/
    msgqnum_t msg_qnum ; /* Current number of
    messages in queue */
    msglen_t msg_qbytes ; /* Maximum number of bytes
    allowed in queue */
    pid_t msg_lspid ; /* PID of last msgsnd (2) */
    pid_t msg_lrpid ; /* PID of last msgrcv (2) */
};
```
## Code snippet

client.c

```C 
#include <stdio.h> 
#include <sys/ipc.h> 
#include <sys/msg.h> 
  
// message queue structure
struct mesg_buffer { 
    long mesg_type; 
    char mesg_text[100]; 
} message; 
  
int main() 
{ 
    key_t key; 
    int msgid; 
  
    // generate unique key 
    key = ftok("somefile", 65); 
  
    // create a message queue and return identifier 
    msgid = msgget(key, 0666 | IPC_CREAT); 
    message.mesg_type = 1; 
  
    printf("Insert message  : "); 
    gets(message.mesg_text); 
  
    // send message 
    msgsnd(msgid, &message, sizeof(message), 0); 
  
    // display the message 
    printf("Message sent to server : %s\n", message.mesg_text); 
  
    return 0; 
}
```
server.c

```C
#include <stdio.h> 
#include <sys/ipc.h> 
#include <sys/msg.h> 
  
// structure for message queue 
struct mesg_buffer { 
    long mesg_type; 
    char mesg_text[100]; 
} message; 
  
int main() 
{ 
    key_t key; 
    int msgid; 
  
    // generate unique key 
    key = ftok("somefile", 65); 
  
    // create a message queue and return identifier 
    msgid = msgget(key, 0666 | IPC_CREAT); 
    
    printf("Waiting for a message from client...\n");

    // receive message 
    msgrcv(msgid, &message, sizeof(message), 1, 0); 
  
    // display the message 
    printf("Message received from client : %s\n",message.mesg_text); 
  
    // to destroy the message queue 
    msgctl(msgid, IPC_RMID, NULL); 
  
    return 0; 
} 
```

start client and server, enter the message to be sent to server
<Image>




