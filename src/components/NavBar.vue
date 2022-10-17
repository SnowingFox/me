<script setup lang="ts">
import { isDark } from '~/logics'
</script>

<template>
  <header class="header z-40">
    <router-link
      class="w-10 h-10 absolute lg:fixed m-6 select-none outline-none"
      to="/"
      focusable="false"
    >
      <img
        src="/avatar.jpeg?url"
        :class="isDark ? 'op30' : 'op80'"
        alt="logo"
        transition-all
        delay-200
        scale-150
      >
    </router-link>
    <nav class="nav">
      <div class="spacer" />
      <div class="right">
        <router-link to="/posts">
          <span class="lt-md:hidden">Blog</span>
          <div i-ri-article-line md:hidden />
        </router-link>
        <router-link to="/algorithm" title="Algorithm">
          <span class="lt-md:hidden">Algorithm</span>
          <div i-clarity:atom-line md:hidden />
        </router-link>
        <router-link to="/bookmarks" title="Bookmarks">
          <div i-ri-bookmark-line />
        </router-link>
        <router-link to="/notes" title="Notes">
          <div i-ri-sticky-note-line />
        </router-link>
        <a href="https://twitter.com/FoxSnowing" target="_blank" title="Twitter" class="lt-md:hidden">
          <feather-twitter />
        </a>
        <a href="https://github.com/snowingfox" target="_blank" title="GitHub" class="lt-md:hidden">
          <div i-uil-github-alt />
        </a>
        <toggle-theme />
        <router-link to="/projects">
          <span class="lt-md:hidden">Projects</span>
          <div i-ri-lightbulb-line class="md:hidden" />
        </router-link>
      </div>
    </nav>
  </header>
</template>

<style scoped>
.header h1 {
  margin-bottom: 0;
}

.logo {
  position: absolute;
  top: 1.5rem;
  left: 1.5rem;
}

.nav {
  padding: 2rem;
  width: 100%;
  display: grid;
  grid-template-columns: auto max-content;
  box-sizing: border-box;
}

.nav > * {
  margin: auto;
}

.nav img {
  margin-bottom: 0;
}

.nav a {
  cursor: pointer;
  text-decoration: none;
  color: inherit;
  transition: opacity 0.2s ease;
  opacity: 0.6;
  outline: none;
}

.nav a:hover {
  opacity: 1;
  text-decoration-color: inherit;
}

.nav .right {
  display: grid;
  grid-gap: 1.2rem;
  grid-auto-flow: column;
}

.nav .right > * {
  margin: auto;
}
</style>
