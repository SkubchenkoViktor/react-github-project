# React + GitHub API project

In this project, we're going to take a small, existing React application and add new features to it.

Here's what the application will look like once you are done:

![react github project](http://i.imgur.com/cSckwUo.gif)

The code you are given for the project implements the search form and the loading of basic user info. You'll have to do all the rest.

Let's take a look at the code that's already there. Many of the starter files should already be familiar to you if you completed [the previous workshop](https://github.com/ziad-saab/react-intro-workshop).

* `server.js`: Tiny Express app that will serve our application files
* `webpack.config.js`: Configuration for bundling our code files to `app-bundle.js`
* `package.json`: Configuration file for NPM, contains dependencies and project metadata
* `.gitignore`: Files that should be ignored by Git. `node_modules` can always be regenerated, and so can `app-bundle.js`.
* `src/index.html`: File that gets served by Express to load our app
* `src/js/app.js`: This file is the entry point for Webpack. It puts our app on the screen.
* `src/js/app-bundle.js`: Never need to touch this file. It gets generated by Webpack and is loaded by the browser.
* `src/js/app-bundle.js.map`: Contains source mapping info for the browser's Developer Tools
* `src/js/components/*`: All the components of our application.
* `src/css/app.css`: The styles for our app. Check it out to see how your starter app is being styled, and add to it to complete the project.

In `app.js` we have the following route structure:

```javascript
<Route path="/" component={App}>
  <IndexRoute component={Search}/>
  <Route path="user/:username" component={User}/>
</Route>
```

The top route says to load the `App` component. Looking at the code of `App.jsx`, you'll see that its `render` method outputs `{this.props.children}`. If the URL happens to be only `/`, then React Router will render an `<App/>` instance, and will pass it a `<Search/>` as its child. If the route happens to be `/user/:username`, React Router will display `<App/>` but will pass it `<User />` as a child.

When the `Search` component is displayed, it has a form and a button. When the form is submitted, we use React Router's `browserHistory` to **programmatically change the URL**. Look at the `Search`'s `_handleSubmit` method to see how that happens.

Once we navigate to the new URL, React Router will render a `User` component. Looking at the `componentDidMount` method of the `User`, you'll see that it does an AJAX call using `this.props.params.username`. The reason why it has access to this prop is because the Router passed it when it mounted the component.

In the `render` method of the `User` component, we are displaying the user info, and we have three links that don't lead anywhere for the moment:

![links](http://i.imgur.com/3CFG1ir.png)

If you click on followers, notice that the URL of the page changes to `/users/:username/followers`. If you have your dev tools open, React Router will give you an error message telling you that this route does not exist.

**The goal of this workshop** is to implement the three links above. To do this, we'll start by implementing the followers page together with step by step instructions. Then, your job will be to implement the two remaining screens and fix any bugs.

## Implementing the Followers page
When clicking on the followers link in the UI, notice that the URL changes to `/user/:username/followers`. Currently this results in a "not found" route. Let's fix this.

![followers page](http://i.imgur.com/IwkBOUc.png)

### Step 1: adding the route
In `app.js`, you currently have your user route setup like this:

```javascript
<Route path="user/:username" component={User} />
```

Let's change it to a route with a nested route

```javascript
<Route path="user/:username" component={User}>
  <Route path="followers" component={Followers} />
</Route>
```

For this to do anything, we first have to implement the `Followers` component.

### Step 2: adding the `Followers` component
Create a component called `Followers`. Since this component is also a route component, it will receive the same `this.props.params.username`. In this component, we're eventually going to do an AJAX call to grab the followers of the user.

For the moment, create the component only with a `render` function. In there, use your props to return the following:

```html
<div className="followers-page">
  <h3>Followers of USERNAME</h3>
</div>
```

### Step 3: displaying the nested component inside its parent
When the URL changes to `followers`, we want to display the followers alongside the current `User` component. This is why we are nesting the followers route inside user.

To reflect this nesting in our tree of components, we have to add a `{this.props.children}` output to our `User` component.

Modify the `User` component to make it display its children just before the closing `</div>` in the `render` method.

When this is done, go back to your browser. Search for a user, and click on FOLLOWERS. The followers component should be displayed below the user info.

### Step 4: loading GitHub data in the `Followers` component:
We want to load the followers of the current user as soon as the `Followers` component is mounted in the DOM. In the `componentDidMount` of `Followers`, use jQuery's `getJSON` to make a request to GitHub's API for the followers. Simply add `/followers` to the GitHub API URL for the user e.g. https://api.github.com/users/ziad-saab/followers

In the callback to your AJAX request, use `setState` to set a `followers` state on your component.

### Step 5: displaying the followers data in the `Followers` component:
Using the `this.state.followers` in your `render` method, display the followers that you receive from GitHub. We'll do this in a few steps.

1. Create a new pure component called `GithubUser`. It should receive a `user` prop, and use its `avatar_url` and `login` properties to display one GitHub user. The whole display should link back to that user's page in your app, using React Router's `Link` component. Here's what a sample output of your `GithubUser` component should look like:

```javascript
<Link to="/user/ziad-saab">
  <img src="AVATAR URL"/>
  ziad-saab
</Link>
```

And here's a visual example of four `GithubUser` instances (remember to use `vertical-align` in your CSS to align the image and the name):

![GithubUser component](http://i.imgur.com/dWp7NIc.png)

2. In `Followers`, use require to import your `GithubUser` component.
3. In the `render` method of `Followers`, use `map` to take the array at `this.state.followers`, and map it to an array of `<GithubUser />` elements, passing the `user` prop. The code of `Followers`' `render` method should look like this:

```javascript
function render() {
  if (!this.state.followers) {
    return <div>LOADING FOLLOWERS...</div>
  }
    
  return (
    <div className="followers-page">
        <h2>Followers of {this.props.params.username}</h2>
        <ul>
            {this.state.followers.map(/* INSERT CODE HERE TO RETURN A NEW <GithubUser/> */)}
        </ul>
    </div>
  );
}
```

Having done this, you should have a full `Followers` component ready to go.

### Step 6: :warning: A wild bug has appeared!
Try to click on a follower in the followers list. Notice that the URL changes to match the user you clicked, but the display does not change to reflect that. [We had the same problem in the previous workshop](https://github.com/ziad-saab/react-intro-workshop#advanced-inter-component-communication). If you recall, it was due to us fetching the data in `componentDidMount`, but sometimes a component's props change while it's still mounted.

Here's what's happening in this case:

1. User is on `/` and does a search for "gaearon"
2. User gets redirected to `/user/gaearon` and React Router **mounts** an instance of the `User` component, passing it "gaearon" as `this.props.params.username`. The `User` component's `componentDidMount` method kicks in and fetches data with AJAX
3. User clicks on FOLLOWERS, gets redirected to `/users/gaearon/followers`. React Router keeps the instance of `User` mounted, and passes it a new instance of `Followers` as `this.props.children`. The `Followers` instance is mounted and its `componentDidMount` kicks in, fetching the followers data.
4. User clicks on one follower called "alexkuz" and the URL changes to `/users/alexkuz`. React Router **does not mount** a new `User` instance. Instead, it changes the `params` prop of the existing `User` instance to make it `{username: "alexkuz"}`.
5. Since `componentDidMount` of `User` is not called, no AJAX call occurs.

To fix this bug, follow the same instructions you did in yesterday's workshop:

1. Move the logic from `componentDidMount` to another method called `fetchData`
2. Call `fetchData` from `componentDidMount`
3. Implement `componentDidUpdate` and call `fetchData` again but **conditionally**, only if the `username` prop has changed.

:warning: `componentDidUpdate` gets called **frequently**, whether the props or the state changed. That's why it's important to always check the new vs. old state/props before calling `setState` again.

## Implementing the following page
Implementing the following page is an exact copy of the followers page. The only differences are:

1. Use `/following` instead of `/followers` in your AJAX call
2. The title of the page and its URL will be different

When displaying the following list, note that you can -- and *should* -- reuse the same `GithubUser` presentational component.

![following page](http://i.imgur.com/1bFxwc7.png)

## Implementing the repos page
Implementing the repos page is similar to the other two pages you implemented. The only differences are:

1. Use `/repos` in your AJAX call
2. Title and URL are different
3. Instead of using a `<Link>` element to link to the repo, use a regular `<a href>` since you're linking to an external resource.
4. You'll need a new `GithubRepo` component that will act similar to the `GithubUser` component you used to display the followers/following.

![repos page](http://i.imgur.com/kxvnCun.png)

When you finish everything, your end-result should look and behave like this:

![react github project](http://i.imgur.com/cSckwUo.gif)
