# test_mock_note
## require
- react 
- vitest

## TLDR
use default export.

## tree
```bash 
── src
│   ├── child.jsx
│   ├── default_child.jsx
│   ├── props_child_component.jsx
│   ├── parent.jsx
│   └── parent.test.jsx
```

### parent.jsx
```jsx
import { ChildComponent, ChildComponent2 } from './child'
import ChildComponent_default from './default_child'

export function ParentComponent () {
  return (
    <div className='ParentComponent'>
      <div>Parent Component</div>
      <ChildComponent />
      <ChildComponent2 />
      <ChildComponent_default />
    </div>
  )
}
```
### child.jsx  (test1)
```jsx
export function ChildComponent () {
  return (
    <div className='ChildComponent'>
      Child component
    </div>
  )
}

export function ChildComponent2 () {
  return (
    <div className='ChildComponent2'>Child component 2</div>
  )
}
```
### default_child.jsx (test2)
```jsx
function ChildComponent_default () {
  return (
    <div className='ChildComponent_default'>
      Child component default
    </div>
  )
}
export default ChildComponent_default
```
### props_child_component.jsx (test3)
```jsx
function ChildComponent_props (props) {
  return (
    <div className='ChildComponent_props'>
      props name is {props.name}
    </div>
  )
}
export default ChildComponent_props
```

### parent.test.jsx
```jsx
import { screen, render } from '@testing-library/react'
import { ParentComponent } from './parent'
import { vi } from 'vitest'
import { ChildComponent, ChildComponent2 } from './child'
import ChildComponent_default from './default_child'
import ChildComponent_props from './props_child_component'

test('no test', () => {
  render(<ParentComponent />)
  screen.debug()
})
```
### screen.debug()
```html
<body>
  <div>
    <div
      class="ParentComponent"
    >
      <div>
        Parent Component
      </div>
      <div
        class="ChildComponent"
      >
        Child component
      </div>
      <div
        class="ChildComponent2"
      >
        Child component 2
      </div>
      <div
        class="ChildComponent_default"
      >
        Child component default
      </div>
    </div>
  </div>
</body>
```
# 
## test 1 : mock the name import module (child.jsx)
### auto mock :
It set all export in the file to vi.fn()
```javascript
test('auto mock', () => {
  vi.mock('./child.jsx')

  render(<ParentComponent />)
  // Check if the mock function was called
  expect(ChildComponent).toHaveBeenCalled()
  // check if it is mock function
  expect(vi.isMockFunction(ChildComponent)).toBe(true)
  // check it return undefined ,since we not given value ,vi.fn()
  expect(ChildComponent()).toBeUndefined()
  expect(ChildComponent).toHaveReturnedWith(undefined)

  screen.debug()
})
```
and you can use mock function 
```javascript
test('auto mock using mock function', () => {
  vi.mock('./child.jsx')

  ChildComponent.mockReturnValue(<button className='ChildComponent'>2</button>)
  ChildComponent2.mockReturnValue(<div className='ChildComponent2'>hi</div>)

  render(<ParentComponent />)

  expect(ChildComponent).toHaveReturnedWith(<button className='ChildComponent'>2</button>)
  expect(ChildComponent2).toHaveReturnedWith(<div className='ChildComponent2'>hi</div>)
  screen.debug()
})
```
### custom mock :
return a object with setting mock function,every export must be set.
```javascript
test('custom mock', () => {
  vi.mock('./child.jsx', () => {
    return {
      ChildComponent: vi.fn(() => <div className='ChildComponent'>123</div>),
      ChildComponent2: vi.fn(() => <div className='ChildComponent2'>456</div>)
    }
  })

  render(<ParentComponent />)
  screen.debug()
})
```
if you forgot one of exports ( e.g. ChildComponent2) will fail.
```typescript
FAIL  src/parent.test.jsx [ src/parent.test.jsx ]
Error: [vitest] No "ChildComponent2" export is defined on the "./child.jsx" mock. Did you forget to return it from "vi.mock"?
If you need to partially mock a module, you can use "vi.importActual" inside:

vi.mock("./child.jsx", async () => {
  const actual = await vi.importActual("./child.jsx")
  return {
    ...actual,
    // your mocked methods
  },
})
```
if you want partially mock a module e.g. ChildComponent2
```js
test('custom mock only ChildComponent2', () => {
  // partial
  vi.mock('./child.jsx', async () => {
    const actual = await vi.importActual('./child.jsx')
    return {
      ...actual,
      ChildComponent2: vi.fn(() => <div className='ChildComponent2'>123</div>)
    }
  })
  render(<ParentComponent />)
  screen.debug()
})
```
## test 2 : mock the default import (default_child.jsx) 
### auto mock : 
same as name import
```javascript
vi.mock('./default_child.jsx')
```
### custom mock :
object name is 'default'
```javascript
vi.mock('./default_child.jsx', () => {
    return {
      default: vi.fn(() => 
      <button className='ChildComponent_default'>2</button>)
    }
  })
```
if you want to use mock function ,need to use the origin function name .
```javascript
test('custom mock only ChildComponent2', () => {
  vi.mock('./default_child.jsx')
  ChildComponent_default.mockReturnValue(<button className='ChildComponent_default'>after</button>)
  render(<ParentComponent />)
  screen.debug()
})
```
## test 3 : mock the props component (props_child_component.jsx)
```javascript
test('custom mock only ChildComponent2', () => {
  vi.mock('./props_child_component', () => {
    return {
      default: vi.fn(({ name, id, color }) =>
        <div className='use mock'>
          props name is {name}
          id is {id}
          color is {color}
        </div>
      )
    }
  })
  render(<ParentComponent />)
  screen.debug()
})
```

## note
### [vi.mock](https://vitest.dev/api/vi.html#vi-mock) 
```javascript
vi.mock('path',()=>{
  return {/*this is object*/}           
})
```
### in the object
```javascript
{
    default:vi.fn(()=> implementation ),
    //or
    modulename:vi.fn(()=>implementation)
}
```
[mock guide](https://vitest.dev/guide/mocking.html#mocking)
pay attenttion to how to partial mock module