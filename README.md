# ReactNativeSampleChat
A collection of hopefully not-terrible practices to consider when building a React Native app

## The Big Idea

There's plenty of React and React Native tutorials out there that exercise a particular concept or two. I've been looking at a lot of these, and wanted to collect together everything I liked into one project; in this case, something that looks vaguely like a first grader building a social networking client. Consider this part-starter pack, navigation/ MobX/ whatever tutorial, demonstration of folder structure, design patterns, and ES6 constructs. Maybe I'm the only one in the world who has this issue, but sometimes I find it a bit difficult to take all of these little bits and robotically apply them wholistically to a new project without missing something, or at least a bit tedious rebuilding these bits in yet another project after doing something else for the last six months or so. So here it is, standing as either a testament to either my foresight or forgetfulness.

A few notes:
* Most of this stuff I haven't just made up. I try to give credit where it's due.
* This app is intentionally shallow and ugly. I build just enough to demonstrate the concept and precious little more than that.
* I will go into agonizing detail about things you might find trivial (looking at you, Mr. Level 97 React Wizard). This is a reflection of stupid questions I've stumbled on and smart questions asked of me when trying to help others with this stuff.

## Running the Sample

### Prerequisites
* You did all [this stuff](https://facebook.github.io/react-native/docs/getting-started.html) to get React Native working.

### Steps (iOS simulator)
1. Run `npm install`
2. Run `react-native link`
3. Run `react-native run-ios`

### Steps (Android simulator)
1. Start your Android emulator
2. Run `npm install`
3. Run `react-native link`
4. Run `react-native run-android`

Note that you only need to run npm install and react-native link once for both platforms.

Installing Android Studio from scratch accomplished almost all of the steps for setting up React Native for Android. The only other things I had to do was a) install the Marshmallow SDK, and b) add the ~/.profile file specified in the React Native instructions for Android.

## Concepts

### Project Structure

Credit to [Spencer Carli's Organizing a React Native Project](https://medium.com/the-react-native-log/organizing-a-react-native-project-9514dfadaa0)

#### Major Points
* Nothing of real consequence in the index.ios.js and index.android.js files. Instead, they point to the real starting point at app/index.js.
* The components/ config/ routes/ layouts structure. I may not be following these exactly, but my interpretation of layouts is that it is for stitching together scenes inside navigation containers, like stacks, and tab bars. Meanwhile, routes are the individual scenes inside of each of those navigation containers (thinking about renaming this to scenes). If this were an iOS storyboard, maybe layouts would be the trunk/ forks in the navigation tree while routes would be branches.
* Breaking up components/ groups of components into separate component, stylesheet, and index files. You can import from the index in a folder simply by referencing the folder (e.g., `../timeline`). This should make it easy to find the styles in effect for a component and limits the refactoring needed to be done when exports change. <s>I don't think the article said one way or the other, but I've decided things are easier if these indexes are made up of non-default exports (so I don't have to worry about whether or not to use curly braces)</s> The linter doesn't like this, I'll fix single imports to be default. routes/timeline (used in layouts/main) is the best example of this so far.
* Added my own "stores" folder for MobX stores, similar to what the author does for Redux (more on that later).

#### Stuff To Do (Better)
* Too much going on in layouts/main. Since the Timeline and Settings can each have their own navigation, I think they ought to have their own layout folders.

### Separating Markup/ Styling/ Logic

Credit to [chantastic's Container Components](https://medium.com/@learnreact/container-components-c0e67432e005), [Gustavo Machado's Tips for Styling your React Native Apps](https://medium.com/the-react-native-log/tips-for-styling-your-react-native-apps-3f61608655eb), as well as Mr. Carli above.

A big point of React is that separation of concerns in frameworks that keep HTML and JS in separate file isn't the holy grail, because those two "separate" files are tightly-coupled, anyway. Vue.js buffs say React got too much HTML in my JavaScript, upsetting non-JS-savvy designers who wanted to tweak the markup and CSS [1]. It seems like you can have your React and eat it too (mmm... React) if you follow some rules about what kind of JSX goes where, so, even though it's all JavaScript, at least you're not mixing the right and wrong JavaScripts together:
1. Separate the "stylesheets" (which are just JS objects) in their own files.
2. Use common values that are referenced by those indiviual stylesheets (this is what the app/config folder above is for). These are really just glorified constants.
3. Most of your components should have very little or no state. Ideally, a component is just a render() function. Even if you have some branching, if you just have a render function that references some props, is it really all that different from Vue.js or Angular markup extensions? This should make it fairly straightforward for someone to change markup without being burdened by having to pick the markup out of business logic.
4. Building on #3, delay integrating in data sources, like web methods and local stores, until the highest-level component possible. This is the Container pattern. Say we have a UI construct like a social network timeline. Even though a timeline definitely needs data to be useful, we don't put any code about actually fetching the data in the Timeline component itself. We make a Timeline that renders posts based on a prop containing posts. Then we have a TimelineContainer that fetches the data and passes it to the Timeline. I think I did a not-awful job of this in routes/timeline.

#### Examples
* All components/ layouts/ routes have markup and styles in separate files.
* Container pattern demonstrated in routes/timeline folder.
* Very primitive common styling demonstrated on components/button. components/button's styles file gets the color of the button from config/common-styles.js. You'll see the button is yellow in the Settings view.

#### Other Details
* The Timeline is using a ListView (which does UITableViewController-like stuff in iOS), which requires some state to be maintained. Some folks argue for bringing in the ListView.DataSource from an outside source, but it feels too ListView-specific for me. Nevertheless, I could still pass in a raw array as a prop that could be "cloned" into the DataSource in state, so it's kind of stateless from the consumer's perspective.

#### Stuff To Do (Better)
* More global styling

[1] *I've never actually heard a designer complain about this, only programmers saying that designers will complain about this. I'd like to believe that designers find procedural programming logic embedded in HTML tags to be as awkward as I do.*

### State/ Data/ Services Layer

Credit to [this awesome MobX tutorial](https://mobx.js.org/getting-started.html).
Something else cool [Why one project chose MobX](https://formidable.com/blog/2016/06/02/why-we-chose-mobx-over-redux-for-spectacle-editor/).

I'm sure Redux is great. Lots of folks swear by it for scaling huge projects with lots of developers. I like a well-thought-out state layer as much as anyone, but I found my brain exhausted by Redux, and in particular, thinking about how I would explain it to someone else. My breaking point was that, just as I was starting to understand the Todo app example, I find that a real example with asynchronous loading is a completely different animal. I really wish I could will myself to be more passionate about immutability/ pure functions. Someday I'll be able to sit with the cool kids.

Really all I wanted was something that would a) maintain a global state, and b) automatically force components to react to changes in that state. That's basically all MobX does. It does it using a shared store concept similar to what's often done in native iOS.

The example so far includes the TimelineContainer and DetailContainer reading messages from the stores/ChatStore.js. In particular TimelineContainer is actually reacting to changes (though there are no real changes happening yet), by observing the ChatStore using ES.next decorators (.e.g, `@observer`). There's a case to be made that a library isn't even necessary yet. Take away the decorators, and ChatStore is just a plain old singleton that could be used anywhere (TimelineContainer would have to manually refresh itself somehow in such a case). The biggest win is that TimelineContainer can read from the same place for both an entire timeline and a single message without having to be aware of each other.

#### Stuff To Do (Better)
* Demonstrate changing data (will do this with the compose view).
* Better strategy for referencing single records in the observable (list) variable in a store. For instance, indexing with dictionaries may make sense.
* Wire up mock web services. Interested in finding a best practice for dependency injection of web request components into the store, so the store can be unit-tested. I'm too used to Angular requiring this ;-).

### Navigation Containers

Two types of navigation containers are demonstrated- StackNavigator, and TabNavigator. The tab navigation is laid out in layouts/main, while each tab's stack navigation and tab-specific properties are in layouts/timeline and layouts/settings. It really does kind of feel like an all-text version of an iOS storyboard. Timeline demonstrates sending messages to navigation in order to push a new view, showing an individual post when you tap on a post or showing a compose screen when tapping on the compose button in the Timeline header.

One recently-added nuance- adorning container components with navigation-specific properties in the layout component. See layouts/timeline. This seems to make sense for keeping the layout stuff with the layout stuff.

#### Stuff To Do (Better)
* Tab bar icons can be arbitrary React views. This can facilitate stuff like a number badge on a tab. I have an example of this elsewhere and would like to add it here. Actually, NativeBase also has this and may be better to learn how to use theirs.
* Show Compose view as a modal instead of putting it onto the stack.

### Libraries with Native Linking

The tab bar icons use react-native-vector-icons. Among other things, these require .ttf files to actually be included in the iOS/ Android native projects. `react-native link` actually does this for us. It was failing for about a week because the latest version of the library had issues with a tvOS build scheme or something like that. It seems inevitable that eventually libraries with more complex native integration will need to be added. Since there's reasons to recreate the iOS/ Android projects (e.g., new version of XCode), I think it's important to stick with stuff that reliably works with `react-native link`.

### iOS/ Android-specific UI

This is accomplished simply with the .ios/ .android sub-extensions (is that a term?). So, Button.android.js is used for Android and Button.ios.js is used for iOS. This is working now with components/button. This control can be seen on the Settings tab.

Once I finally ran this on Android, I noticed how grossly my UI violated [Material Design guidelines for tab bars](https://material.io/guidelines/components/tabs.html). Usually an Android tab bar is a sub category below some stack-like navigation, rather than the tab being king like it is in iOS. In Android, the tabs were on top, with the title bar for the stack navigation direcly underneath. It didn't look good. Even moving the tab bar to the bottom for Android didn't help much. Still confusing and cluttered.

I'd still like to figure out how to make tab navigation work well across both platforms (seems like an obvious scenario). For the time being, I used this opportunity to provide different navigation that looks better in Android. In layouts/main, now there are separate Main.platform.js files, and I made some changes to TimelineTab and SettingsTab. Android now uses drawer navigation, while iOS uses a tab bar.

One ugly thing at the moment about this is that there is code in each -Tab file for adding the drawer menu button to the title bar. It's not as easy to componentize these things, because there's just one title bar, but there must be some way to encapsulate things such that, if you add a component to a DrawerNavigator, it automatically gets a drawer open button.

An ugly bit on the iOS side is that the compose window doesn't appear as a modal. Ideally, it would slide from the bottom and not have back button. It seems that one could do such a thing with [this workaround](https://github.com/react-community/react-navigation/issues/707).

### Third-party control libraries

There's tons of stuff in in [NativeBase](https://nativebase.io/docs/v0.5.7/guide/what-is-nativebase). So far, I'm just using their Button for the Timeline right header bar button. Zero-fuss native look-and-feel for a bar button.

### Linting

Whoa I just got brutalized by the linter. Tons of stuff learned in the span of 5 minutes fixing like 2% of my errors.

Check out React Native Linting setup recommendations at: [Linting for React Native](https://medium.com/pvtl/linting-for-react-native-bdbb586ff694). These use (AirBnB's pretty exhaustive JS and React standards)[https://github.com/airbnb/javascript].

Quick hits:
* Apparently if there's only one export from a file I should make it a default. I kind of don't like this because it breaks importing files once I add the second export, but I probably need to eat my brussels sprouts here.
* Got burned on using PropTypes.object. They want you to use PropTypes.shape() to explicitly define the type of object to expect. Check the [bottom of this thread](https://github.com/yannickcr/eslint-plugin-react/issues/904) for recommendations when you can't possibly know the structure of the object because it's not your's to define. Note to self: look up how to define my own prop types to avoid redundant shape() calls. UPDATE: I overrode this rule because I can't find the PropType for stuff like the "navigation" prop.
* I setup [ESLint in Visual Studio Code](http://shripalsoni.com/blog/configure-eslint-in-visual-studio-code/). Only automatically lints the open file without using a task runner, but that in and of itself is pretty nice.

### Unit Testing

[Snapshot-based testing with Jest](https://facebook.github.io/jest/blog/2016/07/27/jest-14.html)

[Testing with shallow rendering](http://airbnb.io/enzyme/docs/api/shallow.html)

More here soon. I started down the snapshot-based testing, got one to work (may have broken it since then). It looks great, pretty easy to work with. The part I worry about is, say, changing a button and having that invalidate the snapshot of everything that used a button. Therefore, I wonder if shallow rendering (which doesn't render sub-components) could work with snapshotting. Each test would take a snapshot of a shallow render, failing if the code has changed the shallow render since the snapshot. That would avoid the issue of invalidating many tests with a single component change.

### Package management

I added Yarn. Not looking back.

### More decisions to make

#### PropTypes vs Flow
See [this](https://stackoverflow.com/questions/36065185/react-proptypes-vs-flow)

### Other Fun Stuff

* Check app/index.js for warning suppressions due to react-navigation using deprecated libraries. Stuff is churning so quick that this seems to inevitably happen.
* Scores of UNMET PEER DEPENDENCY NPM errors. Seems like NativeBase wants older libraries for a few things. I don't want to roll stuff back because so far it's working well. Other folks at least appear to be in the same boat.
