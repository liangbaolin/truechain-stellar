<template>
  <div id="tc-app">
    <nav :class="{'tc-nav-login': $route.name === null}">
      <img class="tc-nav-logo" src="./assets/logo.png" alt="logo" height="60">
      <transition name="fly-right">
        <div style="position: absolute; width: 100%;" v-if="$route.name !== null">
          <div
            class="tc-nav-title"
            v-for="item in routes"
            :key="routes.indexOf(item)"
            :class="{'tc-nav-title-focus': $route.name === item.name}"
            @click="$router.push(item.path)">
            {{ $t(`App.router.${item.name}`) }}
          </div>
        </div>
      </transition>
      <div class="tc-nav-lang">
        <span :class="{'focus': languageTag === 'en'}" @click="setLanguage('en')">English</span>
        <span :class="{'focus': languageTag === 'sc'}" @click="setLanguage('sc')">简体中文</span>
      </div>
    </nav>
    <div id="tc-content">
      <hello v-if="$route.name === null"></hello>
      <div id="tc-user" @mouseleave="setHoverObjIndex(0)" :class="{
        'tc-user-hide': $route.name === null,
        'tc-user-ctrl-open': hoverNav === 2
      }">
        <div class="tc-user-net" :class="{ 'tc-user-net-hover': hoverNav === 1 }" @mouseenter="setHoverObjIndex(1)">
          {{ $t(`App.network.${providerTag}`) }}
        </div>
        <div class="tc-user-netstatus" :style="{ 'background-color': netWorking ? '#009000' : '#d41400' }" />
        <transition name="netlist" :duration="400">
          <div class="tc-user-netlist" v-if="hoverNav === 1">
            <ul>
              <li
                v-for="(item,index) in networkSet"
                :key="index"
                @click="changeNetWork(index)">
                {{ $t(`App.network.${item.tag}`) }}
              </li>
            </ul>
          </div>
        </transition>
        <div class="tc-user-info" @mouseover="setHoverObjIndex(2)">
          {{ $tc('App.account', accountsCount, {count: accountsCount}) }}
        </div>
        <div class="tc-user-ctrl">
          <span @click.stop="goBack">{{ $t('App.back') }}</span>
        </div>
      </div>
      <router-view ref="view" v-if="!beforeRefresh" @routerinit="initRouterEl"/>
    </div>
    <div id="tc-mask" v-if="accountsDialogIsOpen">
      <div class="tc-mask-box">
        <span class="tc-mask-close" @click="endAddAccounts">{{ $t('App.close') }} X</span>
        <signin-account @done="endAddAccounts" @pass="endAddAccounts"></signin-account>
      </div>
    </div>
    <div id="tc-notice"></div>
  </div>
</template>

<script>
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex'
import routes from '@/router/routes.js'
import Hello from '@/components/Hello'
import LoadingAnimation from '@/components/common/gui/Loading'
import SigninAccount from '@/components/common/function/SigninAccount'
import networkSet from 'static/network.json'

const ETH_NETWORK_UPDATE_CYCLE = 10000

export default {
  name: 'App',
  data: () => {
    return {
      routes,
      beforeRefresh: false,
      hoverNav: 0,
      noticeBox: null,
      noticeBoxTimer: 0,
      networkSet,
      netWorking: true
    }
  },
  computed: {
    ...mapState({
      languageTag: state => state.languageTag,
      providerTag: state => state.providerTag,
      accountsDialogIsOpen: state => state.accounts.accountsDialogIsOpen
    }),
    ...mapGetters([
      'allAccounts',
      'accountsCount'
    ])
  },
  created () {
    this.update()
    setInterval(this.update, ETH_NETWORK_UPDATE_CYCLE)
  },
  mounted () {
    const noticeBox = this.$el.querySelector('#tc-notice')
    this.bindNoticeBox(noticeBox)
    window.addEventListener('resize', this.onresize)
  },
  methods: {
    ...mapActions([
      'updateEthereumInfo'
    ]),
    ...mapMutations([
      'setWeb3Provider',
      'setLanguage',
      'endAddAccounts',
      'bindNoticeBox'
    ]),
    goBack () {
      this.$router.push('/')
    },
    onresize () {
      const updateSize = this.$refs.view.updateSize
      if (typeof updateSize === 'function') {
        updateSize()
      }
    },
    setHoverObjIndex (index) {
      this.hoverNav = index
    },
    changeNetWork (index) {
      this.hoverNav = 0
      this.setWeb3Provider(this.networkSet[index])
      this.update()
      this.beforeRefresh = true
      this.$nextTick(() => {
        this.beforeRefresh = false
      })
    },
    initRouterEl (obj) {
      obj.updateSize = function () {
        obj.height = obj.$el.getBoundingClientRect().height
        obj.onMousewheel()
      }
      obj.onMousewheel = function (e) {
        if (e && e.cancelable) {
          obj.updateSize()
        }
        if (e && e.target && e.target.nodeName === 'TEXTAREA') {
          return
        }
        if (e) {
          obj.pageTranslateY += e.deltaY > 0 ? -90 : 90
        }
        if (obj.pageTranslateY > 0) {
          obj.pageTranslateY = 0
        }
        let contentH = window.innerHeight - 160
        if (obj.height + obj.pageTranslateY < contentH) {
          obj.pageTranslateY = Math.min(contentH - obj.height, 0)
        }
      }
    },
    update () {
      // const time = new Date()
      this.updateEthereumInfo().then(() => {
        this.netWorking = true
      }).catch(() => {
        this.netWorking = false
      })
      // .then(() => {
      //   const finTime = new Date()
      //   console.log(`Ethereum info update at ${time}. elapsed ${finTime - time} ms`)
      // })
    }
  },
  beforeDestroy () {
    window.removeEventListener('resize', this.onresize)
  },
  components: {
    Hello,
    LoadingAnimation,
    SigninAccount
  }
}
</script>

<style lang="stylus">
#tc-app
  font-family 'Avenir', Helvetica, Arial, sans-serif
  -webkit-font-smoothing antialiased
  -moz-osx-font-smoothing grayscale
  color #333
  display flex
nav
  color #fff
  width 200px
  height 100vh
  flex 0 0 200px
  background-color #014676
  box-shadow 4px 0 8px #0001
  position relative
  z-index 10
  transition flex 1s
  overflow hidden
.tc-nav-login
  flex 0 0 200px
.tc-nav-logo
  display block
  margin auto
  height 60px
  padding 30px 0
.tc-nav-title
  cursor pointer
  padding-left 20px
  height 50px
  line-height 50px
  opacity .6
  transition opacity 1s, background 0.6s
  margin 5px 0
.tc-nav-title-focus
  opacity 1
  color #014676
  background-color #fff
.tc-nav-lang
  position absolute
  left 10px
  bottom 10px
  font-size 14px
  span
    transition padding-bottom .4s
    cursor pointer
    padding-bottom 10px
    border-bottom solid 4px #fff
  .focus
    padding-bottom 6px

#tc-content
  position relative
  flex auto
  background-color #f0f4f8
#tc-user
  position relative
  z-index 9
  height 60px
  background-color #fff
  box-shadow 0 2px 4px #0001
  transition transform .4s
.tc-user-hide
  transform translateY(-70px)
.tc-user-net
  float left
  padding 0 20px
  margin-left 10px
  line-height 60px
  transition background-color .6s
.tc-user-net-hover
  background-color #eef4f9
.tc-user-netstatus
  width 8px
  height 8px
  border-radius 4px
  background-color green
  position absolute
  top 26px
  left 16px
  animation breath 2s infinite alternate
  transition background-color .6s
.tc-user-netlist
  position absolute
  padding-top 2px
  top 60px
  left 0
  margin 0 9px
  overflow hidden
  ul
    background-color #FFF
    border-radius 3px
    border solid 1px #ddd
    transition transform .4s
  li
    cursor pointer
    line-height 40px
    padding 0 20px
    &:nth-child(even)
      background-color #fbfbfb
.netlist-enter, .netlist-leave-to
  ul
    transform translate3d(0, -110%, 0)
.tc-user-info
  float right
  padding 0 20px
  margin-right 10px
  line-height 60px
  cursor pointer
  background-color #fff
  position relative
  z-index 11
  transition background-color .6s
.tc-user-ctrl
  line-height 60px
  float right
  transition .6s
  transform translateX(100%)
.tc-user-ctrl span
  padding 0 10px
  font-size 14px
  color #666
  cursor pointer
  transition color .6s
.tc-user-ctrl spanhover
  color #014676
.tc-user-ctrl-open .tc-user-info
  background-color #eef4f9
.tc-user-ctrl-open .tc-user-ctrl
  transform translateX(0)

#tc-mask
  position fixed
  width 100%
  height 100%
  z-index 19
  background-color #0008
.tc-mask-close
  line-height 30px
  color #fff
  cursor pointer
.tc-mask-box
  width 360px
  position absolute
  top 50%
  left 50%
  transform translate3d(-50%, -50%, 0)

#tc-notice
  position fixed
  width 100vw
  min-height 60px
  bottom 0
  left 0
  z-index 20
  box-shadow 0px -4px 4px #0001
  transition transform .6s
  transform translateY(110%)
  padding 20px
  box-sizing border-box
  line-height 20px
  color #fff

@keyframes breath
  from
    opacity 1
  to
    opacity .6

.fly-right-enter-active, .fly-right-leave-active
  transition transform 1s
.fly-right-enter, .fly-right-leave-to
  transform translateX(100%)

.fly-left-enter-active, .fly-left-leave-active
  transition transform 1s, opacity 1s
.fly-left-enter, .fly-left-leave-to
  transform translateX(-50%)
  opacity 0
</style>
