# How to improve RN's FlatList performance?

React Native's `FlatList` is great! But when it comes to big lists it have some flaws. There are a lot of issues and blog posts talking about how you can improve it. Here, however, as someone really concerned about this, i'll make an attempt to create an exhaustive, comprehensive and collaborative documentation about this.

I hate to be corny, but there is no silver bullet (or "magic combination") for this issue. You have to consider the trade offs of every approach and what you think is a good experience for your audience and the best strategy for your app. But fortunatelly there are several tweaks you can try and improve your `FlatList`.

PR's are really needed for fixing my english, fixing enventual wrong concepts, adding relevant information and techniques, adding links.

----------


## Terms and meanings
There are a lot of terms used (on [docs](https://facebook.github.io/react-native/docs/virtualizedlist.html#windowsize) or some issues) that were confusing for me at first. So, let's get this out of the way from the start.

 - **VirtualizedList** is the component behind `FlatList`, and is React Native's implementation of the '[virtual list](https://bvaughn.github.io/react-virtualized/#/components/List)' concept.

 - **Performance**, in this context, imply a smooth (not chopy) scroll (and navigation in or out of your list) experience.
 - **Memory consumption**, in this context, is how much information about your list is being stored in memory, which could lead to a app crash.
 - **Blank areas** means that the VirtualizedList couldn't render your items fast enough, so you enter on a part of your list with non rendered components.
 - **Window** here is not your viewport but rather, size of the area in which items should be rendered.


----------


## Props
One way to improve your `FlatList` is by tweaking it's props. Here are a list of props that can help your with that.

### removeClippedSubviews
You can set the `removeClippedSubviews` prop to true, which unmount components that are off of the window.

**Win:** This is very memory friendly, as you will always have a little rendered list.

**Trade offs:** Be aware, that this implementation can have bugs, such as missing content if you use it on a component that will not unmount (such as a navigation route).
It also can be less performant, having choppy scroll animations for big lists with complex items on not-so-good devices, as it make crazy amounts of calculations per scroll.

### maxToRenderPerBatch
You can set the `maxToRenderPerBatch={number}`, which is a `VirtualizedList` prop that can be passed directly to `FlatList`. With this, you can control the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

**Win:** Setting a bigger number means less visual blank areas when scrolling (a better the fill rate).

**Trade offs:** More items per batch means less javascript performance, which means less responsiveness (clicking a item and opening the detail). If you have a static and non-interactive list, this could be the way to go.

### initialNumToRender
You can set the `initialNumToRender={number}`. This means the initial amount of items to render.

**Win:** You can set this value to the precise number of items that would cover the screen for every device. This can be a big performance boost when rendering the list component.

**Trade offs:** You are most likely to see blank areas when setting a low `initialNumToRender`.

### windowSize
You can set the `windowSize={number}`. The number passed here is a measurement unit where 1 is equivalent to your viewport height. The default value is 21, being 10 viewports above, 10 below, and one in between.

**Win:** If you're worried mainly about performance, you can set a bigger `windowSize` so your list will run smoothly and with less blank space. If you're mainly worried about memory consumption, you can set a lower `windowSize` so your rendered list will be smaller.

**Trade offs:** For a bigger `windowSize`, you will have a bigger memory consumption. For a lower `windowSize`, you will have lower performance and bigger change of seeing blank areas.

### legacyImplementation
[This prop](https://facebook.github.io/react-native/docs/flatlist.html#legacyimplementation), when true, make your `FlatList` rely on the older `ListView`, instead of `VirtualizedList`.

**Win:** This will make your list definitely perform better, as it removes virtualization and render all your items at once.

**Trade offs:** Your memory consumption goes to the roof and chances are good that a big list (100+) with complex items will crash your app.
It also fires a warning that the above tweaks will not work, because you're now on a ListView.

### disableVirtualization
You will see people advocation the use of this prop on some issues. But [this is deprecated now](https://facebook.github.io/react-native/docs/virtualizedlist.html#disablevirtualization). This used to do something similar to `legacyImplementation`.


----------


## List items

There are also some win-win strategies that involves your list item components. They are being managed by VirtualizedList a lot, so they need to be fast.

### Use simple components
The more complex your components are, the slower they will render. Try to avoid a lot of logic and nesting in your list items. If you are reusing this list item component a lot in your app, create a duplicate just for your big lists and make them with less logic as possible and less nested as possible.

### Use light components
The heavier your components are, the slower they render. Avoid heavy images (use a cropped version for list items, as small as possible). Talk to your design team, use as little effects and interactions and information as possible in your list. Save them to your item's detail.

### Use shouldComponentUpdate
Implement update verification to your components. React's PureComponent is mostly for when you don't have time to think. If you're reading this, you do have time, so, create the strictest rules for your list item components. If your list is simple enough, you could even use

    shouldComponentUpdate() {
      return false
    }

----------

## Links
[The main thread about the topic](https://github.com/facebook/react-native/issues/13413)

[Very well documented memory leak](https://github.com/facebook/react-native/issues/16590)

[VirtualizedList doc](https://facebook.github.io/react-native/docs/virtualizedlist.html#disablevirtualization)

[FlatList doc](https://facebook.github.io/react-native/docs/flatlist.html#legacyimplementation)

[Optimizing list render performance in React Native (by @shichongrui)](http://matthewsessions.com/2017/05/15/optimizing-list-render-performance.html)

[React Native 100+ items flatlist very slow performance (on StackOverflow)](https://stackoverflow.com/questions/44384773/react-native-100-items-flatlist-very-slow-performance)
