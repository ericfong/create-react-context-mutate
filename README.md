wrap new React.createContext() to provide context with a set function for Consumer



# NOTE

According to [0002-new-version-of-context](https://github.com/reactjs/rfcs/blob/master/text/0002-new-version-of-context.md#relies-on-strict-comparison-of-context-values), mutation is discouraged. But in some situations, Consumer really need a way to update the Provider. So, I come up with this module.



# Usage

```js
import createMutableContext from 'create-mutable-context'

const { Provider, Consumer } = createMutableContext()

<Provider value={1}>
  <Consumer>
    {ctx => {
      // ctx contain value and a set function
      return (
        <button onClick={() => ctx.set(ctx.value + 1)}>
          {ctx.value}
        </button>
      )
    }}
  </Consumer>
</Provider>
```



## ctx.set()

```js
set(updater[, callback])
```

set function interface is similar to react component's setState(), which accept updater as `object` or `function`.

updater as function with the signature:
```js
(prevValue, providerProps) => newValue
```

**set will just replace the value. It will NOT merge newValue to prevValue.**



## keep value in your own state by Provider.onChange

you can also keep value in your own state (like Input)

```js
import createMutableContext from 'create-mutable-context'

const { Provider, Consumer } = createMutableContext()

class App extends React.Component {
  state = { valueA: 1 }
  render() {
    return (
      <Provider
        value={this.state.valueA}
        onChange={valueA => this.setState({ valueA })}
      >
        <Consumer>
          {ctx => {
            return (
              <button onClick={() => ctx.set(ctx.value + 1)}>
                {ctx.value}
              </button>
            )
          }}
        </Consumer>
      </Provider>
    )
  }
}
```


## use enhancer to add functions to ctx

createMutableContext signature:
```js
createMutableContext(defaultValue, defaultEnhancer)
```

enhancer is a function that accept ctx and return modified ctx
```js
(ctx, providerProps) => modifiedCtx
```

You can also assign enhancer to Provider props.

Both enhancers are run in Provider constructor once (with provider instance props at that time). defaultEnhancer will be run before Provider enhancer.

```js
import createMutableContext from 'create-mutable-context'

const defaultValue = 1
const defaultEnhancer = ctx => {
  ctx.inc1 = () => ctx.set(prevValue => prevValue + 1)
  return ctx
}
const C = createMutableContext(defaultValue, defaultEnhancer)

const App = () => (
  <C.Provider
    // add enhancer in Provider level
    enhancer={ctx => {
      ctx.inc2 = () => ctx.set(prevValue => prevValue + 2)
      return ctx
    }}
  >
    <C.Consumer>
      {ctx => (
        <button
          // ctx get both inc1 and inc2 functions
          onClick={ctx.inc1}
          onMouseOver={ctx.inc2}
        >
          {ctx.value}
        </button>
      )}
    </C.Consumer>
  </C.Provider>
)
```
