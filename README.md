# Astro.build official tutorial: Extend with View Transitions

**View transitions** are a way to control what happens when visitors navigate between pages on your site. Astro’s View Transitions API allows you to add optional navigation features including smooth page transitions and animations, controlling the browser’s history stack of visited pages, and preventing full-page refreshes in order to persist some page elements and state while updating the content displayed.

> GET READY TO...<br>
&bull; Import and add the `<ViewTransitions />` router to a common `head` element<br>
&bull; Add event listeners during the navigation process to trigger `<script>`s when needed<br>
&bull; Add page transition animations using transition directives<br>
&bull; Opt out of client-side routing for an individual page link

#### Prerequisites

You will need **an existing Astro project with a common base layout or `<Head />` component**.

This tutorial uses the [Build a Blog tutorial’s finished project](https://github.com/withastro/blog-tutorial-demo) to demonstrate adding view transitions (client-side routing) to an existing Astro project. You can fork and use that codebase locally, or complete the tutorial in the browser by [editing the blog tutorial’s code in StackBlitz](https://stackblitz.com/github/withastro/blog-tutorial-demo/tree/complete?file=src%2Fpages%2Findex.astro).

You can instead follow these steps with your own Astro project, but you will need to adjust the instructions for your codebase.

We recommend using our sample project to complete this short tutorial first. Then, you can use what you have learned to create view transitions in your own project.

#### Build a Blog Tutorial Code

In the [Build a Blog introductory tutorial](https://docs.astro.build/en/tutorial/0-introduction/), you learned about Astro’s [built-in file-based routing](https://docs.astro.build/en/guides/routing/#static-routes): any `.astro`, `.md`, or `.mdx` file anywhere within the `src/pages/` folder automatically became a new page on your site.

To navigate between these pages, you used the standard HTML `<a>` element. For example, to create a link to your About page, you added `<a href="/about/">About</a>` to your page header. When a visitor to your site clicked that link, the browser refreshed and loaded a new page with entirely new content.

#### Full-page navigation vs client-side routing (SPA mode)

When a browser refreshes and loads a new page, there is no continuity between the old page and the new page. During **client-side routing**, a new page is displayed without a full-page browser refresh.

Client-side routing is a feature of single-page application (SPA) sites, where your entire site or app is “one page” of JavaScript whose content is updated based on visitor interaction.

Because each new page does not require a full browser refresh, client-side routing allows you to control page transitions in several ways. **Persistent elements**, such as a common page header, do not have to be entirely rerendered on the screen. The transition from one page to another can appear much smoother. And, **state can be preserved**, allowing you to transfer values from one page to the next, or even keep a video playing as your visitors navigate pages!

There are times when you will want or need a full-page browser refresh. For example, when a link takes a visitor to a `.pdf` document, you will need the browser to load that new page from the server. Even with view transitions enabled in your Astro project, you will be able to specify how the browser should navigate both by default and on a per-link basis, even opting out of client-side routing entirely.

Read more about [Astro’s view transitions](https://docs.astro.build/en/guides/view-transitions/) in our guide, or dive into the instructions below to extend the blog with view transitions.
`
#### Test your knowledge

1. Adding view transitions to my astro site…

    [] Requires more than 2 lines of code to implement site-wide by default

    [x] Changes the default type of page navigation on a page that contains the `<ViewTransitions />` router in its `<head>`

    [] Does not add back accessibility features normally provided by the browser, such as route announcement and respect for `prefers-reduced-motion`

2. Which is **not** a benefit of Astro’s view transitions?

    [x] Sending some additional JavaScript to the browser for client-side routing

    [] The option to persist individual elements and components as a visitor visits a new page

    [] Controlling which kind of navigation to use on a per-link basis

3. The view transitions router…

    [] Requires me to use a UI framework such as React

    [] Is not ready for production sites

    [x] Includes fallback behavior for browsers that do not yet fully support view transitions

## Extending the blog tutorial with view transitions

The steps below show you how to extend the final product of the Build a Blog tutorial by adding client-side routing to enhance page transitions.

### Upgrade dependencies

1. Upgrade to the latest version of Astro, and upgrade all integrations to their latest versions by running the following commands in your terminal:

```sh
# Upgrade to Astro v4.x
npm install astro@latest

# Example: upgrade the blog tutorial Preact integration
npm install @astrojs/preact@latest
```

> **Tip**<br>
If you are using your own project, then be sure to update any dependencies you have installed. The example blog tutorial codebase only uses the Preact integration.


### Add the `<ViewTransitions />` router

2. Import and add the `<ViewTransitions />` component to the `<head>` of your page layout.

In the Blog tutorial example, the `<head>` element is found in `src/layouts/BaseLayout.astro`. The `ViewTransitions` router must be first imported into the component’s frontmatter. Then, add the routing component inside the `<head>` element.

```jsx
---
import { ViewTransitions } from "astro:transitions";
import Header from "../components/Header.astro";
import Footer from "../components/Footer.astro";
import "../styles/global.css";
const { pageTitle } = Astro.props;
---
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width" />
    <meta name="generator" content={Astro.generator} />
    <title>{pageTitle}</title>
    <ViewTransitions />
  </head>
  <body>
    <Header />
    <h1>{pageTitle}</h1>
    <slot />
    <Footer />
    <script>
      import "../scripts/menu.js";
    </script>
  </body>
</html>
```

No other configuration is necessary to enable Astro’s default client-side navigation! Astro will create default page animations based on the similarities between the old and new page, and will also provide fallback behavior for unsupported browsers.

3. Navigate between pages in your site preview.

View your site preview at a large screen size, such as desktop mode. As you move between pages on your site, notice that the old page content appears to fade out as the new page content fades in. Use the view transitions guide to add custom behavior if you are not satisfied with the defaults.

View your site preview at a smaller screen size, and try to use the hamburger menu to navigate between pages. Notice that your menu will no longer work after the first page load.

### Update scripts

With view transitions, some scripts may no longer re-run after page navigation like they do with full-page browser refreshes. There are several [events during client-side routing that you can listen for](https://docs.astro.build/en/guides/view-transitions/#lifecycle-events), and fire events when they occur. The scripts in your project will now need to hook into two events to run at the right time during page navigation: `astro:page-load` and `astro:after-swap`.

4. Make the script controlling the `<Hamburger />` mobile menu component available after navigating to a new page.

To make your mobile menu interactive after navigating to a new page, add the following code that listens for the `astro:page-load` event which runs at the end of page navigation, and in response, runs the existing script to make the hamburger menu function when clicked:

```jsx
document.addEventListener('astro:page-load', () => {
  document.querySelector('.hamburger').addEventListener('click', () => {
    document.querySelector('.nav-links').classList.toggle('expanded');
  });
});
```

5. Make the script controlling the theme toggle available after page navigation (in `src/components/ThemeIcon.astro`).

The `<script>` that controls the light/dark theme toggle is located in the `<ThemeIcon />` component. For the theme toggle to continue to function on every page, remove the `is:inline` attribute from the script and add the same event listener as in the previous example so that `astro:page-load` event can trigger your existing function.

Update the existing script tag so that your function runs in response to the `astro:page-load` event, making your theme toggle interactive after the new page is fully loaded and visible to the user:

```jsx
---

---

<button id="themeToggle">
  <svg width="30px" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
    <path
      class="sun"
      fill-rule="evenodd"
      d="M12 17.5a5.5 5.5 0 1 0 0-11 5.5 5.5 0 0 0 0 11zm0 1.5a7 7 0 1 0 0-14 7 7 0 0 0 0 14zm12-7a.8.8 0 0 1-.8.8h-2.4a.8.8 0 0 1 0-1.6h2.4a.8.8 0 0 1 .8.8zM4 12a.8.8 0 0 1-.8.8H.8a.8.8 0 0 1 0-1.6h2.5a.8.8 0 0 1 .8.8zm16.5-8.5a.8.8 0 0 1 0 1l-1.8 1.8a.8.8 0 0 1-1-1l1.7-1.8a.8.8 0 0 1 1 0zM6.3 17.7a.8.8 0 0 1 0 1l-1.7 1.8a.8.8 0 1 1-1-1l1.7-1.8a.8.8 0 0 1 1 0zM12 0a.8.8 0 0 1 .8.8v2.5a.8.8 0 0 1-1.6 0V.8A.8.8 0 0 1 12 0zm0 20a.8.8 0 0 1 .8.8v2.4a.8.8 0 0 1-1.6 0v-2.4a.8.8 0 0 1 .8-.8zM3.5 3.5a.8.8 0 0 1 1 0l1.8 1.8a.8.8 0 1 1-1 1L3.5 4.6a.8.8 0 0 1 0-1zm14.2 14.2a.8.8 0 0 1 1 0l1.8 1.7a.8.8 0 0 1-1 1l-1.8-1.7a.8.8 0 0 1 0-1z"
    ></path>
    <path
      class="moon"
      fill-rule="evenodd"
      d="M16.5 6A10.5 10.5 0 0 1 4.7 16.4 8.5 8.5 0 1 0 16.4 4.7l.1 1.3zm-1.7-2a9 9 0 0 1 .2 2 9 9 0 0 1-11 8.8 9.4 9.4 0 0 1-.8-.3c-.4 0-.8.3-.7.7a10 10 0 0 0 .3.8 10 10 0 0 0 9.2 6 10 10 0 0 0 4-19.2 9.7 9.7 0 0 0-.9-.3c-.3-.1-.7.3-.6.7a9 9 0 0 1 .3.8z"
    ></path>
  </svg>
</button>

<style>
  #themeToggle {
    border: 0;
    background: none;
  }

  .sun {
    fill: black;
  }
  .moon {
    fill: transparent;
  }

  :global(.dark) .sun {
    fill: transparent;
  }
  :global(.dark) .moon {
    fill: white;
  }
</style>

<script>
  document.addEventListener("astro:page-load", () => {
    const theme = (() => {
      if (
        typeof localStorage !== "undefined" &&
        localStorage.getItem("theme")
      ) {
        return localStorage.getItem("theme");
      }
      if (window.matchMedia("(prefers-color-scheme: dark)").matches) {
        return "dark";
      }
      return "light";
    })();

    if (theme === "light") {
      document.documentElement.classList.remove("dark");
    } else {
      document.documentElement.classList.add("dark");
    }

    window.localStorage.setItem("theme", theme);

    const handleToggleClick = () => {
      const element = document.documentElement;
      element.classList.toggle("dark");

      const isDark = element.classList.contains("dark");
      localStorage.setItem("theme", isDark ? "dark" : "light");
    };

    document
      .getElementById("themeToggle")
      .addEventListener("click", handleToggleClick);
  });
</script>
```

Now, the theme toggle is interactive on every page when using the `<ViewTransitions />` router, after the page has finished loading.

6. Check for theme earlier to prevent flashing in dark mode.

The theme toggle works on every page, but its script is loaded at the end of the navigation process, **after the new page has fully loaded in the browser**. There may be a flash of the light theme version of the site before this theme toggle script runs and checks which theme it should use on the page.

To check for, and if necessary set, dark mode earlier in the navigation process, create a function that will run in response to the `astro:after-swap` event. The following function to check the browser’s `localStorage` for dark theme will run **immediately after the new page has replaced the old page**, before the DOM elements are painted to the screen.

Add this new script to the `<ThemeIcon />` component, in addition to the script that controls the theme toggle (in `src/components/ThemeIcon.astro`).

```jsx
---

---

<button id="themeToggle">
  <svg width="30px" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
    <path
      class="sun"
      fill-rule="evenodd"
      d="M12 17.5a5.5 5.5 0 1 0 0-11 5.5 5.5 0 0 0 0 11zm0 1.5a7 7 0 1 0 0-14 7 7 0 0 0 0 14zm12-7a.8.8 0 0 1-.8.8h-2.4a.8.8 0 0 1 0-1.6h2.4a.8.8 0 0 1 .8.8zM4 12a.8.8 0 0 1-.8.8H.8a.8.8 0 0 1 0-1.6h2.5a.8.8 0 0 1 .8.8zm16.5-8.5a.8.8 0 0 1 0 1l-1.8 1.8a.8.8 0 0 1-1-1l1.7-1.8a.8.8 0 0 1 1 0zM6.3 17.7a.8.8 0 0 1 0 1l-1.7 1.8a.8.8 0 1 1-1-1l1.7-1.8a.8.8 0 0 1 1 0zM12 0a.8.8 0 0 1 .8.8v2.5a.8.8 0 0 1-1.6 0V.8A.8.8 0 0 1 12 0zm0 20a.8.8 0 0 1 .8.8v2.4a.8.8 0 0 1-1.6 0v-2.4a.8.8 0 0 1 .8-.8zM3.5 3.5a.8.8 0 0 1 1 0l1.8 1.8a.8.8 0 1 1-1 1L3.5 4.6a.8.8 0 0 1 0-1zm14.2 14.2a.8.8 0 0 1 1 0l1.8 1.7a.8.8 0 0 1-1 1l-1.8-1.7a.8.8 0 0 1 0-1z"
    ></path>
    <path
      class="moon"
      fill-rule="evenodd"
      d="M16.5 6A10.5 10.5 0 0 1 4.7 16.4 8.5 8.5 0 1 0 16.4 4.7l.1 1.3zm-1.7-2a9 9 0 0 1 .2 2 9 9 0 0 1-11 8.8 9.4 9.4 0 0 1-.8-.3c-.4 0-.8.3-.7.7a10 10 0 0 0 .3.8 10 10 0 0 0 9.2 6 10 10 0 0 0 4-19.2 9.7 9.7 0 0 0-.9-.3c-.3-.1-.7.3-.6.7a9 9 0 0 1 .3.8z"
    ></path>
  </svg>
</button>

<style>
  #themeToggle {
    border: 0;
    background: none;
  }

  .sun {
    fill: black;
  }
  .moon {
    fill: transparent;
  }

  :global(.dark) .sun {
    fill: transparent;
  }
  :global(.dark) .moon {
    fill: white;
  }
</style>

<script>
  document.addEventListener("astro:page-load", () => {
    const theme = (() => {
      if (
        typeof localStorage !== "undefined" &&
        localStorage.getItem("theme")
      ) {
        return localStorage.getItem("theme");
      }
      if (window.matchMedia("(prefers-color-scheme: dark)").matches) {
        return "dark";
      }
      return "light";
    })();

    if (theme === "light") {
      document.documentElement.classList.remove("dark");
    } else {
      document.documentElement.classList.add("dark");
    }

    window.localStorage.setItem("theme", theme);

    const handleToggleClick = () => {
      const element = document.documentElement;
      element.classList.toggle("dark");

      const isDark = element.classList.contains("dark");
      localStorage.setItem("theme", isDark ? "dark" : "light");
    };

    document
      .getElementById("themeToggle")
      .addEventListener("click", handleToggleClick);
  });
</script>

<script>
  document.addEventListener("astro:after-swap", () => {
    localStorage.theme === "dark"
      ? document.documentElement.classList.add("dark")
      : document.documentElement.classList.remove("dark");
  });
</script>

```

Now, every page change **that uses the `<ViewTransitions />` router for client-side navigation** (and therefore access to the `astro:after-swap` event) will be able to detect `theme: dark` from the browser’s `localStorage` and update the current page accordingly before the page is rendered for the viewer.

#### Test your knowledge

CONTINUAR

---

####

```astro
```

> GET READY TO...<br>
&bull; x<br>
&bull; x<br>
&bull; x

---

**Index**

UNIT 1 - SETUP

1.1. Prepare dev environment

-Terminal
-Node.js
-VS Code

-GitHub account
-Netlify account

1.2. Launch astro setup wizard
1.3. Write your first line of Astro
1.4. Store your repository online 
1.5. Deploy your site to the web

UNIT 2 - PAGES

2.1. Create your first Astro page
2.2. Write your first Markdown blog post
2.3. Add dynamic content about you
2.4. Style your About page
2.5. Add site-wide styling

UNIT 3 - COMPONENTS

3.1. Make a reusable Navigation component
3.2. Create a social media footer
3.3. Build it yourself - Header
3.4. Send your first script to the browser

UNIT 4 - LAYOUTS

4.1. Build your first layout
4.2. Create and pass data to a custom blog layout
4.3. Combine layouts to get the best of both worlds

UNIT 5 - ASTRO API

5.1. Create a blog post archive
5.2. Generate tag pages
5.3. Build a tag index page
5.4. Add an RSS feed

UNIT 6 - ASTRO ISLANDS

6.1. Build your first Astro island
6.2. Back on dry land. Take your blog from day to night, no island required!
6.3. Congratulations!

---

## Unit 1 - Setup: create and deploy your first Astro site

### 1.1. Prepare your dev environment

> GET READY TO...<br>
&bull; Install any tools that you will use to build your Astro website

Get the dev tools you need:

- terminal
- Node.js (version v18.14.1 or later)
- VS Code Editor

### 1.2. Create your first Astro project

> GET READY TO...<br>
&bull; Run the `create astro` setup wizard to create a new Astro project<br>
&bull; Start the Astro server in development (dev) mode<br>
&bull; View a live preview of your website in your browser

#### Launch the Astro setup wizard

1. `npm create astro@latest`
2. "Where would you like to create your new project?": `./astro-tut1-build-a-blog`
3. Templates: `Empty`
4. TypeScript: `n`
5. "Would you like to install dependencies?": `y`
6. "“Would you like to initialize a new git repository": `y`

#### Open your project in VS Code

`code ./astro-tut1-build-a-blog`

#### Run Astro in dev mode (start the dev server)

`npm run dev`

#### View a preview of your website

`http://localhost:4321`

### 1.3. Write your first line of Astro

> GET READY TO...<br>
&bull; Make your first edit to your new website

#### Edit your home page

Replace <s>`<h1>Astro</h1>`</s> with `<h1>My Astro Site</h1>`

</body>
```

### 1.4. Store your repository online

> GET READY TO...<br>
&bull; Put your project repository online

#### Create a repository on GitHub

#### Commit your local code to GitHub

### 1.5. Deploy your site to the web

> GET READY TO...<br>
&bull; Add your GitHub repository as a new Netlify app<br>
&bull; Deploy your Astro site to the web

#### Create a new Netlify site

1. Go to: https://netlify.com/ and make a note of your username: `https://app.netlify.com/teams/knnv-ar/`
2. Click `Add new site > Import an existing project`, then choose your Astro project’s GitHub repository from the list provided.
3. At the final step, Netlify will show you your app’s site settings. The defaults should be correct for your Astro project, so you can scroll down and click `Deploy site`.

#### Change your project name

#### Visit your new website

## Unit 2 - Pages: add, style and link to pages on your site

### 2.1. Create your first Astro page

> GET READY TO...<br>
&bull; Create two new pages on your website: About and Blog<br>
&bull; Add navigation links to your pages<br>
&bull; Deploy an updated version of your website to the web

#### Create a new `.astro` file

1. In `src/pages/`, add `about.astro`
2. Copy, or retype the contents of `index.astro` into your new `about.astro` file.
3. Test `http://localhost:4321/about`

#### Edit your page

Replace:

```html
<body>
  <h1>My Astro Site</h1>
</body>
```

With:

```html
<body>
  <h1>About Me</h1>
  <h2>... and my new Astro site!</h2>

  <p>
    I am working through Astro's introductory tutorial. This is the second page
    on my website, and it's the first one I built myself!
  </p>

  <p>
    This site will update as I complete more of the tutorial, so keep checking
    back and see how my journey is going!
  </p>
</body>
```

#### Add navigation links

Add at the top of pages `index.astro` and `about.astro`:

```html
<a href="/">Home</a> <a href="/about/">About</a>
```

#### Add a blog page

```html
<body>
  <a href="/">Home</a>
  <a href="/about/">About</a>
  <a href="/blog/">Blog</a>

  <h1>My Astro Learning Blog</h1>
  <p>This is where I will post about my journey learning Astro.</p>
</body>
```

#### Publish your changes to the web

### 2.2. Write your first Markdown blog post

> GET READY TO...<br>
&bull; Make a new folder and create a new post<br>
&bull; Write some Markdown content<br>
&bull; Link to your blog posts on your Blog page

#### Create your first .md file

Create `src/pages/posts/post-1-md` and test `http://localhost:4321/posts/post-1`

#### Write Markdown content

```md
---
title: "My First Blog Post"
pubDate: 2022-07-01
description: "This is the first post of my new Astro blog."
author: "Astro Learner"
image:
  url: "https://docs.astro.build/assets/full-logo-light.png"
  alt: "The full Astro logo."
tags: ["astro", "blogging", "learning in public"]
---

# My First Blog Post

Published on: 2022-07-01

Welcome to my _new blog_ about learning Astro! Here, I will share my learning journey as I build a new website.

## What I've accomplished

1. **Installing Astro**: First, I created a new Astro project and set up my online accounts.

2. **Making Pages**: I then learned how to make pages by creating new `.astro` files and placing them in the `src/pages/` folder.

3. **Making Blog Posts**: This is my first blog post! I now have Astro pages and Markdown posts!

## What's next

I will finish the Astro tutorial, and then keep adding more posts. Watch this space for more to come.
```

#### Link to your posts

1. Link to your first post with an anchor tag in `src/pages/blog.astro`:

```html
---
---

<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>Astro</title>
  </head>
  <body>
    <a href="/">Home</a>
    <a href="/about/">About</a>
    <a href="/blog/">Blog</a>

    <h1>My Astro Learning Blog</h1>
    <p>This is where I will post about my journey learning Astro.</p>
    <ul>
      <li><a href="/posts/post-1/">Post 1</a></li>
    </ul>
  </body>
</html>
```

### 2.3. Add dynamic content about you

> GET READY TO...<br>
&bull; Define your page title in frontmatter, and use it in your HTML<br>
&bull; Conditionally display HTML elements<br>
&bull; Add some content about you

#### Define and use a variable

In `src/pages/about.astro` add the variable `pageTitle` between the code fences and use it with curly braces in the body:

```jsx
---
const pageTitle = "About Me";
---

<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width" />
    <meta name="generator" content={Astro.generator} />
    <title>{pageTitle}</title>
  </head>
  <body>
    <a href="/">Home</a>
    <a href="/about/">About</a>
    <a href="/blog/">Blog</a>

    <h1>{pageTitle}</h1>
    <h2>... and my new Astro site!</h2>

    <p>
      I am working through Astro's introductory tutorial. This is the second
      page on my website, and it's the first one I built myself!
    </p>

    <p>
      This site will update as I complete more of the tutorial, so keep checking
      back and see how my journey is going!
    </p>
  </body>
</html>
```

#### Write JavaScript expressions in Astro

1. In `src/pages/about.astro` add the following code between the code fences, below your existing content:

```jsx
const identity = {
  firstName: "Sarah",
  country: "Canada",
  occupation: "Technical Writer",
  hobbies: ["photography", "birdwatching", "baseball"],
};

const skills = ["HTML", "CSS", "JavaScript", "React", "Astro", "Writing Docs"];
```

2. In the same page add the following code in your HTML template, below your existing content:

```jsx
<p>Here are a few facts about me:</p>
<ul>
  <li>My name is {identity.firstName}.</li>
  <li>I live in {identity.country} and I work as a {identity.occupation}.</li>
  {identity.hobbies.length >= 2 &&
    <li>Two of my hobbies are: {identity.hobbies[0]} and {identity.hobbies[1]}</li>
  }
</ul>
<p>My skills are:</p>
<ul>
  {skills.map((skill) => <li>{skill}</li>)}
</ul>
```

#### Conditionally render elements

You can also use your script variables to choose **whether or not** to render individual elements of your HTML `<body>` content.

1. Add the following lines to your frontmatter script to **define variables**, below your existing content:

```js
const happy = true;
const finished = false;
const goal = 3;
```

2. Add the following lines below your existing paragraphs.

```astro
{happy && <p>I am happy to be learning Astro!</p>}

{finished && <p>I finished this tutorial!</p>}

{goal === 3 ? <p>My goal is to finish in 3 days.</p> : <p>My goal is not 3 days.</p>}
```

Then, check the live preview in your browser tab to see what is displayed on the page:

3. Commit your changes to GitHub before moving on.

### 2.4. Style your About page

> GET READY TO...<br>
&bull; Style items on a single page<br>
&bull; Use CSS variables

#### Style an individual page

Using Astro’s own <style></style> tags, you can style items on your page. Adding attributes and directives to these tags gives you even more ways to style.

1. Copy the following `<style>` code and paste it into `src/pages/about.astro`:

```html
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>{pageTitle}</title>
    <style>
      h1 {
        color: purple;
        font-size: 4rem;
      }

      .skill {
        color: green;
        font-weight: bold;
      }
    </style>
  </head>
</html>
```

2. Add the class name `skill` to the generated `<li>` elements on your About page, so you can style them. Your code should now look like this:

```jsx
<p>My skills are:</p>
<ul>
  {skills.map((skill) => <li class="skill">{skill}</li>)}
</ul>
```

3. Add the `skill` class code to your existing style tag:

```css
<style>
  h1 {
    color: purple;
    font-size: 4rem;
  }
  .skill {
    color: green;
    font-weight: bold;
  }
</style>
```

4. Visit your About page in your browser again, and verify, through visual inspection or dev tools, that each item in your list of skills is now green and bold.

#### Use your first CSS variable

The Astro `<style>` tag can also reference any variables from your frontmatter script using the `define:vars={ {...} }` directive. You can **define variables within your code fence**, then **use them as CSS variables in your style tag**.

1. Define a `skillColor` variable by adding it to the frontmatter script of `src/pages/about.astro` like this:

```jsx
---
const pageTitle = "About Me";

const identity = {
  firstName: "Sarah",
  country: "Canada",
  occupation: "Technical Writer",
  hobbies: ["photography", "birdwatching", "baseball"],
};

const skills = ["HTML", "CSS", "JavaScript", "React", "Astro", "Writing Docs"];

const happy = true;
const finished = false;
const goal = 3;

const skillColor = "navy";
---
```

2. Update your existing `<style>` tag below to first define, then use this `skillColor` variable inside double curly braces.

```css
<style define:vars={{skillColor}}>
  h1 {
    color: purple;
    font-size: 4rem;
  }
  .skill {
    color: var(--skillColor);
    font-weight: bold;
  }
</style>
```

3. Check your About page in your browser preview. You should see that the skills are now navy blue, as set by the `skillColor` variable passed to the `define:vars` directive.

### 2.5. Add site-wide styling

> GET READY TO...<br>
&bull; Apply styles globally

#### Add a global stylesheet

1. Create a new file at the location `src/styles/global.css` (You’ll have to create a `styles` folder first.)

2. Copy the following code into your new file, `global.css`:

```css
html {
  background-color: #f1f5f9;
  font-family: sans-serif;
}

body {
  margin: 0 auto;
  width: 100%;
  max-width: 80ch;
  padding: 1rem;
  line-height: 1.5;
}

* {
  box-sizing: border-box;
}

h1 {
  margin: 1rem 0;
  font-size: 2.5rem;
}
```

3. In `about.astro`, add the following import statement to your frontmatter:

```jsx
---
import '../styles/global.css';

const pageTitle = "About Me";

const identity = {
  firstName: "Sarah",
  country: "Canada",
  occupation: "Technical Writer",
  hobbies: ["photography", "birdwatching", "baseball"],
};

const skills = ["HTML", "CSS", "JavaScript", "React", "Astro", "Writing Docs"];

const happy = true;
const finished = false;
const goal = 3;

const skillColor = "navy";
const fontWeight = "bold";
const textCase = "uppercase";
---
```

## 3. Build and design with Astro UI components

## Unit 3 - Components: build and design with Astro UI components

### 3.1. Make a reusable Navigation component

> GET READY TO...<br>
&bull; Create a new folder for components<br>
&bull; Build an Astro component to display your navigation links<br>
&bull; Replace existing HTML with a new, reusable navigation component

#### Create a new `src/components/` folder

To hold .`astro` files that will generate HTML but that will not become new pages on your website, you will need a new folder in your project: `src/components/`.

#### Create a Navigation component

1. Create a new file: `src/components/Navigation.astro`.

2. Copy your links to navigate between pages from the top of any page and paste them into your new file, `Navigation.astro`:

```html
<a href="/">Home</a>
<a href="/about/">About</a>
<a href="/blog/">Blog</a>
```

#### Import and use `Navigation.astro`

1. Go back to `index.astro` and import your new component inside the code fence:

```jsx
---
import Navigation from '../components/Navigation.astro';
---
```

2. Then below, replace the existing navigation HTML link elements with the new navigation component you just imported:

Replace:
```html
<a href="/">Home</a>
<a href="/about/">About</a>
<a href="/blog/">Blog</a>
```

With:
```html
<Navigation />
```

3. Check the preview in your browser and notice that it should look exactly the same… and that’s what you want!

Your site contains the same HTML as it did before. But now, those three lines of code are provided by your `<Navigation />` component.

### 3.2. Create a social media footer

> GET READY TO...<br>
&bull; Create a Footer component<br>
&bull; Create and pass props to a Social Media component

#### Create a Footer Component

1. Create a new file at the location `src/components/Footer.astro`.

2. Copy the following code into your new file, `Footer.astro`:

```jsx
---
const platform = "github";
const username = "withastro";
---

<footer>
  <p>Learn more about my projects on <a href={`https://www.${platform}.com/${username}`}>{platform}</a>!</p>
</footer>
```

#### Import and use Footer.astro



1. Add the following import statement to the frontmatter in each of your three Astro pages (`index.astro`, `about.astro`, and `blog.astro`):

```jsx
import Footer from '../components/Footer.astro';
```

2. Add a new `<Footer />` component in your Astro template on each page, just before the closing `</body>` tag to display your footer at the bottom of the page.

```html
    <Footer />
  </body>
</html>
```

3. In your browser preview, check that you can see your new footer text on each page.

#### Create a Social Media component

Since you might have multiple online accounts you can link to, you can make a single, reusable component and display it multiple times. Each time, you will pass it different properties (`props`) to use: the online platform and your username there.

1. Create a new file at the location `src/components/Social.astro`. 

2. Copy the following code into your new file, `Social.astro`:

```jsx
---
const { platform, username } = Astro.props;
---
<a href={`https://www.${platform.toLowerCase()}.com/${username}`}>{platform}</a>
```

#### Import and use `Social.astro` in your Footer

1. Change the code in `src/components/Footer.astro` to import, then use this new component three times, passing different **component attributes** as props each time:

Replace:

```jsx
const platform = "github";
const username = "withastro";
```

With:

```jsx
import Social from './Social.astro';
```

and replace:

```jsx
<p>Learn more about my projects on <a href={`https://www.${platform}.com/${username}`}>{platform}</a>!</p>
```

With:

```jsx
  <p>
    Learn more about my projects on <Social
      platform="GitHub"
      username="withastro"
    />, <Social platform="Twitter" username="astrodotbuild" /> and <Social
      platform="Youtube"
      username="astrodotbuild"
    />.
  </p>
```

2. Check your browser preview, and you should see your new footer displaying links to these three platforms on each page.

#### Style your Social Media Component

1. Customize the appearance of your links by adding a `<style>` tag to `src/components/Social.astro`.

```jsx
---
const { platform, username } = Astro.props;
---
<a href={`https://www.${platform.toLowerCase()}.com/${username}`}>{platform}</a>

<style>
  a {
    padding: 0.5rem 1rem;
    color: white;
    background-color: #4c1d95;
    text-decoration: none;
  }
</style>
```

2. Add a `<style>` tag to `src/components/Footer.astro` to improve the layout of its contents.

```astro
---
import Social from "./Social.astro";
---

<footer>
  <p>
    Learn more about my projects on <Social
      platform="GitHub"
      username="withastro"
    />, <Social platform="Twitter" username="astrodotbuild" /> and <Social
      platform="Youtube"
      username="astrodotbuild"
    />.
  </p>
</footer>

<style>
  a {
    padding: 0.5rem 1rem;
    color: white;
    background-color: #4c1d95;
    text-decoration: none;
  }
</style>
```

3. Check your browser preview again and confirm that each page shows an updated footer.

### 3.3. Build it yourself - Header

> GET READY TO...<br>
&bull; Create a Header for your site that contains the Navigation component<br>
&bull; Make the Navigation component responsive

#### Try it yourself - Build a new Header component

1. Create a new Header component. Import and use your existing `Navigation.astro` component inside a `<nav>` element which is inside a `<header>` element.


```jsx
---
import Navigation from './Navigation.astro';
---
<header>
  <nav>
    <Navigation />
  </nav>
</header>
```

#### Try it yourself - Update your pages

1. On each page (`index.astro`, `about.astro` and `blog.astro`), replace your existing `<Navigation/>` component with your new header.

Replace: 

```jsx
import Navigation from '../components/Navigation.astro';
```

With:

```jsx
import Header from '../components/Header.astro';
```

And replace:

```jsx
<Navigation />
```

With:

```jsx
<Header />
```

#### Add responsive styles

1. Update `Navigation.astro` with the CSS class to control your navigation links. Wrap the existing navigation links in a ' ' with the class `nav-links`.

```jsx
---
---
<div class="nav-links">
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/blog">Blog</a>
</div>
```

2. Copy the CSS styles below into `global.css`. These styles:

    - Style and position the navigation links for mobile
    - Include an `expanded` class that can be toggled to display or hide the links on mobile
    - Use a `@media` query to define different styles for larger screen sizes

```css
html {
  background-color: #f1f5f9;
  font-family: sans-serif;
}

body {
  margin: 0 auto;
  width: 100%;
  max-width: 80ch;
  padding: 1rem;
  line-height: 1.5;
}

* {
  box-sizing: border-box;
}

h1 {
  margin: 1rem 0;
  font-size: 2.5rem;
}

/* nav styles */

.nav-links {
  width: 100%;
  top: 5rem;
  left: 48px;
  background-color: #ff9776;
  display: unset;
  margin: 0;
}

.nav-links a {
  display: block;
  text-align: center;
  padding: 10px 0;
  text-decoration: none;
  font-size: 1.2rem;
  font-weight: bold;
  text-transform: uppercase;
}

.nav-links a:hover,
.nav-links a:focus {
  background-color: #ff9776;
}

.expanded {
  display: unset;
}

@media screen and (min-width: 636px) {
  .nav-links {
    margin-left: 5em;
    display: block;
    position: static;
    width: auto;
    background: none;
  }

  .nav-links a {
    display: inline-block;
    padding: 15px 20px;
  }

}
```

Resize your window and look for different styles being applied at different screen widths. Your header is now **responsive** to screen size through the use of `@media` queries.

### 3.4. Send your first script to the browser

Let’s add a hamburger menu to open and close your links on mobile screen sizes, requiring some client-side interactivity!

> GET READY TO...<br>
&bull; Create a hamburger menu component<br>
&bull; Write a `<script>` to allow your site visitors to open and close the navigation menu<br>
&bull; Move your JavaScript to its `.js` file

#### Build a Hamburger component

Create a `<Hamburger />` component to open and close your mobile menu.

1. Create a file named `Hamburger.astro` in `src/components/`

2. Copy the following code into your component. This will represent your 3-line “hamburger” menu to open and close your navigation links on mobile. (You will add the new CSS styles to `global.css` later.)

3. Place this new `<Hamburger />` component just before your `<Navigation />` component in `Header.astro`.

```jsx
---
import Navigation from "./Navigation.astro";
import Hamburger from "./Hamburger.astro";
---

<header>
  <nav>
    <Hamburger />
    <Navigation />
  </nav>
</header>
```

4. Add the following styles for your Hamburger component:

```css
/* nav styles */
/* ADD THE FOLLOWING*/
.hamburger {
  padding-right: 20px;
  cursor: pointer;
}

/* ADD THE FOLLOWING*/
.hamburger .line {
  display: block;
  width: 40px;
  height: 5px;
  margin-bottom: 10px;
  background-color: #ff9776;
}

.nav-links {
  width: 100%;
  top: 5rem;
  left: 48px;
  background-color: #ff9776;
  display: none;
  margin: 0;
}

.nav-links a {
  display: block;
  text-align: center;
  padding: 10px 0;
  text-decoration: none;
  font-size: 1.2rem;
  font-weight: bold;
  text-transform: uppercase;
}

.nav-links a:hover, a:focus {
  background-color: #ff9776;
}

.expanded {
  display: unset;
}

@media screen and (min-width: 636px) {
  .nav-links {
    margin-left: 5em;
    display: block;
    position: static;
    width: auto;
    background: none;
  }

  .nav-links a {
    display: inline-block;
    padding: 15px 20px;
  }

  /* ADD THE FOLLOWING*/
  .hamburger {
    display: none;
  }
}
```

#### Write your first script tag

Your header is not yet **interactive** because it can’t respond to user input, like clicking on the hamburger menu to show or hide the navigation links.

Adding a `<script>` tag provides client-side JavaScript to “listen” for a user event and then respond accordingly.

1. Add the following `<script>` tag to `index.astro`, just before the closing `</body>` tag.

```jsx
  <Footer />
  <script>
    document.querySelector('.hamburger').addEventListener('click', () => {
      document.querySelector('.nav-links').classList.toggle('expanded');
    });
  </script>
</body>
```

2. Check your browser preview again at various sizes, and verify that you have a working navigation menu that is both responsive to screen size and responds to user input on this page.

#### Importing a `.js` file

Instead of writing your JavaScript directly on each page, you can move the contents of your `<script>` tag into its own `.js` file in your project.

1. Create `src/scripts/menu.js` (you will have to create a new `/scripts/` folder) and move your JavaScript into it.

```js
document.querySelector('.hamburger').addEventListener('click', () => {
  document.querySelector('.nav-links').classList.toggle('expanded');
});
```

2. Replace the contents of the `<script>` tag on `index.astro` with the following file import:

```jsx
  <Footer />
  <script>
    import "../scripts/menu.js";
  </script>
</body>
```

3. Check your browser preview again at a smaller size and verify that the hamburger menu still opens and closes your navigation links.

4. Add the same `<script>` with import to your other two pages, `about.astro` and `blog.astro` and verify that you have a responsive, interactive header on each page.

```jsx
  <Footer />
  <script>
    import "../scripts/menu.js";
  </script>
</body>
```

> IMPORTANT: **The JavaScript in a `<script>` tag is sent to the browser**, and is available to run, based on user interactions like refreshing a page or toggling an input.

## Unit 4 - Layouts: save time and energy with reusable page layouts

Now that you can build with components, it’s time to create some custom layouts!

### Looking ahead

In this unit, you’ll build layouts to share common elements and styles across your pages and blog posts.

To do this, you will:

- Create reusable layout components
- Pass content to your layouts with `<slot />`
- Pass data from Markdown frontmatter to your layouts
- Nest multiple layouts

### 4.1. Build your first layout

> GET READY TO...<br>
&bull; Refactor common elements into a page layout<br>
&bull; Use an Astro `<slot />` element to place page contents within a layout<br>
&bull; Pass page-specific values as props to its layout

You still have some Astro components repeatedly rendered on every page. It’s time to refactor again to create a shared page layout!

#### Create your first layout component

1. Create a new file at the location `src/layouts/BaseLayout.astro`. (You will need to create a new `layouts` folder first.)

2. Copy the entire contents of `index.astro` into your new file, `BaseLayout.astro`.

```jsx
---
import Header from '../components/Header.astro';
import Footer from '../components/Footer.astro';
import '../styles/global.css';
const pageTitle = "Home Page";
---
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width" />
    <meta name="generator" content={Astro.generator} />
    <title>{pageTitle}</title>
  </head>
  <body>
    <Header />
    <h1>{pageTitle}</h1>
    <Footer />
    <script>
      import "../scripts/menu.js";
    </script>
  </body>
</html>
```

#### Use your layout on a page

3. Replace the code at `src/pages/index.astro` with the following:

```jsx
---
import BaseLayout from '../layouts/BaseLayout.astro';
const pageTitle = "Home Page";
---
<BaseLayout>
  <h2>My awesome blog subtitle</h2>
</BaseLayout>
```

4. Check the browser preview again to notice what did (or, spoiler alert: did not!) change.

5. Add a `<slot />` element to `src/layouts/BaseLayout.astro` just above the footer component, then check the browser preview of your Home page and notice what really did change this time!

```jsx
---
import Header from '../components/Header.astro';
import Footer from '../components/Footer.astro';
import '../styles/global.css';
const pageTitle = "Home Page";
---
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width" />
    <meta name="generator" content={Astro.generator} />
    <title>{pageTitle}</title>
  </head>
  <body>
    <Header />
    <h1>{pageTitle}</h1>
    <slot />
    <Footer />
    <script>
      import "../scripts/menu.js";
    </script>
  </body>
</html>
```

The `<slot />` allows you to inject (or “slot in”) child content written between opening and closing `<Component></Component>` tags to any `Component.astro` file.

#### Pass page-specific values as props

1. Pass the page title to your layout component from `index.astro` using a component attribute:

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
const pageTitle = "Home Page";
---
<BaseLayout pageTitle={pageTitle}>
  <h2>My awesome blog subtitle</h2>
</BaseLayout>
```

2. Change the script of your `BaseLayout.astro` layout component to receive a page title via `Astro.props` instead of defining it as a constant.

Replace:

```jsx
const pageTitle = "Home Page";
```

With:

```jsx
const { pageTitle } = Astro.props;
```

3. Check your browser preview to verify that your page title has not changed. It has the same value, but is now being rendered dynamically. And now, each individual page can specify its own title to the layout.

#### Try it yourself - Use your layout everywhere

**Refactor** your other pages (`blog.astro` and `about.astro`) so that they use your new `<BaseLayout>` component to render the common page elements.

Don’t forget to:

- Pass a page title as props via a component attribute.
- Let the layout be responsible for the HTML rendering of any common elements.
- Delete anything from each page that that page is no longer responsible for rendering, because it is being handled by the layout, including:

  - HTML elements
  - Components and their imports
  - CSS rules in a `<style>` tag (e.g. `<h1>` in your About page)
  - `<script>` tags

### 4.2. Create and pass data to a custom blog layout

> GET READY TO...<br>
&bull; Create a new blog post layout for your Markdown files<br>
&bull; Pass YAML frontmatter values as props to layout component

#### Add a layout to your blog posts

When you include the `layout` frontmatter property in an `.md` file, all of your frontmatter YAML values are available to the layout file.

1. Create a new file at `src/layouts/MarkdownPostLayout.astro`

2. Copy the following code into `MarkdownPostLayout.astro`

```jsx
---
const { frontmatter } = Astro.props;
---
<h1>{frontmatter.title}</h1>
<p>Written by {frontmatter.author}</p>
<slot />
```

3. Add the following frontmatter property in `post-1.md`

```jsx
---
layout: ../../layouts/MarkdownPostLayout.astro
title: 'My First Blog Post'
pubDate: 2022-07-01
description: 'This is the first post of my new Astro blog.'
author: 'Astro Learner'
image:
    url: 'https://docs.astro.build/assets/full-logo-light.png'
    alt: 'The full Astro logo.'
tags: ["astro", "blogging", "learning in public"]
---
```

4. Check your browser preview again at http://localhost:4321/posts/post-1 and notice what the layout has added to your page.

5. Add the same layout property to your two other blog posts `post-2.md` and `post-3.md`. Verify in your browser that your layout is also applied to these posts.

#### Try it yourself - Customize your blog post layout

**Challenge**: Identify items common to every blog post, and use `MarkdownPostLayout.astro` to render them, instead of writing them in your Markdown in `post-1.md` and in every future blog post.

Here’s an example of refactoring your code to include the `pubDate` in the layout component instead of writing it in the body of your Markdown:

Delete in `post-1.md`:

```jsx
Published on: 2022-07-01
```

Add in `src/layouts/MarkdownPostLayout.astro`:

```jsx
---
const { frontmatter } = Astro.props;
---
<h1>{frontmatter.title}</h1>
<p>Published on: {frontmatter.pubDate.toString().slice(0,10)}</p>
<p>Written by {frontmatter.author}</p>
<slot />
```

### 4.3. Combine layouts to get the best of both worlds

Now that you have added a layout to each blog post, it’s time to make your posts look like the rest of the pages on your website!

> GET READY TO...<br>
&bull; Nest your blog post layout inside your main page layout

#### Nest your two layouts

You already have a `BaseLayout.astro` for defining the overall layout of your pages.

`MarkdownPostLayout.astro` gives you some additional templating for common blog post properties such as `title` and `date`, but your blog posts don’t look like the other pages on your site. You can match the look of your blog posts to the rest of your site by `nesting layouts`.

1. In `src/layouts/MarkdownPostLayout.astro`, import `BaseLayout.astro` and use it to wrap the entire template content. Don’t forget to pass the `pageTitle` prop:

```jsx
---
import BaseLayout from './BaseLayout.astro';
const { frontmatter } = Astro.props;
---
<BaseLayout pageTitle={frontmatter.title}>
  <h1>{frontmatter.title}</h1>
  <p>{frontmatter.pubDate.toString().slice(0,10)}</p>
  <p><em>{frontmatter.description}</em></p>
  <p>Written by: {frontmatter.author}</p>
  <img src={frontmatter.image.url} width="300" alt={frontmatter.image.alt} />
  <slot />
</BaseLayout>
```

2. Check your browser **preview at http**://localhost:4321/posts/post-1. Now you should see **content **rendered by**:

  - Your **main page layout**, including your styles, navigation links, and social footer.
  - Your **blog post layout**, including frontmatter properties like the description, date, title, and image.
  - Your **individual blog post Markdown content**, including just the text written in this post.

3. Notice that your page title is now displayed twice, once by each layout.

Remove the line that displays your page title from `MarkdownPostLayout.astro`:

Remove:

```jsx
<h1>{frontmatter.title}</h1>
```

4. Check your browser preview again at http://localhost:4321/posts/post-1 and verify that this line is no longer displayed and that your title is only displayed once. Make any other adjustments necessary to ensure that you do not have any duplicated content.

Make sure that:

- Each blog post shows the same page template, and no content is missing. (If one of your blog posts is missing content, check its frontmatter properties.) 
- No content is duplicated on a page. (If something is being rendered twice, then be sure to remove it from `MarkdownPostLayout.astro`.)

If you’d like to customize your page template, you can.

## Unit 5 - Astro API

Now that you have some blog posts, it’s time to use Astro’s API to work with your files!

#### Looking ahead

In this unit, you’ll supercharge your blog with an index page, tag pages, and an RSS feed.

Along the way, you’ll learn how to use:

- `Astro.glob()` to access data from files in your project
- `getStaticPaths()` to create multiple pages (routes) at once
- The Astro RSS package to create an RSS feed

### 5.1. Create a blog post archive 

Now that you have a few blog posts to link to, it’s time to configure the Blog page to create a list of them automatically!

> GET READY TO...<br>
&bull; Access data from all your posts at once using `Astro.glob()`<br>
&bull; Display a dynamically generated list of posts on your Blog page<br>
&bull; Refactor to use a `<BlogPost />` component for each list item


#### Dynamically display a list of posts

1. Add the following code to `blog.astro` to return information about all your Markdown files. `Astro.glob()` will return an array of objects, one for each blog post.

```jsx
---
import BaseLayout from '../layouts/BaseLayout.astro'
const allPosts = await Astro.glob('../pages/posts/*.md');
const pageTitle = "My Astro Learning Blog";
---
<BaseLayout pageTitle={pageTitle}>
  <p>This is where I will post about my journey learning Astro.</p>
  <ul>
    <li><a href="/posts/post-1/">Post 1</a></li>
    <li><a href="/posts/post-2/">Post 2</a></li>
    <li><a href="/posts/post-3/">Post 3</a></li>
  </ul>
</BaseLayout>
```

2. To generate the entire list of posts dynamically, using the post titles and URLs, replace your individual `<li>` tags with the following Astro code in `src/pages/blog.astro`:

Replace:

```html
  <li><a href="/posts/post-1/">Post 1</a></li>
  <li><a href="/posts/post-2/">Post 2</a></li>
  <li><a href="/posts/post-3/">Post 3</a></li>
```

With:

```jsx
{allPosts.map((post) => <li><a href={post.url}>{post.frontmatter.title}</a></li>)}
```

Your entire list of blog posts is now being generated dynamically, by mapping over the array returned by `Astro.glob()`.

3. Add a new blog post by creating a new `post-4.md` file in `src/pages/posts/` and adding some Markdown content. Be sure to include at least the frontmatter properties used below.

```jsx
---
layout: ../../layouts/MarkdownPostLayout.astro
title: My Fourth Blog Post
author: Astro Learner
description: "This post will show up on its own!"
image:
    url: "https://docs.astro.build/default-og-image.png"
    alt: "The word astro against an illustration of planets and stars."
pubDate: 2022-08-08
tags: ["astro", "successes"]
---
This post should show up with my other blog posts, because `Astro.glob()` is returning a list of all my posts in order to create my list.
```

4. Revisit your blog page in your browser preview at http://localhost:4321/blog and look for an updated list with four items, including your new blog post!

#### Challenge: Create a BlogPost component

Try on your own to make all the necessary changes to your Astro project so that you can instead use the following code to generate your list of blog posts:

Replace:

```jsx
<ul>
  {allPosts.map((post) => <li><a href={post.url}>{post.frontmatter.title}</a></li>)}
</ul>
```

With:

```jsx
<ul>
  {allPosts.map((post) => <BlogPost url={post.url} title={post.frontmatter.title} />)}
</ul>
```

Solution:

src/components/BlogPost.astro:

```jsx
---
const { title, url } = Astro.props
---
<li><a href={url}>{title}</a></li>
```

src/pages/blog.astro

```jsx
---
import BaseLayout from '../layouts/BaseLayout.astro';
import BlogPost from '../components/BlogPost.astro';
const allPosts = await Astro.glob('../pages/posts/*.md');
const pageTitle = "My Astro Learning Blog"
---
<BaseLayout pageTitle={pageTitle}>
  <p>This is where I will post about my journey learning Astro.</p>
  <ul>
    {allPosts.map((post) => <BlogPost url={post.url} title={post.frontmatter.title} />)}
  </ul>
</BaseLayout>
```

### 5.2. Generate tag pages

> GET READY TO...<br>
&bull; Create a page to generate multiple pages<br>
&bull; Specify which page routes to build, and pass each page its own props

#### Dynamic page routing

You can create entire sets of pages dynamically using `.astro` files that export a `getStaticPaths()` function.

#### Create pages dynamically

1. Create a new file at `src/pages/tags/[tag].astro`. (You will have to create a new folder.) Notice that the file name (`[tag].astro`) uses square brackets. Paste the following code into the file:


```jsx
---
import BaseLayout from '../../layouts/BaseLayout.astro';

export async function getStaticPaths() {
  return [
    { params: { tag: "astro" } },
    { params: { tag: "successes" } },
    { params: { tag: "community" } },
    { params: { tag: "blogging" } },
    { params: { tag: "setbacks" } },
    { params: { tag: "learning in public" } },
  ];
}

const { tag } = Astro.params;
---
<BaseLayout pageTitle={tag}>
  <p>Posts tagged with {tag}</p>
</BaseLayout>
```

The `getStaticPaths` function returns an array of page routes, and all of the pages at those routes will use the same template defined in the file.

2. If you have customized your blog posts, then replace the individual tag values (e.g. “astro”, “successes”, “community”, etc.) with the tags used in your own posts.

3. Make sure that every blog post contains at least one tag, written as an array, e.g. `tags: ["blogging"]`.

4. Visit http://localhost:4321/tags/astro in your browser preview and you should see a page, generated dynamically from `[tag].astro`. Check that you also have pages created for each of your tags at `/tags/successes`, `/tags/community`, and `/tags/learning%20in%20public`, etc., or at each of your custom tags. You may need to first quit and restart the dev server to see these new pages.

#### Use props in dynamic routes

1. Add the following props to your `getStaticPaths()` function in order to make data from all your blog posts available to each page route.

Be sure to give each route in your array the new props, and then make those props available to your component template outside of your function.

```jsx
---
import BaseLayout from '../../layouts/BaseLayout.astro';

export async function getStaticPaths() {
  const allPosts = await Astro.glob('../posts/*.md');

  return [
    {params: {tag: "astro"}, props: {posts: allPosts}},
    {params: {tag: "successes"}, props: {posts: allPosts}},
    {params: {tag: "community"}, props: {posts: allPosts}},
    {params: {tag: "blogging"}, props: {posts: allPosts}},
    {params: {tag: "setbacks"}, props: {posts: allPosts}},
    {params: {tag: "learning in public"}, props: {posts: allPosts}}
  ]
}

const { tag } = Astro.params;
const { posts } = Astro.props;
---
```

2. Filter your list of posts to only include posts that contain the page’s own tag.

```jsx
---
const { tag } = Astro.params;
const { posts } = Astro.props;
const filteredPosts = posts.filter((post) => post.frontmatter.tags?.includes(tag));
---
```

3. Now you can update your HTML template to show a list of each blog post containing the page’s own tag. Add the following code to `[tag].astro`:

```jsx
<BaseLayout pageTitle={tag}>
  <p>Posts tagged with {tag}</p>
  <ul>
    {filteredPosts.map((post) => <li><a href={post.url}>{post.frontmatter.title}</a></li>)}
  </ul>
</BaseLayout>
```

4. You can even refactor this to use your `<BlogPost />` component instead! (Don’t forget to import this component at the top of `[tag].astro`.)

```jsx
<BaseLayout pageTitle={tag}>
  <p>Posts tagged with {tag}</p>
  <ul>
    {filteredPosts.map((post) => <BlogPost url={post.url} title={post.frontmatter.title}/>)}
  </ul>
</BaseLayout>
```

5. Check your browser preview for your individual tag pages, and you should now see a list of all of your blog posts containing that particular tag.

> **Takeaway**<br>
-If you need information to construct the page routes, write it **inside** `getStaticPaths()`.<br>
-To receive information in the HTML template of a page route, write it **outside** `getStaticPaths()`.

#### Advanced JavaScript: Generate pages from existing tags

Your tag pages are now defined statically in [tag].astro. If you add a new tag to a blog post, you will also have to revisit this page and update your page routes.

The following example shows how to replace your code on this page with code that will automatically look for, and generate pages for, each tag used on your blog pages.

>**Note**<br>
Even if it looks challenging, you can try following along with the steps to build this function yourself! If you don’t want to walk through the JavaScript required right now, you can skip ahead to the [finished version of the code](https://docs.astro.build/en/tutorial/5-astro-api/2/#final-code-sample) and use it directly in your project, replacing the existing content.

#### 1. Check that all your blog posts contain tags

Revisit each of your existing Markdown pages and ensure that every post contains a `tags` array in its frontmatter. Even if you only have one tag, it should still be written as an array, e.g. `tags: ["blogging"]`.

#### 2. Create an array of all your existing tags

In `src/pages/tags/[tag].astro` add the following code to provide you with a list of every tag used in your blog posts.

```jsx
---
import BaseLayout from '../../layouts/BaseLayout.astro';

export async function getStaticPaths() {
  const allPosts = await Astro.glob('../posts/*.md');

  const uniqueTags = [...new Set(allPosts.map((post) => post.frontmatter.tags).flat())];
```

Tell me what this line of code (the one with `uniqueTags`) is doing in more detail!

It’s OK if this isn’t something you would have written yourself yet!

It goes through each Markdown post, one by one, and combines each array of tags into one single larger array. Then, it makes a new `Set` from all the individual tags it found (to ignore repeated values). Finally, it turns that set into an array (with no duplications), that you can use to show a list of tags on your page.

You now have an array `uniqueTags` with element items `"astro"`, `"successes"`, `"community"`, `"blogging"`, `"setbacks"`, `"learning in public"`

#### 3. Replace the `return` value of the `getStaticPaths` function

Replace:

```jsx
  return [
        {params: {tag: "astro"}, props: {posts: allPosts}},
        {params: {tag: "successes"}, props: {posts: allPosts}},
        {params: {tag: "community"}, props: {posts: allPosts}},
        {params: {tag: "blogging"}, props: {posts: allPosts}},
        {params: {tag: "setbacks"}, props: {posts: allPosts}},
        {params: {tag: "learning in public"}, props: {posts: allPosts}}
      ]
```

With:

```jsx
  return uniqueTags.map((tag) => {
    const filteredPosts = allPosts.filter((post) => post.frontmatter.tags.includes(tag));
    return {
      params: { tag },
      props: { posts: filteredPosts },
    };
  });
```

A `getStaticPaths` function should always return a list of objects containing `params` (what to call each page route) and optionally any `props` (data that you want to pass to those pages). Earlier, you defined each tag name that you knew was used in your blog and passed the entire list of posts as props to each page.

Now, you generate this list of objects automatically using your `uniqueTags` array to define each parameter.

And, now the list of all blog posts is filtered **before** it is sent to each page as props. Be sure to remove the previous line of code filtering the posts, and update your HTML template to use `posts` instead of `filteredPosts`.

Remove:

```jsx
const filteredPosts = posts.filter((post) => post.frontmatter.tags?.includes(tag));
```

And replace:

```jsx
<ul>
    {filteredPosts.map((post) => <BlogPost url={post.url} title={post.frontmatter.title}/>)}    
</ul>
```

With:

```jsx
<ul>
    {posts.map((post) => <BlogPost url={post.url} title={post.frontmatter.title}/>)}
</ul>
```

#### Final code sample

To check your work, or if you just want complete, correct code to copy into `[tag].astro`, here is what your Astro component should look like:

```jsx
---
import BaseLayout from '../../layouts/BaseLayout.astro';
import BlogPost from '../../components/BlogPost.astro';

export async function getStaticPaths() {
  const allPosts = await Astro.glob('../posts/*.md');

  const uniqueTags = [...new Set(allPosts.map((post) => post.frontmatter.tags).flat())];

  return uniqueTags.map((tag) => {
    const filteredPosts = allPosts.filter((post) => post.frontmatter.tags.includes(tag));
    return {
      params: { tag },
      props: { posts: filteredPosts },
    };
  });
}

const { tag } = Astro.params;
const { posts } = Astro.props;
---
<BaseLayout pageTitle={tag}>
  <p>Posts tagged with {tag}</p>
  <ul>
    {posts.map((post) => <BlogPost url={post.url} title={post.frontmatter.title}/>)}
  </ul>
</BaseLayout>
```

Now, you should be able to visit any of your tag pages in your browser preview.

Navigate to http://localhost:4321/tags/community and you should see a list of only your blog posts with the tag community. Similarly http://localhost:4321/tags/learning%20in%20public should display a list of the blog posts tagged `learning in public`.

In the next section, you will create navigation links to these pages.

#### Resources:

- [Dynamic Page Routing in Astro](https://docs.astro.build/en/guides/routing/#dynamic-routes)
- [`getStaticPaths()` API documentation](https://docs.astro.build/en/reference/api-reference/#getstaticpaths)

### 5.3. Build a tag index page

Now that you have individual pages for every tag, it’s time to make links to them.

> GET READY TO...<br>
&bull; Add a new page using the `/pages/folder/index.astro` routing pattern<br>
&bull; Display a list of all your unique tags, linking to each tag page<br>
&bull; Update your site with navigation links to this new Tags page

#### Use the `/pages/folder/index.astro` routing pattern

To add a Tag Index page to your website, you could create a new file at `src/pages/tags.astro`.

But, since you already have the directory `/tags/`, you can take advantage of another routing pattern in Astro, and keep all your files related to tags together.

#### Try it yourself - Make a Tag Index page

1. Create a new file `index.astro` in the directory `src/pages/tags/`.

2. Navigate to http://localhost:4321/tags and verify that your site now contains a page at this URL. It will be empty, but it will exist.

3. Create a minimal page at `src/pages/tags/index.astro` that uses your layout. You have done this before!

```jsx
---
import BaseLayout from "../../layouts/BaseLayout.astro";

const pageTitle = "Tag Index";
---

<BaseLayout pageTitle={pageTitle}>
  <p>This is the tag index content.</p>
</BaseLayout>

```

#### Create an array of tags

You have previously displayed items in a list from an array using `map()`. What would it look like to define an array of all your tags, then display them in a list on this page?

One way to deal with it is to create an array with static tags, but it is not an operative way:

```jsx
---
import BaseLayout from '../../layouts/BaseLayout.astro';
const tags = ['astro', 'blogging', 'learning in public', 'successes', 'setbacks', 'community']
const pageTitle = "Tag Index";
---
<BaseLayout pageTitle={pageTitle}>
  <ul>
    {tags.map((tag) => <li>{tag}</li>)}
  </ul>
</BaseLayout>
```

You could do this, but then you would need to come back to this file and update your array every time you use a new tag in a future blog post.

Fortunately, you already know a way to grab the data from all your Markdown files in one line of code, then return a list of all your tags.

1. In `src/pages/tags/index.astro`, add the `allPost` line of code to the frontmatter script that will give your page access to the data from every `.md` blog post file:

```jsx
---
import BaseLayout from '../../layouts/BaseLayout.astro';
const allPosts = await Astro.glob('../posts/*.md');
const pageTitle = "Tag Index";
---
```

2. Next, add the following line of JavaScript to your page component. This is the same one you used in `src/pages/tags/[tag]`.astro to return a list of unique tags.

```jsx
---
import BaseLayout from '../../layouts/BaseLayout.astro';
const allPosts = await Astro.glob('../posts/*.md');
const tags = [...new Set(allPosts.map((post) => post.frontmatter.tags).flat())];
const pageTitle = "Tag Index";
---
```

#### Create your list of tags

Instead of creating items in an unordered list this time, create one `<p>` for each item, inside a `<div>`. The pattern should look familiar!

```jsx
<BaseLayout pageTitle={pageTitle}>
  <div>{tags.map((tag) => <p>{tag}</p>)}</div>
</BaseLayout>
```

In your browser preview, verify that you can see your tags listed.

2. To make each tag link to its own page, add the following `<a>` link to each tag name:

```jsx
<BaseLayout pageTitle={pageTitle}>
  <div>
    {tags.map((tag) => (
      <p><a href={`/tags/${tag}`}>{tag}</a></p>
    ))}
  </div>
</BaseLayout>
```

#### Add styles to your tag list

1. Add the following CSS classes to style both your `<div>` and each `<p>` that will be generated. Note: Astro uses HTML syntax for adding class names!

```jsx
<BaseLayout pageTitle={pageTitle}>
  <div class="tags">
    {tags.map((tag) => (
      <p class="tag"><a href={`/tags/${tag}`}>{tag}</a></p>
    ))}
  </div>
</BaseLayout>
```

2. Define these new CSS classes by adding the following `<style>` tag to this page:
#### 

```css
<style>
  a {
    color: #00539F;
  }

  .tags {
    display: flex;
    flex-wrap: wrap;
  }

  .tag {
    margin: 0.25em;
    border: dotted 1px #a1a1a1;
    border-radius: .5em;
    padding: .5em 1em;
    font-size: 1.15em;
    background-color: #F8FCFD;
  }
</style>
```

3. Check your browser preview at http://localhost:4321/tags to verify that you have some new styles and that each of the tags on the page has a working link to its own individual tag page.

#### Code Check-In

Here is what your new page should look like:

```jsx
---
import BaseLayout from '../../layouts/BaseLayout.astro';
const allPosts = await Astro.glob('../posts/*.md');
const tags = [...new Set(allPosts.map((post) => post.frontmatter.tags).flat())];
const pageTitle = "Tag Index";
---
<BaseLayout pageTitle={pageTitle}>
  <div class="tags">
    {tags.map((tag) => (
      <p class="tag"><a href={`/tags/${tag}`}>{tag}</a></p>
    ))}
  </div>
</BaseLayout>
<style>
  a {
    color: #00539F;
  }

  .tags {
    display: flex;
    flex-wrap: wrap;
  }

  .tag {
    margin: 0.25em;
    border: dotted 1px #a1a1a1;
    border-radius: .5em;
    padding: .5em 1em;
    font-size: 1.15em;
    background-color: #F8FCFD;
  }
</style>
```

#### Add this page to your navigation

Right now, you can navigate to http://localhost:4321/tags and see this page. From this page, you can click on links to your individual tag pages.

But, you still need to make these pages discoverable from other pages on your website.

1. In your `Navigation.astro` component, include a link to this new tag index page.

```html
<a href="/">Home</a>
<a href="/about/">About</a>
<a href="/blog/">Blog</a>
<a href="/tags/">Tags</a>
```

#### Challenge: Include tags in your blog post layout

You have now written all the code you need to also display a list of tags on each blog post, and link them to their tag pages. You have existing work that you can reuse!

Follow the steps below, then check your work by comparing it to the final code sample.

1. Copy the `<div class="tags">...</div>` and `<style>...</style>` from `src/pages/tags/index.astro` and reuse it inside `MarkdownPostLayout.astro`:

```jsx
---
import BaseLayout from './BaseLayout.astro';
const { frontmatter } = Astro.props;
---
<BaseLayout pageTitle={frontmatter.title}>
  <p><em>{frontmatter.description}</em></p>
  <p>{frontmatter.pubDate.toString().slice(0,10)}</p>

  <p>Written by: {frontmatter.author}</p>

  <img src={frontmatter.image.url} width="300" alt={frontmatter.image.alt} />

  <div class="tags">
    {tags.map((tag) => (
      <p class="tag"><a href={`/tags/${tag}`}>{tag}</a></p>
    ))}
  </div>

  <slot />
</BaseLayout>
<style>
  a {
    color: #00539F;
  }

  .tags {
    display: flex;
    flex-wrap: wrap;
  }

  .tag {
    margin: 0.25em;
    border: dotted 1px #a1a1a1;
    border-radius: .5em;
    padding: .5em 1em;
    font-size: 1.15em;
    background-color: #F8FCFD;
  }
</style>
```

Before this code will work, you need to make **one small edit** to the code you pasted into `MarkdownPostLayout.astro`. Can you figure out what it is?

**Give me a hint**: How are the other props (e.g. title, author, etc.) written in your layout template? How does your layout receive props from an individual blog post?

**Give me another hint!** In order to use props (values passed) from a .md blog post in your layout, like tags, you need to prefix the value with a certain word.

**Show me the code!**

```jsx
    <div class="tags">
      {frontmatter.tags.map((tag) => (
        <p class="tag"><a href={`/tags/${tag}`}>{tag}</a></p>
      ))}
    </div>
```

#### Code Check-in: MarkdownPostLayout

To check your work, or if you just want complete, correct code to copy into `MarkdownPostLayout.astro`, here is what your Astro component should look like:

```jsx
---
import BaseLayout from './BaseLayout.astro';
const { frontmatter } = Astro.props;
---
<BaseLayout pageTitle={frontmatter.title}>
  <p><em>{frontmatter.description}</em></p>
  <p>{frontmatter.pubDate.toString().slice(0,10)}</p>

  <p>Written by: {frontmatter.author}</p>

  <img src={frontmatter.image.url} width="300" alt={frontmatter.image.alt} />

  <div class="tags">
    {frontmatter.tags.map((tag) => (
      <p class="tag"><a href={`/tags/${tag}`}>{tag}</a></p>
    ))}
  </div>

  <slot />
</BaseLayout>
<style>
  a {
    color: #00539F;
  }

  .tags {
    display: flex;
    flex-wrap: wrap;
  }

  .tag {
    margin: 0.25em;
    border: dotted 1px #a1a1a1;
    border-radius: .5em;
    padding: .5em 1em;
    font-size: 1.15em;
    background-color: #F8FCFD;
  }
</style>
```

#### Test your knowledge

Match each file path with a second file path that will create a page at the same route.

1. `src/pages/categories.astro`

    [] `src/pages/posts/post.astro`

    [] `src/pages/posts/index.astro`

    [] `src/components/shoes/Shoe.astro`

    [x] `src/pages/categories/index.astro`


2. `src/pages/posts.astro`

    [] `src/pages/products/shoes.astro`

    [] `src/pages/posts/post.astro`

    [x] `src/pages/posts/index.astro`

    [] `src/pages/categories/index.astro`

3. `src/pages/products/shoes/index.astro`

    [x] `src/pages/products/shoes.astro`
    [] `src/pages/posts/post.astro`
    [] `src/pages/posts/index.astro`
    [] `src/components/shoes/Shoe.astro`

#### Checklist

[x] I can use Astro’s /pages/folder/index.astro routing feature.

#### Resources
[Static Routing in Astro](https://docs.astro.build/en/guides/routing/#static-routes)

### 5.4. Add an RSS feed

> GET READY TO...<br>
&bull; Install an Astro package for creating an RSS feed for your website<br>
&bull; Create a feed that can be subscribed to and read by RSS feed readers

#### Install Astro’s RSS package

Astro provides a custom package to quickly add an RSS feed to your website.

This official package generates a non-HTML document with information about all of your blog posts that can be read by **feed readers** like Feedly, The Old Reader, and more. This document is updated every time your site is rebuilt.

Individuals can subscribe to your feed in a feed reader, and receive a notification when you publish a new blog post on your site, making it a popular blog feature.

1. Quit the Astro development server and run the following command in the terminal to install Astro’s RSS package.

```bash
npm install @astrojs/rss
```

2. Restart the dev server to begin working on your Astro project again.

```bash
npm run dev
```

#### Create an .xml feed document

1. Create a new file in `src/pages/` called `rss.xml.js`

2. Copy the following code into this new document. Customize the `title` and `description` properties, and if necessary, specify a different language in `customData`:

```js
import rss, { pagesGlobToRssItems } from '@astrojs/rss';

export async function GET(context) {
  return rss({
    title: 'Astro Learner | Blog',
    description: 'My journey learning Astro',
    site: context.site,
    items: await pagesGlobToRssItems(import.meta.glob('./**/*.md')),
    customData: `<language>en-us</language>`,
  });
}
```

3. Add the `site` property to the Astro config (`astro.config.mjs`) with your site’s own unique Netlify URL.

```jsx
import { defineConfig } from "astro/config";

export default defineConfig({
  site: "https://astro-tut1-build-a-blog.netlify.app/"
});
```

4. This `rss.xml` document is only created when your site is built, so you won’t be able to see this page in your browser during development. Quit the dev server and run the following commands to first, build your site locally and then, view a preview of your build:

```sh
npm run build

npm run preview
```

5. Visit http://localhost:4321/rss.xml and verify that you can see (unformatted) text on the page with an `item` for each of your `.md` files. Each item should contain blog post information such as `title`, `url`, and `description`.

```xlm
<rss version="2.0">
  <channel>
    <title>Astro Learner | Blog</title>
    <description>My journey learning Astro</description>
    <link>https://astro-tut1-build-a-blog.netlify.app/</link>
    <language>en-us</language>
    <item>
      <title>My First Blog Post</title>
      <link>
        https://astro-tut1-build-a-blog.netlify.app/posts/post-1/
      </link>
      <guid isPermaLink="true">
        https://astro-tut1-build-a-blog.netlify.app/posts/post-1/
      </guid>
      <description>This is the first post of my new Astro blog.</description>
      <pubDate>Fri, 01 Jul 2022 00:00:00 GMT</pubDate>
      <author>Astro Learner</author>
    </item>
    <item>
      <title>My Second Blog Post</title>
      <link>
        https://astro-tut1-build-a-blog.netlify.app/posts/post-2/
      </link>
      <guid isPermaLink="true">
        https://astro-tut1-build-a-blog.netlify.app/posts/post-2/
      </guid>
      <description>After learning some Astro, I couldn't stop!</description>
      <pubDate>Fri, 08 Jul 2022 00:00:00 GMT</pubDate>
      <author>Astro Learner</author>
    </item>
    <item>
      <title>My Third Blog Post</title>
      <link>
        https://astro-tut1-build-a-blog.netlify.app/posts/post-3/
      </link>
      <guid isPermaLink="true">
        https://astro-tut1-build-a-blog.netlify.app/posts/post-3/
      </guid>
      <description>
        I had some challenges, but asking in the community really helped!
      </description>
      <pubDate>Fri, 15 Jul 2022 00:00:00 GMT</pubDate>
      <author>Astro Learner</author>
    </item>
    <item>
      <title>My Fourth Blog Post</title>
      <link>
        https://astro-tut1-build-a-blog.netlify.app/posts/post-4/
      </link>
      <guid isPermaLink="true">
        https://astro-tut1-build-a-blog.netlify.app/posts/post-4/
      </guid>
      <description>This post will show up on its own!</description>
      <pubDate>Mon, 08 Aug 2022 00:00:00 GMT</pubDate>
      <author>Astro Learner</author>
    </item>
  </channel>
</rss>
```

> **View your RSS feed in a reader**<br>
Download a feed reader, or sign up for an online feed reader service and subscribe to your site by adding your own Netlify URL. You can also share this link with others so they can subscribe to your posts, and be notified when a new one is published.

6. Be sure to quit the preview and restart the dev server when you want to view your site in development mode again.

#### Checklist

[x] I can install an Astro package using the command line.
[x] I can create an RSS feed for my website.

#### Resources

[RSS item generation in Astro](https://docs.astro.build/en/guides/rss/#using-glob-imports)

## Unit 6 - Astro Islands

Now that you have a fully functioning blog, it’s time to add some interactive islands to your site!

#### Looking ahead

In this unit, you’ll use **Astro islands** to bring frontend framework components into your Astro site.

You will:

- Add a UI framework, Preact, to your Astro project
- Use Preact to create an interactive greeting component
- Learn when you might not choose islands for interactivity

#### Checklist

[x] I am ready to add some interactivity to my site, and start living that island life!

### 6.1. Build your first Astro island

Use a Preact component to greet your visitors with a randomly-selected welcome message!

> GET READY TO...<br>
&bull; Add Preact to your Astro project<br>
&bull; Include Astro islands (Preact `.jsx` components) on your home page<br>
&bull; Use `client:` directives to make islands interactive

#### Add Preact to your Astro project

1. If it’s running, quit the dev server to have access to the terminal (keyboard shortcut: Ctrl + C).

2. Add the ability to use Preact components in your Astro project with a single command:

```sh
npx astro add preact
```

3. Follow the command line instructions to confirm adding Preact to your project.

#### Include a Preact greeting banner

This component will take an array of greeting messages as a prop and randomly select one of them to show as a welcome message. The user can click a button to get a new random message.

1. Create a new file in `src/components/` named `Greeting.jsx`

    Note the `.jsx` file extension. This file will be written in Preact, not Astro.

2. Add the following code to `Greeting.jsx`:

```jsx
import { useState } from 'preact/hooks';

export default function Greeting({messages}) {

  const randomMessage = () => messages[(Math.floor(Math.random() * messages.length))];

  const [greeting, setGreeting] = useState(messages[0]);

  return (
    <div>
      <h3>{greeting}! Thank you for visiting!</h3>
      <button onClick={() => setGreeting(randomMessage())}>
        New Greeting
      </button>
    </div>
  );
}
```

3. Import and use this component on your Home page `index.astro`.

```jsx
---
import BaseLayout from '../layouts/BaseLayout.astro';
import Greeting from '../components/Greeting';
const pageTitle = "Home Page";
---
<BaseLayout pageTitle={pageTitle}>
  <h2>My awesome blog subtitle</h2>
  <Greeting messages={["Hi", "Hello", "Howdy", "Hey there"]} />
</BaseLayout>
```

Check the preview in your browser: you should see a random greeting, but the button won’t work!

4. Add a second `<Greeting />` component with the `client:load` directive.

```jsx
---
import BaseLayout from '../layouts/BaseLayout.astro';
import Greeting from '../components/Greeting';
const pageTitle = "Home Page";
---
<BaseLayout pageTitle={pageTitle}>
  <h2>My awesome blog subtitle</h2>
  <Greeting messages={["Hi", "Hello", "Howdy", "Hey there"]} />
  <Greeting client:load messages={["Hej", "Hallo", "Hola", "Habari"]} />
</BaseLayout>
```

5. Revisit your page and compare the two components. The second button works because the `client:load` directive tells Astro to send and rerun its JavaScript on the _client_ when the page _loads_, making the component interactive. This is called a **hydrated** component.

6. Once the difference is clear, remove the non-hydrated Greeting component.

```jsx
---
import BaseLayout from '../layouts/BaseLayout.astro';
import Greeting from '../components/Greeting';
const pageTitle = "Home Page";
---
<BaseLayout pageTitle={pageTitle}>
  <h2>My awesome blog subtitle</h2>
  <Greeting messages={["Hi", "Hello", "Howdy", "Hey there"]} />
  <Greeting client:load messages={["Hej", "Hallo", "Hola", "Habari"]} />
</BaseLayout>
```

#### Analyze the Pattern

There are other `client:` directives to explore. Each sends the JavaScript to the client at a different time. `client:visible`, for example, will only send the component’s JavaScript when it is visible on the page.

Consider an Astro component with the following code:

```jsx
---
import BaseLayout from '../layouts/BaseLayout.astro';
import AstroBanner from '../components/AstroBanner.astro';
import PreactBanner from '../components/PreactBanner';
import SvelteCounter from '../components/SvelteCounter.svelte';
---
<BaseLayout>
  <AstroBanner />
  <PreactBanner />
  <PreactBanner client:load />
  <SvelteCounter />
  <SvelteCounter client:visible />
</BaseLayout>
```

1. Which of the five components will be **hydrated** islands, sending JavaScript to the client?

`<PreactBanner client:load />` and `<SvelteCounter client:visible />` will be hydrated islands.

2. In what way(s) will the two `<PreactBanner />` components be the same? In what way(s) will they be different?

**Same**: They both show the same HTML elements and look the same initially.<br>
**Different**: The component with the `client:load` directive will rerender after the page is loaded, and any interactive elements that it has will work.

3. Assume the `SvelteCounter` component shows a number and has a button to increase it. If you couldn’t see your website’s code, only the live published page, how would you tell which of the two `<SvelteCounter />` components used `client:visible`?

Try clicking the button, and see which one is interactive. If it responds to your input, it must have had a `client:` directive.

#### Test your knowledge

For each of the following components, identify what will be sent to the browser:

1. `<ReactCounter client:load />`

    [] HTML and CSS only

    [x] HTML, CSS, and JavaScript

2. `<SvelteCard />`

    [x] HTML and CSS only

    [] HTML, CSS, and JavaScript

#### Checklist

[x] I can install an Astro integration.
[x] I can write UI framework components in their own language.
[x] I can use a `client:` directive for hydration on my UI framework component.

#### Resources

[Astro Integrations Guide](https://docs.astro.build/en/guides/integrations-guide/)
[Using UI Framework Components in Astro](https://docs.astro.build/en/guides/framework-components/#using-framework-components)
[Astro client directives reference](https://docs.astro.build/en/reference/directives-reference/#client-directives)

#### 6.2. Back on dry land. Take your blog from day to night, no island required!

Now that you can build Astro islands for interactive elements, don’t forget that you can go pretty far with just vanilla JavaScript and CSS!

Let’s build a clickable icon to let your users toggle between light or dark mode using another `<script>` tag for interactivity… with no framework JavaScript sent to the browser.

> GET READY TO...<br>
&bull; Build an interactive theme toggle with only JavaScript and CSS<br>
&bull; Send as little JavaScript to the browser as possible!

#### Add and style a theme toggle icon
Section titled Add and style a theme toggle icon

1. Create a new file at `src/components/ThemeIcon.astro` and paste the following code into it:

```jsx
---
---
<button id="themeToggle">
  <svg width="30px" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
    <path class="sun" fill-rule="evenodd" d="M12 17.5a5.5 5.5 0 1 0 0-11 5.5 5.5 0 0 0 0 11zm0 1.5a7 7 0 1 0 0-14 7 7 0 0 0 0 14zm12-7a.8.8 0 0 1-.8.8h-2.4a.8.8 0 0 1 0-1.6h2.4a.8.8 0 0 1 .8.8zM4 12a.8.8 0 0 1-.8.8H.8a.8.8 0 0 1 0-1.6h2.5a.8.8 0 0 1 .8.8zm16.5-8.5a.8.8 0 0 1 0 1l-1.8 1.8a.8.8 0 0 1-1-1l1.7-1.8a.8.8 0 0 1 1 0zM6.3 17.7a.8.8 0 0 1 0 1l-1.7 1.8a.8.8 0 1 1-1-1l1.7-1.8a.8.8 0 0 1 1 0zM12 0a.8.8 0 0 1 .8.8v2.5a.8.8 0 0 1-1.6 0V.8A.8.8 0 0 1 12 0zm0 20a.8.8 0 0 1 .8.8v2.4a.8.8 0 0 1-1.6 0v-2.4a.8.8 0 0 1 .8-.8zM3.5 3.5a.8.8 0 0 1 1 0l1.8 1.8a.8.8 0 1 1-1 1L3.5 4.6a.8.8 0 0 1 0-1zm14.2 14.2a.8.8 0 0 1 1 0l1.8 1.7a.8.8 0 0 1-1 1l-1.8-1.7a.8.8 0 0 1 0-1z"/>
    <path class="moon" fill-rule="evenodd" d="M16.5 6A10.5 10.5 0 0 1 4.7 16.4 8.5 8.5 0 1 0 16.4 4.7l.1 1.3zm-1.7-2a9 9 0 0 1 .2 2 9 9 0 0 1-11 8.8 9.4 9.4 0 0 1-.8-.3c-.4 0-.8.3-.7.7a10 10 0 0 0 .3.8 10 10 0 0 0 9.2 6 10 10 0 0 0 4-19.2 9.7 9.7 0 0 0-.9-.3c-.3-.1-.7.3-.6.7a9 9 0 0 1 .3.8z"/>
  </svg>
</button>

<style>
  #themeToggle {
    border: 0;
    background: none;
  }
  .sun { fill: black; }
  .moon { fill: transparent; }

  :global(.dark) .sun { fill: transparent; }
  :global(.dark) .moon { fill: white; }
</style>
```

2. Add the icon to `Header.astro` so that it will be displayed on all pages. Don’t forget to import the component.

```jsx
---
import Hamburger from './Hamburger.astro';
import Navigation from './Navigation.astro';
import ThemeIcon from './ThemeIcon.astro';
---
<header>
  <nav>
    <Hamburger />
    <ThemeIcon />
    <Navigation />
  </nav>
</header>
```

3. Visit your browser preview at http://localhost:4321 to see the icon now on all your pages. You can try clicking it, but you have not written a script to make it interactive yet.

#### Add CSS styling for a dark theme

Choose some alternate colors to use in dark mode.

1. In `global.css`, define some dark styles. You can choose your own, or copy and paste:

```css
html.dark {
  background-color: #0d0950;
  color: #fff;
}

.dark .nav-links a {
  color: #fff;
}
```

#### Add client-side interactivity

To add interactivity to an Astro component, you can use a `<script>` tag. This script can check and set the current theme from `localStorage` and toggle the theme when the icon is clicked.

1. Add the following `<script>` tag in `src/components/ThemeIcon.astro` after your `<style>` tag:

```jsx
<style>
  .sun { fill: black; }
  .moon { fill: transparent; }

  :global(.dark) .sun { fill: transparent; }
  :global(.dark) .moon { fill: white; }
</style>

<script is:inline>
  const theme = (() => {
    if (typeof localStorage !== 'undefined' && localStorage.getItem('theme')) {
      return localStorage.getItem('theme');
    }
    if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
      return 'dark';
    }
      return 'light';
  })();

  if (theme === 'light') {
    document.documentElement.classList.remove('dark');
  } else {
    document.documentElement.classList.add('dark');
  }

  window.localStorage.setItem('theme', theme);

  const handleToggleClick = () => {
    const element = document.documentElement;
    element.classList.toggle("dark");

    const isDark = element.classList.contains("dark");
    localStorage.setItem("theme", isDark ? "dark" : "light");
  }

  document.getElementById("themeToggle").addEventListener("click", handleToggleClick);
</script>
```

2. Check your browser preview at http://localhost:4321 and click the theme icon. Verify that you can change between light and dark modes.

#### Test your knowledge

Choose whether each of the following statements describes Astro `<script>` tags, UI framework components, or both.

1. They allow you to include interactive UI elements on your website.

    [] Astro `<script>` tags

    [] UI framework components

    [x] both

2. They will create static elements on your site unless you include a `client:` to send their JavaScript to the client and run in the browser.

    [] Astro `<script>` tags

    [x] UI framework components

    [] both

3. They allow you to “try out” a new framework without requiring you to start an entirely new project using that tech stack.

    [] Astro `<script>` tags

    [x] UI framework components

    [] both

4. They allow you to reuse code you have written in other frameworks and you can often just drop them right into your site.

    [] Astro `<script>` tags

    [x] UI framework components

    [] both

5. They allow you to add interactivity without needing to know or learn any other JavaScript frameworks.

    [x] Astro `<script>` tags

    [] UI framework components

    [] both

> **Checklist**<br>
[x] I can use JavaScript for interactivity when I don’t want to add a framework.

#### Resources

[Client-side `<script>` in Astro](https://docs.astro.build/en/guides/client-side-scripts/)

### 6.3. Congratulations!

There’s one more edit to make…

```jsx
---
import BaseLayout from "../layouts/BaseLayout.astro";
const pageTitle = "About Me";
const happy = true;
const finished = true;
const goal = 3;
const identity = {
  firstName: "Sarah",
  country: "Canada",
  occupation: "Technical Writer",
  hobbies: ["photography", "birdwatching", "baseball"],
};
const skills = ["HTML", "CSS", "JavaScript", "React", "Astro", "Writing Docs"];
const skillColor = "navy";
const fontWeight = "bold";
const textCase = "uppercase";
---
```

We hope you learned a little about the basics of Astro, and had fun along the way!

You can find the code for the project in this tutorial on [GitHub](https://github.com/withastro/blog-tutorial-demo/tree/complete) or [StackBlitz](https://stackblitz.com/github/withastro/blog-tutorial-demo/tree/complete?file=src/pages/index.astro).

Check out our docs for guides and reference material, and visit our Discord to ask questions, get help or just hang out!

Welcome to the universe, astronaut. 👩🏼‍🚀👨🏿‍🚀🧑‍🚀👩🏾‍🚀

#### Next Steps

Continue with either of our tutorial extensions to [add view transitions to this project](https://docs.astro.build/en/tutorials/add-view-transitions/) or to [add a content collection to manage your blog posts](https://docs.astro.build/en/tutorials/add-content-collections/).
---

####

```astro
```

> GET READY TO...<br>
&bull; x<br>
&bull; x<br>
&bull; x

---
