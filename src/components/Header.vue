<template>
  <div v-if="hasGroupIcons || hasSearch" class="v3-header">
    <div v-if="hasGroupIcons" class="v3-groups">
      <button
        v-for="group of orderedGroups"
        :key="group.key"
        type="button"
        class="v3-group"
        :class="{
          'v3-is-hidden': !icons[group.key],
        }"
        @click="updateActiveGroup(group.key)"
      >
        <span :title="group.title" class="v3-icon">
          <img :src="icons[group.key]" alt="" />
        </span>
      </button>
    </div>
    <div v-if="hasGroupIcons && hasSearch" class="v3-spacing" />
    <div v-if="hasSearch" class="v3-search">
      <input
        v-model="searchValue"
        type="text"
        :placeholder="placeholder"
        :class="searchClass"
      />
    </div>
  </div>
</template>

<script lang="ts">
/**
 * External dependencies
 */
import { computed, type ComputedRef, defineComponent, inject } from 'vue'

/**
 * Internal dependencies
 */
import { Group, Store } from '../types'
import { snakeToCapitalizedCase } from '../helpers'

/**
 * Group/Category Images
 */
import smileys_emotion from '../svgs/groups/smileys_emotion.svg'
import people_body from '../svgs/groups/people_body.svg'
import animals_nature from '../svgs/groups/animals_nature.svg'
import food_drink from '../svgs/groups/food_drink.svg'
import activities from '../svgs/groups/activities.svg'
import travel_places from '../svgs/groups/travel_places.svg'
import objects from '../svgs/groups/objects.svg'
import symbols from '../svgs/groups/symbols.svg'
import flags from '../svgs/groups/flags.svg'
import recent from '../svgs/groups/recent.svg'

export default defineComponent({
  // eslint-disable-next-line vue/no-reserved-component-names
  name: 'Header',
  setup(props) {
    const { state, updateSearch, updateActiveGroup } = inject('store') as Store

    const hasSearch = computed(() => !state.options.hideSearch)
    const hasGroupIcons = computed(() => !state.options.hideGroupIcons)
    const orderedKeys = JSON.parse(JSON.stringify(state.orderedGroupKeys))
    const placeholder = computed(
      () => state.options.staticTexts.placeholder || ''
    )

    const searchValue = computed({
      get: () => state.search,
      set: (value: string) => updateSearch(value),
    })

    const groups: Group[] = [
      ...state.groups,
      ...Object.keys(state.options.additionalGroups).map((g) => ({
        key: g,
        title: state.options.groupNames[g]
          ? state.options.groupNames[g]
          : snakeToCapitalizedCase(g),
      })),
    ] as Group[]

    const orderedGroups: Group[] = []

    orderedKeys.forEach((key: string) => {
      const index = groups.findIndex((group) => group.key === key)
      if (index === -1) return
      orderedGroups.push(groups[index])
      groups.splice(index, 1)
    })

    return {
      orderedGroups,
      orderedKeys,
      searchValue,
      updateActiveGroup,
      hasSearch,
      hasGroupIcons,
      searchClass: state.options.searchClass,
      placeholder,
      icons: {
        smileys_emotion,
        people_body,
        animals_nature,
        food_drink,
        activities,
        travel_places,
        objects,
        symbols,
        flags,
        ...state.options.groupIcons,
        recent,
      } as Record<string, string>,
    }
  },
})
</script>
