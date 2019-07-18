# React 写法

使用子组件 <组件名/>



父组件传递数据至子组件  <Square *value*={i}/>

子组件接收: this.props.value

子组件如果是受控组件 不能this



父组件传递函数至子组件  达成子组件向父组件传递数据

Square:子组件  Board 父组件   

父组件通过  向子组件 传递 “数据”  回调方式 拿到子组件数据

```react
        return <Square 

        value={this.state.squares[i]}

        onClick={() => this.handleClick(i)}/>;
```

每一个 Square 被点击时，Board 提供的 `onClick` 函数就会触发。我们回顾一下这是怎么实现的：

1. 向 DOM 内置元素 `<button>` 添加 `onClick` prop，让 React 开启对点击事件的监听。
2. 当 button 被点击时，React 会调用 Square 组件的 `render()` 方法中的 `onClick` 事件处理函数。
3. 事件处理函数触发了传入其中的 `this.props.onClick()` 方法。这个方法是由 Board 传递给 Square 的。
4. 由于 Board 把 `onClick={() => this.handleClick(i)}` 传递给了 Square，所以当 Square 中的事件处理函数触发时，其实就是触发的 Board 当中的 `this.handleClick(i)` 方法。

子组件的Button 规定点击触发的内容是props  让React开启对该事件的监听

当子组件的Button 被点击 React调用render()处理点击事件

处理事件 的 函数 *onClick*={()=>this.props.onClick()}>  此方法由父组件传入

当上面的函数执行 实际上是执行父组件的方法

而父组件传递的方法  *onClick*={() => this.handleClick(i)}/>;

所以转而执行 handleClick(i)

数据流向：

