# MemGPT
The main way to control a behaviour of an LLM agent is by changing its input or its context window. However its always not so simple to determine how to construct an optimal context window for an LLM, particularly with complex LLM agents.
Context windows need to include everything an agent needs to know to complete a task, including information from various external data sources 
- user data
- prior messages
- results from tool calls
- previous reasoning steps, etc

The MemGPT research paper (https://arxiv.org/pdf/2310.08560) shows how to build an OS for your LLM that manages the context window. In other words, an LLM OS that performs memory management. In MemGPT this OS is also itself an LLM agent. So the memory management is done automatically. 

## Key ideas behind MemGPT
### Self Editing Memory
This refers to the ability of an LLM to edit its own memory. In many LLM applications the system instructions or personalization information of the LLM is fixed. In MemGPT the agent can update its own instructions or personalization information based on things it learns during a chat.
### Inner Thoughts/ Inner Monologue
Agents are always thinking to themselves even if they dont reply directly to a user.
### Every output is a tool
In MemGPT agents always calls tools. For example, when the agent wants to communicate with the user, it has to call the send message tool. The only time an agent does not call a tool is when its outputting inner thoughts.
### Looping via Heartbeats
MemGPT agents are designed to run many LLM steps off of a single user input. For example, if a user ask the agent to do a complicated task, you would expect the agent to run for many steps as it breaks down the different aspects of the problem into sub tasks. In MemGPT, agents are able to loop through the use of heartbeats.
Whenever a MemGPT agent calls a tool, it can add a special heartbeat request to any tool which will trigger a follow up call. 

## Breaking down the context window
In MemGPT, we create a special section of the context window called core memory. Core memory is used to store important information about the user to personalize the agent. Think about how your conversation with your friends is different from your conversation with the strangers.
In MemGPT, the system prompt also includes the information about how to edit the core memory. So the MemGPT agent will see both the special reserved section of the context for long term memory, as well as understand that it has the power to edit its memory if it sees fit.

Core memory can be customized. For example one section can store information about the user and another section can store information about the agent. Core memory in MemGPT is what gives the agent the ability to learn overtime. 
It isn't just another message in the chat history. It is a special reserved section of the context window that is always visible to the agent no matter what. 

### What happens when we run out of space in chat history?
Context windows are finite. So no matter how large your context window in the base LLM is, if your conversation goes on for long enough you'll eventually run out of space. 

In MemGPT, when you run out of space, you first flush or evict a chunck of the messages in the chat history and replace them with a recursive summary. This summary is called recursive because it summarizes all of the messages that were evicted, which may itself include
a previously generated summary. In MemGPT, you never delete a message history. Instead all the messages that are evicted from the context window get inserted into a persistent database called recall memory. 

If an agent wants to use some messages from the recall memory, it can use some tool to search the database for old messages. 

### What happens when we run out of space in core memory?
Core memory is also limited in size similar to chat history. Each section of the core memory has an associated character limit. The fundamental constraint is that the combined system prompt, core memory, summary and chat history must all together fit inside 
the context window of the base LLM that you are using. 

Core memory also has a second tier of unlimited storage, similar to recal memory, called archival memory. A MemGPT agent decides what information is most important to keep in core memory and what should be stored outside of the context window. 
You can think of archival memory as a general datastore for the MemGPT agent. Its all the general information that's not important enough to kepp inside of core memory that is always pinned to the context window at all times.

Because archival memory is a general concept it means you can use it for many different things. You can use the archival memory to store code or PDF documents. 

### External memory statistics
Because archival and recall memory are in external storage, the agent actually cannot see the contents of them unless it explicitly uses some search tools to get the information. This poses a problem.
If valuable information is stored inside an external storage but all the external storage is outside of the context window, how will the agent know where to look in the first place?
This is where the external memory statistics comes in. In MemGPT, there is a special section of the context window that provides statistics about the external memory. For example, if a user asks a questions that is not defined in the core memory or the chat
history, if the agent sees that there is a lot of external memory, it will first check the external memory to see if there is some relevant information.

<img width="600" height="600" alt="Screenshot 2025-09-06 at 2 54 18 PM" src="https://github.com/user-attachments/assets/53e6b7c2-5f50-4131-b73d-666b125da2d2" />

