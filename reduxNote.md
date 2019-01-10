[Redux 入门教程（一）：基本用法](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)<br>
[Redux 入门教程（二）：中间件与异步操作](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html)<br>
[Redux 入门教程（三）：React-Redux 的用法](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_three_react-redux.html)

# 一、redux

## 1.一些概念
* store = createStore()创建一个store，包含应用的所有数据
* store.getState();获取当前状态
* store.dispatch(action);发送action
* reducer(nowState,action)=>newState,更新state
* store.subscribe(listener);监听，state发生改变时触发listener

## 2.概念的补充
* action是一个对象，一定有type属性
* Action creater动作生成器
* store = createStore(reducer)，把reducer作为参数传入，以后每当dispatch发送过来新的action，就会自动调用reducer
* combineReducers({a,b,...c})把几个子reducer合并成一个大的，参数是一个对象

# 二、中间件和异步

## 1.一些概念
* 通过修改dispatch实现异步，修改后的dispatch就是中间件
* 有现成的中间件，直接调用即可。
```
import createLogger from 'redux-logger'
const logger = createLogger();
store = createStore(
    reducer,
    applyMiddleware(logger)
)
```
* applyMiddleware(a,b,...c)将所有中间件组成一个数组并依次执行，中间件有次序讲究。
* 异步发送两个action，一个是告知操作开始，state更新为正在操作，渲染一次，一个是告知操作结束，state更新为操作结束，再渲染一次。
* 用户触发第一个action，改造Action creater，在其中触发第二个action。
* redux-thunk中间件，使dicpatch参数支持函数
* redux-promise中间件，使dispatch参数支持promise对象

## 2.异步操作的解决方案：
* 第一种：让Action creater返回一个函数。这个函数执行后，先发出一个action，然后进行异步操作，拿到结果后，将结果转为JSON格式，然后再发出一个action。
```
//fetchPosts就是一个Action creater
const fetchPosts = postTitle => (dispatch, getState) => {
  dispatch(requestPosts(postTitle));
  return fetch(`/some/API/${postTitle}.json`)
    .then(response => response.json())
    .then(json => dispatch(receivePosts(postTitle, json)));
  };
};
// 使用方法一
store.dispatch(fetchPosts('reactjs'));
// 使用方法二
store.dispatch(fetchPosts('reactjs')).then(() =>
  console.log(store.getState())
)
```
* 第二种：让Action creater返回一个promise对象
```
//fetchPosts就是一个Action creater
//第一种写法：返回值是一个Promise对象
const fetchPosts = 
  (dispatch, postTitle) => new Promise(function (resolve, reject) {
     dispatch(requestPosts(postTitle));
     return fetch(`/some/API/${postTitle}.json`)
       .then(response => {
         type: 'FETCH_POSTS',
         payload: response.json()
       });
});
//第二种写法：Action 对象的payload属性是一个 Promise 对象，这里需要redux-actions模块引入createAction方法
import { createAction } from 'redux-actions';

class AsyncApp extends Component {
  componentDidMount() {
    const { dispatch, selectedPost } = this.props
    // 发出同步 Action
    dispatch(requestPosts(selectedPost));
    // 发出异步 Action
    dispatch(createAction(
      'FETCH_POSTS', 
      fetch(`/some/API/${postTitle}.json`)
        .then(response => response.json())
    ));
  }
```

# 三、react-redux
## 1.一些概念
* 将组件分成两大类，UI组件和容器组件。
* UI组件由用户提供，容器组件由react-redux自动生成。
* connect()用来生成容器组件
```
const VisibleTodoList = connect(
    mapStateToProps,
    mapDispatchToProps
)(TodoList)
```
* mapStateToProps是一个函数，将外部state映射到UI组件的props对象上
```
const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
const getVisibleTodos = (todos, filter) => {//从state算出todos的值
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
    default:
      throw new Error('Unknown filter: ' + filter)
  }
}
```
以上mapStateToProps会订阅store，每当state更新的时候，就会自动执行，从而触发UI组件重新渲染。<br>
mapStateToProps可以接收第二参数，代表容器组件的props对象，接收这个参数后，如果容器组件的参数发生变化，也会触发UI组件重新渲染。<br>
connect()省略mapStateToProps的话，UI组件就不会订阅store。<br>
* mapDispatchToProps定义哪些用户的操作应该当做action传给store,它可以是一个函数，也可以是一个对象。
* 如果是函数，它有两个参数，dispatch和ownProps（容器组件的props对象）
```
const mapDispatchToProps = (
  dispatch,
  ownProps
) => {
  return {
    onClick: () => {
      dispatch({
        type: 'SET_VISIBILITY_FILTER',
        filter: ownProps.filter
      });
    }
  };
}
```
* 如果是一个对象，它的键名对于UI组件的同名参数
```
const mapDispatchToProps = {
  onClick: (filter) => {
    type: 'SET_VISIBILITY_FILTER',
    filter: filter
  };
}
```

## 2.`<Provider>`组件
功能：就是传入store<br>
用法：在根组件外面包一层<Provider><br>
原理：利用react的context属性。<br>
```
    class Provider extends Component {
      getChildContext() {
        return {
          store: this.props.store
        };
      }
      render() {
        return this.props.children;
      }
    }
    Provider.childContextTypes = {
      store: React.PropTypes.object
    }
    class VisibleTodoList extends Component {
      componentDidMount() {
        const { store } = this.context;
        this.unsubscribe = store.subscribe(() =>
          this.forceUpdate()
        );
      }
      render() {
        const props = this.props;
        const { store } = this.context;
        const state = store.getState();
        // ...
      }
    }
    VisibleTodoList.contextTypes = {
      store: React.PropTypes.object
    }
```
