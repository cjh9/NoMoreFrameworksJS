# NoMoreFrameworks.js
Embrace the web 🌍 Keep it simple ✨ 

What will happen in 5 years? You don't know if your framework will survive, but JavaScript will for sure! ✌ And a lot more functionality will come with new JS standards. These are some suggestions when building your web application. ✅ Just copy-paste the demos and add them to a html file.

## 1. Build views with virutal DOM 🎨
They are pretty fast (fast enough), [Benchmark](https://stefankrause.net/js-frameworks-benchmark8/table.html). A lot of frameworks nowadays use it. A view is just a function that returns virtual dom, which is an object (or array of objects) which describes how the html should look like. Recalclulate the virtual dom for the whole application at each render. Inefficient? ([Root of all evil](http://wiki.c2.com/?PrematureOptimization) + moore's law + webworkers) 
``` js
<body><script type="module">
import { h, setup } from "https://unpkg.com/lifecycle-petit-dom?module"

const MyFancyComponent = () => 
  h('div', {}, [
    h('h1', {}, 'Hello World!'),
    h('button', {}, "Button that doesn't do anything"),
    h('p', {}, [
      h('b', {}, 'Bold text')
    ])
  ])

setup(MyFancyComponent)
</script></body>
```

## 2. Use hyperscript for the views ☝️
Don't use JSX, hyperscript works great! The first argument should be the tag name, second should be an object with properties (so you can attach event listeners) or attributes, and the third should be the contents of the domnode. 
```  js
<body><script type="module">
import { h, setup } from "https://unpkg.com/lifecycle-petit-dom?module"

var isChecked = false

const MyFancyComponent = () => h('div', {}, [
  h('input', {
    type:'checkbox',
    key:'sdf',
    checked:isChecked,
    onclick: ()=> {
      isChecked = !isChecked
      render()
    }
  }, []),
  isChecked && "Hello!",
  h('br', {}, []),
  "Please check the box"
])

const render = setup(MyFancyComponent)
</script></body>
```

## 3. Use DOM lifecycle hooks for animation ▶️ 
The view library should support onremove and oncreate callbacks so you can make nice animations. If an element is removed all its ancestors should be notified as well. You may want to write some utility function for this.
```  js
<body><script type="module">
import { h, setup } from "https://unpkg.com/lifecycle-petit-dom?module"

var isChecked = false

var enterFrom = `transform: translateX(-100px)`
var enterTo = `transition: all 0.3s; transform: translateX(0px)`
var exitTo = `transition: all 0.3s 0.3s; transform: translateX(-100px)`

var enterFromInner = `display:inline-block;transform: rotate(90deg);`
var enterToInner = `display:inline-block; transition:all 0.3s 0.3s; transform:rotate(0deg);`
var exitToInner = `transition:all 0.3s; transform: rotate(90deg);display:inline-block;`

const MyFancyComponent = () => h('div', {}, [
  h('input', {
    type:'checkbox',
    checked:isChecked,
    onclick: ()=> {
      isChecked = !isChecked
      render()
    }
  }, []),
  isChecked && h('div',
    {
      oncreate: elm => {
        elm.style.cssText = enterFrom
        setTimeout(()=>elm.style.cssText = enterTo,0)
      },
      onremove: (elm, done) => {
        elm.style.cssText = exitTo
        setTimeout(done, 600)
      }
    },
    h('div', {
      oncreate: elm => {
        elm.style.cssText = enterFromInner
        setTimeout(() => elm.style.cssText = enterToInner,0)
      },
      onremove: (elm, dn) => {
        elm.style.cssText = exitToInner
      }
    }, 'Hello animations')
  )
])

const render = setup(MyFancyComponent)
</script></body>
```

## 4. Build single file components in ES6 modules 📁
Just like in Vue.js. A component should export it's _view function_ and an optional _factory to create its model_. Perhaps some functions to manipulate a components model. 💥💥💥[**Compose views and models.**](https://reactjs.org/docs/composition-vs-inheritance.html)💥💥💥
```  js
<body><script type="module">
import { h, setup } from "https://unpkg.com/lifecycle-petit-dom?module"

const randX = () => window.innerWidth*Math.random()
const randY = () => window.innerHeight*Math.random()
const randColor = (current) => 
['red', 'green', 'purple', 'yellow']
.filter( c => c !== current)
[Math.round(Math.random()*2)]

/* ########################### In Ball.js ########################### */

const createBallModel = ()=> ({
    x:Math.random(),
    y:Math.random(),
    dx: Math.random(),
    dy: Math.random(),
    color:randColor(),
    speed:0.005,
    size:100,
})

const updateBall = (mdl) => {
    const {dx, dy, x, y, speed} = mdl
    mdl.x += dx * speed
    mdl.y += dy * speed

    if( x > 1 || x < 0) {
        mdl.dx = -dx
        mdl.x = Math.round(x)
    }
    if(  y > 1 || y < 0 ) {
        mdl.dy = -dy
        mdl.y = Math.round(y)        
    }
}

const ballStyle = ({size, x, y, color}) => `
width:${size};
height:${size};
display:flex;
justify-content:center;
cursor:pointer;
user-select:none;
align-items:center;
font-weight:bold;
border-radius:50%;
position: absolute;
transform: translate(${x*window.innerWidth-size/2}px, ${y*window.innerHeight-size/2}px);
background:${color};
`

const Ball = (mdl)  => 
h('div',
    {
        onclick: ()=> {
            mdl.color = randColor(mdl.color)
            render()
        },
        style: ballStyle(mdl)
    },
    ['Click me']
)

/* ########################### In BallField.js ########################### */

const createBallFieldModel = ()=> ({
    balls:[
        createBallModel(),
        createBallModel(),
        createBallModel(),
    ]
})

var ballFieldModel = createBallFieldModel()

const BallField = () => h('div', 
    {
        style:`width:100%;height:100%;`
    },
    h('button', {
        onclick: ()=> {
            ballFieldModel.balls.push(createBallModel()),
            render()
        }
    }, ['Add']),
    ballFieldModel.balls.map( ballModel => Ball(ballModel) )
)

const draw = ()=>{
    ballFieldModel.balls.forEach( ballModel => updateBall(ballModel)),
    render()
    requestAnimationFrame(draw)
}
requestAnimationFrame(draw)

const render = setup(BallField)
</script></body>
```

## 5. Use template strings to generate stylestrings 👁️
Don't use css with classes:s or id:s. Keep your single file componens isolated and pluggable.

## 6. Don't worry about state 😟
Try to make the application modular and pluggable, use ES6 modules! Encapsulate state. Don't build in hard dependecies to some  fancy but hard to learn state library. Keep it simple. Object orientation works 😃

## 7. Conventions 🤝
Use [named function parameters](http://2ality.com/2011/11/keyword-parameters.html), at least for the views. `nameRenderFn` : A composeable renderfunction (similar to children in react),  for example `Drawer({activeIdx, contentRenderFn, drawerModel})`. `activeIdx` : An integer which is and index, should end with "Idx". View function should be capitalized, `Drawer` an the model factory should be like this : `createDrawerModel`. 

## Please send your components ✉️
They should be in a single file, pluggable, expose its view, and a factory function to create its model. You can assume that the view is called with it's model. Perhaps you can let the view register its own state, and only supply an id to the view. To keep it completely isolated from the outside world.
```  js
<body><script type="module">
import { h, setup } from "https://unpkg.com/lifecycle-petit-dom?module"

// In Counter.js

const createCounterModel = ()=>({value:0})
var models = {}

const Counter = ({id}) => {
    models[id] = models[id] || createCounterModel()
    var mdl = models[id]
    return h('div', {
            key:`counter-${id}`,
            onremove: (elm, done) => {
                //Clean up the state
                delete models[id]
                done()
            } 
        },
        h('button', {onclick:()=> {
            mdl.value++
            render()
        }}, ['+']),
        h('button', {onclick:()=> {
            mdl.value--
            render()
        }}, ['-']),
        h('span', {}, ['' + mdl.value])
    )
}

// In Main.js

var numberOfCounters = 3
const Root = ()=> 
h('div', {}, [
    h('button', {
        onclick:()=>{
            numberOfCounters++
            render()
        }
    }, ['add']),
    h('button', {
        onclick:()=>{
            numberOfCounters--
            render()
        }
    }, ['remove']),
    Array(numberOfCounters)
    .fill('')
    .map( (_, idx) => Counter({id:idx}))
])

const render = setup(Root)
</script></body>
```

### Don't like to render? Hook in to the events 🔨 
Or do some thing else, create utility functions as much as possible.
```  js
<body><script type="module">
import { h as hframework, setup } from "https://unpkg.com/lifecycle-petit-dom?module"

const h = (a,b,c) => {
    if(typeof b.oninputr === 'function') {
        b.oninput = (...args)=> {
            b.oninputr(...args)
            render()
        }
    }
    return hframework(a, b, c)
}

var txt = ''
const Root = ()=> 
h('div', {}, [
    h('div', {}, txt),
    h('input', {
        oninputr: (evt) => txt = evt.target.value
    })
])

const render = setup(Root)
</script></body>
```

## 6. What else? Please tell me :) Give suggestions 💡



