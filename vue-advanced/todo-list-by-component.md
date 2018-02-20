# 组件化 Todo List 编写笔记

## 前言

徐老师在过年期间开的这门 Vue.js 的进阶课，是以组件化方式开发的 Todo List 为主线。自己虽然也能编码实现，但如果不做笔记，只是写代码，学习的效果还不够好。只有把自己的实现思路记录下来，遇到的问题和解决方法也记录下来，用文字把这个过程梳理清楚，才能对整个项目有更加清晰、准确的认识。

PS：文章内容是按照组件进行分隔的，但并不是写完一个组件的所有功能才写另外一个组件，所以在阅读时可能需要在文章的不同位置之间来回跳。

PPS：自己在做这个组件化的 Todo List 的时候，并没有完全按照老师的示例来做，所以有些地方会不一样。

## TodoItem 组件

### 显示待办事项清单（用本地数据）

先写一个最简单的组件，就是用 `v-for` 指令显示待办事项清单。数据也是用的本地的数据，这样在这一步能够把更多的精力放在学习组件的编写上。

首先，当然是在 `components` 目录下新建 `TodoItem.vue` 文件，用来显示待办事项清单，代码如下：

```html
<template>
  <ul>
    <li
      v-for="task in tasks"
      :key="task.id">
        {{ task.title }}
      </li>
  </ul>
</template>

<script>
export default {
  name: 'TodoItem',
  props: {
    tasks: Array
  }
}
</script>
```

在 `script` 中，`name` 选项定义了组件的名称 `TodoItem`，`props` 选项则定义了组件所接收数据的名称 `tasks` 和类型：数组（Array）。

在 `template` 中，则在根元素 `ul` 内，通过 `li` 元素显示待办事项的名称 `task.title`。`v-for` 语句大家都很熟悉，就不用讲了。加了另一条语句 `:key="task.id"`，是因为 Vue 建议在用 `v-for` 遍历时，为所遍历的每一项提供一个唯一的 `key` 属性（参考：[key](https://cn.vuejs.org/v2/guide/list.html#key)）。这一项不加也完全没关系，只不过 `vue-cli` 附带的 ESLint 会有错误提示，所以我这里就加上了。

另外这里还有个小知识点，Vue 规定组件的 `template` 中只能有一个根元素，也就是说下面这种写法是会报错的。个人猜测，之所以会有这种规定，也是为了最终渲染出来的 HTML 结构能更加清晰。仔细想想，这个理念也和组件化是相通的，不是嘛？

```html
<!-- 错误写法 -->
<template>
  <div></div>
  <div></div>
</template>
```

这个组件最基本的内容已经写好了，接下来就在 `App.vue` 中引入它。

```html
<script>
import TodoItem from "./components/TodoItem.vue";

export default {
  name: "app",
  components: {
    TodoItem
  }
};
</script>
```

引入组件之后，当然还要为它提供数据，这样组件才有内容可以显示。这里也有个知识点，组件中的**数据对象** `data` 必须是函数，因为这样能够保证组件实例[不会修改同一个数据对象](https://cn.vuejs.org/v2/api/#data)。刚开始写组件的同学可能容易忽略这个知识点，多写几次就记住了。

```javascript
export default {
  name: "app",
  components: {
    TodoItem
  },
  data() {
    return {
      tasks: [
        {
          id: "6b9a86f6-1d1a-558a-83df-f98d84cd87bd",
          title: "JS",
          content: "Learn JavaScript",
          completed: true,
          createdAt: "2017-08-02"
        },
        {
          id: "1211bb33-a249-5782-bd97-0d5652438476",
          title: "Vue",
          content: "Learn Vue.js and master it!",
          completed: false,
          createdAt: "2018-01-02"
        }
      ]
    };
  }
};
```

为组件准备好数据之后，就可以开始用它了。组件的基本用法也很简单，按照它的要求提供数据，然后组件就会按照自己设定的样式把数据显示出来。

```html
<template>
  <div id="app">
    <TodoItem
      :tasks="tasks" />
  </div>
</template>
```

上面的代码中，调用了 `TodoItem` 这个组件，并且将根组件中的数据属性 `tasks` 绑定到 `TodoItem` 这个组件的 `props` 选项上。在 `:tasks="tasks"` 这句代码中，等号前的 `tasks` 是子组件 `TodoItem` 中定义的名称，可以近似地理解为“形参”；等号后面的 `tasks` 则是父组件中的数据属性，可以近似地理解为“实参”。所以这种用法也可以写成 `:形参="实参"`，希望这种写法能够帮大家更容易地理解组件传入数据的语法。而父组件的数据属性和子组件的 `props` 选项都用 `tasks` 这个名称，是为了保持代码上的一致性，大家刚接触组件的时候可能觉得分不清谁是谁，但是代码写多了之后就能体会到这种写法的好处了，根组件只负责提供数据，子组件只负责使用数据，保持一致的命名，阅读和修改代码的时候就能很容易看出来互相之间的关系。

保存代码，然后在终端中执行 `npm run serve`，构建工具就会自动编译，然后在浏览器中打开页面，如果能够看到类似下图中的效果，就说明已经写好了一个最简单的组件了，接下来就要丰富这个 Todo List 的各项功能了。

![List - Component](http://owve9bvtw.bkt.clouddn.com/FhVBhKDMkceIrw-h6RlnIWCHCD-8)

### 样式改进

要使用 Bootstrap 的样式，首先需要把它的 CSS 文件引入进来，编辑 `public` 目录下的 `index.html` 文件，在 `head` 中加入下面的 CSS。后面需要引入 CSS 或者 JS 的时候，都可以在这里引入。当然了，也可以通过 `npm install xxx` 指令以后端库的形式引入，不过这样只能引入 JS，没法引入 CSS。

```html
<!DOCTYPE html>
<html>
  <head>
    <link href="https://cdn.bootcss.com/bootstrap/4.0.0/css/bootstrap.min.css" rel="stylesheet">
  </head>
</html>
```

接下来就是搭框架，先修改 `App.vue`，确定整体框架：

```html
<template>
  <div id="app" class="container">
    <div class="col-md-8 offset-md-2 mt-5">
      <TodoItem
        :tasks="tasks" />
    </div>
  </div>
</template>
```

在根 `div` 中加上 `class="container"`，这样子元素就可以应用 `col-md-8` 这样的网格样式了。然后在子元素中加上 `class="col-md-8 offset-md-2 mt-5"`，`col-md-8` 表示待办事项占12列宽度的网格中的8列，`offset-md-2` 表示往右偏移2列之后显示待办事项，这样就能够居中显示了。`mt-5` 则表示待办事项距离上方有一定空白，“留白”了才好看。

![Grid Setting](http://owve9bvtw.bkt.clouddn.com/FvG2GiNt2LGKup_mXJvHKlxYGkfE)

每个待办事项要显示标题、内容、日期，可以用 Bootstrap 的 [Custom Content 列表](https://getbootstrap.com/docs/4.0/components/list-group/#custom-content)。

![List Group with Custom content](http://owve9bvtw.bkt.clouddn.com/Fjz7jj9Njhxw3G-jwfnZLNotcUJU)

观察上图对应的代码可以知道，`a` 标签内的 `h5` 标签可用于显示待办事项的标题，相邻的 `small` 标签可用于显示时间，`a` 标签内最后的 `small` 标签则可用显示于事项的具体内容，因此 `TodoItem.vue` 组件中可以改成如下内容。

```html
<template>
  <div class="list-group">
    <a
      href="#"
      class="list-group-item list-group-item-action flex-column align-items-start"
      v-for="task in tasks"
      :key="task.id">
      <div class="d-flex w-100 justify-content-between">
        <h5 class="mb-1">{{ task.title }}</h5>
        <small>{{ task.createdAt }}</small>
      </div>
      <small>{{ task.content }}</small>
    </a>
  </div>
</template>
```

在浏览器中看看页面效果，怎么样，还不错吧？

![TodoItem with Style](http://owve9bvtw.bkt.clouddn.com/FoJ36ODt2hkLuk1lal-fhH3qzzhw)

### 待办事项清单的数据从远程获取

在实际业务中，数据都是放在服务器上，前端页面加载完成之后，再向服务器请求数据。这样实现了“前后端分离”，让前端页面只关注界面部分，数据由后端负责提供，将前后端解耦，降低了功能之间的依赖性。

要向服务器请求数据，就要用 axios 这个库，和前面引入 Bootstrap 的 CSS 一样，编辑 `public` 目录下的 `index.html` 文件，将 axios 这个库的链接加进来。

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="https://cdn.bootcss.com/axios/0.17.1/axios.min.js"></script>
  </head>
</html>
```

然后再编辑根组件 `App.vue`，将数据属性 `tasks` 的初始值设置为空数组，在 Vue 实例的 `created` 这个生命周期钩子中获取数据。数据方面参考[徐老师的建议](http://xugaoyang.com/post/5a6c1f298957a723cf8845e2) ，放在 myjson 上。

```javascript
const tasksUrl = "https://api.myjson.com/bins/xxxxx";

export default {
  name: "app",
  components: {
    TodoItem
  },
  data() {
    return {
      tasks: []
    };
  },
  methods: {
    fetchData(jsonUrl, obj) {
      axios
        .get(jsonUrl)
        .then(response => response.data)
        .then(data => {
          data.forEach(ele => {
            obj.push(ele);
          });
        })
        .catch(console.log());
    },
  },
  created() {
    this.fetchData(tasksUrl, this.tasks);
  }
};
```

从上面的代码可以看到，数据属性的值保存在 `tasksUrl` 这个 URL 中，通过 axios 获取数据。老师在前面的课程中也讲过，在 Vue 中更新数组，需要用特定的[变异方法](https://cn.vuejs.org/v2/guide/list.html#%E5%8F%98%E5%BC%82%E6%96%B9%E6%B3%95)，才能触发视图的更新，也就是上面代码中的 `obj.push(ele);`。

另外，将更新数据的代码抽离成一个单独的函数 `fetchData`，这样能够提高代码的可读性。否则如果 `created` 这个钩子中需要执行五六个操作的时候，把具体的代码全放到这里面，那代码就乱得没法看了。

### 用 `v-cloak` 指令，使组件只在数据加载完成之后才显示

为了优化用户体验，可以用 `v-cloak` 指令，实现[组件在数据加载完成之后才显示](https://cn.vuejs.org/v2/api/#v-cloak)的功能。

具体的测试结果，可以看[这个视频](http://7xq4gx.com1.z0.glb.clouddn.com/v-cloak_fast-3g.mp4)。在这个视频中，通过 Chrome 开发者工具将网速限制为 `Fast 3G` 模式，以便更清楚地展示这个过程。然后点击刷新按钮加载页面，能够看到页面在成功获取到服务器上的数据之后，才会渲染组件内容并显示出来，在这之前页面则一直是空白状态。

## TodoMenu 组件

### 显示菜单按钮（用本地数据）

前面知道怎么用组件显示待办事项清单了，那么显示一个菜单列表也就很容易了，照葫芦画瓢就行。

首先需要在根组件 `App.vue` 中准备数据 `menus`。

```javascript
export default {
  name: "app",
  components: {
    TodoItem,
    TodoMenu
  },
  data() {
    return {
      tasks: [],
      menus: [
        { tag: "all", text: "全部" },
        { tag: "doing", text: "未完成" },
        { tag: "done", text: "已完成" }
      ]
    };
  }
}
```

然后就是选择按钮的样式，徐老师之前用的是 [Button Group](https://v4.bootcss.com/docs/4.0/components/button-group/) 这个按钮组，但是自己在用了之后发现按钮的样式不够好看，于是选用了独立且轮廓清晰的 [Outline buttons](https://v4.bootcss.com/docs/4.0/components/buttons/#outline-buttons)，组件的代码像下面这样写：

```html
<template>
  <div>
    <button
      type="button"
      class="btn btn-outline-secondary"
      v-for="menu in menus"
      :key="menu.id">
      {{ menu.text }}
    </button>
  </div>
</template>

<script>
export default {
  name: 'TodoMenu',
  props: {
    menus: {
      type: Array,
      required: true
    }
  }
}
</script>
```

与之前编写 TodoItem 组件时相比，代码上主要的区别在于 `props` 的定义更加详细了，理由参见 Vue.js 官方文档中的风格指南：[Prop 定义](https://cn.vuejs.org/v2/style-guide/#Prop-%E5%AE%9A%E4%B9%89-%E5%BF%85%E8%A6%81)。

做出来的页面效果如下所示：

![TodoMenu](http://owve9bvtw.bkt.clouddn.com/Fnbbeya7caU_rkL1WXXQS6KKF6mg)

### 样式改进

基本的功能做出来了，下面就来改进一下 TodoMenu 组件的样式，让它更好看。

首先是要给按钮之间加上间距，也是前面提到过的“留白”，就跟设计 PPT 一样，把页面塞得满满的其实很难看。查看 Bootstrap 的文档 [Margin and padding](https://getbootstrap.com/docs/4.0/layout/utilities-for-layout/#margin-and-padding)，知道了可以用 `mr-x` 这样的类来设置右边距，测试了几个值之后，最终确定为 `mr-2`。

然后还要给上面的一排按钮和下面的待办事项清单之间也加上间距，这里就用 `mb-3` 设置按钮的下边距，之前在 TodoItem 组件中设置的 `mt-5` 则删掉。

```html
<template>
  <div>
    <button
      type="button"
      class="btn btn-outline-primary mr-2 mb-3"
      v-for="menu in menus"
      :key="menu.id">
      {{ menu.text }}
    </button>
  </div>
</template>
```

现在的页面效果就是这个样子的了：

![TodoMenu Beautified](http://owve9bvtw.bkt.clouddn.com/FgXrOIzUoZb73DGPhVmyzP2hExTe)

### 网页加载完成后突出显示第一个按钮

查看 Bootstrap 的文档可以知道，给按钮添加一个 `active` 类，按钮就会处于被点击的状态。这样一来，只需要修改 `menus` 的数据结构，给每个对象添加一个名为 `active` 的布尔型变量，然后给 TodoMenu 组件动态绑定 `active` 类，就能实现页面加载完成后突出显示第一个按钮的功能了。

```javascript
// App.vue
menus: [
  { tag: "all", text: "全部", active: true },
  { tag: "doing", text: "未完成", active: false },
  { tag: "done", text: "已完成", active: false }
]
```

```html
<!-- TodoMenu.vue 只列出了新增的内容 -->
<template>
  <div>
    <button
      :class="{active: menu.active}">
    </button>
  </div>
</template>
```

![TodoMenu - First Button Actived](TodoMenu - First Button Actived.png)

### 高亮当前所点击的按钮

### 点击按钮，显示对应的待办事项

## Todo Edit 组件

### 点击待办事项后显示编辑界面

### 样式改进

### 点击待办事项，编辑界面在显示/隐藏之间切换

### 点击“保存”按钮，保存更改

## Header 组件

### 添加 Header 及文本内容

### 添加 Icon Font

## Footer 组件

### 添加固定在底部的 Footer

## 参考资料

- [Collapsible contents (code block) in comments / spoiler tag · Issue #166 · dear-github/dear-github](https://github.com/dear-github/dear-github/issues/166)：用 Markdown 语法，实现内容的折叠效果。不过最后呈现出来的效果不好，就没有用上。