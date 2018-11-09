<template>
  <div>
    <h1>{{ msg }}</h1>
    Hello, {{ $route.params.name }}
    <hr />
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'firstroute',
  data () {
    return {
      msg: 'FirstRoute'
    }
  }
}
</script>
