---
title: 一文读懂JavaScript测试框架Jest
date: 2022-12-24
tags: 
  - tutorial
  - jest javascript
categories: technology
keywords: 'jest test javascript'
---

# 一、基础知识
> 可参考下面代码一同学习：
> 1. [reader.test.js](https://github.com/hellooo-stack/hellooo-commons-nodejs/blob/master/__tests__/excel/reader.test.js)
> 2. [writer.test.js](https://github.com/hellooo-stack/hellooo-commons-nodejs/blob/master/__tests__/excel/writer.test.js)

## 1.1 是什么

Jest是一个流行的JavaScript测试框架，由Facebook开发，可以让我们在开发过程中更方便地进行单元测试。它提供了很多有用的功能，例如：

- 快速和可靠的测试运行器：可以快速地运行测试用例，并在测试用例间进行快速切换
- 断言库：在测试用例中进行断言
- 模拟函数：模拟函数的行为
- 异步测试支持：方便地测试异步代码。
- 自动模块打包：在测试用例中轻松加载依赖模块
- 自动运行测试：在文件发生变化时自动运行测试用例
- 可视化报告：生成详细的可视化报告，以更好地理解测试结果

Jest可以运行在NodeJS环境中，也可以运行在浏览器中，因此可以用于测试各种类型的JavaScript代码。它的语法简单易懂，易于学习，因此是一个非常流行的JavaScript测试框架。

## 1.2 使用步骤

```jsx
// 1. 安装
npm i --save-dev jest

// 2. 创建一个包含需要被测试的代码的文件sum.js:
function sum(a, b) {
  return a + b;
}
module.exports = sum;

// 3. 编写测试用例
// 创建一个sum.test.js的文件:
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});

// 4. 在package.json里面添加如下配置:
{
  "scripts": {
    "test": "jest"
  }
}

// 5. 运行npm test, 将打印如下信息:
PASS  ./sum.test.js
✓ adds 1 + 2 to equal 3 (5ms)
```

## 1.3 编写测试用例

在编写Jest测试用例时，通常会使用以下几种函数：

- `describe()`：用于组织测试用例，将相关的测试用例分组。可以将多个`test()`函数包装在`describe()`函数中，并为其提供一个描述性字符串。就是说，这个函数是用来对测试用例进行分组的，它分组之后还可以在该函数内再次调用这个函数进行子分组，如此嵌套。
  
    ```jsx
    // 分组
    describe('group1', () => {
    
        // 子分组嵌套
        describe('sub1 of group1', () => {
    
            // 测试
            test('test1', () => {
                // 返回匹配对象
                let expectationObj = expect(1 + 1);
                // 执行断言操作测试代码
                expectationObj.toBe(2);
            });
        });
    
        // 嵌套的第二个子分组
        describe('sub2 of group1', () => {
    
            test('test2', () => {
                console.log('success of test2');
            });
        });
    });
    ```
    
- `test()`函数：用于执行单个测试用例。
- `it()`函数：等价于`test()`函数，用于执行单个测试用例。
- `expect()`函数：用来返回一个”匹配对象“，然后可以用“匹配器”来对这个“匹配对象”执行一些断言操作，用于测试代码的期望输出。
- `toBe()`、`toEqual()`、`toMatch()`等断言函数：也被称为匹配器。Jest支持的匹配器可通过[该文档](https://jestjs.io/zh-Hans/docs/expect)查询。
- 辅助函数：当`beforeXXX()`函数和`afterXXX()`函数在`describe()`函数内的时，这些设置只会作用于当前的`describe()`分组。
    - `beforeEach()`、`beforeEach()`：用于在每个测试用例之前/之后执行特定的代码。
      
        ```jsx
        beforeEach(() => {
          initializeCityDatabase();
          // 如果这个方法是异步方法, 并且返回promise, 那么可以像测试异步代码那样, 直接返回这个promise
          return initializeCityDatabase();
        });
        
        afterEach(() => {
          clearCityDatabase();
        });
        
        test('city database has Vienna', () => {
          expect(isCity('Vienna')).toBeTruthy();
        });
        
        test('city database has San Juan', () => {
          expect(isCity('San Juan')).toBeTruthy();
        });
        ```
        
    - `beforeAll()`、`afterAll()`：用于在所有测试用例之前/之后执行特定的代码。
      
        ```jsx
        beforeAll(() => {
          return initializeCityDatabase();
        });
        
        afterAll(() => {
          return clearCityDatabase();
        });
        
        test('city database has Vienna', () => {
          expect(isCity('Vienna')).toBeTruthy();
        });
        
        test('city database has San Juan', () => {
          expect(isCity('San Juan')).toBeTruthy();
        });
        ```
    
- `done()`回调函数：用于在异步测试用例执行完毕时通知Jest。
  
    ```jsx
    test('fetches data successfully', (done) => {
      fetchData().then(data => {
        expect(data).toBeDefined();
        done();
      });
    });
    ```
    

## 1.4 测试异步代码

当有异步方式运行的代码时，Jest需要知道当前它测试的代码是否已经完成，然后它可以转移到另一个测试。Jest通过`callback`，`Promises`或`Async/Await`来处理这种情况。

- `callback`：
  
    ```jsx
    // 假如fetchData(callback) 是一个异步函数, 会在获取到数据之后调用callback
    // 假如需要测试获取到的数据是不是'peanut butter', 不要这样做:
    test('the data is peanut butter', () => {
      function callback(data) {
        expect(data).toBe('peanut butter');
      }
    
      // 一旦fetchData执行结束, 此测试就在没有调用回调函数前结束
      fetchData(callback);
    });
    
    // 正确做法:
    test('the data is peanut butter', done => {
      function callback(data) {
        try {
          expect(data).toBe('peanut butter');
          // Jest会等done回调函数执行结束后, 结束测试
          done();
        } catch (error) {
          done(error);
        }
      }
    
      fetchData(callback);
    });
    ```
    
- `Promises`：
  
    ```jsx
    // 如果fetchData不使用回调函数, 而是返回一个Promise, 其解析字符串为‘peanut butter', 那么可以这样测试:
    test('the data is peanut butter', () => {
      // 这里一定要有return, 否则在这个promise被resole, then()有机会执行之前, 测试就已经被视为已经完成了
      return fetchData().then(data => {
        expect(data).toBe('peanut butter');
      });
    });
    ```
    
- `Async/Await`：
  
    ```jsx
    // 改用Async/Await的方式:
    test('the data is peanut butter', async () => {
      const data = await fetchData();
      expect(data).toBe('peanut butter');
    });
    
    test('the fetch fails with an error', async () => {
      expect.assertions(1);
      try {
        await fetchData();
      } catch (e) {
        expect(e).toMatch('error');
      }
    });
    ```
    

## 1.5 Jest配置文件

Jest的配置文件是用于配置Jest运行参数的文件。默认情况下，Jest会在项目根目录中寻找名为`jest.config.js`的文件作为配置文件。在配置文件中，可以包含各种选项，例如：

- `rootDir`：指定项目的根目录。
- `testMatch`：指定要包含在测试运行中的测试文件的匹配模式。
- `moduleNameMapper`：指定将模块名称映射到模块文件路径的对象。
- `transform`：指定要应用于测试文件的转换函数。
- `setupFilesAfterEnv`：指定要在测试环境初始化后运行的脚本文件。

下面是一个简单的Jest配置文件的示例：

```json
module.exports = {
  // 规定哪些文件应该被测试
  testMatch: ['**/*.test.js'],

  // 规定哪些文件应该被忽略
  testPathIgnorePatterns: ['/node_modules/'],

  // 规定如何映射模块的路径
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },

  // 规定在测试运行之前要执行的脚本
  setupFiles: ['<rootDir>/jest.setup.js'],

  // 规定在测试运行之前，且在测试环境被创建之后要执行的脚本
  setupFilesAfterEnv: ['<rootDir>/jest.setupAfterEnv.js'],

  // 规定如何转换源文件
  transform: {
    '^.+\\.jsx?$': 'babel-jest',
  }
}
```

Jest配置文件生成：

1. 确保已在项目中安装了Jest。如果还没有安装，可以在终端中运行`npm install --save-dev jest`来安装
2. 在终端中运行命令`jest --init`，将启动一个向导，提示选择配置选项。可以根据提示选择适合项目的选项。完后会在项目根目录下生成配置文件`jest.config.js`

# 二、最佳实践

## 2.1 目录结构

Jest测试用例通常位于项目中的`__tests__`文件夹中。每个测试用例都是一个独立的JavaScript文件，文件名通常以`.test.js`结尾。例如，如果要测试的是一个名为`add.js`的模块，则可以在`__tests__`文件夹中创建一个名为`add.test.js`的测试用例文件。测试用例文件中的代码应该类似于以下内容：

```bash
const add = require('../add');

test('adds 1 + 2 to equal 3', () => {
  expect(add(1, 2)).toBe(3);
});
```

## 2.2 配置实践

1. 被测试文件`sum.js`路径为：`src/sum.js`，测试文件`sum.test.js`路径为：`__test__/sum.test.js`，`sum.test.js`内导入`sum.js`的方式为：`const { add } = require('../src/add');`，前缀`../`在每个测试文件下都要写一遍，非常冗余，解决方式：
   
    ```json
    // jest.config.js
    module.exports = {
      // ...其他配置
      moduleNameMapper: {
        '^src/(.*)$': '<rootDir>/src/$1',
      },
    };
    ```
    

## 2.3 问题处理

1. WebStorm无法识别Jest函数，报错："Unresolved function or method test()"
    - 原因：WebStorm没有正确识别Jest的类型定义。
    - 解决方案：安装@types/jest包，`npm install @types/jest --save-dev`
