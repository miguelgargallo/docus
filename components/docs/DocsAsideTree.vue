<script setup lang="ts">
import type { PropType } from 'vue'

const props = defineProps({
  links: {
    type: Array as PropType<any>,
    default: () => []
  },
  level: {
    type: Number,
    default: 0
  },
  max: {
    type: Number,
    default: null
  },
  parent: {
    type: Object as PropType<any>,
    default: null
  }
})

const route = useRoute()
const { config } = useDocus()

const collapsedMap = useState(`docus-docs-aside-collapse-map-${props.parent?._path || '/'}`, () => {
  if (props.level === 0) {
    return {}
  }
  return (props.links as any [])
    .filter(link => !!link.children)
    .reduce((map, link) => {
      map[link._path] = true
      return map
    }, {})
})

const isActive = (link: any) => {
  return route.path === link._path
}

const isCollapsed = (link: any) => {
  if (link.children) {
    // Directory has been toggled manually, use its state
    if (typeof collapsedMap.value[link._path] !== 'undefined') {
      return collapsedMap.value[link._path]
    }

    // Check if aside.collapsed has been set in YML
    if ([true, false].includes(link?.aside?.collapsed)) { return link.aside.collapsed }

    // Return value grabbed from the link
    if (link?.collapsed) { return link?.collapsed }

    if (config?.value?.aside?.collapsed) { return config.value.aside?.collapsed }
  }

  return false
}

const toggleCollapse = (link: any) => (collapsedMap.value[link._path] = !isCollapsed(link))

const hasNesting = computed(() => props.links.some((link: any) => link.children))
</script>

<template>
  <ul class="docs-aside-tree">
    <li
      v-for="link in links"
      :key="link._path"
      :class="{
        'has-parent-icon': parent?.icon,
        'has-children': level > 0 && link.children,
        'bordered': level > 0 || !hasNesting,
        'active': isActive(link),
      }"
    >
      <button v-if="link.children" class="title-collapsible-button" @click="toggleCollapse(link)">
        <span class="content">
          <Icon v-if="link?.navigation?.icon || link.icon" :name="link?.navigation?.icon || link.icon" class="icon" />
          <span>{{ link?.navigation?.title || link.title || link._path }}</span>
        </span>
        <span>
          <Icon :name="isCollapsed(link) ? 'lucide:chevrons-up-down' : 'lucide:chevrons-down-up'" class="collapsible-icon" />
        </span>
      </button>

      <NuxtLink
        v-else
        :to="link.redirect ? link.redirect : link._path"
        class="link"
        :exact="link.exact"
        :class="{
          'padded': level > 0 || !hasNesting,
          'active': isActive(link),
        }"
      >
        <span class="content">
          <Icon v-if="link?.navigation?.icon || link.icon" :name="link?.navigation?.icon || link.icon" class="icon" />
          <span>{{ link?.navigation?.title || link.title || link._path }}</span>
        </span>
      </NuxtLink>

      <DocsAsideTree
        v-show="!isCollapsed(link)"
        v-if="link.children?.length && (max === null || level + 1 < max)"
        :links="link.children"
        :level="level + 1"
        :parent="link"
        :max="max"
        class="recursive"
      />
    </li>
  </ul>
</template>

<style scoped lang="ts">
css({
  '.docs-aside-tree': {
    li: {
      '&.bordered': {
        borderLeft: '1px solid {elements.border.primary.static}',
        '&:hover': {
          borderColor: '{elements.border.primary.hover}'
        },
        '&.active': {
          borderColor: '{color.primary.400}',
          '@dark': {
            borderColor: '{color.primary.600}'
          },
        },
        '&.has-children': {
          paddingLeft: '{space.4}'
        },
        '&.has-parent-icon': {
          marginLeft: '{space.2}'
        }
      }
    },
    '.recursive': {
      padding: '{space.2} 0'
    },
    '.title-collapsible-button': {
      display: 'flex',
      alignItems: 'center',
      justifyContent: 'space-between',
      padding: '{space.rem.375} 0',
      fontSize: '{text.sm.fontSize}',
      lineHeight: '{text.sm.lineHeight}',
      fontWeight: '{fontWeight.semibold}',
      width: "100%",
      color: '{color.gray.900}',
      '@dark': {
        color: '{color.gray.50}'
      },
      '.content': {
        display: 'flex',
        alignItems: 'center',
        '.icon': {
          width: '{space.4}',
          height: '{space.4}',
          marginRight: '{space.2}'
        }
      },
      '.collapsible-icon': {
        width: '{space.3}',
        height: '{space.3}',
        color: '{color.gray.400}',
        '@dark': {
          color: '{color.gray.500}',
        }
      }
    },
    '.link': {
      display: 'flex',
      alignItems: 'center',
      justifyContent: 'space-between',
      padding: '{space.rem.375} 0',
      fontSize: '{text.sm.fontSize}',
      lineHeight: '{text.sm.lineHeight}',
      color: '{color.gray.500}',
      '&:hover': {
        color: '{color.gray.900}',
      },
      '@dark': {
        '&:not(.active)': {
          color: '{color.gray.400}',
          '&:hover': {
            color: '{color.gray.50}',
          }
        }
      },
      '&.padded': {
        paddingLeft: '{space.4}'
      },
      '&.active': {
        color: '{color.primary.500}',
        fontWeight: '{fontWeight.medium}'
      },
      '.content': {
        display: 'inline-flex',
        alignItems: 'center'
      },
      '.icon': {
        width: '{space.4}',
        height: '{space.4}',
        marginRight: '{space.1}'
      }
    }
  }
})
</style>
