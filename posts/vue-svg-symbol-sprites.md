# Using SVG symbol sprites with Vue

I recently compared the most popular methods of embedding icons in web application and came to the conclusion, that SVG symbols are probably the best solution in 2019.

There are also:

* Icon Fonts, which usually work fine but are a bit difficult to maintain because you have to generate binary font files. Also in my experience they sometimes fail to load in mysterious ways, even on popular websites. 
* Inline SVGs: Are quite convenient to implement and work with but can't be reused within the project and blow up the HTML, which can be annoying during development and can also hurt performance in the browser, especially when repeated in big lists.
* Component files are equally convenient to use as inline SVGs and can be reused within the project. They are are great solution for dynamic graphics where we want to toggle single paths for example. But they are also hard to cache and can cause performance issues when repeated to often.  
* "Linked" SVGs, as in HTML `<img>` tags and in CSS `background-images` are a good solution for bigger and static graphics. They can easily be cached in the browser, but we are not able to do modifications like changing the `fill` color.   

SVG symbols on the other hand can be cached with a little effort, reused, are accessible for CSS modifications like `fill` and don't hurt the performance.

It works by having one big SVG image in your HTML body which looks like this:

```
<svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
  <symbol viewBox="0 0 1792 1792" id="icon-unicorn">
    <!-- <paths>s -->  
  </symbol>
  
  <!-- more symbols... -->  
</svg>
```

The SVG won't be shown at this place, but provides symbols to be used in other SVGs in the document like this:

```
<svg><use xlink:href="#icon-unicorn"></use></svg>
```

Now you could just have a big file where you manually add the `<symbol>` tags and it would work just fine.
But there is more convenient way by using a Webpack loader which allows you to use original `.svg` files just by importing them with any modification or copy-pasting around. It also provides a way to load the sprite image and cache it in localStorage, which makes sense for sprite images of a certain size. 

I use Vue.js for both work and my side-project [tidyMind][tidymind-intro] and implemented the following solution into both of them. It should be easily adjustable for other frameworks / libraries like React or Angular though.

The plugin & loader combination I use is [svg-symbol-sprite-loader][svg-symbol-sprite-loader].

This is how to set it up with Vue:

## Webpack configuration

You will need a Webpack configuration which already uses `html-webpack-loader`. This is necessary because the plugin uses a hook of html-webpack-loader to insert the filename of the generated filesize into the HTML, to be picked up by the script which loads and caches it later.

```
const SVGSymbolSprite = require('svg-symbol-sprite-loader')

module.exports = {
  module: {
    rules: [
      {
        test: /\.svg$/,
        use: [
          {
            loader: 'svg-symbol-sprite-loader',
            options: {
              symbolId: filePath => `icon-${path.basename(filePath, '.svg')}`,
            },
          },
        ],
      },
    ],
  },
  plugins: [
    // must be included _after_ the HtmlWebpackPlugin
    new new SVGSymbolSprite.Plugin({
      filename: `icon-sprite${process.env.NODE_ENV === 'production' ? '.[chunkhash]' : ''}.svg`
    }),
  ],
}
```

The `symbolId` option is optional, but when you omit it the `id` attribute of the `symbols` will just be the filename of you SVG. This can cause duplicate IDs in the document and [lead to bugs][collisions]

You can also consult the official documentation for this part: <https://github.com/crystal-ball/svg-symbol-sprite-loader#1-configure---webpackconfigjs>

## Load and cache sprite file

[svg-symbol-sprite-loader][svg-symbol-sprite-loader] conveniently provides a function which you just need to call, to fetch and cache the sprite image:

```
import svgSymbolSpriteLoader from 'svg-symbol-sprite-loader'

svgSymbolSpriteLoader({ useCache: process.env.NODE_ENV === 'production' })
```

Just be be aware that this uses the [Fetch API][fetch-api] which is not supported by Internet Explorer.
If you need to support IE, you can either import a [fetch polyfill][fetch-polyfill] or copy and adapt [the function][icon-sprite-loader] to the HTTP client you use. This is what I did for my work project where we support IE, already have bundled [Superagent][superagent] as a HTTP client and did not want to increase the bundle size by including a polyfill which is otherwise not needed.
   
## Create a Vue component

The Vue component should import all necessary icons and expose a simple API to include the icons anywhere needed. I did it like this:

```
<script>
  import unicorn from './icons/unicorn.svg'
  // ...more icons

  const iconMap = {
    unicorn: unicorn.id,
    // ...more icons
  }

  export default {
    props: {
      name: {
        type: String,
        required: true,
        validate: name => Object.keys(iconMap).includes(name),
      }
    },
    computed: {
      iconId() {
        return iconMap[this.name]
      },
    },
  }
</script>

<template>
  <svg class="icon">
    <use :xlink:href="`#${iconId}`" />
  </svg>
</template>

<style scoped>
  .icon {
    fill: currentColor;
  }
</style>
```

The style tag is optional, but allows the icons to automatically inherit the current CSS font color from the parent component which can be quite convenient.

The usage looks like this:

```
// import `Icon` component locally or global

<template>
  <Icon name="unicorn" />
</template>
``` 

## Bonus: Distinguish between icons and other SVGs

If you don't want to load all SVGs sprite `svg-symbol-sprite-loader` I recommend to introduce a special file extension for icon SVGs like `.icon.svg`. Change the `test` regex in the webpack config like to this: ``

Have questions or problems with this? Write me an email or on Twitter 🙂
Found an mistake in this post? Write me an issue or send a PR on GitHub: <https://github.com/sdvg/blog-posts/> 


[tidymind-intro]: tidy-mind-introduction.html
[collisions]: https://github.com/crystal-ball/svg-symbol-sprite-loader/issues/27
[svg-symbol-sprite-loader]: https://github.com/crystal-ball/svg-symbol-sprite-loader
[fetch-api]: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
[fetch-polyfill]: https://github.com/github/fetch
[icon-sprite-loader]: https://github.com/crystal-ball/svg-symbol-sprite-loader/blob/master/src/icon-sprite-loader.js
[superagent]: https://github.com/visionmedia/superagent