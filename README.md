# react项目添加测试用例
## 一 安装的插件
### ① React Testing Library
[React Testing Library](https://testing-library.com/docs/react-testing-library/intro)

安装
```
npm install --save-dev @testing-library/react@12.1.4
```
这里指定版本是因为其他版本会包react-dom的错误

### ② user-event
[user-event](https://testing-library.com/docs/user-event/intro)

通过派发事件模拟用户交互的库

```
npm install --save-dev @testing-library/user-event
```

### ③ jest-dom

[jest-dom](https://testing-library.com/docs/ecosystem-jest-dom/)

为jest提供DOM元素匹配器

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
在根目录添加jest.config.js文件
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
在项目的test目录下添加.test.ts文件

### describe块
`describe(name, fn)`创建一个包含相关测试簇的块，第一个参数是测试描述，第二个参数是毁掉函数。

### test块
`test(name, fn, timeout)`运行一个测试。前两个参数与describe类似，第三个参数可选，表示等待多长时间推出测试，默认5秒。

### beforeEach和beforeAll afterEach和afterAll
beforEach(fn, timeout) 本文件内每个test运行前需要执行的函数，如果该函数返回一个promise，jest等待promise resolve后执行test
beforeAll(fn, timeout) 本文件内的任何测试运行之前执行函数，如果该函数返回一个promise，jest等待promise resolve后执行test
afterEach(fn, timeout) 本文件内每个测试运行完成后执行函数
afterAll(fn, timeout) 本文件内所有测试运行完成后执行函数

以上函数可以用于做测试需要的准备工作和收尾工作。例如：模拟登录，模拟setWaiting，模拟主题等，以及在结束后清除模拟。

### 渲染相关方法
[文档](https://testing-library.com/docs/react-testing-library/api#render)

用于组件测试
以render为例: render(ui:React.ReactElement<any>,options) 渲染React组件并将其挂到容器中，容器默认为document.body。

### 查询方法
[文档](https://testing-library.com/docs/queries/about)

用来页面中找元素的方法
#### 查询单个元素
有三种形式
getBy... 返回匹配到的元素，如果没有匹配到或者匹配多个抛出错误
queryBy... 与getBy的区别是如果没有匹配到返回null，匹配多个抛出错误
findBy... 与getBy区别是返回promise，匹配到resolve 如果超时没匹配到，或者匹配多个reject

#### 查询多个元素
也有三种形式
getAllBy... 返回匹配元素数组，没有匹配到抛出错误
queryAllBy... 没有匹配到返回空数组
findAllBy.. 返回promise 匹配到resolve，没有匹配到reject

