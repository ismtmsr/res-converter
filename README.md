# res-converter

> レス変換

# 作業覚書

```
[ishimoto:adachi]$ vue --version
@vue/cli 4.2.3
[ishimoto:adachi]$ vue init simulatedgreg/electron-vue res-converter

? Application Name res-converter
? Application Id com.five-birds.res-converter
? Application Version 0.0.1
? Project description レス変換
? Use Sass / Scss? Yes
? Select which Vue plugins to install vue-electron, vue-router, vuex, vuex-electron
? Use linting with ESLint? Yes
? Which ESLint config would you like to use? Standard
? Set up unit testing with Karma + Mocha? Yes
? Set up end-to-end testing with Spectron + Mocha? Yes
? What build tool would you like to use? builder
? author masaru.ishimoto@five-birds.com

   vue-cli · Generated "res-converter".

---

All set. Welcome to your new electron-vue project!

Make sure to check out the documentation for this boilerplate at
https://simulatedgreg.gitbooks.io/electron-vue/content/.

Next Steps:

  $ cd res-converter
  $ yarn (or `npm install`)
  $ yarn run dev (or `npm run dev`)

[ishimoto:adachi]$ cd res-converter/
```

##  ReferenceError: process is not defined

[electron-vueで「ReferenceError: process is not defined」が出たときの対処法](https://taklog.me/archives/811)

## [Vuetify] Multiple instances of Vue detected

[Electron-VueでVuetifyを使用した際に「Multiple instances of Vue detected」が発生する時の対処法](https://qiita.com/ryo2132/items/035e9745f941c6c98654)

```
.electreon-vue/webpack.renderer.config.js
- let whiteListedModules = ['vue']
+ let whiteListedModules = ['vue', 'vuetify']
```
