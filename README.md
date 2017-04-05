
### React进阶技巧

**Stateless Functions**

**无状态组件意思就是组件内无state和生命周期，无需考虑组件的检测和内存分配问题，是一个改善React性能的好办法。**
```
import {PropTypes, ContextTypes} from "react";

#最简单的无状态组件
const Greeting = () => <div>Hi there!</div>;

#可以接受两个参数props和context
const Greeting = (props, context) =>
  <div style={{color: context.color}}>Hi {props.name}</div>;

#定义局部变量
const Greeting = (props, context) => {
  const style = {
    fontWeight: "bold",
    color: context.color
  };

  return <div style={style}>{props.name}</div>
};

#或者定义一个外部函数
const getStyle = context => ({
  fontWeight: "bold",
  color: context.color
});

const Greeting = (props, context) =>
  <div style={getStyle(context)}>{props.name}</div>;

#然后定义 defaultProps, propTypes 和 contextTypes.
Greeting.propTypes = {
  name: PropTypes.string.isRequired
};
Greeting.defaultProps = {
  name: "Guest"
};
Greeting.contextTypes = {
  color: PropTypes.string
};
```
**JSX Spread Attributes**

**JSX分离属性，这是JSX的特点，可以通过…操作符把对象所有属性传递到JSX属性。**
```
#下面两句是对等的
let main = () => <main className="main" role="main">{children}</main>;
let main = () => <main {...{className: "main", role: "main", children}} />;

#直接把props转换成JSX，这个很常用
let FancyDiv = (props) => <div className="fancy" {...props} />;
let FancyDiv = () => <FancyDiv data-id="my-fancy-div">So Fancy</FancyDiv>;

#需要注意的是属性顺序，假如已经存在同名的属性，后面的属性会覆盖前面的属性
let FancyDiv = () => <FancyDiv className="my-fancy-div"/>;
const FancyDiv = props => <div {...props} className="fancy"/>;

#下面比较有意思，把className和其他剩余的props单独来传递
const FancyDiv = ({ className, ...props }) => (
  <div
    className={["fancy", className].join(' ')}
    {...props}
  />
);

```
**Destructuring Arguments**

**解构参数，很适合使用在无状态组件中的props解构。**
```
#传递所有props
const Greeting = props => <div>Hi {props.name}!</div>;

#传递props中的name属性
const Greeting = ({ name }) => <div>Hi {name}!</div>;

#通过剩余参数语法，把name从props中独立出来
const Greeting = ({ name, ...props }) => Hi {name}!;

#结合上面的分离属性使用效果更佳
const Greeting = ({ name, ...props }) => <div {...props}>Hi {name}!</div>;
```

**Conditional Rendering**

**巧用条件渲染**
```
#条件成立时渲染
function render() {
  return condition && <span>Rendered when `truthy`</span>
}

#条件不成立时渲染
function render() {
  return condition || <span>Rendered when `falsey`</span>
}

#单行三元运算符
function render() {
  return condition
    ? <span>Rendered when `truthy`</span>
    : <span>Rendered when `falsey`</span>
}

#多行三元运算符
function render() {
  return condition ? (
    <span>
      Rendered when `truthy`
    </span>
  ) : (
    <span>
      Rendered when `falsey`
    </span>
  )
}

#也有一种情况，就是使用非布尔值去条件渲染
const Oops = ({showFirst, dontShowSecond}) => (
  <div>
    {showFirst && 'first'}
    {dontShowSecond || 'second'}
  </div>
)

#使用0或1传值会返回"01", 而不是期望的
<Oops showFirst={0} dontShowSecond={1}/>

#避免这种情况可以使用!!
const Oops = ({showFirst, dontShowSecond}) => (
  <div>
    {!!showFirst && 'first'}
    {!!dontShowSecond || 'second'}
  </div>
)
```
**Children Types**

**React可以渲染很多数据类型，其中最常见的就是字符串和数组了**
```
#字符串
function render() {
  return (
    <div>
      Hello World!
    </div>
  )
}

#数组--下面这句在开发模式会报错，因为没有为数组设key值
function render() {
  return (
    <div>
      {["Hello ", World, "!"]}
    </div>
  )
}

#函数
function render() {
  return (
    <div>
      { (() => "hello world!")() }
    </div>
  )
}
```
**Array As Children**

**这是React渲染列表最常用的方式，JSX有直接渲染数组的能力**
```
#下面两个渲染结果是对等的
(<ul>
  {["first", "second"].map((item) => (
    <li>{item}</li>
  ))}
</ul>)

(<ul>
  {[
    <li>first</li>,
    <li>second</li>
  ]}
</ul>)

#应用剩余参数语法分类属性效果更佳
(<ul>
  {arrayOfMessageObjects.map(({ id, ...message }) =>
    <Message key={id} {...message} />
  )}
</ul>)
Function As Children

//然而并不常用，这种函数逻辑应该在return之前定义好
<div>{() => { return "hello world!" }()}</div>
Render Callback

这是一个使用渲染回调的组件，这不是有用的，但这是一个很容易的例子。

//把children设成一个函数
const Width = ({ children }) => children(500)

//调用
(<Width>
  {width => <div>window is {width}</div>}
</Width>)

//输出结果
<div>window is 500</div>

//一个更复杂的例子
class WindowWidth extends React.Component {
  constructor() {
    super();
    this.state = {
      width: 0
    };
  }

  //没想到this.setState有第二参数，并且是函数
  componentDidMount() {
    this.setState({
      width: window.innerWidth
    }, () => {
      window.addEventListener("resize", ({target}) => this.setState({width: target.innerWidth}))
    })
  }

  render() {
    return this.props.children(this.state.width);
  }
}

//调用
(<WindowWidth>
  {width => <div>window is {width}</div>}
</WindowWidth>)
```
**Children types**

**一般向子组件传递数据用props，但同时也可以用children**
```
#假如存在一个需要返回children的组件
class SomeContextProvider extends React.Component {
  getChildContext() {
    return {some: "context"}
  }

  render() {
    # 最好返回children的方式是什么
  }
}

#选项1：返回一个div包裹
return <div>{this.props.children}</div>;

#选项2：会报错
return this.props.children;

#选项3：使用React提供的React.Children可以更合适的处理
return React.Children.only(this.props.children);

#Children里传递另一个组件
class App extends React.Component {
  render() {
    var title = <h1>Hello there!</h1>;
    return (
      <Header title={ title }>
        <Navigation />
      </Header>
    );
  }
}

export default class Header extends React.Component {
  render() {
    return (
      <h1>
        { this.props.title }
        <hr />
        { this.props.children }
      </h1>
    );
  }
};

#Children里传递一个对象
function UserName(props) {
  return (
    <div>
      <b>{props.children.lastName}</b>
      {props.children.firstName}
    </div>
  );
}

function App() {
  var user = {
    firstName: 'Vasanth',
    lastName: 'Krishnamoorthy'
  };

  return(
    <UserName>{ user }</UserName>
  )
}

#Children里传递一个函数
function TodoList(props) {
  const renderTodo = (todo, i) => {
    return (
      <li key={ i }>
        { props.children(todo) }
      </li>
    );
  };
  return (
    <section className='main-section'>
      <ul className='todo-list'>{ props.todos.map(renderTodo)}</ul>
    </section>
  );
}

function App() {
  const todos = [
    { label: 'Write tests', status: 'done' },
    { label: 'Sent report', status: 'progress' },
    { label: 'Answer emails', status: 'done' }
  ];
  var isCompleted = todo => todo.status === 'done';

  return (
    <TodoList todos={ todos }>
      { todo => isCompleted(todo) ? <b>{ todo.label }</b> : todo.label }
    </TodoList>
  );
}
```

**Proxy Component**

**使用代理组件**
```
#常见的button
<button type="button">

#使用一个高阶的组件代理上面低阶的组件
const Button = props => <button type="button" {...props}>

#调用
<Button />
<Button className="CTA">Send Money</Button>
Style Component

#假如我们需要一个可自定义的按钮组件
<button type="button" className="btn btn-primary">

#定义组件
const PrimaryBtn = props => <Btn {...props} primary/>

const Btn = ({ className, primary, ...props }) => (
  <button
    type="button"
    className={classnames(
      "btn",
      primary && "btn-primary",
      className
    )}
    {...props}
  />
);

#下面输出同样的结果
<PrimaryBtn />
<Btn primary/>
<button type="button" className="btn btn-primary"/>
```
**Event Switch**

**事件切换**
```
#分别定义事件
handleClick() { require("./actions/doStuff")(/* action stuff */) }
handleMouseEnter() { this.setState({ hovered: true }) }
handleMouseLeave() { this.setState({ hovered: false }) }

#使用switch
handleEvent({type}) {
  switch(type) {
    case "click":
      return require("./actions/doStuff")(/* action dates */)
    case "mouseenter":
      return this.setState({ hovered: true })
    case "mouseleave":
      return this.setState({ hovered: false })
    default:
      return console.warn(`No case for event type "${type}"`)
  }
}

#使用对象
const handlers = {
  click: () => require("./actions/doStuff")(/* action dates */),
  mouseenter: () => this.setState({ hovered: true }),
  mouseleave: () => this.setState({ hovered: false }),
  default: () => console.warn(`No case for event type "${type}"`)
};

handleEvent({type}) {
  const NORMALIZED_TYPE = type.toLowerCase();
  const HANDLER_TO_CALL = NORMALIZED_TYPE in handlers ? NORMALIZED_TYPE : 'default';
  handlers[HANDLER_TO_CALL].bind(this)();
}

#巧妙利用try..catch
handleEvent({type}) {
  try {
    handlers[type.toLowerCase()].bind(this)();
  } catch (e) {
    handlers['default'].bind(this)();
  }
}
```
**Layout Component**

**布局组件和内容组件的分离可以更好的控制页面**
```
#把组件当做props传递
<HorizontalSplit
  leftSide={<SomeSmartComponent />}
  rightSide={<AnotherSmartComponent />}
/>

class HorizontalSplit extends React.Component {
  render() {
    <FlexContainer>
      <div style={{ flex: 1 }}>{this.props.leftSide}</div>
      <div style={{ flex: 2 }}>{this.props.rightSide}</div>
    </FlexContainer>
  }
}
```
**Container Component**

**容器组件和UI组件的分离，让容器组件专注于数据的拉取，UI组件专注于展示的复用。**
```
#一个可复用的UI组件
const CommentList = ({ comments }) =>
  <ul>
    {comments.map(comment =>
      <li>{comment.body}-{comment.author}</li>
    )}
  </ul>;

#一个拉取数据的容器组件
class CommentListContainer extends React.Component {
  constructor() {
    super();
    this.state = {comments: []}
  }

  componentDidMount() {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: comments =>
        this.setState({comments: comments});
    });
  }

  render() {
    return <CommentList comments={this.state.comments}/>
  }
}
```

**Higher Order Component**

**高阶组件是把一个组件当做一个参数传入一个函数，然后返回一个新的组件，用于统一为一些组件执行同样的工作，是一个非常实用的功能。**
```
#定义一个函数，传入一个组件，返回一个新组件
export var Enhance = ComposedComponent => class extends Component {
  constructor() {
    this.state = { data: null };
  }
  componentDidMount() {
    #常用于拉取数据，改变对应组件的state
    this.setState({ data: 'Hello' });
  }
  render() {
    return <ComposedComponent {...this.props} data={this.state.data} />;
  }
};

#调用
class MyComponent {
  render() {
    #判断是否含有data，没有则显示等待中
    if (!this.props.data) return <div>Waiting...</div>;
    #有则返回数据
    return <div>{this.props.data}</div>;
  }
}
#注意这里导出的格式
export default Enhance(MyComponent);
```
**State Hoisting**

**状态提升是指把子组件的数据传到父组件，一般通过函数来交流。**
```
//传递一个函数到子组件
class NameContainer extends React.Component {
  render() {
    return <Name onChange={newName => alert(newName)}/>
  }
}

//在子组件触发该函数，父组件会反映出来，这是最常见的方式。
const Name = ({ onChange }) => <input onChange={e => onChange(e.target.value)}/>

//下面就是子组件通过函数传递数据到父组件的例子
class NameContainer extends React.Component {
  constructor() {
    super()
    this.state = {name: ""}
  }

  render() {
    return <Name onChange={newName => this.setState({name: newName})}/>
  }
}
```
**Controlled – Uncontrolled Components**

**受控 – 不受控组件，这个概念有点抽象，主要讨论from表单里的input元素是个不可控组件，大概了解一下即可。**
```
//当这样写的时候，input只能显示最初获取的值，且无法改变
class UncontrolledNameInput extends React.Component {
  constructor() {
    super();
    this.state = {name: ""}
  }

  render() {
    return <input type="text" value={this.state.name} />
  }
};

//这样可以做到双向绑定，让input变得可控
//只要知道一点，当你按下键盘的时候，实际上是先改变this.state.name，才会间接引起input的值变化，并非直接改变。
class ControlledNameInput extends Component {
  constructor() {
    super();
    this.state = {name: ""}
  }

  render() {
    return (
      <input
        value={this.state.name}
        onChange={e => this.setState({name: e.target.value})}
      />
    );
  }
}
```
**Conditionals in JSX**

**分析JSX中的条件渲染**
```
//单条件不建议写法
const sampleComponent = () => {
  return isTrue ? <p>True!</p> : <none/>
};

//单条件建议写法
const sampleComponent = () => {
  return isTrue && <p>True!</p>
};

//多条件不建议写法
const sampleComponent = () => {
  return (
    <div>
      {flag && flag2 && !flag3
        ? flag4
        ? <p>Blah</p>
        : flag5
        ? <p>Meh</p>
        : <p>Herp</p>
        : <p>Derp</p>
      }
    </div>
  )
};

//多条件建议写法
const sampleComponent = () => {
  return (
    <div>
      {
        (() => {
          if (flag && flag2 && !flag3) {
            if (flag4) {
              return <p>Blah</p>
            } else if (flag5) {
              return <p>Meh</p>
            } else {
              return <p>Herp</p>
            }
          } else {
            return <p>Derp</p>
          }
        })()
      }
    </div>
  )
};
```
