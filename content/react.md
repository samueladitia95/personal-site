---
date: 2024-08-08
tags:
  - react
  - research
  - performance
---
# Source

1. [Developer Way](https://www.developerway.com/posts/react-re-renders-guide)
2. Advanced React by Nadia Makarevich
3. [React.Memo Documentation](https://react.dev/reference/react/memo)
4. [React useMemo Documentation](https://react.dev/reference/react/useMemo)
5. [React useCallback Documentation](https://react.dev/reference/react/useCallback)

# React Lifecycle

## 1. Mount

When a component first called to the screen, React will Initialize state, declared all it’s function, run is hooks and rendered all UI elements to the DOM. This process will only run once at the beginning of react component lifecycle.

## 2. Update/Re-Render

If there’s a new information in a component, the component will re-render the component with the new information. Compare to Mount lifecycle, the computation cost of re-rendering is minimal.

## 3. Unmount

After a component no longer used, react will remove every element from the DOM and clean any information store in memory.

# Why React component re-render

## 1. State Changes

A react component will re-render if the state of the component change. if a component re-rendered, it will re-declared every function and re-render everything inside the component.

```jsx
function Component () {
  const [count, setCount] = useState(1)
  
  return (
	  <h1>number Times 2: {randomComputedValue}</h1>
	  <button onClick={() => setCount(count + 1)}>Click Me</button>
  )
}
```

## 2. Parent Component Re-render

A component will also re-render if the parent component re-render even if there’s no state changes in the child component

```jsx
function ChildComponent() {
  return <div>Child Component</div>
}

function ParentComponent() {
  const [count, setCount] = useState(1)

  const randomFunction = () => {}
  const randomComputedValue = count * 2

  return (
    <div>
      <h1>number: {count}</h1>
      <h1>number Times 2: {randomComputedValue}</h1>
      <button onClick={() => setCount(count + 1)}>Click Me</button>
      <ChildComponent /> // will re-rendered every time count change
    </div>
  )
}

```

As example above, every time in count is changing, ChildComponent will be also re-rendered even though there's no change in ChildComponent state. randomFunction and randomComputedValue will also be redeclared every time a component re-rendered.

## 3. Hooks Changes

Any changes in custom hooks *(_Including context)_ will also trigger re-render in any component that uses that hooks.

---

# Component Props

A component will not re-render if a props is changing. **PROPS IS READ ONLY AND IMMUTABLE [source](https://react.dev/learn/passing-props-to-a-component#how-props-change-over-time).** It will also mean that any children a component have will not be re-rendered because the parent component rendered.

```jsx
function ChildComponent() {
  return <div>Child Component</div>
}

function ParentComponent({children}) {
  const [count, setCount] = useState(1)
  return (
    <div>
      <h1>number: {count}</h1>
      <button onClick={() => setCount(count + 1)}>Click Me</button>
      {children}
    </div>
  )
}

function Container() {
  return (
    <ParentComponent>
      <ChildComponent /> // Will not be re-rendered
    </ParentComponent>
  )
}
```

Any component pass through the children of ParentComponent will not be re-rendered.

---

# How Javascript compare value

Small refresher about how javascript compare value. If the variables is primitive (_ex: string, number, boolean_), javascript will directly compare it's value

```jsx
const number1 = 10
const number2 = 10

number1 === number2 // TRUE

const string1 = "string"
const string2 = "string"

string1 === string2 // TRUE
```

If the variables is non-primitive (_ex: object, instance, array_), javascript will not compare it's value, but **COMPARE MEMORY ADDRESS** of the variables. Different variable will always have different memory address even though they have the same value.

```jsx
const object1 = { name: 'Random name' }
const object2 = { name: 'Random name' }

object1 === object2 // FALSE

const arr1 = [1, 2, 3, 4]
const arr2 = [1, 2, 3, 4]

arr1 === arr2 // FALSE
```

---

# Memoization

Memoization is a optimization technique to speed up a program by caching the result of expensive function call or expensive calculation. In react the result of a calculation or function declaration doesn’t automatically be memoized.

There 3 ways to memoize in React

1. if you need to memoize the result expensive calculation, use **useMemo**
2. if you need to memoize the function, use **useCallback**
3. if you need to memoize a component, use **React.Memo**

Let’s discuss it one by one

## 1. useMemo

useMemo is a React Hook that lets you cache the result of a calculation between re-renders.

```jsx
function Component({ props }) {
	const [value, setValue] = useState({...})
  const filteredProps = filterArray(props)
  
  return (
    <div>
	    <button onClick={() => setValue({...newValue})}>Click Me</button>
      {filteredProps.map((el) => {
        ...
      })}
    </div>
  )
}
```

in above example, filteredProps will always re-calculated every time Component re-rendered, even though there are no changes in filteredProps value. in order to cached the result of filteredProps, we need to use useMemo

```jsx
function Component({ props }) {
	const [value, setValue] = useState({...})
  const filteredProps = useMemo(
	  () => {
		  return filterArray(props)
		}, 
		[props] // Dependency
	)
  
  return (
    <div>
	    <button onClick={() => setValue({...newValue})}>Click Me</button>
      {filteredProps.map((el) => {
        ...
      })}
    </div>
  )
}
```

## 2. useCallback

useCallback is a React Hook that lets you cache a function definition between re-renders. function declared using useCallback will not be re-declared as a new function and will have the same memory address. usually used with debounce function because debounce function cant be re-declared.

```jsx
const handleDebounce = useCallback(
  debounce((props1, ...) => {
    ...
  }, 1000),
  [], // Dependency
)
```

## 3. React.memo

React.memo lets you skip re-rendering a component when its props are unchanged. it allow child component to **NOT BE RE-RENDERED** even ****if the parent component is re-rendered.

```jsx

const ChildComponentMemo = React.memo(function ChildComponent ({ element }) {
	return (
		<div>{element}</div>
	)
})

function ParentComponent() {
  const [arr, setArr] = useState([])

  return (
    <div>

      {arr.map((element, index) => (
        <ChildComponentMemo key={index} element={element} />
      ))}
    </div>
  )
}
```

---

# Suggested Pattern

## 1. Don't declare component inside another component

There's a common pattern that we always used in out react project

```jsx
// ANTI PATTERN
function ParentComponent() {
  const [arr, setArr] = useState([])

  const ChildComponent = ({ element }) => {
    return <div>{element}</div>
  }

  return (
    <div>
      {arr.map((element, index) => (
        <ChildComponent key={index} element={element} /> // Will Re-mount
      ))}
    </div>
  )
}
```

The example will cost unnecessary computation cost. there are several reason. Every time ParentComponent re-render, **EVERY FUNCTION** inside ParentComponent, including function ChildComponent, will be redeclared and become a new function. It mean ChildComponent will not be re-rendered, but it will mount the component again, which will be much slower than normal re-rendered.

The solution to this problem is to move ChildComponent outside ParentComponent.

```jsx
const ChildComponent = ({ element }) => {
  return <div>{element}</div>
}

function ParentComponent() {
  const [arr, setArr] = useState([])

  return (
    <div>
      {arr.map((element, index) => (
        <ChildComponent key={index} element={element} />
      ))}
    </div>
  )
}
```

## 2. Moving State Down

Another common pattern what we use in out react project:

```jsx
// ANTI PATTERN
function Component() {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Click Me</button>
      {isOpen && <Modal />}
      <BigComponent />
    </div>
  )
}
```

BigComponent will always re-render every time there’s a state change in Component, even though there’s no changes in BigComponent.

The way to solve this is to move the state down to absolute lowest component so any state changes will not effect any other big components.

![Screenshot 2024-08-16 at 10.56.44.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/03168ceb-481f-49fb-90d1-bdfde85d4163/42ca0275-40ad-44a8-9446-4b0596720e1b/Screenshot_2024-08-16_at_10.56.44.png)

```jsx
function ModalWrapper() {
  const [isOpen, setIsOpen] = useState(false)
 
  return (
    <>
      <button onClick={() => setIsOpen(true)}>Click Me</button>
      {isOpen && <Modal />}
    </>
  )
}

function Component() {
  return (
    <div>
      <ModalWrapper /> 
      <BigComponent />
    </div>
  )
}

```

## 3. Pass Component as props/children

Another way to prevent re-render in big component is to pass the component as children for another component. Because children is considered a props (*props is read only and immutable), it will not be effected by state changes of it’s parent.

so instead writing it like this:

```jsx
// ANTI PATTERN
function ParentComponent() {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Click Me</button>
      {isOpen && <Modal />}
      <BigComponent />
    </div>
  )
}

function Home() {
	return (
		<ParentComponent />
	)
}
```

we can pass BigComponent as a child to ParentComponent:

```jsx
function ParentComponent({children}) {
  const [isOpen, setIsOpen] = useState(false)

  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Click Me</button>
      {isOpen && <Modal />}
      {children}
    </div>
  )
}

function Home() {
	return (
		<ParentComponent>
      <BigComponent />
    </ParentComponent>
	)
}
```

## 4. Move Utils function outside of the component

To prevent declaring a function multiple times every time a component re-render, we should move every utils function outside the component. Another benefit is it force every dependency the function have to pass through function props.

```jsx
const utils2 = (props1, ...) => {...} // Do this

function Component () {
	const utils1 = (props1, ...) => {...} // Dont Do This

	return (
		...
	)
}
```

## 5. Memoize value pass throughs context

If Context is used not in root component, there are a change that the context is re-rendered if parent component re-rendered.

```jsx
function ContextComponent{children}) {
  const [value, setValue] = useState({...})

  return (
    <Context.provider value={value}>
      {children}
    </Context.provider>
  )
}

function PageComponent() {
  return (
    <div>
      <OtherComponent />
      <ContextComponent>
	        ...
      </ContextComponent>
    </div>
  )
}
```

In Order to prevent excessive re-render of ContextComponent, memoize the value of the context with useMemo

```jsx
function ContextComponent{children}) {
  const memoValue = useMemo(() => return {...}, [])

  return (
    <Context.provider value={memoValue}>
      {children}
    </Context.provider>
  )
}
```

## 6. Separate Value and Function in Context Provider

If in Context there is a combination of data and Function they can be split into different Providers under the same component. That way, components that use the function only won’t re-render when the data changes.

```jsx
const Provider = ({ children }) => {
  const [state, setState] = useState(1);

  return (
    <ContextData.Provider value={state}>
      <ContextApi.Provider value={setState}>
	      {children}
	    </ContextApi.Provider>
    </ContextData.Provider>
  );
};
```