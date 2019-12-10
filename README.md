# Hammer Tutorial

Welcome to Hammer! If you haven't yet, check out the [Hammer README](https://github.com/hammerframework/hammer/blob/master/README.md) to get a little background on why we created Hammer and the problems it's meant to solve. Hammer brings several existing technologies together for the first time into what we think is the future of single page applications.

In this tutorial we're going to build a blog engine. In reality a blog is probably not the ideal candidate for a Hammer app: blog articles can be stored in a CMS and statically generated to HTML files and served as flat files from a CDN. But as most developers are familiar with a blog and it uses all of the features we want to demonstrate, we decided to build one anyway.

## Prerequisites

This tutorial assumes you are already familiar with a few core concepts:

- [React](https://reactjs.org/)
- [GraphQL](https://graphql.org/)
- [The JAMstack](https://jamstack.org/)

You could work through this tutorial without knowing anything about these technologies but you may find yourself getting lost in terminology that we don't stop and take the time to explain. It also helps knowing where the line is between what is built into React and what additional features Hammer brings to the table.

## Installation & Starting Development

We have a set of command line tools that aids in creating various parts of your Hammer app. Install with either `node` or `yarn` (the rest of this guide assumes Yarn):

    yarn global add @hammerframework/hammer-cli

Note that we install `hammer-cli` globally so that we can create apps from scratch rather than creating a directory and adding it to just that project. Now you can use the Hammer command line tools everywhere. Let's use it right now to create the basic structure of our app:

    hammer new hammerblog

After some commands fly by you'll have a new directory `hammerblog` containing several directories and files. Change to that directory and let's start the development server:

    cd hammerblog
    hammer dev

Your browser should open to http://localhost:8910 show the Hammer welcome page:

[screenshot]

We've already initialized a git repo for you so you may want to save the current state of the app as your first commit.

    git add .
    git commit -am 'First commit'

## Hammer File Structure

Let's take a look at the files and directories that were created for us (config files have been excluded for now):

<img src="https://user-images.githubusercontent.com/300/70482979-05a2d800-1a9c-11ea-9a38-b3ead0f3b5f1.png" alt="hammerblog directory structure" style="width: 300px">

At the top level we have two directories, `api` and `web`. Hammer separates the backend (`api`) and frontend (`web`) concerns into their own paths in the codebase. Yarn refers to these as "workspaces". When you add packages going forward you'll need to specify which workspace they should go in. For example (don't run these commands, we're just looking at the syntax):

    yarn workspace web add marked
    yarn workspace api add better-fs

Within `api` there are two directories:

- `prisma` will contain our database schema (`schema.prisma`) and migrations (files that track the structure of our database changing over time). If your app doesn't need access to a SQL-like database then you can delete this directory completely and even remove the `prisma2` package.
- `src` contains all other backend code. `api/src` contains three more directories:
  - `functions` will contain any lambda functions your app needs in addition to the `graphql.js` file auto-generated by Hammer. This file is required to use the GraphQL API. If you don't need GraphQL you can remove this function completely (and remove the `apollo` package).
  - `graphql` contains SDL files which define your GraphQL schema and types
  - `services` contain GraphQL resolvers and other business logic

That's it for the backend. Let's take a look at the frontend `web` directory:

- `components` contain your traditional React components as well as Hammer "cells".
- `layouts` contain HTML/components that wrap your content and are shared across pages.
- `pages` contain components and are optionally wrapped inside layouts and are the "landing page" for a given URL (a URL like `/articles/hello-world` will map to one page and `/contact-us` will map to another page)
- `index.html` is the standard React starting point for our app.
- `index.js` contains the bootstraping code to get our Hammer app up and running.
- `Routes.js` contains the route definitions for our app

## Our First Page

Let's give our users something to look at besides the Hammer welcome page. We'll use the `hammer` command line tool to create a page for us:

    hammer page home /

This will do two things:

- create `/web/src/pages/HomePage/HomePage.js`
- add a `<Route>` in `/web/Routes.js` that maps the path `/` to the new _HomePage_ page

In fact this page is already live. If you reload your browser you should see this new page instead of the Hammer welcome page:

![image](https://user-images.githubusercontent.com/300/70484154-d1c9b180-1a9f-11ea-967e-f3af142a5ba4.png)

It's not pretty, but it's a start! Open the page in your editor, change some text and save. Your browser should reload with your new text. Open up `/web/src/Routes.js` and take a look at the route that was created:

    <Route path="/" page={HomePage}>

Try changing the route to something like:

    <Route path="/hello" page={HomePage}>

When the browser reloads you should see the Hammer welcome page again. That's because we abandoned the root URL so Hammer takes over and shows the splash screen again. Change your URL to `http://localhost:8910/hello" and you should see our page again.

Change the route back to `/` before continuing!

## A Second Page and a Link

Let's create an "About" page for our blog so everyone knows about the geniuses behind this achievement. We'll create another page using `hammer`:

    hammer page about /about

http://localhost:8910/about should show our new page. But no one's going to find it by changing the URL so let's add a link from our homepage to the About page and vice versa. We'll start creating a simple header and nav bar at the same time:

```javascript
// /web/src/pages/HomePage/HomePage.js

import { Link, routes } from 'src/lib/HammerRouter'

const HomePage = () => {
  return (
    <header>
      <h1>Hammer Blog</h1>
      <nav>
        <ul>
          <li><Link to={routes.aboutPage()}>About</Link></li>
        </ul>
      </nav>
    </header>
    <main>
      Home
    </main>
  )
}

export default HomePage
```

Let's point out a few things here:

- Hammer loves [Function Components](https://www.robinwieruch.de/react-function-component). We'll make extensive use of [React Hooks](https://reactjs.org/docs/hooks-intro.html) as we go and these are only enabled in functional components. You're free to create class components if you want but we haven't found any cases that functional components + hooks were unable to handle.
- Hammer's `<Link>` tag, in its most basic usage, takes a single `to` attribute. That `to` attribute will point to a _Named Route_. By default the name of the route is name of the page itself, in this case `aboutPage`. Named routes are awesome because if you ever change your route, you only change it `Router.js` and every link using named routes will automatically point to the correct place.

Once we get to the About page we don't have any way to get back so lets add a link there as well:

```javascript
// /web/src/pages/AboutPage/AboutPage.js

import { Link, routes } from 'src/lib/HammerRouter'

const AboutPage = () => {
  return (
    <header>
      <h1>Hammer Blog</h1>
      <nav>
        <ul>
          <li><Link to={routes.aboutPage()}>About</Link></li>
        </ul>
      </nav>
    </header>
    <main>
      <p>
        This site was created to demonstrate my mastery of Hammer:
        Look on my works, ye mighty, and despair!
      </p>
      <Link to={routes.homePage()}>Return home</Link>
    </main>
  )
}

export default AboutPage
```

Great! Try that out in the browser and verify you can get back and forth.

As a world-class developer you probably saw that copy and pasted `<header>` (and even `<main>`) and developed an involuntary facial tick. We feel you. That's why Hammer has a little something called _Layouts_.

## Layouts

One way to solve the `<header>` dilemma would be to create a `<Header>` component and include it in both `HomePage` and `AboutPage`. That works, but you've still copied and pasted, albeit a much shorter block of text. What else can we do?

When you look at these two pages what do they really care about? They have some content they want to display. They really shouldn't have to care what comes "before" (a `<header>`) or "after" (a `<footer>`). That's exactly what layouts do: they wrap your pages in a component that then renders the page as its child:

<img src="https://user-images.githubusercontent.com/300/70486228-dc874500-1aa5-11ea-81d2-eab69eb96ec0.png" alt="Layouts structure diagram" style="width: 300px">

Let's create a component to hold that `<header>`:

    hammer layout blog

That created `/web/src/layouts/BlogLayout/BlogLayout.js`. We're calling this the "blog" layout because we may have other layouts at some point in the future (an "admin" layout, perhaps?).

Copy the `<header>` and opening and closing `<main>` from both `HomePage` and `AboutPage` and add it to the layout instead:

```javascript
// /web/src/layouts/BlogLayout/BlogLayout.js

import { Link, routes } from 'src/lib/HammerRouter'

const BlogLayout = (props) => {
  return (
    <header>
      <h1>Hammer Blog</h1>
      <nav>
        <ul>
          <li><Link to={routes.aboutPage()}>About</Link></li>
        </ul>
      </nav>
    </header>
    <main>
      { props.children }
    </main>
  )
}

export default BlogLayout
```

`props.children` is where the magic will happen. Any page content given to the layout will be rendered here. Back to `HomePage` and `AboutPage`, we add a `<BlogLayout>` wrapper and now they're back to containing only the content they care about:

```javascript
// /web/src/pages/HomePage/HomePage.js
import { Link, routes } from "src/lib/HammerRouter";
import BlogLayout from "src/layouts/BlogLayout";

const HomePage = () => {
  return <BlogLayout>Home</BlogLayout>;
};

export default HomePage;

// /web/src/pages/AboutPage/AboutPage.js
import { Link, routes } from "src/lib/HammerRouter";
import BlogLayout from "src/layouts/BlogLayout";

const AboutPage = () => {
  return (
    <BlogLayout>
      <p>
        This site was created to demonstrate my mastery of Hammer:
        Look on my works, ye mighty, and despair!
      </p>
      <Link to={routes.homePage()}>Return home</Link>
    </BlogLayout>
  );
};

export default AboutPage;
```

Back to the browser and you should see...nothing different. But that's good, it means our layout is working.

## Intermission: Why are things named the way they are?

You may have noticed some duplication in Hammer's file names. Pages live in a directory called `/pages` and also contain `Page` in their name. Same with Layouts. What's the deal?

When you have dozens of files open in your editor it's easy to get lost, especially when you have several files with names that are similar or even the same (they happen to be in different directories). We've found that the extra duplication in the names of files is worth the productivity benefit when scanning through your file list.

This naming convention isn't actually enforced by the framework, but the suffix will be added by any files created by the `hammer` CLI. Feel free to name pages and layouts however you like!

## Getting Dynamic

These two pages are great and all but the real meat and potatoes of a blog are the blog posts. Let's make those next.

For the purposes of our tutorial we're going to get our blog posts from a database.
