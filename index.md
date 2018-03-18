title: "Codemods intro"
controls: false
progress: true
theme: ./cleaver-theme-sunset

---

# Codemods
## A brief introduction
![background](codebg.jpg)

---

### What are codemods?

 - Sometimes called code **transformations**
 - Automatise programming
 - Can be applied to your entire code base
 - Making transformations on **semantic** level
     - Analyse and transform *AST*, not *code* directly

---

### Basic code modifications

Identifier is updated:

```diff
export {
-  foo
+  bar
}
```

Find and replace 💪

```diff
import {
-  foo
+  bar
} from './myLib.js'

// ...

- foo()
+ bar()
```
---

### Why do you need the AST?

Semantics are not visible in code alone:

```js
import { foo as fooA } from './myLib.js'
import { foo as fooB } from './otherLib.js'

obj.foo = 42

sendMessage('foo')
```

- Using regex?
  - Still very limited

---

### Changing code structure

Binding manually:

```js
class MyComponent extends React.Component {
  constructor () {
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick() {
    this.setState({ open: true })
  }
}
```

Using arrow function as class property:

```js
class MyComponent extends React.Component {
  handleClick = () => this.setState({ open: true })
}
```

---

### Reading the AST

```js
class MyComponent extends React.Component {
  constructor () {
    this.handleClick = this.handleClick.bind(this)
  }

  handleClick() {
    this.setState({ open: true })
  }
}
```

- ClassDeclaration (`MyComponent`)
  - MethodDefinition (`constructor`)
    - AssignmentExpression (`=`)
      - left: MemberExpression (`this.handleClick`)
      - right: CallExpression (`bind()`)
  - MethodDefinition (`handleClick`)
    - CallExpression (`setState()`)

---

```js
// moves this.state assignment in constructor to class property
export default function transformer(file, api) {
  const j = api.jscodeshift

  const root = j(file.source)

  const stateAssignments = root
    .find(j.ClassDeclaration)
    .find(j.MethodDefinition, { kind: 'constructor' })
    .find(j.AssignmentExpression, {
      left: {
        object: j.ThisExpression,
        property: { name: 'state' }
      }
    })

  stateAssignments.forEach(stateAssignment => {
      const idName = stateAssignment.node.left.property
      const stateValue = stateAssignment.node.right
      const stateProperty = j.classProperty(
        idName, stateValue, null, false
      )

      j(stateAssignment)
        .closest(j.ClassBody)
        .get('body')
        .insertAt(0, stateProperty)

      j(stateAssignment).remove()
    })

  return root.toSource()
}
```

---

# That's it for today 👌
