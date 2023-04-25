# useCallBack

在 `React` 组件中，当父组件重新渲染时，所有子组件也会重新渲染，即使子组件的 `props` 没有发生变化。这是因为 `React` 会比较父组件传递给子组件的 `props` 是否发生变化，如果发生变化，就会重新渲染子组件。

如果父组件传递给子组件的 `props` 是一个函数，而该函数在父组件重新渲染时也会重新生成，就会导致子组件也重新渲染。这会造成不必要的性能损耗，特别是当父组件包含大量子组件时，影响会更加明显。

这时候就可以使用 useCallback 来缓存函数，避免不必要的重新渲染。useCallback 接受两个参数，第一个参数是要缓存的函数，第二个参数是一个依赖数组。当依赖数组中的值发生变化时，会重新生成缓存的函数，否则会返回缓存的函数。

例如，下面这个例子中，`ChildComponent` 接受了一个 onClick 的 `props`，该 `props` 是一个函数，如果每次父组件重新渲染时都会生成一个新的函数，就会导致 `ChildComponent` 也重新渲染。如果使用 `useCallback` 缓存函数，只有在 count 发生变化时才会重新生成函数，避免了不必要的渲染。

## useCallBack vs useMemo

`useCallback` 的作用是缓存函数，避免不必要的渲染和性能损耗。它接受两个参数，第一个参数是要缓存的函数，第二个参数是一个依赖数组，只有依赖数组中的值发生变化时，才会重新生成缓存的函数。通常情况下，`useCallback` 适用于需要将函数作为 `props` 传递给子组件或者在 `React.memo`中进行优化的场景。

`useMemo` 的作用是缓存计算结果，避免重复计算和性能损耗。它接受两个参数，第一个参数是一个函数，用于计算需要缓存的值，第二个参数是一个依赖数组，只有依赖数组中的值发生变化时，才会重新计算缓存的值。通常情况下，`useMemo` 适用于需要根据一些 `props` 或者 `state` 计算出一个值，并将该值作为渲染结果的一部分展示的场景。

```jsx
import List from './List.js';

export default function App() {
  const [number, setNumber] = useState(1);
  const [dark, setDark] = useState(false);

  const getItems = useCallback((incrementor) => {
    return [number + incrementor, number + 1 + incrementor, number + 2 + incrementor];
  }, [number]);

  const theme = {
    backgroundColor: dark ? '#333' : '#FFF',
    color: dark ? '#FFF' : '#333'
  };

  return (
    <div style={theme}>
      <input
        type="number"
        value={number}
        onChange={e => setNumber(parseInt(e.target.value))}
      />
      <button onClick={() => setDark(prevDark => !prevDark)}>Toggle theme</button>
      <List getItems={getItems} />
    </div>
  );
}
```

```jsx
// List.js
export default function List({ getItems }) {
  const [items, setItems] = useState([]);

  useEffect(() => {
    setItems(getItems(3));
    console.log('Updating Items');
  }, [getItems]);

  return items.map((item) => (
    <div key={item}>{item}</div>
  ));
}
```
