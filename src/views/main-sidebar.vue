<template>
  <aside class="site-sidebar" :class="'site-sidebar--' + sidebarLayoutSkin">
    <div class="site-sidebar__inner">
      <el-menu
        :default-active="menuActiveName || 'home'"
        :collapse="sidebarFold"
        :collapseTransition="false"
        class="site-sidebar__menu">
        <el-menu-item index="home" @click="$router.push({ name: 'home' })">
          <icon-svg name="shouye" class="site-sidebar__menu-icon"></icon-svg>
          <span slot="title">首页</span>
        </el-menu-item>
        <!--<el-submenu index="demo">-->
          <!--<template slot="title">-->
            <!--<icon-svg name="shoucang" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span>demo</span>-->
          <!--</template>-->
          <!--<el-menu-item index="demo-echarts" @click="$router.push({ name: 'demo-echarts' })">-->
            <!--<icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">echarts</span>-->
          <!--</el-menu-item>-->
          <!--<el-menu-item index="demo-ueditor" @click="$router.push({ name: 'demo-ueditor' })">-->
            <!--<icon-svg name="editor" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">ueditor</span>-->
          <!--</el-menu-item>-->
        <!--</el-submenu>-->
        <!--<el-submenu index="sign">-->
          <!--<template slot="title">-->
            <!--<icon-svg name="log" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span>数据管理</span>-->
          <!--</template>-->
          <!--<el-menu-item index="demo-echarts" @click="$router.push({ name: 'studentPlan' })">-->
            <!--<icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">招生目标</span>-->
          <!--</el-menu-item>-->
          <!--<el-menu-item index="demo-echarts" @click="$router.push({ name: 'sign' })">-->
            <!--<icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">报名成单数据</span>-->
          <!--</el-menu-item>-->
          <!--<el-menu-item index="demo-echarts" @click="$router.push({ name: 'refund' })">-->
            <!--<icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">退费监控</span>-->
          <!--</el-menu-item>-->
          <!--<el-menu-item index="demo-echarts" @click="$router.push({ name: 'getData' })">-->
            <!--<icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">获取数据</span>-->
          <!--</el-menu-item>-->
          <!--<el-menu-item index="demo-echarts" @click="$router.push({ name: 'baiduChannel' })">-->
            <!--<icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">百度渠道获取数据</span>-->
          <!--</el-menu-item>-->
          <!--<el-menu-item index="demo-echarts" @click="$router.push({ name: 'courseData' })">-->
            <!--<icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">文化课报名数据</span>-->
          <!--</el-menu-item>-->
        <!--</el-submenu>-->
        <!--<el-submenu index="organ">-->
          <!--<template slot="title">-->
            <!--<icon-svg name="log" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span>机构管理</span>-->
          <!--</template>-->
          <!--<el-menu-item index="demo-echarts" @click="$router.push({ name: 'organ' })">-->
            <!--<icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">机构管理</span>-->
          <!--</el-menu-item>-->
          <!--<el-menu-item index="demo-echarts" @click="$router.push({ name: 'member' })">-->
            <!--<icon-svg name="tubiao" class="site-sidebar__menu-icon"></icon-svg>-->
            <!--<span slot="title">成员管理</span>-->
          <!--</el-menu-item>-->
        <!--</el-submenu>-->
        <sub-menu
          v-for="menu in menuList"
          :key="menu.menuId"
          :menu="menu"
          :dynamicMenuRoutes="dynamicMenuRoutes">
        </sub-menu>
      </el-menu>
    </div>
  </aside>
</template>

<script>
  import SubMenu from './main-sidebar-sub-menu'
  import {isURL} from '@/utils/validate'

  export default {
    data() {
      return {
        dynamicMenuRoutes: []
      }
    },
    components: {
      SubMenu
    },
    computed: {
      sidebarLayoutSkin: {
        get() {
          return this.$store.state.common.sidebarLayoutSkin
        }
      },
      sidebarFold: {
        get() {
          return this.$store.state.common.sidebarFold
        }
      },
      menuList: {
        get() {
          return this.$store.state.common.menuList
        },
        set(val) {
          this.$store.commit('common/updateMenuList', val)
        }
      },
      menuActiveName: {
        get() {
          return this.$store.state.common.menuActiveName
        },
        set(val) {
          this.$store.commit('common/updateMenuActiveName', val)
        }
      },
      mainTabs: {
        get() {
          return this.$store.state.common.mainTabs
        },
        set(val) {
          this.$store.commit('common/updateMainTabs', val)
        }
      },
      mainTabsActiveName: {
        get() {
          return this.$store.state.common.mainTabsActiveName
        },
        set(val) {
          this.$store.commit('common/updateMainTabsActiveName', val)
        }
      }
    },
    watch: {
      $route: 'routeHandle'
    },
    created() {
      this.menuList = JSON.parse(sessionStorage.getItem('menuList') || '[]')
      this.dynamicMenuRoutes = JSON.parse(sessionStorage.getItem('dynamicMenuRoutes') || '[]')
      this.routeHandle(this.$route)
    },
    methods: {
      // 路由操作
      routeHandle(route) {
        if (route.meta.isTab) {
          // tab选中, 不存在先添加
          var tab = this.mainTabs.filter(item => item.name === route.name)[0]
          if (!tab) {
            if (route.meta.isDynamic) {
              route = this.dynamicMenuRoutes.filter(item => item.name === route.name)[0]
              if (!route) {
                return console.error('未能找到可用标签页!')
              }
            }
            tab = {
              menuId: route.meta.menuId || route.name,
              name: route.name,
              title: route.meta.title,
              type: isURL(route.meta.iframeUrl) ? 'iframe' : 'module',
              iframeUrl: route.meta.iframeUrl || '',
              params: route.params,
              query: route.query
            }
            this.mainTabs = this.mainTabs.concat(tab)
          }
          this.menuActiveName = tab.menuId + ''
          this.mainTabsActiveName = tab.name
        }
      }
    }
  }
</script>
