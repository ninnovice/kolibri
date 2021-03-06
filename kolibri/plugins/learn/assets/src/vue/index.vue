<template>

  <core-base :topLevelPageName="topLevelPageName" :appBarTitle="$tr('learnTitle')">
    <div slot="app-bar-actions">
      <channel-switcher @switch="switchChannel"/>
      <!--
      <ui-icon-button
        icon="search"
        type="secondary"
        color="white"
        :ariaLabel="$tr('search')"
        @click="toggleSearch"/>
        -->
    </div>
    <div slot="content">
      <tabs :items="learnTabs" type="icon-and-text" @tabclicked="handleTabClick"/>
      <breadcrumbs/>
      <component :is="currentPage"/>
    </div>
    <div slot="extra" class="search-pane" v-show="searchOpen">
      <search-widget :showTopics="exploreMode"/>
    </div>

  </core-base>

</template>


<script>

  const constants = require('../state/constants');
  const PageNames = constants.PageNames;
  const PageModes = constants.PageModes;
  const getters = require('../state/getters');
  const actions = require('../actions');
  const store = require('../state/store');
  const TopLevelPageNames = require('kolibri.coreVue.vuex.constants').TopLevelPageNames;

  module.exports = {
    $trNameSpace: 'learn',
    $trs: {
      learnTitle: 'Learn',
      recommended: 'Recommended',
      topics: 'Topics',
      search: 'search',
    },
    components: {
      'search-widget': require('./search-widget'),
      'explore-page': require('./explore-page'),
      'content-page': require('./content-page'),
      'learn-page': require('./learn-page'),
      'scratchpad-page': require('./scratchpad-page'),
      'content-unavailable-page': require('./content-unavailable-page'),
      'core-base': require('kolibri.coreVue.components.coreBase'),
      'ui-icon-button': require('keen-ui/src/UiIconButton'),
      'channel-switcher': require('kolibri.coreVue.components.channelSwitcher'),
      'breadcrumbs': require('./breadcrumbs'),
      'tabs': require('kolibri.coreVue.components.tabs'),
    },
    methods: {
      toggleSearch() {
        this.toggleSearch();
      },
      switchChannel(channelId) {
        let rootPage;
        if (this.pageMode === constants.PageModes.EXPLORE) {
          rootPage = constants.PageNames.EXPLORE_CHANNEL;
        } else {
          rootPage = constants.PageNames.LEARN_CHANNEL;
        }
        this.clearSearch();
        this.$router.push({
          name: rootPage,
          params: { channel_id: channelId },
        });
      },
      handleTabClick(tabIndex) {
        switch (tabIndex) {
          case 0:
            this.$router.push({ name: constants.PageNames.LEARN_ROOT });
            return;

          case 1:
            this.$router.push({ name: constants.PageNames.EXPLORE_ROOT });
            return;

          default:
            return;
        }
      },
    },
    computed: {
      topLevelPageName() {
        return TopLevelPageNames.LEARN;
      },
      currentPage() {
        if (this.pageName === PageNames.EXPLORE_CHANNEL ||
          this.pageName === PageNames.EXPLORE_TOPIC) {
          return 'explore-page';
        }
        if (this.pageName === PageNames.EXPLORE_CONTENT ||
          this.pageName === PageNames.LEARN_CONTENT) {
          return 'content-page';
        }
        if (this.pageName === PageNames.LEARN_CHANNEL) {
          return 'learn-page';
        }
        if (this.pageName === PageNames.SCRATCHPAD) {
          return 'scratchpad-page';
        }
        if (this.pageName === PageNames.CONTENT_UNAVAILABLE) {
          return 'content-unavailable-page';
        }
        return null;
      },
      exploreMode() {
        return this.pageMode === PageModes.EXPLORE;
      },
      learnTabs() {
        const isRecommended = this.pageName === constants.PageNames.LEARN_CHANNEL;
        return [{
          title: this.$tr('recommended'),
          icon: 'forum',
          selected: isRecommended,
          disabled: false,
        }, {
          title: this.$tr('topics'),
          icon: 'folder',
          selected: !isRecommended,
          disabled: false,
        }];
      },
    },
    vuex: {
      getters: {
        pageMode: getters.pageMode,
        pageName: state => state.pageName,
        searchOpen: state => state.searchOpen,
      },
      actions: {
        toggleSearch: actions.toggleSearch,
        clearSearch: actions.clearSearch,
      },
    },
    store, // make this and all child components aware of the store
  };

</script>


<style lang="stylus" scoped>

  @require '~kolibri.styles.definitions'
  @require 'learn.styl'

  .search-pane
    background-color: $core-bg-canvas
    overflow-y: scroll
    position: fixed
    top: 0
    left: 0
    height: 100%
    width: 100%
    z-index: 1
    @media screen and (min-width: $portrait-breakpoint + 1)
      padding-left: $nav-width

  .content
    margin: auto

</style>
