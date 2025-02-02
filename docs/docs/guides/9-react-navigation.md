---
title: Integrate AppBar with react-navigation
---

# Integrate AppBar with react-navigation

## Prerequisites

 - `react-native-paper`
 - `react-navigation`
 
We assume that you have already installed the mentioned libraries above, otherwise please check out the guides below.
 
 [React Native Paper - Getting Started](https://callstack.github.io/react-native-paper/getting-started.html)
 
 [React Navigation - Getting Started](https://reactnavigation.org/docs/getting-started/)

## Stack Navigator

We will start with `react-navigation` by creating a basic navigation stack. Stack navigator gives us a possibility of transition between screens in our app and manage navigation's history. In a simple scenario where there is only one stack navigator present in the app, it resembles a navigation state in a browser.
Screens are pushed and popped from the stack while the user navigates to a new screen or go back to the previous one.

Let's create two screens. A main screen named `Home` and details screen named `Details`.

```js
import 'react-native-gesture-handler';
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';

const Stack = createStackNavigator();

export default function App() {
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

At the moment our navigation stack contains two screens and will render `HomeScreen` or `DetailsScreen` components according to the current navigation state. We have not implemented those components yet, so let's do this now:

```js
import React from 'react';
import {View, Text, Button, StyleSheet} from 'react-native';

function HomeScreen() {
  return (
    <View style={style.container}>
      <Text>Home Screen</Text>
    </View>
  );
}

function DetailsScreen() {
  return (
    <View style={style.container}>
      <Text>Details Screen</Text>
    </View>
  );
}

const style = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```

Once we have finished implementing the components, we can run the app and check how Stack looks like.

![navigationAppBar1](../../static/screenshots/react-navigation-appBar1.png)

To navigate from `HomeScreen` to `DetailsScreen` we can use the navigation object provided by `Stack.Screen` component. Every component rendered by `Stack.Screen` has an access to the navigation object via props. Let's modify our `HomeScreen` component:

```js
function HomeScreen({ navigation }) {
  return (
    <View style={style.container}>
      <Text>Home Screen</Text>
      <Button
        title="Go to details"
        onPress={() => navigation.navigate('Details')}
      />
    </View>
  );
}
```

Our result:

![navigationAppBar2](../../static/screenshots/react-navigation-appBar2.gif)

As you can see, we can already navigate between two screens. In the next steps, we will show you how to use Paper's `AppBar` instead of the default header.

### Adding AppBar

We can customize Stack's header by passing custom component:

```js
export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          header: CustomNavigationBar,
        }}>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

Now we will implement `CustomNavigationBar` using `AppBar` component:

```js
import { Appbar } from 'react-native-paper';

function CustomNavigationBar() {
  return (
    <Appbar.Header>
      <Appbar.Content title="My awesome app" />
    </Appbar.Header>
  );
}
```

Current implementation of the `CustomNavigationBar` is simple - we just render a title inside of it. You may notice there is no way to go back to the previous screen, because the back button is not visible in the header. Let's add it now and let's make sure it's visible on all Stack's screens except `Home` screen.

Firstly, pass navigation props to `CustomNavigationBar`:

```js
export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator
        initialRouteName="Home"
        screenOptions={{
          header: (props) => <CustomNavigationBar {...props} />,
        }}>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

Secondly, we check if the navigation bar receives a `back` prop. If it has, it means there is another screen on the stack beneath the current screen and we should render the back arrow button in such a case. (The `back` prop is sent in React Navigation 6.x; in 5.x a prop named `previous` is sent and can be checked for instead.)

```js
function CustomNavigationBar({ navigation, back }) {
  return (
    <Appbar.Header>
      {back ? <Appbar.BackAction onPress={navigation.goBack} /> : null}
      <Appbar.Content title="My awesome app" />
    </Appbar.Header>
  );
}
```

![navigationAppBar3](../../static/screenshots/react-navigation-appBar3.gif)


Another interesting pattern that can be implemented with `react-native-paper` and `react-navigation` is a "hamburger menu". Thanks to the `Menu` component we can add a nice looking pop-up to our `Appbar`. To implement this feature we need to make a couple of changes in `CustomNavigationBar`:
- Render a `Menu` component
- Pass `Appbar.Action` to the anchor prop
- Add a state to control `Menu` visibility

We also want the menu to appear only on `HomeScreen`, which means we will render it conditionally based on the `back` prop.

```js
function CustomNavigationBar({ navigation, back }) {
  const [visible, setVisible] = React.useState(false);
  const openMenu = () => setVisible(true);
  const closeMenu = () => setVisible(false);

  return (
    <Appbar.Header>
      {back ? <Appbar.BackAction onPress={navigation.goBack} /> : null}
      <Appbar.Content title="My awesome app" />
      {!back ? (
        <Menu
          visible={visible}
          onDismiss={closeMenu}
          anchor={
            <Appbar.Action icon="menu" color="white" onPress={openMenu} />
          }>
          <Menu.Item onPress={() => {console.log('Option 1 was pressed')}} title="Option 1" />
          <Menu.Item onPress={() => {console.log('Option 2 was pressed')}} title="Option 2" />
          <Menu.Item onPress={() => {console.log('Option 3 was pressed')}} title="Option 3" disabled />
        </Menu>
      ) : null}
    </Appbar.Header>
  );
}
```

Final result:

![navigationAppBar4](../../static/screenshots/react-navigation-appBar4.gif)

That's all we need! We have app bar that contains everything we need to navigate through screens and access an additional menu on the main screen. As you can see, with Material design `Appbar` provided by `react-native-paper` used together with `react-navigation` we can easily create an app that looks and works great.
