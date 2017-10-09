### `withTabState([mapPropsToTabState], [options])`
HOC that saves, loads and resets data extracted from the **props** in the `tabState` reducer on tab switch.
*returns* a new, component class.

#### Arguments

* [`mapPropsToTabState(ownProps): tabState`] \(*Function*): If this argument is specified, the new component while saving the `tabState`, will call `mapPropsToTabState` with `ownProps`. The results of `mapPropsToTabState` must be a plain object (serves as payload), which will be merged into the `tabState` stored in the redux-store. Every time, the `tabState` is picked up from the redux-store, the callback `onAction`( should be passed down as a prop ) will be called with `{ type: 'VIEW_STATE_UPDATED', payload }`.

* [`options`] *(Object)* If specified, further customizes the behavior of the connector. It accepts these additional options:
  * [`tabStateName`] *(String)*: **Highly recommended to specify it.** Used for making the key for storing the payload in the Redux's store state.
  * [`shouldStoreScroll`] *(Boolean)*: When true, injects a function `setScrollContainerRef`. Use it on the element for which you want to store the scroll position. It will take care of restoring the scroll position ( no callbacks needed ). If you want to just maintain the scroll position without saving any state, pass `mapPropsToTabState` as `null` or `undefined`.


#### Storing `state`

```javascript
class TodoList extends React.PureComponent {
  state = { selectedTodoIds: {} };
  
  handleAction = action => {
    switch(action.type){
      case 'TODO_SELECTED':
        this.setState({
          selectedTodoIds: { 
            ...this.state.selectedTodoIds,
            [action.payload.id] : true,
          },
        });
        break;
      default:
        break;
    }
  }
  
  render(){
    return (
    <div>
      {this.props.todos.map(todo => ( 
          <Todo
            key={todo.id}
            todo={todo}
            selected={this.state.selectedTodoIds[todo.id]}
            onAction={this.handleAction}
          />
      )}
    </div>
    );
  }
}
```

We want to store the `selectedTodoIds`, on switching the tab and get it back on switching back to the tab.


Solution 1

1. Make `TodoListContainer`.
2. Pass down the entire state.
3. Pass down `onAction`.

```javascript
class TodoListContainer extends React.PureComponent {
  state = { selectedTodoIds: {} };
  
  handleAction = action => {
    switch(action.type){
      case 'TODO_SELECTED':
        this.setState({
          selectedTodoIds: { 
            ...this.state.selectedTodoIds,
            [action.payload.id] : true,
          },
        });
        break;
      case 'VIEW_STATE_UPDATED':
        this.setState(action.payload);
        break;
      default:
        break;
    }
  }
  
  render(){
    return (
      <TodoList {...this.props} {...this.state} onAction={this.handleAction} />
    );
  }
}
```

TodoList.js
```javascript
const TodoList = props => (
  <div>
      {props.todos.map(todo => ( 
          <Todo
            key={todo.id}
            todo={todo}
            selected={props.selectedTodoIds[todo.id]}
            onAction={props.onAction}
          />
      )}
);
const mapPropsToTabState = props => _pick(props, 'selectedTodoIds');
export default withTabState(mapPropsToTabState, {tabStateName: 'TodoList'} )(TodoList);
```

Solution 2
1. Use `connectActionsAndTabState`

todoListHandlers.js

```javascript

export default {
  TODO_SELECTED(action, {getState, setState}) {
    setState({
      selectedTodoIds: { 
        ...(getState().selectedTodoIds || {}),
        [action.payload.id] : true,
      },
    });
  },
}
```

TodoList.js
```javascript
const TodoList = props => (
  <div>
      {props.todos.map(todo => ( 
          <Todo
            key={todo.id}
            todo={todo}
            selected={props.selectedTodoIds[todo.id]}
            onAction={props.onAction}
          />
      )}
   </div>
);

TodoList.defaultProps = { selectedTodoIds: {} };

const mapPropsToTabState = props => _pick(props, 'selectedTodoIds');
export default compose(
  connectActionsWithTabState(todoListActionHandlers),
  withTabState(mapPropsToTabState, {tabStateName: 'TodoList'} )
  )(TodoList);
```

![wTS_transparent-todo-mini.png](https://github.com/chinmay17/withTabState/blob/master/wTS_transparent-todo-mini.png?raw=true)

#### Restoring scroll

```javascript
class MyScrollableComponent extends React.PureComponent {
  render(){
    return (
      <div className="ovf-y-auto" ref={this.props.setScrollContainerRef}>
        {this.renderContent()}
      </div>
    );
  }
}

export default withTabState(null, {
  tabStateName: 'MyScrollableComponent',
  shouldStoreScroll: true,
})(MyScrollableComponent);
```

