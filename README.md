# Astro.build official tutorial: Extend with View Transitions

**Index**

Prerequisites

Build a Blog Tutorial Code

1. Full-page navigation vs client-side routing (SPA mode)

    1.1. Test your knowledge

2. Extending the blog tutorial with view transitions

    2.1. Upgrade dependencies

    2.2. Add the `<ViewTransitions />` router

    2.3. Update scripts

    2.4. Test your knowledge

3. Customize Transition animations

    3.1. Try it yourself - Make the navigation links slide in

    3.2 Force a full browser reload for some links

---

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

## Full-page navigation vs client-side routing (SPA mode)

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

Which is the correct order of events after a visitor clicks a link to go to a new page during client-side navigation?

[] 1.`astro:after-swap`, 2.`astro:page-load`, 3.the new page is visible

[x] 1.`astro:after-swap`, 2.the new page is visible, 3.`astro:page-load`

[] 1.`astro:page-load`, 2.the new page is visible, 3.`astro:after-swap`

### Customize Transition animations

7. Change the default `fade` animation to a `slide` for the page title.

With view transitions enabled, you currently have a small fade in and out set for all page transition animations. Astro also provides a built-in `slide` animation. To change the type of animation for a single element, add the `transition:animate=""` directive.

For example, to make the page titles slide in instead of fade in, add `transition:animate="slide"` to the `<h1>` element in the BaseLayout (`src/layouts/BaseLayout.astro`):

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
    <h1 transition:animate="slide">{pageTitle}</h1>
    <slot />
    <Footer />
    <script>
      import "../scripts/menu.js";
    </script>
  </body>
</html>
```

In your browser preview, you will now see the page titles slide on to the screen, while other elements such as the body text continues to fade in and out.

### Try it yourself - Make the navigation links slide in

Add an animation directive to make the `<div>` in `Navigation.astro` containing all the header links slide in on page navigation, following the [same steps as above](https://docs.astro.build/en/tutorials/add-view-transitions/#customize-transition-animations).

```jsx
---
---

<div class="nav-links" transition:animate="slide">
  <a href="/">Home</a>
  <a href="/about/">About</a>
  <a href="/blog/">Blog</a>
  <a href="/tags/">Tags</a>
</div>

```

Check your browser preview and now both the page title and the header links will slide in on every page navigation.

8. Add a longer fade-in on your blog post descriptions.

    You can also customize Astro’s built-in animations by importing them, then providing any CSS animation properties.

    For example, to make the description fade in slowly when you navigate to a blog post, import the `fade` animation in your layout for Markdown blog posts (`src/layouts/MarkdownPostLayout.astro`). Then, add the transition directive for Astro’s `fade` with a duration of `2s`:

```jsx
---
import BaseLayout from "./BaseLayout.astro";

import { fade } from "astro:transitions";

const { frontmatter } = Astro.props;
---

<BaseLayout pageTitle={frontmatter.title}>
  <p>{frontmatter.pubDate.slice(0, 10)}</p>
  <p transition:animate={fade({ duration: "2s" })}>
    <em>{frontmatter.description}</em>
  </p>
  <p>Written by: {frontmatter.author}</p>
  <img src={frontmatter.image.url} width="300" alt={frontmatter.image.alt} />
  <slot />
</BaseLayout>
```

Navigate to any blog post in your browser preview, and you will see the description fade in slower than the rest of the text.

Read more about the different [transition directives](https://docs.astro.build/en/guides/view-transitions/#transition-directives) and [customizing animations](https://docs.astro.build/en/guides/view-transitions/#customizing-animations).

### Force a full browser reload for some links

9. Prevent client-side routing and instead require the browser to reload when navigating to your About page.

    Sometimes you will want a full browser reload when visitors click a certain link. For example, you may be linking to a page that does not also use the `<ViewTransitions />` router, or to a file directly such as a `.pdf`.

    To make it so that your browser refreshes every time you click the navigation link to go to your About page, add the `data-astro-reload` attribute to the `<a>` tag in your `<Navigation />` component (`src/components/Navigation.astro`). This will override the `<ViewTransitions />` router entirely, and any of the view transition animations, for this one link.

```jsx
---
---

<div class="nav-links" transition:animate="slide">
  <a href="/">Home</a>
  <a href="/about/" data-astro-reload>About</a>
  <a href="/blog/">Blog</a>
  <a href="/tags/">Tags</a>
</div>
```

Now, when you click the navigation link to your About page, no animations will occur. The page links and title will not slide in, and the page content will not fade in when you navigate to your About page using this link.

10. Add a link to your About page from your author name in your Markdown layout for blog posts.

    `data-astro-reload` only triggers a full browser refresh when going to a new page **from the link it is added to**. It does not control all instances of navigating to your About page.

    In your `<MarkdownPostLayout />` (`src/layouts/MarkdownPostLayout.astro`) component, add a link to your About page on your author name:

```jsx
---
import BaseLayout from "./BaseLayout.astro";
import { fade } from "astro:transitions";
const { frontmatter } = Astro.props;
---

<BaseLayout pageTitle={frontmatter.title}>
  <p>{frontmatter.pubDate.slice(0, 10)}</p>
  <p transition:animate={fade({ duration: '2s' })} ><em>{frontmatter.description}</em></p>
  <p>Written by: <a href="/about/">{frontmatter.author}</a></p>
  <img src={frontmatter.image.url} width="300" alt={frontmatter.image.alt} />
  <slot />
</BaseLayout>
```

If you visit any blog post in your browser preview, and then click on the linked author name to be taken to the About page, what does the page navigation look like?

When a visitor clicks a link to the About page from an individual blog post, the page title and header navigation links slide in across the screen because the `data-astro-reload` attribute is not set on these links.

There is still so much more to explore! See our [full View Transitions Guide](https://docs.astro.build/en/guides/view-transitions/) for more things you can do with view transitions.

For the full example of the blog tutorial using view transitions, see the [View Transitions branch](https://github.com/withastro/blog-tutorial-demo/tree/view-transitions) of the tutorial repo.
---

####

```astro
```

> GET READY TO...<br>
&bull; x<br>
&bull; x<br>
&bull; x

---