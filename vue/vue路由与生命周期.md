# vue路由与生命周期

```html
<!DOCTYPE html>

<html>

<head>

  <meta charset="utf-8">

  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">

  <title>vue路由与生命周期</title>

  <meta name="description" content="">

  <meta name="keywords" content="">

  <link href="" rel="stylesheet">

  <script src="https://cdn.jsdelivr.net/npm/vue@2.6.14/dist/vue.js"></script>

  <script src="https://unpkg.com/vue-router@3"></script>

</head>

<body>

  <div id="app">

    <router-link to='/app/1'>1</router-link>

    <router-link to='/app/2'>2</router-link>

    <router-link to='/test'>test</router-link>

    <router-view></router-view>

  </div>

  <template id="template">

    <div>

      {{$route.params.id}}

    </div>

  </template>

  <script type="text/javascript">

    const template = {

      // 获取输出，在路由触发前获取数据，但是此时获取不到路由的this对象
      beforeRouteEnter(to, from, next) {
        console.log('beforeRouteEnter 组件中的路由');
        //next();
        next(vm => {
          console.log(vm)
        })

      },

      // 同一个组件不同路由触发
      beforeRouteUpdate(to, from, next) {
        console.log('beforeRouteUpdate 组件中的路由')
        next()

      },

      // 表单修改
      beforeRouteLeave(to, from, next) {
        console.log('beforeRouteLeave 组件中的路由')
        next()
      },
      props: ['name'],
      template: '#template',

    };

    const test = {
      beforeRouteEnter(to, from, next) {
        console.log('test -- beforeRouteEnter 组件中的路由');
        //next();
        next(vm => {
          console.log(vm)

        })

      },

      beforeRouteUpdate(to, from, next) {
        console.log('test -- beforeRouteUpdate 组件中的路由')
        next()

      },

      beforeRouteLeave(to, from, next) {
        console.log('test -- beforeRouteLeave 组件中的路由')
        next()

      },
      template: '<h1>test1</h1>'
    }

    const routes = [
      {
        path: "/app/:id",
        component: template,
        beforeEnter(to, from, next) {
          console.log('beforeEnter 路由中的路由');
          next();
        }
      },

      {

        path: '/test',
        component: test,
        beforeEnter(to, from, next) {
          console.log('test -- beforeEnter 路由中的路由');
          next();

        }

      },

    ];

    const router = new VueRouter({
      routes // (缩写) 相当于 routes: routes
    });

    // 验证用户登录

    router.beforeEach((to, from, next) => {
      console.log('beforeEach 生命周期');
      //next('/login') 接受的参数和router-link一致
      next();

    });

    router.beforeResolve((to, from, next) => {
      console.log('beforeResolve 生命周期');
      next()

    });

    // 跳转之后执行 没有next

    router.afterEach((to, from) => {
      console.log('afterEach 生命周期');

    });

    const app = new Vue({

      el: "#app",

      router,

      beforeCreate() {
        console.log('beforeCreate 生命周期');

      },

      created() {
        console.log('created 生命周期');

      },

      beforeMount() {
        console.log('beforeMount 生命周期');
      },

      mounted() {
        console.log(' mounted 生命周期');
      },

      beforeUpdate() {
        console.log('beforeUpdate 生命周期');
      },

      updated() {
        console.log('updated 生命周期');
      },

      beforeDestroy() {
        console.log('beforeDestroy 生命周期');
      },

      destroyed() {
        console.log('destroyed 生命周期');
      }

    })

  </script>

</body>

</html>

```


