---
title: Projects - Snowingfox
display: Projects
subtitle: List of my projects
description: List of my projects
projects:
  Upcoming:
    - name: 'HFUT-helper'
      link: 'https://github.com/hfut-soft-ware/hfut-helper'
      desc: '合肥工业大学小助手'
      icon: 'i-emojione:watermelon'
    - name: 'HFUT-Tree-Hole'
      link: 'https://github.com/hfut-tree-hole/hfut-tree-hole-react'
      desc: '合肥工业大学树洞'
      icon: 'i-noto:deciduous-tree'
    - name: 'vue-use-form'
      link: 'https://github.com/vue-use-form/vue-use-form'
      desc: 'Vue Compisiton API for form state management and validation'
      icon: 'i-uiw:safety'

  Working:
    - name: 'HFUT-API'
      link: 'https://hfut-api-doc.vercel.app/#/'
      desc: '合肥工业大学教务API'
      icon: 'i-carbon:ibm-cloud-internet-services'

  Toys:
    - name: 'LyricParser'
      link: 'https://github.com/SnowingFox/LyricParser'
      desc: 'An plugin for parsing lyrics from NeteaseCloudMusic(网易云音乐)'
      icon: 'i-ic:outline-lyrics'
---

<ListProjects :projects="frontmatter.projects" />
