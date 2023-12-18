# react项目添加测试用例
## 一 安装的插件
### ① React Testing Library
[React Testing Library](https://testing-library.com/docs/react-testing-library/intro)

安装
```
npm install --save-dev @testing-library/react@12.1.4
```
这里指定版本是因为其他版本会报`react-dom`的错误

### ② user-event
[user-event](https://testing-library.com/docs/user-event/intro)

通过派发事件模拟用户交互的库

```
npm install --save-dev @testing-library/user-event
```

### ③ jest-dom

[jest-dom](https://testing-library.com/docs/ecosystem-jest-dom/)

为`jest`提供DOM元素匹配器

```
npm install --save-dev @testing-library/jest-dom
```

### ④ jest
[jest](https://jestjs.io/)

#### 安装
```
npm install --save-dev jest
```
#### 添加配置文件
在根目录添加`jest.config.js`文件
```js
module.exports = {
  verbose: true, //执行的过程是否把每个测试文件中的测试用例的状态给打印到控制台上
  transform: { '^.+\\.[jt]sx?$': '<rootDir>/test/jestPreprocess.js' }, //定义在运行测试时，如何转换测试文件

  //在每次测试之前自动清除模拟调用、实例和结果
  clearMocks: true,

  // 覆盖率文件的目录
  coverageDirectory: 'coverage',

  //指出是否收集测试时的覆盖率信息
  collectCoverage: true,

  moduleFileExtensions: ['tsx', 'ts', 'js'],//当import没有指定文件后缀时，Jest将从配置的数组中从左到右查找扩展名

  // node_modules
  moduleDirectories: ['node_modules', '../MP/node_modules'],//定义优先加载的依赖的目录路径

//对一些模块的代理处理
  moduleNameMapper: {
    '\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$': '<rootDir>/test/mocks/fileMock.js',
    '\\.(css|less)$': '<rootDir>/test/mocks/styleMock.js',
    '@/(.*)': ['<rootDir>\\src\\$1', '..\\Common\\src\\$1'],
  },

  transformIgnorePatterns: ['node_modules/(?!(@material-ui|@babel)/.*)'],//忽略文件
  testEnvironment: 'jsdom', //测试环境
  // testRegex: ['/test/.*\\.(test)\\.(tsx)$'], //测试文件目录
  testMatch: ['**/test/**/*.test.(ts|tsx)'], //测试单个文件
};
```

## 二 添加测试文件编写测试用例
在项目的test目录下添加`.test.ts`文件

### describe块
`describe(name, fn)`创建一个包含相关测试簇的块，第一个参数是测试描述，第二个参数是毁掉函数。

### test块
`test(name, fn, timeout)`运行一个测试。前两个参数与`describe`类似，第三个参数可选，表示等待多长时间推出测试，默认5秒。

### beforeEach和beforeAll afterEach和afterAll
`beforEach(fn, timeout)` 本文件内每个test运行前需要执行的函数，如果该函数返回一个promise，jest等待promise resolve后执行test
`beforeAll(fn, timeout)` 本文件内的任何测试运行之前执行函数，如果该函数返回一个promise，jest等待promise resolve后执行test
`afterEach(fn, timeout)` 本文件内每个测试运行完成后执行函数
`afterAll(fn, timeout)` 本文件内所有测试运行完成后执行函数

以上函数可以用于做测试需要的准备工作和收尾工作。例如：模拟登录，模拟`setWaiting`，模拟主题等，以及在结束后清除模拟。

### 渲染相关方法
[文档](https://testing-library.com/docs/react-testing-library/api#render)

用于组件测试
以`render`为例: `render(ui:React.ReactElement<any>,options)` 渲染React组件并将其挂到容器中，容器默认为`document.body`。

### 查询方法
[文档](https://testing-library.com/docs/queries/about)

用来页面中找元素的方法
#### 查询单个元素
有三种形式
`getBy...` 返回匹配到的元素，如果没有匹配到或者匹配多个抛出错误
`queryBy...` 与`getBy`的区别是如果没有匹配到返回`null`，匹配多个抛出错误
`findBy...` 与`getBy`区别是返回promise，匹配到resolve 如果超时没匹配到，或者匹配多个reject

#### 查询多个元素
也有三种形式
`getAllBy...` 返回匹配元素数组，没有匹配到抛出错误
`queryAllBy...` 没有匹配到返回空数组
`findAllBy..` 返回promise 匹配到resolve，没有匹配到reject

### 用户交互
[文档](https://testing-library.com/docs/dom-testing-library/api-events)

用户交互就是通过派发事件，模拟用户行为。RTL有两种派发事件的API `fireEvent`和`user-event`。
`fireEvent`可以应付大多数场景，但是`fireEvent`并不能精确反应用户行为。比如点击事件，`fireEvent`只是触发一个`click`事件，而事实上用户在点击按钮时会触发`mouseMove`、`mouseDown`、`mouseUp`、`click`等一系列事件。
所以更推荐使用`user-event`。

[user-event](https://testing-library.com/docs/user-event/intro/)

#### waitFor函数
用户交互有时是异步的，RTL提供`waitFor(callBack, options)`API用来解决异步的情况，任何需要等待的结果都可以使用`waitFor()`.

### 模拟函数
[模拟函数文档](https://www.jestjs.cn/docs/mock-function-api)
对于某些不关心其实现，只需要返回值的函数，例如:后台调用，插件等可以使用模拟函数模拟其返回值。jest会捕捉到函数的调用，并以配置的返回值代替。
#### 模拟后台调用
```js
  import axios from 'axios';
	jest.mock('axios');
	axios.get.mockResolvedValue(resp);
```
`mockResolvedValue()`函数为`axio`s的get配置了一个假的响应结果resp。这样做的好处是速度快，而且便于维护。

#### 模拟模块
```js
  import defaultExport from '../foo-bar-baz';
	jest.mock('../foo-bar-baz', () => {
    const originalModule = jest.requireActual('../foo-bar-baz');
  	return {
      __esModule: true,
    	...originalModule,
    	default: jest.fn(() => 'mocked baz'),
      foo: 'mocked foo'
    }
  })
```
这个例子，模拟了`foo-bar-baz`这个包的默认导出，并将其返回值配置为`'mocked baz'`
### 断言
[文档](https://www.jestjs.cn/docs/expect)
匹配器是jest提供的用来测试结果是否符合预想。jest提供大量的匹配器函数，几乎可以满足所有测试需求
```js
expect(2 + 2).toBe(4);
expect(n).not.toBeUndefined();
```

### 结语
有以上介绍工具，就可以对MP项目进行单元测试和组件测试了。对公用方法单元测试时，应该尽量多的添加测试用例，测试各种情况，充分应用jest的匹配函数。
进行组件测试时要以用户为中心，通过代码如实模拟用户在页面上的操作。




