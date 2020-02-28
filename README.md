I'm proud to announce that this article is now [part of the official docs](https://facebook.github.io/react-native/docs/next/optimizing-flatlist-configuration).

(*Before going on, check out my newest [React/React Native open source project](https://github.com/filipemerker/react-instagram-media)!*)

# How to improve RN's FlatList performance?

React Native's `FlatList` is great! But when it comes to big lists it has some flaws. There are a lot of issues and blog posts talking about how you can improve it. Here, however, as someone really concerned about this, i'll make an attempt to create an exhaustive, comprehensive and collaborative documentation about this.

I hate to be corny, but there is no silver bullet (or "magic combination") for this issue. You have to consider the trade offs of every approach and what you think is a good experience for your audience and the best strategy for your app. But fortunatelly there are several tweaks you can try and improve your `FlatList`.

PR's are really needed for fixing my English, fixing wrong concepts, adding relevant information and techniques, and adding links. Giving a ⭐ is also very helpful (:



## Terms and meanings
There are a lot of terms used (on [docs](https://facebook.github.io/react-native/docs/virtualizedlist.html#windowsize) or some issues) that were confusing for me at first. So, let's get this out of the way from the start.

 - **VirtualizedList** is the component behind `FlatList`, and is React Native's implementation of the '[virtual list](https://bvaughn.github.io/react-virtualized/#/components/List)' concept.

 - **Performance**, in this context, imply a smooth (not chopy) scroll (and navigation in or out of your list) experience.
 - **Memory consumption**, in this context, is how much information about your list is being stored in memory, which could lead to a app crash.
 - **Responsiveness**, in this context, is your apps's ability to respond to interactions. Low responsiveness, for instance, is when you touch on a component and it waits a bit to respond, instead of responding immediately as expected.
 - **Blank areas** means that the VirtualizedList couldn't render your items fast enough, so you enter on a part of your list with non rendered components.
 - **Window** here is not your viewport but, rather, the area size in which items should be rendered.



## Props
One way to improve your `FlatList` is by tweaking it's props. Here are a list of props that can help your with that.

### removeClippedSubviews
You can set the `removeClippedSubviews` prop to true, which unmount components that are off of the window.

[Default: `false`](https://github.com/facebook/react-native/issues/13316#issuecomment-298044067)

**Win:** This is very memory friendly, as you will always have a little amount of rendered items instead of the whole list.

**Trade offs:** Be aware that this implementation can have bugs, such as missing content (mainly observed on iOS) if you use it on a component that will not unmount (such as a root navigation scene).
It also can be less performant, having choppy scroll animations for big lists with complex items on not-so-good devices, as it makes crazy amounts of calculations per scroll.

### maxToRenderPerBatch
You can set the `maxToRenderPerBatch={number}`, which is a `VirtualizedList` prop that can be passed directly to `FlatList`. With this, you can control the amount of items rendered per batch, which is the next chunk of items rendered on every scroll.

[Default: `10`](https://github.com/facebook/react-native/blob/7d741d1119532213e2c30707320351fb56c63953/Libraries/Lists/VirtualizedList.js#L439)

**Win:** Setting a bigger number means less visual blank areas when scrolling (a better the fill rate).

**Trade offs:** More items per batch means less javascript performance, which means less responsiveness (clicking a item and opening the detail). If you have a static and non-interactive list, this could be the way to go.

### updateCellsBatchingPeriod
While `maxToRenderPerBatch` tells the amount of items rendered per batch, setting `updateCellsBatchingPeriod={number}` tells to your VirtualizedList the delay, [in milliseconds](https://github.com/facebook/react-native/blob/d01ab66b47a173a62eef6261e2415f0619fefcbb/Libraries/Interaction/Batchinator.js#L24), between batch renders. In other words, it defines how frequently your component will be rendering the windowed items.

[Default: `50`](https://github.com/facebook/react-native/blob/7d741d1119532213e2c30707320351fb56c63953/Libraries/Lists/VirtualizedList.js#L442)

**Win:** Combining this prop with `maxToRenderPerBatch` gives you the power to, for example, render more items in a less frequent batch, or less items in a more frequent batch. Which works the best for your use case.

**Trade offs:** Less frequent batches may cause blank areas. More frequent batches may cause responsiveness and performance loss.

### initialNumToRender
You can set the `initialNumToRender={number}`. This means the initial amount of items to render.

[Default: `10`](https://github.com/facebook/react-native/blob/7d741d1119532213e2c30707320351fb56c63953/Libraries/Lists/VirtualizedList.js#L428)

**Win:** You can set this value to the precise number of items that would cover the screen for every device. This can be a big performance boost when rendering the list component.

**Trade offs:** You are most likely to see blank areas when setting a low `initialNumToRender`.

### windowSize
You can set the `windowSize={number}`. The number passed here is a measurement unit where 1 is equivalent to your viewport height. The default value is 21, being 10 viewports above, 10 below, and one in between.

[Default: `21`](https://facebook.github.io/react-native/docs/virtualizedlist.html#windowsize)

**Win:** If you're worried mainly about performance, you can set a bigger `windowSize` so your list will run smoothly and with less blank space. If you're mainly worried about memory consumption, you can set a lower `windowSize` so your rendered list will be smaller.

**Trade offs:** For a bigger `windowSize`, you will have a bigger memory consumption. For a lower `windowSize`, you will have lower performance and bigger chance of seeing blank areas.

### legacyImplementation
[This prop](https://facebook.github.io/react-native/docs/flatlist.html#legacyimplementation), when true, make your `FlatList` rely on the older `ListView`, instead of `VirtualizedList`.

[Default: `false`](https://facebook.github.io/react-native/docs/flatlist.html#legacyimplementation)

**Win:** This will make your list definitely perform better, as it removes virtualization and render all your items at once.

**Trade offs:** Your memory consumption goes to the roof and chances are good that a big list (100+) with complex items will crash your app.
It also fires a warning that the above tweaks will not work, because you're now on a ListView.

### disableVirtualization
You will see people advocation the use of this prop on some issues. But [this is deprecated now](https://facebook.github.io/react-native/docs/virtualizedlist.html#disablevirtualization). This used to do something similar to `legacyImplementation`.



## List items

There are also some win-win strategies that involves your list item components. They are being managed by VirtualizedList a lot, so they need to be fast.

### Use simple components
The more complex your components are, the slower they will render. Try to avoid a lot of logic and nesting in your list items. If you are reusing this list item component a lot in your app, create a duplicate just for your big lists and make them with less logic as possible and less nested as possible.

### Use light components
The heavier your components are, the slower they render. Avoid heavy images (use a cropped version for list items, as small as possible). Talk to your design team, use as little effects and interactions and information as possible in your list. Save them to your item's detail.

### Use shouldComponentUpdate
Implement update verification to your components. React's PureComponent implement a `shouldComponentUpdate` with shallow compasion. This is expensive here, because it need to check all your props. If you want a good bit-level performance, create the strictest rules for your list item components, checking only props that could potentially change. If your list is simple enough, you could even use
```js
    shouldComponentUpdate() {
      return false
    }
```

### Use cached optimized images
I personally use [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image) from [@DylanVann](https://github.com/DylanVann). Every image in your list is a `new Image()` instance. The faster it reaches the `loaded` hook, the faster your Javascript thread will be free again.

### Use getItemLayout
You can set the `getItemLayout` to your `FlatList` component. If all your list item components have the same height (or width, for a horizontal list), passing this prop removes the need for your `FlatList` to dynamically calculate it every time. This is a very desirable optimization technique and if your components have dynamic size, and you really need performance, consider asking your design team if they may think of a redesign in order to perform better.
Your method should look like this, for items with height of, say, `70`:
```js
   getItemLayout = (data, index) => ({
     length: 70,
     offset: 70 * index,
     index
   })
```

### Use keyExtractor
You can set the `keyExtractor` to your `FlatList` component. This prop is used for caching and as the React `key` to track item re-ordering. For example, if you're using your item id as the key:
```js
   keyExtractor={item => item.id}
```
----------

## Links
There is a lot of discussion going on about this topic, so below I try to list the most relevant threads

### Articles and posts

[The main thread about the topic](https://github.com/facebook/react-native/issues/13413)

[Very well documented memory leak](https://github.com/facebook/react-native/issues/16590)

[Optimizing list render performance in React Native (by @shichongrui)](http://matthewsessions.com/2017/05/15/optimizing-list-render-performance.html)

[React Native 100+ items flatlist very slow performance (on StackOverflow)](https://stackoverflow.com/questions/44384773/react-native-100-items-flatlist-very-slow-performance)

### Oficial content

[VirtualizedList doc](https://facebook.github.io/react-native/docs/virtualizedlist.html#disablevirtualization)

[VirtualizedList code](https://github.com/facebook/react-native/blob/7d741d1119532213e2c30707320351fb56c63953/Libraries/Lists/VirtualizedList.js)

[FlatList doc](https://facebook.github.io/react-native/docs/flatlist.html#legacyimplementation)

[FlatList code](https://github.com/facebook/react-native/blob/7d741d1119532213e2c30707320351fb56c63953/Libraries/Lists/FlatList.js)
