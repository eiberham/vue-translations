<template>
    <div id="app">
        <router-link to="/">{{ $t('app.home') }}</router-link>|
        <router-link to="/about">{{ $t('app.about') }}</router-link>
        <router-view />
    </div>
</template>

<script>
export default {
    components: {}
}
</script>