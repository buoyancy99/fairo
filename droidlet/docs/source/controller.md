```eval_rst
.. _controller_label:
```
# Controller

In the "abstract" droidlet agent, the controller chooses whether to put Tasks on the Task Stack based on the memory state.   In the locobot agent and the craftassist agent subclasses, it consists of

* a [DSL](https://github.com/facebookresearch/fairo/tree/main/droidlet/documents/logical_form_specification)
* a neural semantic parser, which translates natural language into partially specified programs over the DSL
* a Dialogue Manager, Dialogue Stack, and Dialogue objects.
* the Intepreter, a special Dialogue Object that takes partially specified programs from the DSL and fully specifies them using the Memory
* a set of "default behaviors", run randomly when the Task Stack and Dialogue Stack are empty
```eval_rst
Dialogue Objects behave similarly to :ref:`tasks_label` , except they only affect the agent's environment directly by causing the agent to issue utterances (or indirectly by pushing Task Objects onto the Task Stack).  In particular, each Dialogue Object has a .step() that is run when it is the highest priority object on the Stack. Dialogue Objects, like Task Objects are modular:  a learned model or a heuristic can mediate the Dialogue Object, and the same model or heuristic script can be used across many different agents.
```

The Dialogue Manager puts Dialogue Objects on the Dialogue Stack, either on its own, or at the request of a Dialogue Object.  In the locobot and craftassist agent, the manager is powered by a [neural semantic parser](https://github.com/facebookresearch/fairo/tree/main/droidlet/perception/semantic_parsing/nsp_transformer_model).

A sketch of the controller's operation is then
```
if new utterance from human:
     logical_form = semantic_parser.translate(new command)
     if the logical_form denotes a command:
         push Interpreter(logical_form, agent_memory) onto the DialogueStack
     else if the logical_form denotes some other kind of dialogue the agent can handle:
         push some other appropriate DialogueObject on the DialogueStack
if the Dialogue Stack is not empty:
     step the highest priority DialogueObject
if TaskStack is empty:
     maybe place default behaviors on the stack
```


## Dialogue Manager ##

The Dialogue Manager operates the logical forms and places dialogue tasks in the agent's memory.
```eval_rst
 .. autoclass:: droidlet.dialog.dialogue_manager.DialogueManager
  :members: get_last_m_chats, step
```
### Semantic Parser ###
The training of the semantic parsing model is described in detail [here](https://github.com/facebookresearch/fairo/blob/main/droidlet/perception/semantic_parsing/nsp_transformer_model/train_model.py) and the model architecture has been described [here](https://github.com/facebookresearch/fairo/blob/main/droidlet/perception/semantic_parsing/nsp_transformer_model/encoder_decoder.py) and [here](https://github.com/facebookresearch/fairo/blob/main/droidlet/perception/semantic_parsing/nsp_transformer_model/decoder_with_loss.py).
; the interface is
```eval_rst
 .. autoclass:: droidlet.perception.semantic_parsing.nsp_transformer_model.query_model.NSPBertModel
  :members: parse
```
## Dialogue Task ##
The generic Dialogue Task is a Task
```eval_rst 
:ref:`tasks_label`
```

A Task's main method is .step(),
Some other dialogue tasks:

```eval_rst
 .. autoclass:: droidlet.dialog.dialogue_task.Say
 .. autoclass:: droidlet.dialog.dialogue_task.AwaitResponse
 .. autoclass:: droidlet.dialog.dialogue_task.BotCapabilities
 .. autoclass:: droidlet.dialog.dialogue_task.ConfirmTask
 .. autoclass:: droidlet.dialog.dialogue_task.ConfirmReferenceObject

```



### Interpreter ###
The [Interpreter](https://github.com/facebookresearch/fairo/blob/main/droidlet/interpreter/interpreter.py) is responsible for using the world state \(via [memory](memory.md)\) and a natural language utterance that has been parsed into a logical form over the agent's DSL from the semantic parser to choose a [Task](memory.md) to put on the Task Stack.   The [robot](https://github.com/facebookresearch/fairo/blob/main/droidlet/interpreter/robot/loco_interpreter.py) and [craftassist](https://github.com/fairinternal/minecraft/blob/master/craftassist/agent/dialogue_objects/mc_intepreter.py) Interpreters are not the same, but the bulk of the work is done by the shared subinterpreters (in the files \*\_helper.py) [here](https://github.com/facebookresearch/fairo/blob/main/droidlet/dialog/dialogue_objects/).  The subinterpreters, registered in the main Interpreter [here](https://github.com/facebookresearch/fairo/blob/main/droidlet/dialog/dialogue_objects/intepreter.py#L55) \(and for the specialized versions [here](https://github.com/fairinternal/minecraft/blob/master/locobot/agent/dialogue_objects/loco_intepreter.py#L56) and [here](https://github.com/fairinternal/minecraft/blob/master/craftassist/agent/dialogue_objects/mc_intepreter.py#L61)\), roughly follow the structure of the DSL.  This arrangement is to allow replacing the (currently heuristic) subinterpreters with learned versions or specializing them to new agents.

```eval_rst
 .. autoclass:: droidlet.interpreter.interpreter.Interpreter
```
