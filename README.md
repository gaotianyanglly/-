## 进程和线程

- 进程：可以简单的理解为操作系统分配给程序运行所需的专属内存空间，每个程序至少都有一个进程，进程之间是相互独立的，即使互相通信也需要双方同意并且代价比较大
- 线程：进程内真正运行代码的单位就是线程
  - 前端接触较多的线程间通信
    - 同进程下的线程公用一个资源池或者说是公用一个内存空间，可以通过访问或者读写内存数据进行通信（最常见的）
    - 消息传递，前端来说就是浏览器会分很多进程，进程下也会分很多线程（浏览器主进程、网络进程、渲染进程等），渲染进程又会有一个渲染主线程负责解析 html、css 和运行 js，渲染主线程用来处理各种任务的方式就是队列
      - 微队列：优先级最高，常接触的将一个函数添加到微队列的方式就是 Promise
      - 交互队列：用于存放用户在页面操作后产生的事件任务，优先级低于微队列
      - 延时队列：⽤于存放计时器到达后的回调任务，优先级低于交互队列

## 浏览器任务有优先级吗？

- 任务本身没有优先级，在消息队列中先进先出，但消息队列有优先级，根据 W3C 的最新标准，
  - 每个任务都有一个任务类型，根据任务类型将不同类型的任务分配到不同的消息队列中，在后续的任务循环中，浏览器根据实际情况从不同的队列中取出任务并执行
  - 浏览器必须准备好一个微队列，微队列中的任务是优先于其它所有任务执行的

## http 三次握手四次挥手

- 三次握手：客户端向服务端发送报文，进入 SYN-SENT 状态 => 服务端收到报文将连接放入预备队列中，之后向客户端发送确认报文，并进入 SYN-RECEIVED 状态 => 客户端收到确认报文，向服务端发送确认报文 ESTABLISHED
  - 两次不安全，四次没必要
- 四次挥手：
  - 客户端数据发送成功后，向服务端发送终止请求的报文
  - 服务器收到终止报文后释放 TCP 连接，不再接收数据但是依然可以向客户端传输数据
  - 服务端数据发送完成后向客户端发送终止报文
  - 客户端收到终止报文后，向服务器发送确认终止报文，一段时间后没收到服务端的数据关闭连接，服务端收到终止报文后也关闭连接

## https 为什么更加安全

- 主要是因为 HTTPS 通过 SSL/TLS（加密保护协议）协议进行加密，以及提供了身份认证和完整性保护。公钥和私钥 公钥用来加密 私钥用来解密
  - 数据加密。HTTPS 使用加密算法对传输的数据进行加密，确保即使数据在传输过程中被截获，也无法被解密。
  - 身份认证。HTTPS 通过数字证书  来验证通信对方的身份，防止黑客伪造网站，确保用户连接到正确的网站。
  - 完整性保护。HTTPS 通过摘要算法来检查数据在传输过程中是否被篡改，如果数据被篡改，接收方能够识别出来。

## 输入 URL 后浏览器发生了什么

- 主要负责的进程为 主进程、渲染简称、网络进程
- 输入 URL 后首先会对 URL 进行解析（协议、域名、端口、路径、参数等），输入的如果不是 IP 还会进行 DNS 进行解析
  - DNS 解析优化：使用 CDN 缓存，加快 CDN 找到资源的速度（dns-prefetch）
- 建立 TCP 连接：TCP 三次握手四次挥手
- 拿到需要的基础资源后开始后续的解析
- 解析 html、css 和 js 等资源
- 解析这步又主要分为构建 dom 树和构建 cssom 树（生成 stylesheet，封装组件常用，日常开发不常用）
  - 尽量先加载 CSS 文件，防止页面先渲染没有样式的 html 之后闪屏再渲染样式，优化用户体验
  - CSS 从左向右解析，例如 a .active{...}，CSS 解析器会先遍历所有的 a 标签的祖先节点，因此效率会低一些
- 两个树都生成后开始合并渲染树，根据渲染树的描述确定浏览器每个像素的样式
- 布局和绘制 浏览器根据渲染树进行布局（Layout）和绘制（Paint）
  - 布局就是根据 DOM 树生成 Layout 树，Layout 树跟 DOM 树并不是一一对应的（比如 display：none，before 等，根据实际样式删除或者新增节点）
  - 布局之后进行分层
    - 会影响浏览器分层的 css 属性：position：fixed 或者 position：absolute
    - opacity 透明度会导致浏览器为该元素创建一个新的图层
    - transform 旋转缩放还有 translatez 可以让元素获得 3D 比那换，并导致浏览器单独为它创建一个图层
    - video 和 canvas 元素默认自己有自己的图层，并且它们会享受浏览器内置的 GPU 加速
    - will-change 属性会提示浏览器预先合成一个图层，以优化动画性能
  - 这里的绘制，就是为每一层生成如何绘制的渲染指令，并将指令全部推送给浏览器的渲染主线程
- 绘制结束后 浏览器会进行分块，即将每一层分为多个小块（方便后续的光栅化），分块工作是由渲染主线程外的多个线程同时进行的
- 分块结束后会进行光栅化，即将每个块转变成位图，并且有限处理靠近视口的块，此过程会⽤到 GPU 加速

## 什么是 reflow（回流）和 repaint（重绘）

## 什么是 GPU 加速

- 浏览器将一部分运算密集的任务交给显卡完成的性能优化策略，video 和 canvas 元素，transform、opcity、transition（css3 的动画）

## DOMContentLoad 和 onLoad

- DOMContentLoad 是 dom 树生成完之后
- dom 树生成完不代表页面加载完 还有其他静态资源要下载 这些静态资源下载完之后 执行 onLoad

## 如何避免重绘回流

- https://www.php.cn/faq/618114.html
- will-change
- https://blog.csdn.net/zz_jesse/article/details/132613789
- css 样式 告诉浏览器这个元素将会发生某种变化
- requestAnimationFrame
- getBoundingClientRect 常用来做懒加载 但会引起重绘回流 可以换为 intersectionObserver

## DocumentFragment

- 文档片段/文档碎片 原理就是直接将文档片段替换 DOM 树，不会引起回流和重绘
- 列表优化

## Mutationobserver

- 监控 dom 删改 常用于水印变化 计算首屏时间

## vue2 cumputed 和 watch

- 计算属性：多对一（缓存）dirty
- 监听属性：一对一
  - immediate:立即执行
  - deep：深度监听
  - handler：函数

## 双向绑定/响应式原理

- 在 beforeCreate 和 created 之间初始化 data
- Object.defineProperty 设置 getter、setter
- **ob**对象
- 数组：出于性能考虑 不能每项都设置 getter、setter 新增了一个对象集成 Array 的原型 改写了数组的原型方法
- dep 和 watcher
  - https://blog.csdn.net/weixin_69810763/article/details/135478552

## vue.observeable

- 2.6 新增的 api，将一个对象变为响应式数据，将一个数据共享给各个组件，是 vuex 或 bus 的轻量级的解决方案，缺陷是数据修改太过随意
- vue3 中被其它 api 替代，reactive

## dep、watcher 相互收集

- watcher 为什么收集 dep

## diff 算法

- 虚拟 DOM,对象
- 每个组件都有唯一 key
- Vue 的 diff 算法是一种用于比较虚拟 DOM 树之间差异的高效算法。它是 Vue 的核心特性之一，允许 Vue 以一种有效的方式更新 DOM 以反映数据的最新状态。
- Vue 的 diff 算法采用深度优先的递归方式比较两棵虚拟 DOM 树的差异。它会尽可能地复用老的虚拟节点，只会对差异发生变化的部分进行最小化的 DOM 更新。
- 数组对比：四个指针 新前新后 旧前旧后 就是描述 diff 的具体实现算法：对比新数列头的元素与旧数列头部元素，如果相同新头放在旧数列的头部，新尾放在旧数列的尾部，旧数列则遍历对比新数列中有没有相同项，如果没有就直接删除
- class 不同时 不管 只认 key

## 箭头函数为什么不能作为构造函数

- 箭头函数没有自己的上下文，没有原型
- Promise：https://www.cnblogs.com/lvdabao/p/es6-promise-1.html
  - then 链式
  - 需求：请求超时 怎么用 Promise 实现 race 方法
  - allSettled 方法 https://blog.csdn.net/iloveyu123/article/details/116588214
- async/await https://blog.csdn.net/weixin_45811256/article/details/123638582
  - generator + Promise
  - await + 函数 不等待定时器
  - async 函数执行完返回什么

## 性能优化

- 按钮：防抖
- 懒加载
- vue 的路由懒加载 component:()=>import('url') 这样写只有当访问对应路由时，加载组件的方法才会执行，优化了性能
- 节流 https://blog.csdn.net/m0_64346035/article/details/124293989

## 什么是策略模式，应用场景？

- https://www.51cto.com/article/688731.html
- 比如表单验证

## vue-admin

- 请求封装：axios 响应拦截器和请求拦截器
- 怎么实现取消 axios 的重复请求

## cookie

- cookie 与其他本地缓存的区别

## SEO

- nuxtjs vue 服务器渲染的整合方案
  - vue 普遍开发的是 SPA（单页面应用），打包后只有一个 index.html 作为入口，SPA 的好处就是由于是在一个页面上加载内容，用户在切换页面时不会全部重绘页面，只会部分重绘，因为用户体验更好，但是这些优点是牺牲了首屏加载速度和 SEO 优化的来的，而一般页面想要优化首屏加载速度最常用的方案就是 SSR 即服务器加载（将首屏页面在服务器加载完成后发给客户端，客户端直接加载服务器返回的页面），nuxtjs 就是给 vue 提供的采用类似思路解决 SPA 问题的一种整合后的解决方案
- tdk

## 自定义 SEO 优化

- tdk
- 语义化标签
- h1 不嵌入其它元素
- site-map
- 微数据结构化
  - https://www.cnblogs.com/zdz8207/p/seo-google-rich-snippets.html

## axios.js fetch

## axios 如何做到 nodejs、浏览器通用

- node：http 模块封装 net
- 浏览器：xhr
- fetch

## 大文件的下载和暂停

- 文件是二进制对象 blob
- 继承对象前端可以切片 实现文件上传的断点续传
- buffer
- http：range 部分请求 可用于下载文件断点续传
  - https://blog.csdn.net/m0_62617719/article/details/128324865

## 如何判断一个文件的类型经过非法修改

- 在 JavaScript 中，可以通过读取文件的二进制数据并利用 FileReader 对象的 readAsArrayBuffer 方法来读取文件头部信息，然后根据文件类型的魔数（magic number）进行判断。
  - "D0CF11E0A1B11AE1": "application/wps-office.ppt",
  - "89504E470D0A1A0A": "image/png",
  - "6674797071742020": "video/quicktime",
  - "504B030414000600": "application/wps-office.xlsx",
  - "504B03040A000000": "application/wps-office.docx",
  - "3C68746D6C3E0A0A": "text/html",
  - "3C21646F63747970": "text/html",
  - "526172211A0700": "application/vnd.rar",
  - "7F454C46": "application/x-sharedlib",
  - "504B0304": "application/zip",
  - "464C5601": "video/x-flv",
  - "3C737667": "image/svg+xml",
  - "25504446": "application/pdf",
  - "1A45DFA3": "video/webm",
  - "FFD8": "image/jpeg", // (.jpg, .jpeg, .jfif, .pjpeg, .pjp)
  - "4D5A": "application/x-ms-dos-executable",

## docker

## 技术选型考虑哪些

- 考虑跨平台
- 人气如何 生态
- 扩展性
- 维护活跃度

## 内部组件库如何维护

- menorepo 把所有项目全部放到一个大项目里 全部可以互相引用
- 通过 pnpm 安装子项目中的各个组件然后测试打包发布

## 设计模式

- BV1KJ4m1V7zC
- 工厂模式
  - 比如各种提示框
- 单例模式
  - 比如全局缓存（oa 办公系统里用的最多的就是保存 user 字段，将所有的用户信息放到这个字段中）
- 适配器模式
  - 比如父组件传入了时间，要在子组件内格式化并且回显，一般在 computed 中转换 就是适配器
  - 或者购物车等，从服务器或者列表中用户选择商品后，会有单价、数量、优惠券等信息，最终回显数量、总价、总配送费等
- 外观模式。
  - 提供一个统一的接口隐藏内部逻辑，方便外部调用，如实现兼容多种浏览器的添加事件方法。
- 迭代器模式。
  - 提供一种方法顺序访问聚合对象中的元素，而不暴露其内部表示，如 js 的 Array.prototype.forEach。

## 前端优化

- webpack 可视化工具（webpack-bundle-analyzer）看各个模块的体积
  - https://b23.tv/GjDa3J6
- lodash -> lodash-es
  - https://blog.csdn.net/qq_39335404/article/details/129951010
- 移动端预加载
- 表单细节优化
  - https://b23.tv/DxhW7UI

## 涉及到图形的库 包体积过大 如何解决

- external cdn 引入 不打包到主项目内 通过外链引入
- BV1BD421W7oS 打包体积的分析和优化

## vite 和 webpage

- vite 是基于 ESModules 实现的，更加轻量，开发环境下打包速度更快（简单来说访问什么模块才加载什么模块），webpage 是全量打包（加载完所有模块后才完成打包），因此比较慢

## map 和对象的区别

-

## 二次封装 localStorage 考虑哪些问题

- 存读 JSON
- 存取 map

## 怎么判断对象有环引用

- 深度遍历 如果是对象就放到数组里 然后查重

## nextTick 原理

- 内部实现是用的 promise 实现，如果没有用 MutationObserver，在没有用的是 settimeout 参考事件循环几种任务队列

## 父子组件生命周期执行顺序

- 加载渲染阶段：在这个阶段，父组件的生命周期钩子会先于子组件执行。具体顺序为：父组件的 beforeCreate、created、beforeMount，然后是子组件的 beforeCreate、created、beforeMount，最后是子组件的 mounted。这意味着，在子组件挂载到父组件之前，父组件已经完成了所有的生命周期钩子的执行。
- 更新阶段：当父子组件有数据传递时，会执行到更新阶段的顺序。在这个阶段，父组件和子组件的生命周期钩子会按照它们在代码中的顺序执行。具体顺序为：父组件的 beforeUpdate，子组件的 beforeUpdate，子组件的 updated，最后是父组件的 updated。
- 销毁阶段：在销毁阶段，父组件的生命周期钩子会先于子组件执行。具体顺序为：父组件的 beforeDestroy，子组件的 beforeDestroy，子组件的 destroyed，最后是父组件的 destroyed。
- 需要注意的是，如果子组件是异步组件，那么执行顺序会发生改变，会先执行完父组件的生命周期然后再执行子组件的生命周期。这是因为 Vue 的生命周期钩子是按照组件的嵌套层级顺序依次执行的，确保了在子组件挂载到父组件之前，父组件已经完成了所有的生命周期钩子的执行。
- 父组件 mouted 中是否能获取到子组件中的变量？

## Promise 链式调用

- 为什么可以实现链式调用？每次返回值都是 promise

## CDN 如何配置

## 老项目精度丢失

- webpack 抽象语法树统一替换精度缺失的公式

## 虚拟列表？和缺点

- 需要频繁的计算 dom 消耗性能较大 可能导致白屏
- 如何解决：预加载

## git 的 rebase 和 merge

- 都是合并，merge 会保存所有提交记录
- rebase 不会，会使提交记录看起来更简洁
- 具体用哪个视项目情况而定

## git 的 reset 和 reverse

- reset + commitId soft 模式和 hard
- reverse 生成新提交保存前面的提交

## git stash

- 可以理解为先将本地代码存入栈中，使用最多的场景是本地有 bug 未修完，需要拉去新代码
- BV1nv4y1R78d

## vue 的自定义指令

- directives
  - 常用于组件/按钮等的统一的权限管理
  - https://blog.csdn.net/XH_jing/article/details/118547130

## vuex 修改数据的步骤

- BV1YM411w7Zc p48
- Viwe dispatch 到 Action，commit 到 Mutations， Mutate 到 State，最终 Rander 到 Viwe
- 最简单或者说最常用的修改 vuex 状态的方式是直接在组件中 commit 一次更改到 Mutations
- Action 中常封装一些页面的公共逻辑
- 本质上是通过再生成一个响应式数据来实现的（vue2：`new Vue()`/vue3：`reactive()`），因此命名不能重复，2.6 版本以上有一个更轻量级的方案：vue.observeable

## vuex 的五个属性/方法

- https://blog.csdn.net/qq_44755705/article/details/105291699
- module BV1YM411w7Zc p52

## router 的几种模式

- hash：兼容性最好 但是不利于 SEO 优化 不美观
- history 会向服务器请求资源
- abstrat 上述两个方法都不生效时使用 不常用
- BV1YM411w7Zc p46

## router 的钩子函数

- BV1YM411w7Zc p45

## elementui 页面只有一个 input 框时有什么问题

- 回车会刷新页面，导致这个 bug 的根源原因是包了 form 标签，浏览器默认按回车时触发 submit 事件，由于没有配置 submit 的 url，就会刷新页面，解决方法就是在原生 submit 事件中阻止一下冒泡，类似场景中，不用 element 也可以利用浏览器这一特性实现回车事件的快速改写（表单外套 form 标签，然后监听或者重写 submit 事件），原生环境下开发表单都推荐包一层 form 标签

## vue 组件间传值

- 父子组件：props 和$emit
- 直接用 ref/refs 取组件内的变量
- eventBus 又被称为事件总线
  - 使用 eventBus 有没有遇到过什么问题？出现该 bug 的原理是什么？如何解决？由于 eventBus 是独立于业务组件外的 vue 对象，当前组件销毁并不会触发 eventBus 自动销毁，所以如果未经过正确注销，多次访问调用 eventBus 的组件会导致发布的事件次数增多从而导致监听多次触发（根据访问次数累加），解决方法就是在调用组件注销时手动来销毁 eventBus（调用 bus.$emit 的组件 beforeDistroy 时调用 bus 的 off 方法）
  - 还有个比较常见的问题是使用 eventBus 时要注意监听和发布（bus.$on和bus.$emit）方法时应在哪个钩子函数中，例如我在父组件中 created 里发布方法，子组件中是无法监听到第一次发布的方法的
    - 详情原理参考 [父子组件生命周期执行顺序](#父子组件生命周期执行顺序)
    - https://www.php.cn/faq/382432.html
    - https://www.jb51.net/article/264928.htm

## vue3 相比 vue2 新增的功能或者不同点

- 响应式 defineProps 和 proxy
- diff 算法更新 静态节点提升
- 组合式 API（Composition）

  -

  ```js
  <script setup></script>
  ```

  内即可使用组合式 API，内部的 import 引用自动成位当前组件的子组件，相当于

  ```js
  <script>setup(){}</script>
  ```

  - 使用函数而不是声明选项的方式书写 vue 组件
  - 好处：代码可读性、可维护性更好：选项式(Option)API 无论逻辑代码，位置很可能会分散在多个位置，组合式 API 的代码组织更加自由，有利于维护，相比选项式 API，代码结构更接近于原生 js
  - `setup(){}`中`this`是`undefined`，vue3 中已经弱化`this`了
  - `setup()`的运行时机：beforeCreate 之前（什么时候初始化 data？created 之前，beforeCreate 之后，因此 data 中可以取到 setup 中的数据，反之则不能）
  - setup 的返回值也可以是个 render 函数

- 创建实例：`const app = createApp(App)`
- 组件根标签可以有多个
- reactive() 声明一个响应式数据，参数是对象
  - 需要注意，vue3 中很多函数都需要在用之前从 vue 中引入(如 ref、computed、reactive 等等)
  - 一维数据、多维数据都可以
  - 如果只希望只有第一层数据是响应式的，vue3 还提供了 shallowReactive 的 API
- ref() 声明一个基础数据类型的响应式
  - 不需要额外引入
  - 原理是 ref 会自动生成一个空对象，并将参数赋值给该对象的 value 字段，所以用 ref 声明响应式数据后操作的是.value，这样做的原因是原生 js 无法监听基础数据类型的 getter、setter，因此可以利用创建一个空对象然后监听他的 value 字段从而实现响应式
- readonlay() 只读字段不允许更改，因此不需要做响应式处理
  - 只有根属性只读：shallowReadonly，第二层数据依然是非响应式，但可以更改
- computed() 与 vue2 使用方式类似，语法发生变化

  - 有缓存，大致原理是有一个 dirty（脏数据名称的由来）字段用于标识 dep 中该字段是否发生了变化，computed 计算属性时如果该字段为 true，则使用最新数据，计算完成后将 dirty 字段改为 false，否则则使用缓存数据进行计算，这样的好处是当同一个 computed 字段在页面内多次被调用时，只会进行一次计算，也就只会触发一次所需响应式数据的 getter 方法，优化性能
  - ````js
            <script setup>
                const content = ref('测试文本')
                const textLng = computed(() => {
                    conso.log('computed')
                    return content.value.length
                })
                <template>
                    {{textLng}}
                    {{textLng}}
                    {{textLng}}
                </template>
            </script>
        ```
    调用三次但只有第一次时数据发生了变化会进行计算，否则使用缓存数据，因此只会打印一次（机制原理与vue2相同）
    ````
  - 可写计算属性

  ```js
  <script>
  import { ref, computed } from 'vue'

  const firstName = ref('John')
  const lastName = ref('Doe')

  const fullName = computed({
      // getter
      get() {
          return firstName.value + ' ' + lastName.value
      },
      // setter
      set(newValue) {
          // 注意：我们这里使用的是解构赋值语法
          [firstName.value, lastName.value] = newValue.split(' ')
      }
  })
  </script>
  ```

  - 要注意，使用可写计算属性时，不要改变其他状态、在 getter 中做异步请求或者更改 DOM！计算属性本身也尽量不要直接修改，仅用来读取计算属性

- watch() 监听器
  - 监听 ref 值：
  ```js
  const content = ref("测试文本");
  watch(content, (newVal, oldVal) => {
    console.log(newVal);
  });
  ```
  - 監聽 reactive 中的某個字段时，需要以函数的形式 return：
  ```js
  const myData = shallowReactive({
    name: "张三",
    age: 18,
    friends: ["李雷"],
  });
  watch(
    () => myData.age,
    (newVal, oldVal) => {
      console.log(newVal);
    }
  );
  ```
- watchEffect()
  - https://cn.vuejs.org/api/reactivity-core.html#watcheffect
  - 简单使用：
  ```js
  watchEffect(() => {
    console.log("watchEffect:" + content.value);
    console.log("watchEffect:" + myData.age);
  });
  ```
  函数中用到了哪些响应式数据就会监听哪些响应式数据，并且立即执行一次（immediate）
- watch()和 watchEffect()配置中都有 flush 字段用来指明是否访问更新后的 DOM（`post`异步监听器和`sync`同步监听器）

  ```js
  watch(source, callback, {
    flush: "post",
  });

  watchEffect(callback, {
    flush: "post",
  });
  watch(source, callback, {
    flush: "sync",
  });

  watchEffect(callback, {
    flush: "sync",
  });
  ```

  - vue3 还提供了 watchPostEffect 和 watchSyncEffect 方法，与上述两种 watchEffect 相同
  - 同步侦听器不会进行批处理，每当检测到响应式数据发生变化时就会触发。可以使用它来监视简单的布尔值，但应避免在可能多次同步修改的数据源 (如数组) 上使用。
  - watchEffect 的返回值为该侦听器的停止方法，大部分情况下在`setup()`或者`<script setup>`中以同步的形式创建的侦听器是不需要特别关心何时停止的，因为这样创建的侦听器会绑定在宿主组件上，组件卸载时就会自动停止，但当在异步回调中创建一个侦听器时，它不会绑定在当前组件上，必须手动停止以防内存泄漏

  ```js
  <script setup>
    import {watchEffect} from 'vue' // 它会自动停止 watchEffect(() => {}) //
    ...这个则不会！ setTimeout(() => {watchEffect(() => {})}, 100)
  </script>;

  // 手动停止
  const unwatch = watchEffect(() => {});

  // ...当该侦听器不再需要时
  unwatch();
  ```

  尽可能同步创建侦听器，如果是处理异步请求的数据可以利用 if

  ```js
  // 需要异步请求得到的数据
  const data = ref(null);

  watchEffect(() => {
    if (data.value) {
      // 数据加载后执行某些操作...
    }
  });
  ```

## Pinia

- 轻量级的状态管理库

## CSS 样式权重排序

- 内联样式（行内样式）> ID 选择器 > 类选择器 > 标签选择器
- 权重相同时后面的规则会被应用
- 强行改变权重的方式：!important 尽量少用

## http、tcp、ucd 的异同

- https://www.cnblogs.com/gaopeng527/p/5255827.html
- http：基于 tcp 的用于万维网浏览的数据传输协议
- tcp：服务器间建立连接需要的协议
- ucd：“面向非连接”就是在正式通信前不必与对方先建立连接，不管对方状态就直接发送。比如，我们经常使用“ping”命令来测试两台主机之间 TCP/IP 通信是否正常

## http1.x 和 http2.x 的区别

- https://www.cnblogs.com/wzj4858/p/11411457.html

想要以 key value 的形式存储数组有哪些方法
map 的方法（set 等）
如何快速更改 echarts 主题

## vite 中的 import.meta.glob 方法

- vue3 中使用 require.context 会报错, 可以改用 import.meta.glob 方法
- 注意 glob 方法接收的参数必须为不含有任何变量的字符串

## vue3 中的 setup 函数是个很特殊的函数，在其中使用 await 时需要特别注意

- 如果 setup 中使用了 await 后页面无法正常渲染并且控制台中出现如下报错：
  ```
  Component <Anonymous>: setup function returned a promise, but no <Suspense> boundary was found in the parent component tree. A component with async setup() must be nested in a <Suspense> in order to be rendered.
  at <Index onVnodeUnmounted=fn<onVnodeUnmounted> ref=Ref< undefined > >
  at <RouterView>
  at <App>
  ```
  就说明需要在用 await 的组件外包裹`<Suspense></Suspense>`标签

## 高德地图 webapi

- 获得地图中点相对于地图容器的位置坐标 ：`map.lngLatToContainer(new AMap.LngLat(lng, lat))`

## vite 中的 import.meta 方法（类似于 webpack 中的 require）

- `import.meta` 是 JavaScript 中的一个特殊对象，用于获取关于模块自身的元信息。它包含了一些有用的属性，可以在模块内部使用。

  目前主要有两个属性：

  1. `import.meta.url`：返回当前模块文件的 URL 地址。
  2. `import.meta.env`：返回一个对象，包含了当前环境的一些信息，比如 `MODE`、`BASE_URL` 等。

  在 Vite 中，`import.meta` 特别常用，因为 Vite 的热更新和模块热替换（HMR）功能会利用这个特性。例如，`import.meta.url` 可以用于动态地构建资源路径，而 `import.meta.env` 则可以用于在不同环境下提供不同的配置。

  以下是一个示例：

  ```javascript
  console.log(import.meta.url); // 返回当前模块文件的 URL 地址
  console.log(import.meta.env.MODE); // 返回当前环境的模式（development、production 等）
  console.log(import.meta.env.BASE_URL); // 返回当前环境的基础 URL
  ```

- `import.meta.glob` 是 Vite 中的一个特殊语法，用于匹配指定目录下的文件，并返回一个对象，该对象的键是匹配到的文件路径，值是一个函数，调用这个函数可以动态导入对应的模块。
  这个语法的基本用法是：

  ```javascript
  const modules = import.meta.glob("./path/to/files/*.js");
  ```

  上面的代码将会匹配 ./path/to/files/ 目录下所有以 .js 结尾的文件，并返回一个对象 modules，其中包含了每个匹配到的文件路径及其对应的动态导入函数。

## markdown 文档导出方案

- 利用 typora 导出规范的文档

## css3 新特性

- Flexbox 布局
- Grid 布局 `display: grid`
- 多列布局
  ```javascript
  .column {
    column-count: 3; /* 三列 */
    column-gap: 20px; /* 列间距 */
  }
  ```
- CSS 变量

  ```css
  :root {
    --main-color: #3498db;
  }

  .element {
    color: var(--main-color);
  }
  ```

- 变换 (Transform)
- 动画 (Animations)
- 过渡 (Transitions)
- 伪类和伪元素
- 媒体查询 (Media Queries)
  ```css
  @media (max-width: 600px) {
    .container {
      flex-direction: column; /_ 在小屏幕上纵向排列 _/
    }
  }
  ```
- 文字效果

  ```css
  .element {
    text-shadow: 2px 2px 5px grey; /* 文字阴影 */
    font-family: "Open Sans", sans-serif; /* 自定义字体 */
  }
  ```
