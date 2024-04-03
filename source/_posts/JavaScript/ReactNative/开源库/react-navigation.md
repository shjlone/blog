---
title: react-navigation
tags: React Native 
---

[https://reactnavigation.org/docs/](https://reactnavigation.org/docs/)

@react-navigation/native、@react-navigation/native-stack跟react-navigation的区别？

react-navigation是早期的版本，@react-navigation是新的架构，采用组件化的设计，每种导航器都是一个单独的包

@react-navigation/stack
@react-navigation/elements
@react-navigation/bottom-tabs
@react-navigation/native-stack
@react-navigation/native
@react-navigation/drawer
@react-navigation/routers
@react-navigation/material-top-tabs
@react-navigation/devtools
@react-navigation/compat


```shell
# 安装
npm install --save @react-navigation/stack



```

React Navigation默认的navigator：

- StackNavigator：堆栈导航，createNativeStackNavigator创建
- TabNavigator：tab页导航，createBottomTabNavigator、createMaterialBottomTabNavigator、createMaterialTopTabNavigator创建
- DrawerNavigator：抽屉导航，createDrawerNavigator创建



## 基本用法


```JavaScript


function DetailsScreen({ navigation }) {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Details Screen</Text>
      <Button
        title="Go to Details... again"
        onPress={() => navigation.push('Details')}
      />
      <Button title="Go to Home" onPress={() => navigation.navigate('Home')} />
      <Button title="Go back" onPress={() => navigation.goBack()} />
    </View>
  );
}

function HomeScreen({ navigation }) {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Home Screen</Text>
      <Button
        title="Go to Details"
        onPress={() => navigation.navigate('Details')}
      />
    </View>
  );
}



const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator initialRouteName="Home">
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}



```

## 常用属性

- navigate：路由跳转
- state：当前页面的状态
- setParams：修改路由参数
- goBack：关闭当前路由
- dispatch：分发事件


路由的切换过程中的生命周期回调函数还是基于React，并没有像Android端onPause、onResume等函数，可以通过以下方式来处理

- 监听focus变化，注意不同的版本的key不一样，6.x使用'focus'，4.x使用'didFocus'
- 使用useFocusEffect