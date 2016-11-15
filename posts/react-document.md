#Learn Something New From React Document

断断续续用了一年react，一直也没看过文档，读一遍，看看有什么收获。

##Quick Start

###Introducing JSX

- **jsx**是js的语法扩展，编译之后jsx为js对象，所以可以把它当作变量来用

  **jsx** is a syntax extension to js, it bacomes regular js objects after compiling, so you can use it as variable

- **babel**把jsx转成调用React.createElement()

  **babel** compiles jsx down to React.createElement() calls

###Rendering Elements

- **react element**是不可变的,改变ui的唯一方法就是创建一个新的element

  **react element** is immutable, the only way to change update the ui is to create a new element

- **ReactDom**会将新老虚拟dom进行对比，只在dom中改虚拟dom改变了的部分

  **ReactDom** will compare the element and its children to the previous one, only update the changing parts of element to the dom

###Components and Props

- **components**将ui拆分成独立可复用的模块

  **components**split ui into independent, reusable pieces

- react将自定义component的属性包装成一个单独的对象，**props**

  react pass all attributes to the user-defined components as a single object which is called **props**

- **components**必须为纯函数，即它的输入不能被更改，即**props**不能被更改，是只读的

  **components** must be pure, its inputs-props mustn't be changed，the props is read-only

###State and Lifecycle

- 将**函数式component**变为**class component**需要将函数的内容放到class的render方法中

  To convert a **functional component** to a **class component** you should move the body of the function into the render method.

- 在一个有很多component的应用中，移除component的过程中清空component的数据很重要

  In an app with many components, it's important to clear up the data when remove a component

- **componentDidMount**发生在component的dom渲染完成之后，**componentWillUnmount**发生在component被移除之前

  **componentDidmount** hook run after the component dom finish being rendered, **componentWillUnmount** run before the component dom removed

- setState的值涉及state或者props可以采用第二种方法，防止同时进行多个setState操作导致操作失败

  ```js
  // Wrong
  this.setState({
    counter: this.state.counter + this.props.increment,
  });
  // Correct
  this.setState((prevState, props) => ({
    counter: prevState.counter + props.increment
  }));
  ```

###Handling Events

- 不需要等dom创建之后addEventListener，只需要在首次渲染时监听事件即可

  You dont need to addEventListener after the dom is created, just provide event listener when the element is initially rendered.

- class里的method里的this默认是undefined，所以必须要自己绑一下或者用箭头函数使其没有自己的this，里面的this直接指向外层this

  **this** in method of class is undefined by default, if you want to use this.setState, you need to bind this to methods

- 不建议在向子component传props的时候对props的值进行计算，可能会导致子component重复渲染，导致性能问题

  When pass some props to lower component, the valud of the props shouldn't do some caculation, it may cause the lower component do an extra re-rendering

  ```js
  render () {
    return <UserDefinition onClick={(e) => {this.handleClick(e)}}>OK</UserDefinition> // wrong
  }
  ```

###Conditional Rendering

- js中，true && expression ==> expression, false && expression ==> false

###Lists and Keys

- 数组中的elements需要有**key**属性，react借助它标示数组中元素的变化

  Element items in an array should have **key** property, react use it to identify which item have been changed, added or removed

- 不能通过props拿到key属性，想访问key的值要通过其它属性名

  cannot get props.key，set another property to get the value of key

- 嵌套过深就可以考虑抽象出一个component

  if a part of code is too nested, you should consider extracting a component

###Forms

- 在html中，<input><textarea><select>有自己的状态，根据用户输入更新状态，react的装套存在state中并且通过setState更新
  
  In html <input><textarea><select> have their own state and update it accroding to user input, but in react mutable state typically kept in state property state of components and update only by setState

- **controlled component**把input的value绑到state上，通过onChange改变state从而把form的状态也集中到了**state**的上
  
  **controlled component** tie the value of element to the **state** and centralize the state of form to the state of react component

###Lifting State Up

###Composition vs Inheritance

- **props.children**表示该component的所有子component

  **props.children** is equal to all children component of this component

- 通过**components**内置的**props**属性可以替代**继承**所起的作用

  You can do everything that **inheritance** can do through the **props** of **components**

##Advanced Guides

###JSX in Depth

- JSX 只是React.createElement(component, props, ..children)的语法糖

  JSX is just syntacitc sugar of React.createElement