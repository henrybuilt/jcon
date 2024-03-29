# JCON - Json Component Notation

JCON is inspired by React, but is much easier than JavaScript to parse and build tools around.

It provides an intuitive way to specify/represent UI components and apps in JSON, allowing you to inject code (JavaScript & CSS currently—soon TypeScript and maybe other languages) as needed.

## Example

```js
{
  type: 'Component',
  name: 'Counter',
  imports: [
    {import: {useState: {}}, from: 'react'}
  ],
  expressions: [
    '{var [count, setCount] = useState(1)}'
  ],
  children: [
    {
      type: 'div',
      props: {
        onClick: '{() => setCount(count + 1)}'
      },
      children: '{count}'
    }
  ]
}
```

#### app.jcon

Components are the primary focus of most front-end applications, but apps also have logic/helpers, assets, and dependencies.

JCON is typically used in a larger app.jcon file that describes the entire app—metadata, dependencies, components, etc.

JCON files can be edited directly, but their core value is that they're easy to build tools around, so we recommend using a tool like Scaffolding to edit them.

JCON apps are run by generating a repository (i.e. NextJS or React Native repo), which can be deployed or developed traditionally. This means all of a supported framework's features are supported without performance costs because a native, unopinionated repository can be generated and deployed traditionally.

JCON apps can use any existing npm module including existing component libraries because of this.

Please note the Further Development & Standardization section at the end for more details on the future of the standard.

```js
{
  name: 'Todo App',
  type: 'app',
  platform: 'cross-platform',
  dependencies: {
    'react': '18.0.0',
    'expo': '42.0.0',
    'react-native': '0.66.0',
    'react-native-web': '0.17.1',
    '@scaffolding-components/ui': '1.0.0'
  },
  imports: [
    {import: {useState: {}}, from: 'react'},
    {import: {View: {}, Text: {}, TextInput: {}, Button: {}, Image: {}, ScrollView: {}}, from: '@scaffolding-components/ui'},
  ],
  rootComponent: 'App',
  components: [
    {
      type: 'Component',
      name: 'App',
      expressions: [
        {type: 'state', var: ['tasks', 'setTasks'], initialState: []},
        {type: 'var', var: 'updateTask', value: `{(id, updates) => {
          setTasks(tasks.map(task => task.id === id ? {...task, ...updates} : task));
        }}`}
      ],
      render: [
        {type: 'ScrollView', props: {style: {}, contentContainerStyle: {}}, children: [
          {type: 'Array', data: '{tasks}', var: ['task', 'index'], children: [
            {type: 'Task', props: {task: '{task}', index: '{index}'}}
          ]}
        ]}
      ]
    },
    {
      type: 'Component',
      name: 'Task',
      imports: [
        {import: 'formatTaskName', from: 'formatTaskName'},
        {import: 'checkIcon', from: 'assets/check.png'}
      ],
      expressions: [
        {type: 'var', var: 'task', value: '{props.task}'}
      ],
      render: [
        {type: 'View', props: {style: {flexDirection: 'row'}}, children: [
          {type: 'Text', children: '{`${props.index + 1}:`}'},
          {type: 'TextInput', props: {value: '{task.title}', onChange: '{event => updateTask(task.id, {title: event.target.value})}'}}
          {type: 'Button', props: {onPress: '{() => updateTask(task.id, {completed: !task.completed})}'}, children: [
            {type: 'Image', props: {source: '{checkIcon}'}}
          ]}
        ]}
      ]
    }
  ]
}
```

The above example app would be transformed into the following JS:

```js
import {useState} from 'react';
import {View, Text, TextInput, Button, Image, ScrollView} from '@scaffolding-components/ui';
import checkIcon from 'assets/check.png';

function App() {
  var [tasks, setTasks] = useState([]);

  var updateTask = (id, updates) => {
    setTasks(tasks.map(task => task.id === id ? {...task, ...updates} : task));
  }

  return (
    <ScrollView style={{}} contentContainerStyle={{}}>
      {tasks.map((task, index) => (
        <Task task={task} index={index}/>
      ))}
    </ScrollView>
  );
}

function Task(props) {
  var {task} = props;

  return (
    <View style={{flexDirection: 'row'}}>
      <Text>{`${props.index + 1}:`}</Text>
      <TextInput value={task.title} onChange={event => updateTask(task.id, {title: event.target.value})}/>
      <Button onPress={() => updateTask(task.id, {completed: !task.completed})}>
        <Image source={checkIcon}/>
      </Button>
    </View>
  );
}
```

## Component Specification

#### `type`

The type of the component—typically what you'd put between `<` and `>` in JSX.

#### `props`

Properties to pass to component instances when rendering them.

e.g. `{type: 'Text', props: {text: 'Hello World'}}` => `<Text text="Hello World"/>`

Props support a single object shown above, but also support an array syntax that compiles to spread syntax:

`{type: 'Text', props: [{text: 'Hello World'}, '{isActive ? {fontWeight: 'bold'} : {}}'`

=> 

`<Text props={{text: 'Hello World', ...(isActive ? {fontWeight: 'bold'} : {})}}/>`

#### `children`

Child components to render inside the component, or a primitive like a string or number.

e.g. `{type: 'div', children: 'Hello World'}` => `<div>Hello World</div>`
e.g. `{type: 'div', children: [{type: 'input'}]}` => `<div><input /></div>`

#### `name`

A unique name is required for component definitions but is optional elsewhere (though it is helpful for clarity, searching, and debugging). We recommend StartCase for component names.

e.g. `{type: 'Component', name: 'App'}` => `function App() {`

#### `style`

There are many approaches to styling a website - from inline styles, to CSS/SASS, to various CSS-in-JSS approaches (i.e. StyledComponents).

We aim to cover all common use-cases, including CSS/SCSS stylesheets (unprocessed) and modules (autogenerated unique class names).

You can always just apply a style object:

```js
{
  type: 'Text',
  style: {fontSize: 10}
}
```

But the below example app is showing many possible use-cases including a global stylesheet, SCSS modules, and JCON style syntax (which generates CSS for static styles not relying on scripts).


**app.jcon**
```js
{
  styleSheet: `.dark-theme {
    background-color: black;
    color: white;
  }`,
  styleModule: `.flex {
    display: flex;
  }`,
  styleFormats: {
    wide: {minWidth: 1024},
    narrow: {maxWidth: 1023}
  },
  components: [
    {
      name: 'MyComponent',
      type: 'Component',
      styleSheet: ``,
      styleModule: ``,
      expressions: [
        {type: 'state', var: ['isExpanded', 'setIsExpanded'], initialState: true}
      ],
      children: [
        {
          type: 'View',
          name: 'MyView', //optional name for code organization - will use autogenerated name at compile time otherwise
          props: {
            className: ['.dark-theme', '{classes.flex}']
          },
          style: [
            {style: {height: 10, width: '{props.size * 10}'}},
            {condition: '{isExpanded}', style: {display: 'block'}}, //conditional styles
            {format: 'wide', style: {width: 30}}, //responsive layouts - css on web, js on cross-platform (see styleFormats above)
            {selector: '&:hover', style: {opacity: 0.5}}, //static sass-selector-based styles (web only)
            `&::-webkit-scrollbar { background-color: rgba(0, 0, 0, 0.1); }`, //static sass string (web only)
            `{props.style}`, //dynamic js script
          ],
          children: [
            {
              name: 'MyText',
              type: 'Text',
              style: {color: 'black'}, //plain object also works
              children: 'Hello world'
            },
            { //no name - will use autogenerated name at compile time
              type: 'Text',
              props: {style: {fontSize: 10}}, //you can use props.style to explicity pass inline styles
              style: {color: 'black'}, //plain object also works
              children: 'Hello again'
            }
          ]
        }
      ]
    }
  ]
}
```

This compiles to the following:

**global.scss**
```css
.dark-theme { /* from styleSheet */
  background-color: black;
  color: white;
}
```

**App.module.scss**
```css
.flex { /* from styleModule */
  display: flex;
}

.MyView {
  height: 10px; /* cross-platform uses styles.MyView instead for this static style */

  &:hover { /* cross-platform supports css in a web environment only (not ios/android) */
    opacity: 0.5;
  }

  @media (min-width: 1024px) { /* format: wide */
    width: 30px !important;
  }

  &::-webkit-scrollbar {
    background-color: rgba(0, 0, 0, 0.1);
  }
}
```

**App.js (NextJS)**
```js
import { useState } from 'react';
import classes from 'App.module.scss';
import 'global.scss';

export default function App() {
  return <MyComponent />
}

function MyComponent(props) {
  var [isExpanded, setIsExpanded] = useState(true);

  return (
    <div
      style={{height: 10, width: props.size * 10, ...(isExpanded ? {display: 'block'} : {display: 'none'}), ...props.style}}
      className={[classes.MyView, '.dark-theme', classes.flex].join(' ')}
    >
      <span className={classes.MyText}>Hello World</span>
    </div>
  )
}
```

**App.js (Expo/React Native)**
```js
import { useState } from 'react';
import { View, Text } from 'react-native';
import { useFormats, getUseStyle, useFormat } from 'jcon-react';
import classes from 'App.module.scss'; //works in web environment only
import 'global.scss'; //works in web environment only

export default function App() {
  return <MyComponent />
}

function MyComponent(props) {
  var [isExpanded, setIsExpanded] = useState(true);
  var useStyle = getUseStyle({classes, styles});

  // useStyle conditionally decides to
  // use css or js styles based on the environment
  // and reduces boilerplate

  return (
    <View
      style={useStyle([
        styles.MyView,
        {width: props.size * 10},
        isExpanded ? {display: 'block'} : {},
        useFormat('wide', {width: 30}),
        props.style
      ], {className: [classes.MyText, '.dark-theme', classes.flex]})}
    >
      <Text style={styles.MyText}>Hello world</Text>
      <Text style={[styles.Text1, {color: 'black'}]}>Hello again</Text>
    </View>
  );
};

var styles = {
  MyView: {
    height: 10
  },
  MyText: {
    color: 'black'
  },
  Text1: {
    color: 'black'
  }
};
```

#### `styleFormats`

It's common that apps support specific screen orientations/sizes - vertical/horizontal, wide/narrow.

You can acheive this with CSS media queries and classNames in web apps, or conditional logic/libraries in cross-platform apps, but we think it should be native behavior because most apps need some form of it.

You can define a format like so at the top level:

```js
{
  formats: {
    wide: {minWidth: 1024}
  }
}
```

...and then use them on a given component like so:

```js
{
  type: 'View',
  style: [
    {format: 'wide', style: {
      width: 1000
    }}
  ]
}
```

These formatted styles compile to CSS on web apps, and are managed by the useStyle hook in cross-platform apps.

#### `expressions` (type: Component only)

Expressions are used to inject logic into components. They are typically used to define state, variables, and functions. Some common patterns are supported natively for convenience when editing a JCON app using an interface.

Example expressions:

```js
{
  type: 'Component',
  name: 'Task',
  expressions: [
    '{var [counter, setCounter] = useState(1)}', // script
    {type: 'state', var: ['tasks', 'setTasks'], initialState: []}, // shorthand for useState - destructuring array
    {type: 'var', var: 'myValue', value: 1}, //variable
    {type: 'var', var: {task: {}, index: {alias: 'i', defaultValue: 0}, restProps: '...'}, value: '{props}'}, // complex object destructuring
    {type: 'var', var: ['x', {y: {defaultValue: 0}}, {restArray: '...'}}], value: [0, 0, 0]} //complex array destructuring
    {type: 'ref', var: 'inputRef', initialValue: null}, //ref
    {type: 'effect', setup: `{() => inputRef.current?.focus()}`, dependencies: []} //effect
  ],
  children: []
}
```

=>

```js
function Task(props) {
  var [counter, setCounter] = useState(1);
  var [tasks, setTasks] = useState([]);

  var updateTask = (id, updates) => {
    setTasks(tasks.map(task => task.id === id ? {...task, ...updates} : task));
  }

  var {task, index: i = 0, ...restProps} = props;
  var [x = 0, ...restArray] = [0, 0, 0];
  var inputRef = useRef();

  useEffect(() => {
    inputRef.current.focus();
  }, []);

  return null;
}
```

#### `imports` (type: Component only)

Imports are used to import dependencies, assets, and other components.

e.g.

```js
{
  type: 'Component',
  name: 'App',
  imports: [
    {import: {useState: {}}, from: 'react'},
    {import: {View: {}, Button: {}}, from: '@scaffolding-components/ui'},
    {import: '_', from: 'lodash'},
    {import: {'*': {alias: 'THREE'}}, from: 'three'}
  ],
  children: []
}
```

=>

```js
import {useState} from 'react';
import {View, Button} from '@scaffolding-components/ui';
import _ from 'lodash';
import * as THREE from 'three';

function App {
  return null;
}
```

#### `provideContexts` & `useContexts` (type: Component only)

React has a mechanism called "[contexts](https://react.dev/learn/passing-data-deeply-with-context)" to allow a component higher up in the tree (i.e. a root App component) down to components lower in the tree.

JCON has first-class support for contexts:

```js
[
  {
    type: 'Component',
    name: 'App',
    expressions: [{type: 'var', var: 'myValue', value: 1}],
    provideContexts: ['myValue'], // => <MyValueContext.Provider value={myValue}>{children}</MyValueContext.Provider>
    children: [
      {type: 'ChildComponent'}
    ]
  },
  {
    type: 'Component',
    name: 'ChildComponent',
    children: [
      {type: 'GrandchildComponent'}
    ]
  },
  {
    type: 'Component',
    name: 'ChildComponent',
    useContexts: ['myValue'], // => var myValue = useContext(MyValueContext);
    children: [
      {type: 'Text', props: {text: '{myValue}'}}
    ]
  }
]
```

Contexts generated using provideContexts must have unique names in the app, but eliminate significant boilerplate and setup, making contexts much more appealing to use and get the benefits of performance optimization/organization. 

You can always create your own contexts manually using expressions if you prefer.

## Native Component Types

#### `Component`

A component definition that can be rendered.

e.g.

```js
{
  type: 'Component',
  name: 'App',
  imports: [
    {import: {useState: {}}, from: 'react'}
  ],
  expressions: [
    '{var [count, setCount} = useState(1)}'
  ],
  children: [
    {type: 'div', props: {onClick: '{() => setCount(count + 1)}'}, children: '{count}'}
  ]
}
```

=>

```js
import {useState} from 'react';

function App() {
  var [count, setCount] = useState(1);

  return (
    <div onClick={() => setCount(count + 1)}>{count}</div>
  );
}
```

#### `Map`

Display child components for each item in an array.

e.g. `{type: 'Map', data: '{tasks}', var: ['task', 'index'], children: []}` => `{tasks.map((task, index) => (`

#### `If`

Conditionally render components.

e.g. `{type: 'If', condition: '{isEnabled}', children: []}` => `{isEnabled && (`

#### `Script`

Write JS inline in JSX.

e.g. `{type: 'Script', script: '{isEnabled && counter + 1}'}` => `{isEnabled && counter + 1}`

## Further Development & Standardization

JCON is still in early development, and we're looking for contributors to help build out the ecosystem.

What we have so far is a strong baseline for further development of JCON as a standard because it:

- covers compatibility for a large portion of front-end apps
- is easy to understand (JSON) & familiar (like React/JSX)
- is easy to extend (much easier than introducing a new ECMA standard)
- is easy to build better tools for developers around (improving the Javascript + filesystem workflow is really hard—building around JSON is much more manageable)

The above standard is quite stable, but some app-level properties like `files` and `rootComponent` may be subject to change. Remaining V1 standardization efforts are focused on JS/React (specifically NextJS for web-only apps and Expo/React Native for cross-platform apps), but we're also interested in supporting other languages/frameworks.

More documentation on the JCON spec is coming soon as it is built out.

Contact support@symbolicframeworks.com if you're interested in contributing or using JCON in your project—or create an Issue/PR if you have an idea/improvement.

## License

MIT © [Symbolic Frameworks](https://www.symbolicframeworks.com) 2024
