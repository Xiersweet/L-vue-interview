### 1.v-if和v-for哪个优先级高？如果两个同时出现，应该怎么优化得到更好的性能？

1. 当它们处于同一节点时，v-for比v-if优先级更高。因为v-for需要将模板转换为VNode，而v-if是VNode中负责控制是否渲染成真实Dom的对象，所以v-for比v-if优先级更高
2. 优化方案：
   - 如果是为了过滤掉数据结构中某个状态数据的话，可以使用computed计算属性来对v-for要迭代的数据进行过滤，则可以省略v-if判断，只迭代出想要的数据项。
   - 如果v-if是为了控制v-for渲染后的整体结果显示隐藏的话，那么建议将v-if提升至v-for的父容器上，这样可以避免v-if条件不成立时，仍然会去进行v-for的渲染，造成无意义的性能浪费。

### 2.Vue组件data选项为什么必须是个函数而Vue的根实例则没有此限制？

1. vue组件的data选项，是用来管理组件中需要绑定的数据。而组件有可能会被同时、多次调用，此时如果data属性为对象的话，一旦修改其中一个组件的数据，其他组件相同数据就会被改变。而如果data作为函数的话，每次使用组件return data对象时都会在栈中开辟一块新的内存空间，所以相同组件多次使用，组件间不会互相影响。
2. 根实例是在应用初始化时，就对data中的数据进行响应式处理，且初始化完成之后无法在根实例的data上继续添加响应式属性，所以不存在响应冲突的问题，因此vue根实例的data可以是对象形式。

- Vue组件可能存在多个实例，如果使用对象形式定义data，则会导致他们公用一个data对象，那么状态变更将会影响所有组件实例，这是不合理的；采用函数形式定义，在initData时会将其作为一个工厂函数返回全新对象，有效规避多实例之间状态污染的问题。而在Vue根实例创建过程中则不存在该限制，因为根实例创建，是单例模式，实例只能有一个，所以不需要担心这种情况。

### 3.你知道vue中key的作用和工作原理吗？说说你对它的理解。

- key是为了配合vue的diff算法，管理可复用的元素，减少不必要的元素的重新渲染,也要让必要的元素能够重新渲染。

- 在高度复用的组件或者循环渲染的元素上，建议增加一个值唯一的key属性，目的是在于当vue响应数据变化时，一来可以根据key，更快更精确地定位到VNode。其次，相同标签名元素的过渡切换时，也会使用到key属性，其目的也是为了让vue可以区分它们，否则vue只会替换其内部属性而不会触发过渡效果。

1. key的作用主要是为了高效地更新虚拟DOM，其原理是vue在patch的过程中通过key可以精准判断两个节点是否是同一个，从而避免频繁更新不同元素，使得整个patch过程更加高效，减少DOM操作量，提高性能。
2. 另外，若不设置key还可能在列表更新时引发一些隐蔽的bug
3. vue中在使用相同标签名元素的过渡切换时，也会用到key属性，其目的也是为了让vue可以区分它们，否则vue只会替换其内部属性而不会触发过渡效果。

### 4.你怎么理解vue中的diff算法？

- vue是依靠虚拟dom来进行高效的页面渲染，当依赖绑定的数据发生变化时，需要更新某一个或多个组件来进行响应。此时，就必须要对新、老两套dom元素进行比较，这个比较的过程就是diff算法。

- 在进行diff算法比较时，首先会比较同级元素，如果两个元素未发生改变且均包含子元素，则会递归向下比较。

- vue对diff算法进行了改进，会从新老VNode树的头尾开始向中间比较，如果找到，则将VNode进行位移，如果未找到对应匹配，则开始双重循环比较，这样的改进优化了算法的执行效率，对于变化不大的视图来说，可以减少循环次数，节省性能

1. diff算法是虚拟DOM技术的必然产物：通过新旧虚拟DOM作对比（即diff），将变化的地方更新在真实DOM上；另外，也需要diff高效的执行对比过程，从而降低时间复杂度为O(n)。
2. vue 2.x中为了降低Watcher粒度，每个组件只有一个Watcher与之对应，只有引入diff才能精确找到变化的地方。
3. vue中diff执行的时刻是组件实例执行其更新函数时，它会比对上一次渲染结果oldVNode和新的渲染结果newVNode，此过程称为patch。
4. diff过程整体遵循深度优先，同级比较的策略；两个节点之间比较会根据它们是否拥有子节点或者文本节点做不同操作；比较两组子节点是算法的重点，首先假设头尾节点可能相同做4次对比尝试，如果没有找到相同节点才按照通用方式遍历查找，查找结束再按情况处理剩余节点；借助key通常可以非常精确找到相同节点，因此整个patch过程非常高效。

### 5.谈一谈对vue组件化的理解？

1. 组件是独立和可复用的代码组织单元。组件系统是vue核心特性之一，它使开发者使用小型、独立和通常可复用的组件构建大型应用；
2. 组件化开发能大幅度提高应用开发效率、测试性、复用性等；
3. 组件使用按分类有：页面组件、业务组件、通用组件；
4. vue的组件是基于配置的，我们通常编写的组件是组件配置而非组件，框架后续会生成其构造函数，它们基于VueComponent，扩展于Vue；
5. vue中常见的组件化技术有：属性prop、自定义事件、插槽等，它们主要用于组件通信、扩展等；
6. 合理地划分组件，有助于提高应用性能；
7. 组件应该是高内聚，低耦合的；
8. 遵循单向数据流的原则。

### 6.谈一谈对vue设计原则的理解？

- vue作为一个渐进式JavaScript框架，有着灵活、易用和高效的特点：
    - 渐进式的设计，使得一个vue项目可以作为自底向上的逐层应用。
    - 易用性：vue提供数据响应式，声明式模板语法和基于配置的组件系统等核心特性。这些使得我们可以只关注应用的核心业务即可，只要会写js、css和html就可以轻松编写vue应用。
    - 灵活性：渐进式框架最大的优点就是灵活性，如果应用足够小，那么我们可能仅需要使用vue核心特性即可完成实现；随着应用规模不断扩大，我们才可能逐渐引入路由、状态管理、vue-cli等库和工具，不管是应用体积还是学习难度都是一个逐渐增加的平和曲线。
    - 高效性：vue超快的虚拟DOM和diff算法使得应用拥有最佳的性能表现。未来vue3中引入Proxy对数据响应式改进以及编译器中对于静态内容编译的改进都会让vue更加高效，追求高效的过程还在继续。

### 7.vue为什么要求组件模板只能有一个根元素？

- 在vue中，每一个组件都会被视为一个vnode，当使用diff算法对vnode进行比较时，深度优先，同级比较的策略会对组件vnode的数据结构进行限制，如果此时template下有多个节点的话，那么vue就无法得知从哪一个节点开始进行深度递归。所以可将数据结构理解为一棵“树”，遍历的起始点就是一个唯一的“根”

### 8.谈谈你对MVC、MVP和MVVM的理解？

1. MVC模式：
    - 主要分为：Models: 数据层，负责数据的请求和存储以及处理；View: 展示层，负责View和动画效果的展示，以及用户的交互；Controller: 控制器层，负则连接View和Model，对view的交互事件有controller层处理传递给model，model处理再交互传递给Controller，Controller传递给view
    - 是早期前后端分离解耦概念的设计模式
    - 优点：
        - 增强代码复用性耦合性低
        - 解决代码臃肿
        - 易拓展
        - 可维护
    - 缺点
        - 简单的小型项目，使用MVC设计反而会降低开发效率层和层虽然相互分离，但是之间关联性太强，没有做到独立的重用。
        - 对于简单的界面，严格遵循MVC，使模型、视图与控制器分离，会增加结构的复杂性，并可能产生过多的更新操作，降低运行效率
        - 视图与控制器是相互分离，但却是联系紧密的部件，视图没有控制器的存在，其应用是很有限的，反之亦然，这样就妨碍了他们的独立重用。
        - 依据模型操作接口的不同，视图可能需要多次调用才能获得足够的显示数据。对未变化数据的不必要的频繁访问，也将损害操作性能。
2. MVP模式：
    - MVC的缺点在于并没有隔离View和Model层, MVP针对以上缺点做了优化, 它将业务逻辑和业务展示也做了一层隔离, 对应的就变成了MVCP
    - MVP从视图层中分离了行为（事件响应）和状态（属性，用于数据展示），它创建了一个视图的抽象，也就是presenter层，而视图就是P层的『渲染』结果。P层中包含所有的视图渲染需要的动态信息，包括视图的内容（text、color）、组件是否启用（enable），除此之外还会将一些方法暴露给视图用于某些事件的响应。
    - MVP相对于MVC, 它其实只做了一件事情, 即分割业务展示和业务逻辑. 展示和逻辑分开后, 只要我们能保证V在收到P的数据更新通知后能正常刷新页面, 那么整个业务就没有问题. 因为V收到的通知其实都是来自于P层的数据获取/更新操作, 所以我们只要保证P层的这些操作都是正常的就可以了. 即我们只用测试P层的逻辑, 不必关心V层的情况
3. MVVM模式：
    - MVVM隔层职责和MVP类似，VM对应P层
    - MVVM相对于MVC的改进是对VM/P和view做了双向的数据和命令绑定
MVP的view触发P的业务逻辑，然后P再回调改变View的显示的操作，使用MVVM的数据绑定来实现让逻辑更加清晰，代码也更少。这就是MVVM相对于MVP的改进之处

- 这三者都是框架模式，它们设计的目标都是为了解决Model和View的耦合问题。
- MVC模式出现比较早主要应用在后端，如Spring MVC、ASP.NET MVC等，在前端领域的早期也有应用，如Backbone.js。它的优点是分层清晰，缺点是数据流混乱，灵活性带来的维护问题。
- MVP模式是MVC模式的进化形式，Presenter作为中间层负责MV通信，解决了两者耦合问题，但P层过于臃肿会导致维护问题。
- MVVM模式在前端有广泛应用，它不仅解决MV耦合问题，还同时解决了维护两者映射关系的大量繁杂代码和DOM操作代码，在提高开发效率、可读性同时还保持了优越的性能表现。

### 9.谈谈你对vue组件之间通信的理解？
- vue组件之间的通信有如下几种方式：
    - props
    - eventBus事件总线
    - 自定义事件（$emit/$on）
    - vuex
    - $root/$parent/$children/$refs
    - provide/inject
    - $attrs/$listeners
- 定义props可以预设组件入参，规范参数类型及默认值。便于整理组件使用API，清晰调用方式
- eventBus事件总线是一个挂载在全局vue对象上的一个事件监听器，可以使组件跨层级通信。但还是建议遵循单一数据流原则来使用，否则会使原始数据发生不可控的变更，不利于追踪。
- 自定义事件，用于由子组件派发和监听一个特定事件，通知父组件进行逻辑处理，不会跨层级
- vuex是应用级数据状态管理，控制了数据变更方式，保证数据修改遵循单一数据流。
- $root/$parent/$children是vue组件对象内置属性，用于获取当前组件的根节点、父节点和子节点，编写业务层组件时不常使用，因为业务层组件因为架构的缘故，有更好的通信方式可选。但在编写通用组件库时，为了弱化侵入性，解耦等目的，使用得频率较高
- provide/inject数据依赖注入，可以实现跨层级数据传递，有较高的优先级，但是传递下来的数据在子节点中不能修改，业务组件使用频率极低
- $attrs/$listeners包含了父作用域中不作为 prop 被识别 (且获取) 的特性绑定 ( class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 ( class 和 style 除外)，并且可以通过 vbind="$attrs" 传入内部组件——在创建高级别的组件时非常有用。

### 10.你了解哪些vue性能优化方法？
- 合理使用v-if和v-show指令，避免预留过多真实dom元素
- 使用计算属性处理列表数据，减少多余循环次数
- 静态资源使用懒加载策略
- 路由组件按需引用
- 代码压缩、拆分
- ssr

------分割-------

- 路由懒加载
- keep-alive缓存页面
- 使用v-show复用DOM
- v-for避免同时使用v-if
- 长列表性能优化
    - 如果列表是纯粹的展示，不会有任何改变，就取消它的响应化，使用object.freeze进行数据冻结
    - 如果是大数据长列表，可采用虚拟滚动，只渲染少部分区域内容
- 组件销毁时，手动注销自定义的定时器等，避免内存泄漏
- 图片懒加载v-lazy
- 第三方组件库按需引入，减少包体积
- 无状态的组件标记为函数式组件
- 子组件分割
- 变量本地化
- ssr

### 11.vue3有哪些新特性吗？他们会带来什么影响？

1. 检测机制
    - vue2中基于Object.defineProperty的observer实现，vue3中则基于Proxy的observer实现
    - 对属性的添加、删除动作的监测
    - 对数组基于下表的修改，对于length修改的监测
    - 对Map、Set的支持
    - 默认为惰性监测
        - 在2.x版本中，响应式数据都会在启动的时候进行监测，如果数据量较大，会有严重的性能消耗
        - 在3.x版本中，只有应用初始可见部分所用到的数据会被监测到
    - 更精准的变动通知
        - 在2.x版本中，通过Vue.set强制添加一个新的属性，所有依赖这个对象的watch函数都被执行一次
        - 在3.x中，只有依赖这个具体属性的watch函数被通知
    - 更好的调试
        - 通过使用新增的renderTracked和renderTriggered钩子，可以精确的追踪到一个组件发生重新渲染的触发时机及原因
    - 暴露出observable的api来创建响应式对象，可以替代掉event bus,做一些跨组件的通信
2. 性能优化
    - 组件渲染
        - 在2.x版本中，父组件重新渲染时，其子组件也必须重新渲染(前提是修改的数据是子组件的props，才会触发子组件的重新渲染)
        - 在3.x版本中，可以单独重新渲染子组件或者父组件
    - 静态树提升
        - 在3.x版本中，编译器可以检测到静态组件，将其提升，降低渲染成本
    - 静态属性提升

### 12.vue如果想扩展某个现有的组件时应该怎么做？

- 使用mixins和extend
    - extend在编译的时候优先级会高于mixins，做的都是配置合并。但是mixins由于参数传入的类型不同，所以做的是更细致的配置合并。所以mixins在组件部分配置扩展时会更建议使用，反之推荐使用extend

### 13.watch和computed的区别以及怎么选用?

- computed计算属性
    - 是由data中的已知值，得到的一个新值
    - 新值只会根据已知值的变化而变化，其他不相关的数据的变化不会影响该新值
- watch属性监听
    - 监听data中数据的变化
    - 被监听的数据需要在data中定义
- 当一个属性受多个属性影响的时候就需要用到computed
- 当一条数据影响多条数据的时候就需要用watch

### 14.谈谈你对vue生命周期的理解？
- vue组件从创建到销毁的过程，就称之为生命周期
- 通常我们所说的生命周期分为8个：分别为在初始化实例时，默认调用的beforeCreate、created、beforeMount、mounted这四个钩子函数，还有当更新数据时，更新之前会触发beforeUpdate这个钩子函数，更新完成之后，会触发updated这个钩子函数；当vue的实例销毁时，会调用beforeDestroy和destroyed这两个钩子函数。除此之外还有不常用的activated、deactivated、errorCaptured这三个钩子函数
- 钩子函数的暴露，给了开发者能在精确的环节做相应的逻辑处理的机会

### 15.谈谈你对vuex使用及其理解？
- Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
- vuex在main.js中与vue根实例一同初始化，需要配置：state、mutations、actions、getters（可选）
    - state是vuex需要管理的元数据声明
    - mutations用于修改state中的数据（单项）
    - 如果需要同时修改多个state中的数据，则在actions中commit多个mutation进行修改
    - getters可以对state中的数据进行包装、计算后返回，可以理解为vuex的计算属性，非必须配置项

### 16.你知道nextTick的原理吗？
- JS是基于事件循环单线程执行的
    1. 所有同步任务都在主线程上执行，形成一个执行栈
    2. 主线程之外，会存在一个任务队列，只要异步任务有了结果，就在任务队列中放置一个事件
    3. 当执行栈中的所有同步任务执行完后，就会读取任务队列。那些对应的异步任务，会结束等待状态，进入执行栈
    4. 主线程不断重复第三步
- 这里主线程的执行过程就是一个tick，而所有的异步结果都是通过任务队列来调度。Event Loop 分为宏任务和微任务，无论是执行宏任务还是微任务，完成后都会进入到一下tick，并在两个tick之间进行UI渲染。
- 由于Vue DOM更新是异步执行的，即修改数据时，视图不会立即更新，而是会监听数据变化，并缓存在同一事件循环中，等同一数据循环中的所有数据变化完成之后，再统一进行视图更新。为了确保得到更新后的DOM，所以设置了 Vue.nextTick()方法。
[参考文档](https://segmentfault.com/a/1190000020499713)