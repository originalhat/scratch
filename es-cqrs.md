# es/cqrs

to be found in `retro.js`

## user action (e.g. react button click)

```js
function newTaskSubmit(state) {
    const addTaskCommand = retro.addTask({id: guid(), name: state.uiState.newTaskName});

    return {
        type: 'commandQueued',
        command: addTaskCommand,
    };
}
```

>this can be wired into the view, providing an entry for your data...

```js
const onNewTaskInput = (newTaskName) => store.dispatch(newTaskInput(newTaskName));
const onNewTaskSubmit = () => store.dispatch(newTaskSubmit(store.getState()));

ReactDOM.render(
    <App onNewTaskInput={onNewTaskInput}
         onNewTaskSubmit={onNewTaskSubmit}/>,
    document.getElementById('root')
);
```

## commands (user intent)

```js
export function addTask(taskId, taskName) {
    return {
        type: 'addTask',
        taskId,
        taskName,
    }
}
```
## "handle commands"

>injected in the view...

```js
function handleCommands(commands, validationState, viewState) {
    for (let i = 0; i < commands.length; i++) {
        const {type, v: eventResult} = list.commandHandlers.addTask(commands[i], validationState);

        if (type === 'ok') {
            const event = eventResult;
            viewState = viewStateHandlers[event.type](event, viewState, true);
            validationState = list.eventHandlers[event.type](event, validationState);
        } else {
            console.log('error: ', eventResult);
            break;
        }
    }

    return viewState;
}
```

## command handler

```js
export const commandHandlers = {
    createTask: (command, state) => {
        const event = taskCreated(command.taskId, command.taskName);

        if (state.created) {
            return err(ERROR_CANNOT_CREATE_ALREADY_CREATED_LIST);
        }

        return ok(event);
    },
}
```

## event handler

```js
export const eventHandlers = {
    itemAdded: (event, state) => ({...state, itemIds: state.itemIds.concat(event.item.id)}),
};
```

## events

```js
export function taskCreated(taskId, taskName) {
    return {
        type: 'taskCreated',
        taskId,
        taskName,
    }
}
```

