<template>
  <div id="app" :style="{'background-color': Ibg}" :class="{ marL: leftmenu,Inight: night }" @click="hideMenu">
<!--     <img src="./assets/logo.png"> -->
    <header id="header">
      <img src="./assets/menu.png" @click.stop="leftMenus">
      <img src="./assets/portmenu.png"  @click.stop="rightMenus">
    </header>

    <leftMenu
     v-on="{setting:Setting,showSave:showSave}"
     :class="{ L : leftmenu }"
     ></leftMenu>
    <rightMenu
     :article="article"
     :save="isSaved"
     v-on="{
      hidee:hideRight, 
      Save:Save, 
      removeSave:removeSave,
      article:NPArticle, 
      random:randomArticle, 
      today:getArticle}" 
     :class="{ R : rightmenu }"
     ></rightMenu>
    <router-view :article1="article" :night="night"></router-view>
    <setting
     v-on="{setSize:setSize, setNight:setNight,setBg: setBg}" 
     :class="{hideSetting:setting}"
     ></setting>
    <saveArticle v-on="{hide:showSave,go:NPArticle}" :saveArticles="saveTotal" :class="{showSave :showS}"></saveArticle>
  </div>
</template>

<script>
import rem from '../static/rem'
import storage from '../static/storage'
import leftMenu from '~/leftMenu'
import rightMenu from '~/rightMenu'
import setting from '~/setting'
import saveArticle from '~/saveArticle'
export default {
  name: 'app',
  created () {
    rem.defaultFontSize
    if (storage.get('articles').length > 0) {
      this.saveTotal = storage.get('articles')
    }
  },
  mounted () {
    this.getArticle()
  },
  data () {
    return {
      article: '',
      leftmenu: false,
      rightmenu: false,
      night: false,
      setting: false,
      Ibg: '#fafafa',
      saveTotal: [],
      showS: false
    }
  },
  components: {
    leftMenu,
    rightMenu,
    setting,
    saveArticle
  },
  methods: {
  // 左右菜单栏
    leftMenus () {
      this.leftmenu = !this.leftmenu
      if (this.leftmenu) {
        document.body.style.overflow = 'hidden'
      } else {
        document.body.style.overflow = 'auto'
      }
    },
    rightMenus () {
      this.rightmenu = !this.rightmenu
    },
    hideMenu () {
      if (this.rightmenu || this.leftmenu || this.setting) {
        this.leftmenu = false
        this.rightmenu = false
        this.setting = false
      }
    },
    showSave () {
      this.leftmenu = false
      this.showS = !this.showS
    },
    hideRight () {
      this.rightmenu = false
    },
  // 获取文章
    getArticle () {
      this.$ajax({
        methods: 'get',
        baseURL: '/article',
        url: 'today',
        params: {
          dev: 1
        }
      }).then((res) => {
        this.article = res.data.data
      })
      document.body.scrollTop = document.documentElement.scrollTop = 0
    },
    NPArticle (data) {
      if (data > new Date().Format('yyyyMMdd')) {
        alert('已是最后一天，请浏览之前的文章')
        return
      }
      document.body.style.overflow = 'auto'
      this.showS = false
      this.$ajax({
        methods: 'get',
        baseURL: '/article',
        url: 'day',
        params: {
          dev: 1,
          date: data
        }
      }).then((res) => {
        this.article = res.data.data
      })
      document.body.scrollTop = document.documentElement.scrollTop = 0
    },
    randomArticle () {
      this.$ajax({
        methods: 'get',
        baseURL: '/article',
        url: 'random',
        params: {
          dev: 1
        }
      }).then((res) => {
        this.article = res.data.data
      })
      document.body.scrollTop = document.documentElement.scrollTop = 0
    },
    Save (data) {
      this.saveTotal.unshift(data)
      storage.set('articles', this.saveTotal)
    },
    removeSave (data) {
      for (let item in this.saveTotal) {
        if (this.saveTotal[item].title === data) {
          this.saveTotal.splice(item, 1)
        }
      }
      storage.set('articles', this.saveTotal)
    },
  // 阅读设置
    Setting () {
      document.body.style.overflow = 'auto'
      this.setting = true
      this.leftmenu = false
    },
    setSize (val) {
      document.querySelector('.content').style.fontSize = val
    },
    setBg (val) {
      this.night = false
      this.Ibg = val
    },
    setNight () {
      this.night = !this.night
    }
  },
  computed: {
    isSaved () {
      for (let item in this.saveTotal) {
        if (this.saveTotal[item].title === this.article.title) {
          return true
        }
      }
      return false
    }
  }
}
</script>

<style>
 @import '../static/normal.css';
  [v-cloak]{
    display: none;
  }
  *{
    margin: 0;
    padding: 0;
  }
  body{
    font-size: .35rem;
    margin: 0 auto;
    /*overflow: hidden;*/
  }
  ul li{
    list-style: none;
  }
  img{
    cursor: pointer;
  }
  .Inight{
    background:#444 !important;
  }
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    position: relative;
    transition: .3s;
    width: 100%;
    background: #fafafa;
  }
  #header{
    display: flex;
    justify-content: space-between;
    box-sizing: border-box;
    padding:.15rem .2rem;
    position: fixed;
    top: 0;
    width: 100%;
    background: inherit;
  }
  .marL{
    margin-left: 40%;
  }
  .content_c p{
    text-indent: 2em;
    line-height: .5rem;
    margin: .4rem 0 .2rem;
    /*font-weight: 600;*/
  }
  .hideSetting {
    bottom:0 !important;
  }
  div.showSave {
    opacity: 1 !important;
    top: 0 !important;
    left: 0 !important;
    transition:opacity .3s,top .3s, left 0s !important;
  }
</style>
