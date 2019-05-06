[React component patterns - William Whatley](https://medium.com/teamsubchannel/react-component-patterns-e7fb75be7bb0)

## Common component patterns are:

### Container (stateful: render, state, lifecycle events, [context, props])

    “A container does data fetching and then renders its corresponding sub-component. That’s it.” — Jason Bonta
### Presentational (stateless: render, props, context)

    Presentational components utilize props, render, and context (stateless API’s) and can be the syntactically-pretty functional, stateless component
### Higher order components (HOC’s)

    A higher order component is a function that takes a component as an argument and returns a new component.
### Render callback

    Similar to higher order components, render callbacks or render props are used to share or reuse component logic. 

[Video: Never Write a another HoC](https://www.youtube.com/watch?v=BcVAq3YFiuc)

## Principles to write a resilient component
* **Don’t stop the data flow.** Props and state can change, and components should handle those changes whenever they happen.
* **Always be ready to render.** A component shouldn’t break because it’s rendered more or less often.
* **No component is a singleton.** Even if a component is rendered just once, your design will improve if rendering twice doesn’t break it.
* **Keep the local state isolated.** Think about which state is local to a particular UI representation — and don’t hoist that state higher than necessary.

## Function component
```jsx
function Greeting() {
  return <div>Hi there!</div>;
}

// Collect props from the first argument of your function.

function Greeting(props) {
  return <div>Hi {props.name}!</div>;
}
// Define any number of local variables to do what you need in your function components.
// Always return your React Component at the end.

function Greeting(props) {
  let style = {
    fontWeight: "bold",
    color: context.color
  };

  return <div style={style}>Hi {props.name}!</div>;
}

// Set defaults for any required props using defaultProps.

function Greeting(props) {
  return <div>Hi {props.name}!</div>;
}
Greeting.defaultProps = {
  name: "Guest"
};
```

## Destructuring props
```jsx
// Destructuring assignment is used a lot in function components.
// These component declarations below are equivalent.

function Greeting(props) {
  return <div>Hi {props.name}!</div>;
}

function Greeting({ name }) {
  return <div>Hi {name}!</div>;
}

// There's a syntax for collecting remaining props into an object.
// It's called rest parameter syntax and looks like this.

function Greeting({ name, ...restProps }) {
  return <div>Hi {name}!</div>;
}
```

## JSX spread attributes
```jsx
function Greeting({ name, ...restProps }) {
  return <div {...restProps}>Hi {name}!</div>;
}
// This makes Greeting super flexible.
// We can pass DOM attributes to Greeting and trust that they'll be passed through to div.

<Greeting name="Fancy pants" className="fancy-greeting" id="user-greeting" />

// Avoid forwarding non-DOM props to components.
// Destructuring assignment is popular because it gives you a way to separate component-specific props from DOM/platform-specific attributes.

function Greeting({ name, ...platformProps }) {
  return <div {...platformProps}>Hi {name}!</div>;
}
```

## Merge destructured props with other values
```jsx
// Consider this component that uses a class attribute for style a button.

function MyButton(props) {
  return <button className="btn" {...props} />;
}

// This works great until we try to extend it with another class.

<MyButton className="delete-btn">Delete...</MyButton>
// In this case, delete-btn replaces btn.

// Order matters for JSX spread attributes.
// The props.className being spread is overriding the className in our component.

// We can change the order but now the className will never be anything but btn.

function MyButton(props) {
  return <button {...props} className="btn" />;
}

// We need to use destructuring assignment to get the incoming className and merge with the base className.
// We can do this simply by adding all values to an array and joining them with an space.

function MyButton({ className, ...props }) {
  let classNames = ["btn", className].join(" ");

  return <button className={classNames} {...props} />;
}

// To guard from undefined showing up as a className,
// Use default values.

function MyButton({ className = "", ...props }) {
  let classNames = ["btn", className].join(" ");

  return <button className={classNames} {...props} />;
}

```

## Conditional rendering
```jsx
// You can't use if/else statements inside a component declarations.
// So conditional (ternary) operator and short-curcuit evaluation are your friends.

// if
{
  condition && <span>Rendered when `truthy`</span>;
}
// unless
{
  condition || <span>Rendered when `falsy`</span>;
}
// if-else
{
  condition ? (
    <span>Rendered when `truthy`</span>
  ) : (
    <span>Rendered when `falsy`</span>
  );
}
```

## Children types
```jsx
// React can render children from most types.
// In most cases it's either an array or a string.

// String
<div>Hello World!</div>
// Array
<div>{["Hello ", <span>World</span>, "!"]}</div>
```

## Array as children
```jsx
// Array as children
// Providing an array as children is a very common.
// It's how lists are drawn in React.

// We use map() to create an array of React Elements for every value in the array.

<ul>
  {["first", "second"].map(item => (
    <li>{item}</li>
  ))}
</ul>

// That's equivalent to providing a literal array.

<ul>{[<li>first</li>, <li>second</li>]}</ul>

// This pattern can be combined with destructuring, JSX Spread Attributes, and other components, for some serious terseness.

<ul>
  {arrayOfMessageObjects.map(({ id, ...message }) => (
    <Message key={id} {...message} />
  ))}
</ul>

```

## Function as children
React components don't support functions as children. However, render props is a pattern for creating components that take functions as children.

## Render prop
```jsx
// Here's a component that uses a render callback.
// It's not useful, but it's an easy illustration to start with.

const Width = ({ children }) => children(500);

// The component calls children as a function, with some number of arguments. Here, it's the number 500.

// To use this component, we give it a function as children.

<Width>{width => <div>window is {width}</div>}</Width>

// We get this output.

<div>window is 500</div>

// With this setup, we can use this width to make rendering decisions.

<Width>
  {width => (width > 600 ? <div>min-width requirement met!</div> : null)}
</Width>

// If we plan to use this condition a lot, we can define another components to encapsulate the reused logic.

const MinWidth = ({ width: minWidth, children }) => (
  <Width>{width => (width > minWidth ? children : null)}</Width>
);

// Obviously a static Width component isn't useful but one that watches the browser window is. Here's a sample implementation.

class WindowWidth extends React.Component {
  constructor() {
    super();
    this.state = { width: 0 };
  }

  componentDidMount() {
    this.setState(
      { width: window.innerWidth },
      window.addEventListener("resize", ({ target }) =>
        this.setState({ width: target.innerWidth })
      )
    );
  }

  render() {
    return this.props.children(this.state.width);
  }
}
```

## Children pass-through
```jsx
// You might create a component designed to apply context and render its children.

class SomeContextProvider extends React.Component {
  getChildContext() {
    return { some: "context" };
  }

  render() {
    // how best do we return `children`?
  }
}
/* You're faced with a decision. Wrap children in an extraneous <div /> or return children directly. The first options gives you extra markup (which can break some stylesheets). The second will result in unhelpful errors.*/

// option 1: extra div
return <div>{children}</div>;

// option 2: unhelpful errors
return children;

/* It's best to treat children as an opaque /data type. React provides React.Children for dealing with children appropriately.*/

// return React.Children.only

(this.props.children);
```

## Proxy component
```jsx
/*Buttons are everywhere in web apps. And every one of them must have the type attribute set to "button".*/

<button type="button" />

/*Writing this attribute hundreds of times is error prone. We can write a higher level component to proxy props to a lower-level button component.*/

const Button = props =>
  <button type="button" {...props} />

/*We can use Button in place of button and ensure that the type attribute is consistently applied everywhere.*/

<Button />

/* <button type="button"></button> */

<Button className="CTA">Send Money</Button>

/* <button type="button" class="CTA">Send Money</button> */
```

## Style component
```jsx
// This is a Proxy component applied to the practices of style.

// Say we have a button. It uses classes to be styled as a "primary" button.

<button type="button" className="btn btn-primary" />

// We can generate this output using a couple single-purpose components.

import classnames from "classnames";

const PrimaryBtn = props => <Btn {...props} primary />;

const Btn = ({ className, primary, ...props }) => (
  <button
    type="button"
    className={classnames("btn", primary && "btn-primary", className)}
    {...props}
  />
);

// It can help to visualize this.

/*
PrimaryBtn()
  ↳ Btn({primary: true})
    ↳ Button({className: "btn btn-primary"}, type: "button"})
      ↳ '<button type="button" class="btn btn-primary"></button>'
*/

// Using these components, all of these result in the same output.

<PrimaryBtn />
<Btn primary />
<button type="button" className="btn btn-primary" />

// This can be a huge boon to style maintenance. It isolates all concerns of style to a single component.
```

## Event switch
```jsx
// When writing event handlers it's common to adopt the handle{eventName} naming convention.

handleClick(e) { /* do something */ }

// For components that handle several event types, these function names can be repetitive. The names themselves might not provide much value, as they simply proxy to other actions/functions.

handleClick() { require("./actions/doStuff")(/* action stuff */) }
handleMouseEnter() { this.setState({ hovered: true }) }
handleMouseLeave() { this.setState({ hovered: false }) }

// Consider writing a single event handler for your component and switching on event.type.

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

// Alternatively, for simple components, you can call imported actions/functions directly from components, using arrow functions.

<div onClick={() => someImportedAction({ action: "DO_STUFF" })}

// Don't fret about performance optimizations until you have problems. Seriously don't.
```

## Layout component
```jsx
// Layout components result in some form of static DOM element. It might not need to update frequently, if ever.

// Consider a component that renders two children side-by-side.

<HorizontalSplit
  leftSide={<SomeSmartComponent />}
  rightSide={<AnotherSmartComponent />}
/>

// We can aggressively optimize this component.

// While HorizontalSplit will be parent to both components, it will never be their owner. We can tell it to update never, without interrupting the lifecycle of the components inside.

class HorizontalSplit extends React.Component {
  shouldComponentUpdate() {
    return false;
  }

  render() {
    <FlexContainer>
      <div>{this.props.leftSide}</div>
      <div>{this.props.rightSide}</div>
    </FlexContainer>;
  }
}

```

## Container component
```jsx
// "A container does data fetching and then renders its corresponding sub-component. That’s it."—Jason Bonta

// Given this reusable CommentList component.

const CommentList = ({ comments }) => (
  <ul>
    {comments.map(comment => (
      <li>
        {comment.body}-{comment.author}
      </li>
    ))}
  </ul>
);

// We can create a new component responsible for fetching data and rendering the CommentList function component.

class CommentListContainer extends React.Component {
  constructor() {
    super()
    this.state = { comments: [] }
  }

  componentDidMount() {
    $.ajax({
      url: "/my-comments.json",
      dataType: 'json',
      success: comments =>
        this.setState({comments: comments});
    })
  }

  render() {
    return <CommentList comments={this.state.comments} />
  }
}
// We can write different containers for different application contexts
```

## Higher-order component
```jsx
// A higher-order function is a function that takes and/or returns a function. It's not more complicated than that. So, what's a higher-order component?

// If you're already using container components, these are just generic containers, wrapped up in a function.

// Let's start with our Greeting component.

const Greeting = ({ name }) => {
  if (!name) {
    return <div>Connecting...</div>;
  }

  return <div>Hi {name}!</div>;
};

// If it gets props.name, it's gonna render that data. Otherwise it'll say that it's "Connecting...". Now for the the higher-order bit.

const Connect = ComposedComponent =>
  class extends React.Component {
    constructor() {
      super();
      this.state = { name: "" };
    }

    componentDidMount() {
      // this would fetch or connect to a store
      this.setState({ name: "Michael" });
    }

    render() {
      return <ComposedComponent {...this.props} name={this.state.name} />;
    }
  };

// This is just a function that returns component that renders the component we passed as an argument.

// Last step, we need to wrap our our Greeting component in Connect.

const ConnectedMyComponent = Connect(Greeting);

// This is a powerful pattern for providing fetching and providing data to any number of function components.
```

## State hoisting
```jsx
/*
function-component don't hold state (as the name implies).

Events are changes in state. Their data needs to be passed to stateful container components parents.

This is called "state hoisting". It's accomplished by passing a callback from a container component to a child component.
*/

class NameContainer extends React.Component {
  render() {
    return <Name onChange={newName => alert(newName)} />;
  }
}

const Name = ({ onChange }) => (
  <input onChange={e => onChange(e.target.value)} />
);

/*
Name receives an onChange callback from NameContainer and calls on events.

The alert above makes for a terse demo but it's not changing state. Let's change the internal state of NameContainer.
*/

class NameContainer extends React.Component {
  constructor() {
    super();
    this.state = { name: "" };
  }

  render() {
    return <Name onChange={newName => this.setState({ name: newName })} />;
  }
}

/*
The state is hoisted to the container, by the provided callback, where it's used to update local state. This sets a nice clear boundary and maximizes the re-usability of function component.

This pattern isn't limited to function components. Because function components don't have lifecycle events, you'll use this pattern with component classes as well.

Controlled input is an important pattern to know for use with state hoisting

(It's best to process the event object on the stateful component)
*/
```

## Controlled input
```jsx
/*
It's hard to talk about controlled inputs in the abstract. Let's start with an uncontrolled (normal) input and go from there.
*/

<input type="text" />

/*
When you fiddle with this input in the browser, you see your changes. This is normal.

A controlled input disallows the DOM mutations that make this possible. You set the value of the input in component-land and it doesn't change in DOM-land.
*/

<input type="text" value="This won't change. Try it." />

// Obviously static inputs aren't very useful to your users. So, we derive a value from state.

class ControlledNameInput extends React.Component {
  constructor() {
    super();
    this.state = { name: "" };
  }

  render() {
    return <input type="text" value={this.state.name} />;
  }
}

// Then, changing the input is a matter of changing component state.

return (
  <input
    value={this.state.name}
    onChange={e => this.setState({ name: e.target.value })}
  />
);

// This is a controlled input. It only updates the DOM when state has changed in our component. This is invaluable when creating consistent UIs.
```