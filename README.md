# edit-rc
Want to edit react component and generate a page with preview instantly?

# Why
最近一段时间一直在忙于国际化官网的建设，通过对业务的熟悉，总结出前线运营对于官网的一些需求。其中最重要的就是对于页面静态内容的管控，比如图片的替换和文案的修改。如果每次对于这种需求都通过代码的修改和上线无疑是低人效的做法，所以一个 CMS 是必须的。而官网重展示而非后台重交互（通过交互来完成对复杂数据的处理），一个好用的 CMS 就必须满足**所见即所得**，即不懂代码的运营人员可以将自己的改动立即呈现出来，从而做到心中有数而不是经过发布到生产（或预览）环境的上线流程才看到自己所做的修改。这样运营上手修改，甚至从头配置一个页面，从而将前端人力从官网项目中解放出来，不用每天应对类似文案更新这样的重复需求。 

为了一套代码生成多个国家官网，现有官网采用了 React 技术栈搭建了 SPA，所以本人通过对 React 组件的可视化编辑实现了满足以上需求的 CMS 来配置和管理官网项目。

页面编辑如何抽象？一个页面可以看做是一堆数据的集合，数据即为内容（文字和图片等）和样式的集合。编辑页面，本质上是在编辑数据。根据粒度划分页面，粗至组件，细到 DOM 元素，我们需要在某个程度上来完成编辑的行为。

我们在用代码实现页面的过程，往往是将页面直接划分为大块，大块再细分为小块，再用 DOM 元素去组织这些小块。块即为 Block，借鉴 CSS 的一种命名约定 BEM，React 组件就是块（B）的实现形式，而 Element 和 Modifier 可以看做是组件内部通过属性的变化对展现做出的反应，这也就是 React 函数化思想的本质。所以我们在组件层做抽象无疑是最合适的，更细粒度到 DOM 元素都可以通过数据的变化来实现，React 接收不同数据就会帮我们渲染出不同的页面。

反过来说，如果我们细粒度到 DOM 元素，就必然会将元素相关的样式和内容信息维护起来，然后手动创建这些标签来完成页面渲染。将本可以维护在组件中的巨量信息暴露出来这无疑大大增加了复杂性和工作量，CMS 需要实现一个类似原型工具的画布来记录编辑者对于组件的位置信息、样式和内容的修改，而像运营人员这样的编辑者不关心这些信息，因为页面经前端开发后一开始发布出去就几乎定型，编辑样式也不是他们的工作。

所以页面抽象到组件，组件编辑后产生数据，再由这些包含了组件布局和组件属性的数据反推（SSR 或 SPA 都可以）出页面即可。

# How

以下将通过对 AntD 项目中展示类组件的编辑来说明实现过程。以 Avatar 为例，组件需要 icon、shape、size、src、srcSet、alt 和 onError 属性，分类一下：

|分类|属性|属性元类型|
|--|--|--|
|text|icon、alt|string|
|select|shape、size|Enum|
|link|src、srcSet|string|
|function|onError|function|

_注：本质上数字和链接这样的文本输入都可以简化为 text，但分开更容易做样式优化和类型验证这样的特殊处理_

#### 第一步
将以上分类信息挂载到组件上，这时我们约定如下格式的对象：
```
Avatar.info = {
  props: {
    icon: { type: 'text' },
    shape: { 
      type: 'select',
      options: ['circle', 'square']  
    },
    size: { 
      type: 'select',
      options: ['large', 'small', 'default'] 
    },
    src: { type: 'link' },
    srcSet: { type: 'link' },
    alt: { type: 'text' },
    onError: { type: 'function' },
  },
};
```

#### 第二步
在编辑页面的页面遍历以上信息来生成编辑表单：
```
function getFormItems(info) {
  return Object.keys(info.props).map(prop => {
      // 判断 prop 的 type 并渲染表单项
  })
}
```
表单项的渲染过程参考 [storybook/knobs-addon](https://github.com/storybookjs/storybook/tree/master/addons/knobs) 
从而实现以下控制组件的效果：
![knobs-effect](https://www.learnstorybook.com/design-systems-for-developers/storybook-addon-knobs.gif)

#### 第三步
表单编辑过程由于编辑平台也是 React 应用，直接引入组件库 `Groot`，遍历添加的组件即可创建组件，所需的 `props` 从表单获取即可：
```
[
 {
  componentName: 'aComponent',
  props: { foo: 1, bar: 2 }
 }, {
  componentName: 'bComponent',
  props: { foo: 3, bar: 4 }
 }
].map(item => React.createElement(Groot[item.componentName], item.props)) 
```
把渲染数据做好状态管理从而实现_所见即所得_的效果。

#### 第四步
完成后点击预览，数据同步到后端，采用前后端同构的 SSR 渲染，即可产出 HTML 文件。跟 [jamstack](https://jamstack.org/) 异曲同工。

# Next

- [ ] 引入栅格系统来满足对布局的需求
- [ ] 组件属性分类来实现自动化分类
- [ ] 选择组件属性的类型来手动分类
- [ ] 可编辑区域处理来实现组件嵌套
