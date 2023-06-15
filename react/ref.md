# ref

## 创建及获取 ref 的方式

- 在类组件中可以使用 `React.createRef()` 创建 ref 对象
- 在函数组件中可以使用 `useRef()` 创建 ref 对象
- ref 可以是一个对象或者回调函数，通过 `ref.current` 获取绑定的 DOM 元素或者组件实例等

```jsx
// 在类组件中创建 ref
class MyComponent extends Component {
  ref1 =React.createRef(null) // ref1是一个对象
  ref2 =null // ref2是一个函数
  render=()=> {
    <div>
     <div ref={this.ref1}>Hello, ref1!</div>
      {/* ref2是一个回调函数  */}
     <div ref={(node)=>{
               this.ref2 = node
               console.log(this.node) 
               // 这里第一次打印是div,会在render之后以及componentDidMount之前执行。
               // 更新后会在getSnapshotBeforeUpdate之后、componentDidUpdate之前执行
               // 打印结果为null和<div>ref元素节点</div>，更新先会执行commitDetachRef方法把refs置为null,
               //然后再通过commitAttachRef绑定为更新后的值
            }}>Hello, ref2!</div> 
    </div>
  }
}

// 在函数组件中创建 ref
function MyComponent() {
  const childRef = useRef(null);
  const divRef = useRef(null);
  const handleClick = () => {
    console.log(childRef.current); // 获取Child组件实例
    console.log(divRef.current); // 获取div实例
  };
    render=()=> {
    <div>
     <div ref={childRef}>Hello, childRef!</div>;
     <div ref={divRef}>Hello, div!</div>;
     <Child ref={childRef} />
    </div>
  }
}
```

## forwardRef 的使用场景

- 组件获取子中的DOM或者组件实例；
- 函数组件没有实例，只能通过 `forwardRef` 来获取`ref`；
- 高阶组件是在原组件外层又包裹了一层组件，直接通过`ref`绑定，`ref.current` 会是高阶组件的实例而不是原组件。如果想要绑定原组件，可以使用 `forwardRef`转发。
  
```jsx
// 组件想要获取子中的 DOM 或者组件实例；
// 函数组件没有实例，只能通过 forwardRef 来获取 ref；
const Child = forwardRef((props, ref) => {
    const handleClick = () => {
        console.log('Hello, World!');
    };
    return <div ref={ref} onClick={handleClick}>Click Me</div>;
});
const Parent = () => {
    const childRef = React.useRef(null);
    const handleClick = () => {
        console.log(childRef.current); // <div>Click Me</div>
    };
    return (
        <div>
            <Child ref={childRef} />
            <button onClick={handleClick}>Get Child Ref</button>
        </div>
    );
};
```

`forwardRef`转发高阶组件

```jsx
// 高阶组件
function HOC(WrappedComponent) {
  // 使用 forwardRef 来将 ref 传递给子组件
  const HOC = forwardRef((props, ref) => {
    return <WrappedComponent {...props} forwardedRef={ref} />;
  });

  return HOC;
}

// 子组件
const Child = ({ forwardedRef, ...props }) => {
  // 点击事件处理函数
  const handleClick = () => {
    console.log('Hello, World!');
  };

  // 将 ref 绑定到 div 元素上
  return <div ref={forwardedRef} onClick={handleClick}>Click Me</div>;
};
// 使用高阶组件封装子组件
const HocChild = HOC(Child);
// 父组件
const Parent = () => {
  // 创建一个引用变量
  const childRef = React.useRef(null);

  // 点击事件处理函数
  const handleClick = () => {
    console.log(childRef.current);
  };

  return (
    <div>
      <HocChild forwardedRef={childRef} />
      <button onClick={handleClick}>Get Child Ref</button>
    </div>
  );
};
```

## ref 组件通信

将 ref 绑定到类组件时，`ref.current` 是组件实例，可以直接操作实例里的方法；对于函数组件可以使用 `React.forwardRef` 和 `useImperativeHandle` 自定义对外暴露的 ref 内容。

```jsx
import { useRef, useImperativeHandle, forwardRef, useState } from 'react';

// 子组件
function Son(props, ref) {
  const [count, setCount] = React.useState(0);
  // 使用 useImperativeHandle 来声明可以被外部调用的方法
  useImperativeHandle(ref, () => ({
    incrementCount() {
      setCount(count + 1);
    }
  }));
s  return <span>Count: {count}</span>;
}

const ForwardSon = forwardRef(Son);

// 父组件
function Father() {
  const childRef = useRef(null);
  const handleClick = () => {
    childRef.current.incrementCount();
  }
  return (
    <div>
      <ForwardSon ref={childRef} />
      <button onClick={handleClick}>操控子组件</button>
    </div>
  );
}

```

## 使用ref缓存数据

ref返回的对象始终是不变的，改变 current 并不会导致组件重新渲染，可以利用这个特点去缓存数据。
