# bff_event_handson

できればブラウザはchromeでvue.jsの拡張を入れるとわかりやすいです。

## Nuxt.jsのプロジェクトを作成する

```
$ npx create-nuxt-app <project-name>
```

もしくは
```
$ yarn create nuxt-app <project-name>
```

コマンドを実行するとNuxt.jsの初期設定についていくつか聞かれるので以下のように答えてください。
```
? Project name authTestApp
? Project description My astounding Nuxt.js project
? Use a custom server framework none
? Use a custom UI framework Element UI
? Choose rendering mode Universal
? Use axios module yes
? Use eslint yes
? Use prettier yes
? Author name myname
? Choose a package manager yarn
```

その後に
```
$ cd authTestApp
$ yarn run dev
```
を実行して、localhost:3000にアクセスできれば成功です。

## Auth0をNuxt.jsで使うために必要なAuth Moduleを追加する
```
$ yarn add @nuxtjs/auth
```

```js
//nuxt.config.js

  modules: [
    '@nuxtjs/axios',
    '@nuxtjs/auth', // 追加
  ]
```

nuxt.config.jsに追加したモジュールを追記する。

続いてAuth0に必要なルーティングの設定を追加していく。
```js
//nuxt.config.js

  auth: {
    redirect: {
      login: '/',  // 未ログイン時のリダイレクト先
      logout: '/logout',  // ログアウト処理を実行した直後のリダイレクト先
      callback: '/callback',  // コールバックURL
      home: '/mypage',  // ログイン後に遷移するページ
    },
  }
```


## Auth0を作成する
Auth0のページにアクセスして、アカウントを作成すると、clientIDとDomainが取得できるので、これをNuxt.js側に設定していく。

```js
//nuxt.config.js

  auth: {
    strategies: {
        auth0: {
          domain: 'Domain',  // 追加
          client_id: 'Client ID'  // 追加
        }
    },
    redirect: {
      login: '/login',  // 未ログイン時のリダイレクト先
      logout: '/logout',  // ログアウト処理を実行した直後のリダイレクト先
      callback: '/callback',  // コールバックURL
      home: '/mypage',  // ログイン後に遷移するページ
    },
  },
```


## 各ページを実装していく
```js
// layout/deault.vue

<template>
  <div>
    <div class="line"></div>
    <el-menu
      mode="horizontal"
      background-color="#545c64"
      text-color="#fff"
      active-text-color="#ffd04b">
      <el-menu-item><nuxt-link to="/">Home</nuxt-link></el-menu-item>
      <el-menu-item><nuxt-link to="/mypage">Mypage</nuxt-link></el-menu-item>
      <el-menu-item><nuxt-link to="/login">Login</nuxt-link></el-menu-item>
    </el-menu>
    <el-card>
      <el-container fluid fill-height>
        <nuxt/>
      </el-container>
    </el-card>
  </div>
</template>
```

```js
// page/login.vue

<template>
  <el-container>
    <el-form>
      <el-button type="primary" @click="loginWithAuthZero">Auth0でログイン 
      </el-button>
    </el-form>
  </el-container>
</template>

<script>
export default {
  methods: {
    loginWithAuthZero: function () {
      this.$auth.loginWith('auth0')
    }
  }
}
</script>
```

上記のlogin.vueでエラーが出る人は
```js
// page/login.vue

<template>
  <div>
    <button @click="loginWithAuthZero">Auth0でログイン</button>
  </div>
</template>

<script>
export default {
  methods: {
    loginWithAuthZero: function () {
      this.$auth.loginWith('auth0')
    }
  }
}
</script>
```

こちらも試してみてください。

```js
// page/callback.vue

<template>
  <el-container>
    <p>Please Wating・・・・</p>
  </el-container>
</template>
```

認証を通らないと見れないページ
```js
// page/mypage.vue

<template>
  <el-container>
    <h1>Hello, {{this.$auth.$state.user.family_name}}</h1>
  </el-container>
</template>
```

これで必要なページの実装は完了したので、`$ yarn run dev`を実行して実際にログイン/新規登録してみましょう。


## middlewareを使ってログインチェックしてみる
```js
// middleware/auth.js

export default function({store, redirect}) {
  if (!store.state.auth.loggedIn) {
    redirect('/login');
  }
}
```


```js
// mypage.vue

<script>
export default {
  middleware: 'auth'
}
</script>
```

このように認証していないとアクセスさせたくないページのmiddlewareに宣言すると、アクセスを制限できる。
