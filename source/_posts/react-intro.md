title: React生命周期
date: 2015-11-26 18:56:26
categories: React
tags: [react]
---

## React生命周期
#### 1.创建类
var Lifecycle = React.createClass({})
#### 2.实例化
 getInitialState()  初始化state
 componentWillMount()  渲染之前处理业务
 render()  渲染页面
 componentDidMount()  渲染之后执行，比如请求网络
#### 3.更新
componentWillRecieveProps()  props改变会回调
shouldComponentUpdate()     是否执行更新周期
componentWillUpdate()   页面重新渲染之前处理逻辑
render: 渲染页面
componentDidUpdate:              重新渲染页面后执行
#### 4.销毁
componentWillUnmount()         销毁回调
```javascript
/**
 * 学习react生命周期
 * auth honaf
 */
'use strict';

var React = require('react-native');
var {
  AppRegistry,
  StyleSheet,
  Text,
  View,
} = React;
var Util = require('ShopMall/views/Util');
var className = 'Lifecycle==>';
var Lifecycle = React.createClass({
  getDefaultProps(){
    Util.log(className+'getDefaultProps()');
  },

  getInitialState(){
    Util.log(className+'getInitialState()');
    return{

    }
  },

  componentWillMount(){
    Util.log(className+'componentWillMount()');
  },

  componentDidMount(){
    Util.log(className+'componentDidMount()');
  },

  componentWillReceiveProps(){
    Util.log(className+'componentWillReceiveProps()');
  },

  shouldComponentUpdate(){
    Util.log(className+'shouldComponentUpdate()');
  },

  componentWillUpdate(){
    Util.log(className+'componentWillUpdate()');
  },

  componentDidUpdate(){
    Util.log(className+'componentDidUpdate()');
  },

  render: function() {
    Util.log(className+'render');
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
      </View>
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#F5FCFF'
  }
});

module.exports = Lifecycle;
```
1.直接运行之后我们可以看到，先创建后实例化
打印结果：
Lifecycle==>getDefaultProps()
Lifecycle==>getInitialState()
Lifecycle==>componentWillMount()
Lifecycle==>render
Lifecycle==>componentDidMount()
2.当我们setState之后,这里强调一点，只要调用了setState，不管里面的值有没有更改，都会触发更新的生命周期（如下代码及打印）
```javascript
componentDidMount(){
    Util.log(className+'componentDidMount()');
    this.setState({loaded:true});
  },
```
运行结果：
Lifecycle==>getDefaultProps()
Lifecycle==>getInitialState()
Lifecycle==>componentWillMount()
Lifecycle==>render
Lifecycle==>componentDidMount()
Lifecycle==>shouldComponentUpdate()
Lifecycle==>componentWillUpdate()
Lifecycle==>render
Lifecycle==>componentDidUpdate()
3.此时如果在shouldComponentUpdate中返回false,后续更新生命周期将不会执行,当然看自己需求，一般不建议这么干。（如下代码及打印）

```javascript
  shouldComponentUpdate(){
    Util.log(className+'shouldComponentUpdate()');
    return false;
  },
```
 运行结果：
 Lifecycle==>getDefaultProps()
 Lifecycle==>getInitialState()
 Lifecycle==>componentWillMount()
 Lifecycle==>render
 Lifecycle==>componentDidMount()
 Lifecycle==>shouldComponentUpdate()
4.我们添加一个子组件再来测试一下,现在头部导入子组件，然后再添加到view中，可以看到当头部require了子组件（LifecycleItem统称为子组件）时，子组件的getDefaultProps()执行了,然后就是当前组件创建，执行getDefaultProps，执行实例化周期，到render渲染之时再实例化子组件，然后再调用当前组件componentDidMount，其中有setState,必然会执行更新周期，由于shouldComponentUpdate中返回false,就终止了更新的周期。（如下代码及打印）
```javascript
var LifecycleItem = require('./LifecycleItem');

render: function() {
    Util.log(className+'render');
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>
          Welcome to React Native!
        </Text>
        <LifecycleItem/>
      </View>
    );
  }
```
打印结果：
 ~~LifecycleItem==>getDefaultProps()
 Lifecycle==>getDefaultProps()
 Lifecycle==>getInitialState()
 Lifecycle==>componentWillMount()
 Lifecycle==>render
   ~~LifecycleItem==>getInitialState()
   ~~LifecycleItem==>componentWillMount()
   ~~LifecycleItem==>render
   ~~LifecycleItem==>componentDidMount()
 Lifecycle==>componentDidMount()
 Lifecycle==>shouldComponentUpdate()
5.此时我们将shouldComponentUpdate中返回false,可以看到当前组件更新render之时也触发了子组件的props change的周期，而实际上props我们没有做任何更改，这个大家一起讨论交流
![](http://img.blog.csdn.net/20151126184328776)
```javascript
shouldComponentUpdate(){
    Util.log(className+'shouldComponentUpdate()');
    return true;
},
```
打印结果：
    ~~LifecycleItem==>getDefaultProps()
 Lifecycle==>getDefaultProps()
 Lifecycle==>getInitialState()
 Lifecycle==>componentWillMount()
 Lifecycle==>render
   ~~LifecycleItem==>getInitialState()
   ~~LifecycleItem==>componentWillMount()
   ~~LifecycleItem==>render
   ~~LifecycleItem==>componentDidMount()
 Lifecycle==>componentDidMount()
 Lifecycle==>shouldComponentUpdate()
 Lifecycle==>componentWillUpdate()
 Lifecycle==>render
  ~~LifecycleItem==>componentWillReceiveProps()
   ~~LifecycleItem==>shouldComponentUpdate()
   ~~LifecycleItem==>componentWillUpdate()
   ~~LifecycleItem==>render
   ~~LifecycleItem==>componentDidUpdate()
 Lifecycle==>componentDidUpdate()
