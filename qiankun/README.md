# qiankun（乾坤）源码阅读



### 基本使用

在主应用中注册子应用

```
import { registerMicroApps, start } from 'qiankun';

registerMicroApps([
  {
    name: 'react app', // app name registered
    entry: '//localhost:7100',
    container: '#yourContainer',
    activeRule: '/yourActiveRule',
  },
  {
    name: 'vue app',
    entry: { scripts: ['//localhost:7100/main.js'] },
    container: '#yourContainer2',
    activeRule: '/yourActiveRule2',
  },
]);

start();
```



在子应用中，不需要引入任何依赖即可接入 `qiankun` 主应用，但是子应用需要在自己的入口 js (通常就是你配置的 webpack 的 entry js) 导出 `bootstrap`、`mount`、`unmount` 三个生命周期钩子，以供主应用在适当的时机调用

```js
/**
 * bootstrap 只会在微应用初始化的时候调用一次，下次微应用重新进入时会直接调用 mount 钩子，不会再重复触发 bootstrap。
 * 通常我们可以在这里做一些全局变量的初始化，比如不会在 unmount 阶段被销毁的应用级别的缓存等。
 */
export async function bootstrap() {
  console.log('react app bootstraped');
}

/**
 * 应用每次进入都会调用 mount 方法，通常我们在这里触发应用的渲染方法
 */
export async function mount(props) {
  ReactDOM.render(<App />, props.container ? props.container.querySelector('#root') : document.getElementById('root'));
}

/**
 * 应用每次 切出/卸载 会调用的方法，通常在这里我们会卸载微应用的应用实例
 */
export async function unmount(props) {
  ReactDOM.unmountComponentAtNode(
    props.container ? props.container.querySelector('#root') : document.getElementById('root'),
  );
}

/**
 * 可选生命周期钩子，仅使用 loadMicroApp 方式加载微应用时生效
 */
export async function update(props) {
  console.log('update props', props);
}
```



### qiankun 与 single-spa 关系

`qiankun`是基于 `single-spa` 实现的，而 `single-spa` 规定入口 js 文件必须通过 export 形式导出三个函数：`bootstrap`、`mount`、`unmount`，这也就不难理解为什么 `qiankun` 也需要导出这三个函数了



同时，`qiankun` 对 `single-spa` 做了包装，降低了上手成本，并且优化了 `singe-spa` 只能加载 js 的缺点，可以加载 html 文件，这更加符合开发。因为目前大多数前端项目都是基于 webpack 打包成的单页面应用，入口都是 html 文件



当然，`qiankun` 做的不仅仅是这些，下面，就看看，`qiankun` 基于 `single-spa` 做了哪些优化



### qiankun 解读



