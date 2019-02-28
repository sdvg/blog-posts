---
title: Using SVG symbol sprites with Vue
date: 2019-02-28
---

I recently compared the most popular methods of embedding icons in web application and came to the conclusion, that SVG symbols are probably the best solution in 2019.

Other options are:

* Icon Fonts, which usually work fine but are a bit difficult to maintain because you have to regenerate binary font files every time you add or change an icon. Also in my experience they sometimes fail to load in mysterious ways.
* Inline SVGs are easy to implement and to work with but can't be reused within a project and blow up the HTML, which can be annoying during development and can also hurt performance in the browser, especially when being repeated in long lists.
* Component files are equally handy to use as inline SVGs and can be reused within a project. They are great solution for dynamic graphics where you want to toggle single paths for example. But they are also hard to cache and can cause performance issues when repeated too often.  
* Lazy-loaded SVGs, as in HTML `<img>` tags and in CSS `background-image`s are a good solution for bigger and static graphics. They can easily be cached in the browser, but can't be manipulated, like changing the color using a CSS `fill` property.

SVG symbols on the other hand can be cached with a little effort, be reused, are accessible for CSS modifications and don't hurt the performance.

The idea is to have one big SVG image in your HTML body which looks like this:

```
<svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
  <symbol viewBox="0 0 1792 1792" id="icon-unicorn">
    <!-- paths and shapes -->
  </symbol>
  
  <!-- more symbols... -->
</svg>
```

The SVG won't be shown at this place, but provides symbols to be used in other SVGs within the document like this:

```
<svg>
  <use xlink:href="#icon-unicorn"></use>
</svg>
```

Now you could just have a big file where you manually add the `<symbol>` tags and this would work just fine.  
But there is more user-friendly way by using a Webpack loader which allows you to use your original `.svg` files just by importing them without any modifications or copy-pasting parts of them around. It also provides a simple way to fetch the sprite image and cache it in localStorage, which makes sense for sprite images of a certain size.

I use Vue.js for both work and my side-project [tidyMind][tidymind-intro] and implemented the following solution into both of them. It should be easily adjustable for other frameworks / libraries like React or Angular as well.

The plugin and loader combination I use is [svg-symbol-sprite-loader][svg-symbol-sprite-loader].

This is how to set it up with Vue:

## Install dependency

```
npm i -D svg-symbol-sprite-loader
```

## Webpack configuration

You will need a Webpack configuration which uses `html-webpack-loader`. This is necessary because the plugin uses a hook of html-webpack-loader to insert the filename of the generated sprite image into the HTML, to be picked up by the script which loads and caches it later.

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
    // ⚠️ must be included _after_ the HtmlWebpackPlugin
    new SVGSymbolSprite.Plugin({
      filename: `icon-sprite${process.env.NODE_ENV === 'production' ? '.[chunkhash]' : ''}.svg`
    }),
  ],
}
```

The `symbolId` option is optional, but when you omit it the `id` attributes of the `symbols` will just be the filenames of your SVGs (`unicorn.svg` becomes `id="unicorn"`). This can cause duplicate IDs in the document and [lead to bugs][collisions].

You can also consult the [official documentation][webpack-config] for this part.

## Load and cache the sprite file

[svg-symbol-sprite-loader][svg-symbol-sprite-loader] already provides a function which you just need to call to fetch and cache the sprite image:

```
import svgSymbolSpriteLoader from 'svg-symbol-sprite-loader'

svgSymbolSpriteLoader({ useCache: process.env.NODE_ENV === 'production' })
```

Just be aware that this uses the [Fetch API][fetch-api] which is not supported by Internet Explorer.

If you need to support IE, you can either import a [fetch polyfill][fetch-polyfill] or copy and adapt [the function][icon-sprite-loader] to the HTTP client you use. I have done the latter for my work projects. Since IE is supported there and [Superagent][superagent] is already bundled as HTTP client, we didn't want to increase the bundle size by an otherwise not needed polyfill.
   
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
      },
    },
    computed: {
      iconId () {
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

The `style` tag is optional, but allows the icons to automatically inherit the current CSS font color from parent components, which can be quite convenient.

The usage within other components looks like this:

```
// import `Icon` component locally or global

<Icon name="unicorn" />
``` 

## Bonus: Distinguish between icons and other SVGs

If you don't want to load all SVGs using `svg-symbol-sprite-loader` I recommend to introduce a special file extension for icon SVGs like `.icon.svg`. Change the `test` regexes in the Webpack config like to this:

Icon loader: `test: /\.icon\.svg$/,`  
File loader: `test: /.((?<!\.icon.)svg|png|jpg|gif|ico|eot|ttf|woff2?)$/`

This uses a negative lookbehind operator which matches `*.svg` but not `*.icon.svg`.

## Bonus: Use it with Vue CLI

When using Vue CLI you can't configure Webpack directly. But it allows you to manipulate and extend the config using [webpack-chain][webpack-chain].

Just create a `vue.config.js` file with the following content:

```
const path = require('path')
const SVGSymbolSprite = require('svg-symbol-sprite-loader')

module.exports = {
  chainWebpack: config => {
    config
      .plugin('svg-symbol-sprite-loader')
      .after('html')
      .use(SVGSymbolSprite.Plugin)
      .end();

    const svgRule = config.module.rule('svg')

    svgRule.uses.clear()

    svgRule
      .use('svg-symbol-sprite-loader')
      .loader('svg-symbol-sprite-loader')
      .options({
        symbolId: filePath => `icon-${path.basename(filePath, '.svg')}`,
      })
  }
}
```

I put together a small demo project which does this and which also includes the component: [sdvg/vue-cli-svg-symbol-sprite][cli-demo].

Got any questions or problems with this? Write me [an email][email] or a message on [Twitter][twitter] 🙂  
Found an mistake in this post? Write me an issue or send a PR on GitHub: [sdvg/blog-posts][github]

Thanks to [Alex][alex] for reviewing this!

[tidymind-intro]: tidy-mind-introduction.html
[collisions]: https://github.com/crystal-ball/svg-symbol-sprite-loader/issues/27
[svg-symbol-sprite-loader]: https://github.com/crystal-ball/svg-symbol-sprite-loader
[fetch-api]: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API
[fetch-polyfill]: https://github.com/github/fetch
[icon-sprite-loader]: https://github.com/crystal-ball/svg-symbol-sprite-loader/blob/master/src/icon-sprite-loader.js
[superagent]: https://github.com/visionmedia/superagent
[webpack-config]: https://github.com/crystal-ball/svg-symbol-sprite-loader#1-configure---webpackconfigjs
[email]: mailto:mail@stefan-dietz.eu
[twitter]: https://twitter.com/sd_vg
[github]: https://github.com/sdvg/blog-posts/
[webpack-chain]: https://github.com/neutrinojs/webpack-chain
[cli-demo]: https://github.com/sdvg/vue-cli-svg-symbol-sprite-demo
[alex]: https://github.com/alexanderstudte