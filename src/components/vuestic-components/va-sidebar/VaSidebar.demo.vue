<template>
  <VbDemo>
    <VbCard title="Sidebar">
      <va-sidebar
        style="position: static; height: auto;"
        :minimized="minimized"
      >
        <template slot="menu">
          <template v-for="(item, index) in items">
            <sidebar-link-group
              :key="index"
              :icon="[ 'sidebar-menu-item-icon vuestic-iconset', item.meta.iconClass ]"
              v-if="item.children">
              <span slot="title">{{ item.displayName }}</span>
              <sidebar-link
                v-for="(subMenuItem, index) in item.children"
                :key="index"
                :to="{ name: subMenuItem.name }"
              >
                <div slot="title">
                  <span>{{ subMenuItem.displayName }}</span>
                </div>
              </sidebar-link>
            </sidebar-link-group>

            <sidebar-link
              v-else
              :key="index"
              :icon="[ 'sidebar-menu-item-icon vuestic-iconset', item.meta.iconClass ]"
              :to="{ name: item.name }">
              <span slot="title">{{ item.displayName }}</span>
            </sidebar-link>
          </template>
        </template>
      </va-sidebar>
    </VbCard>

    <VbCard title="Sidebar minimised">
      <va-sidebar
        style="position: static; height: auto;"
        minimized
      >
        <template slot="menu">
          <template v-for="(item, index) in items">
            <sidebar-link-group
              :key="index"
              :icon="[ 'sidebar-menu-item-icon vuestic-iconset', item.meta.iconClass ]"
              v-if="item.children"
              minimized
            >
              <span slot="title">{{ item.displayName }}</span>
              <sidebar-link
                v-for="(subMenuItem, index) in item.children"
                :key="index"
                :to="{ name: subMenuItem.name }"
                minimized
              >
                <div slot="title">
                  <span>{{ subMenuItem.displayName }}</span>
                </div>
              </sidebar-link>
            </sidebar-link-group>

            <sidebar-link
              v-else
              :key="index"
              :icon="[ 'sidebar-menu-item-icon vuestic-iconset', item.meta.iconClass ]"
              :to="{ name: item.name }"
              minimized
            >
              <span slot="title">{{ item.displayName }}</span>
            </sidebar-link>
          </template>
        </template>
      </va-sidebar>
    </VbCard>
  </VbDemo>
</template>

<script>
import VaSidebar from '../../vuestic-components/va-sidebar/VaSidebar'
import SidebarLinkGroup from './VaSidebarLinkGroup'
import SidebarLink from './VaSidebarLink'
import { navigationRoutes } from './navigation-router-example'

export default {
  components: {
    VaSidebar,
    SidebarLinkGroup,
    SidebarLink,
  },
  data () {
    return {
      value: 60,
      icon: 'iconicstroke iconicstroke-info',
      theme: 'info',
      items: navigationRoutes.routes,
      hoverState: false,
      minimized: false,
    }
  },
}
</script>
