---
layout:     post
title:      Adding keyboard shortcuts in React
date:       2019-06-25 20:24:52
summary:
categories:
---

Keyboard shortcuts are a great feature for your React app. They help power users to get the most from your application, and can greatly reduce the time spent navigating menus and suchlike.  I am currently building a web based markdown editor (because the world definitely need another one of those). I'm aiming for complete minimalism so that when writing it's just the raw text, with a preview pane which shows up when you use a keyboard shortcut.  

A quick Google threw up [react-hotkeys](https://github.com/greena13/react-hotkeys) which seemed great for the task. It's super simple to configure and set up.

```react
const keyMap = {
    PREVIEW: "ctrl+shift+p",
};
class App extends Component {

    handlers = {
        PREVIEW: event => this.setState((state) => {
            return {
                ...state,
                preview: !state.preview
            }
        })

    };
...
<div className="App">
                <GlobalHotKeys keyMap={keyMap} handlers={this.handlers}/>
... the rest of your code here....

```
The ```keymap``` is a list of the actions you want to make available (you can make up any name you want for your commands) mapped to the keyboard shortcut you want for it.  ```handlers``` is an object, mapping the command names to an event handler function.  In the sample I update the preview field in my state, which is used to toggle a className in my app to either display or hide the preview div.

After dropping this code into my app the shortcut worked first time, __except for when my cursor was in a text area__.  Given that my app is almost 100% textarea this was a problem!  I discovered that react-hotkeys disables shortcuts from firing from inside textareas and other inputs by default, which makes 
 total sense. The standard when building webapps is for shortcuts to be single letters (eg. 'c' will compose a new email in gmail), and if you're typing a message out you don't want it to fire your shortcuts.

react-hotkeys has a configure method which allows you to modify the defaults.  Placing this configuration in my component makes hotkeys work irrelevant of focus:

```javascript
configure({
    ignoreTags: []
})
```

Simple!
