---
title: Tailwind CSS Experience
slug: tailwind_css
date: 2020-10-03
tags:
- web
description: Finally, a CSS toolkit that is technically sound and easy to use
---

I've gone through a few CSS toolkits this year, such as: [PureCSS](https://purecss.io/), [Milligram](https://milligram.io/) and now: [TailwindCSS](https://tailwindcss.com/). I am now fully sold on tailwind CSS. If you are like me who is doing solo projects and like to tweak, there is nothing better in the CSS world than Tailwind.

## Toolchain ##

Tailwind is built on top of [PostCSS](https://postcss.org/) framework. I always dislike earlier CSS macro languages like SASS or SCSS because the toolchain is of questionable taste. Do you know that the tool [node-sass](https://github.com/sass/node-sass/releases) actually download another binary from GitHub under the hood? If you are not on the support list there is no easy way to install SASS/SCSS. You don't have to try very hard to be unsupported: node 10.x + 32-bit x86 Linux does not sound very exotic, right? It is unsupported.

On the other hand, PostCSS is a pure js toolchain, so as long as you have node.js and NPM, everything works. Also, you can tailor the functionalities by configured plugins. So far I only have 3 plugins enabled:

 * [autoprefixer](https://github.com/postcss/autoprefixer), which handles the "-webkit-" and "-moz-" craps for you;
 * [postcss](https://github.com/postcss/postcss-nested), which gives you the ability to have nested styles;
 * [tailwindcss](https://tailwindcss.com/), which gives you everything you will ever need to write your own style in a concise way

The downside is it feels slow compared to SCSS processing. Well, it is a build time job so I can live with that.

## No default styles ##

Tailwind is not a CSS file you directly use, but a bunch of directives that you include in your CSS file to make a style that is completely yours. If you don't use its classes it has nothing, it even reset every style included with your browser so you start really from scratch. Compared to Milligram or PureCSS which gives you a foundation of styles, Tailwind gives you only building blocks in the form of utility classes, and there are **lots** of them. If you include everything from it, the resulting CSS file will be several megabytes long.

Obviously, you don't want to have a megabytes long CSS file. So the method employed by Tailwind is call purging. The idea is to figure out all the classes you really used in your HTML and purge everything unneeded. The resulting CSS will just contain the styles you used, nothing more, nothing less.

## Utility vs Semantics ##

When you style an element, you typically add one or more classes to it in the HTML. The classes are defined in the CSS to give them styles. The name of the classes, however, could be utility names or semantical names. For example:

``` html
<div class="column-33 text-align-right">
...
</div>
```

vs

``` html
<div class="sidebar">
...
</div>
```

All CSS packages provide are utility classes, because they have no idea of how the classes should map to your HTML components. Most CSS packages' classes are opinionated and already usable as-is. However, Tailwind is different. Its classes are basic and orthogonal; you most likely have to use multiple classes to achieve what you want. In milligram you do:

``` html
<button class="button">Default Button</button>
```

To achieve the same, you will have to do:

``` html
<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
  Button
</button>
```

On one hand, you have the convenience and on the other controllability. What I had before in my HTML is a mixture of utility and semantical classes: The utility classes are straight from the CSS packages if they fit my need already; and the semantical classes and composed by me to suit my particular needs. It has always felt a little hacky to me, but it works.

With Tailwind, I settled down to a strict separation of semantical and utility classes. In my HTML, I only use semantical classes; there are no Tailwind defined classes at all. In my CSS file, I package Tailwind's utility classes into semantical components. It is like writing CSS from scratch, but using Tailwind provides utility classes as a funky macro language:

``` css
@layer components {
    .button {
		@apply flex-none px-6 py-2 inline-block rounded appearance-none
		font-bold whitespace-no-wrap text-sm text-center uppercase tracking-wide;
    }
    .viewport {
		@apply relative;
		.header {
			@apply flex flex-col text-center;
		}
		.content {
			form {
				@apply flex flex-wrap mb-4;
 				.button[type="reset"] {
					@apply text-white bg-pink-600 mr-1;
				}
				.button[type="submit"] {
					@apply text-white bg-purple-600 mr-1;
				}
				.button:focus, .button:hover {
					@apply bg-gray-600;
				}
			}
		}
		.footer {
			@apply flex flex-col text-center text-sm;
		}
	}
}
```

As you can see, by using nested CSS classes, I nearly recreated my HTML element hierarchy in my CSS and explicitly styled everything. I could have done it without Tailwind, but that would be very verbose. With Tailwind, I can express the same idea using much fewer lines and have a CSS file that is readable and maintainable.

Also a few words on layers. The layer is a concept from Tailwind's classes organization. It defines 3 layers, base, components, and utilities. The majority of its classes are in the utility layer. By defining my own classes in the components layer, I achieved separation, so I can easily purge the CSS without looking at my HTML files. Here is my tailwind.config.js file:

``` javascript
module.exports = {
    future: {
		removeDeprecatedGapUtilities: true,
		purgeLayersByDefault: true,
    },
    purge: {
		enabled: true,
		layers: ['utilities'],
		content: [],
    },
    theme: {
		extend: {},
    },
    variants: {},
    plugins: [],
}
```

As you can see, I didn't give Tailwind any HTML files to parse. I don't want to trust Tailwind's ability to parse HTML files that can come from anywhere among templates, serverside scripts, or frontend scripts. The objective of purging is to keep the CSS file size small; and since most of the size of Tailwind is in the utility layer, purging all of it is good enough.

## Summary ##

I believe one should control every single line of their CSS files. Tailwind and PostCSS help you to achieve this goal by leaving you in firm control of what is included. The method I settled for is not exactly the same one suggested by Tailwind in their examples, but like this:

 * Write your HTML file with only semantical and as few as possible classes
 * Write your CSS file with PostCSS-nested that mimic your HTML's hierarchy
 * Use Tailwind to style your elements and classes explicitly and concisely
 * Use Tailwind's purge functionality to remove unneeded styles
