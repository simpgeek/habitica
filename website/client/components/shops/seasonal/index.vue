<template lang="pug">
  .row.seasonal
    .standard-sidebar
      .form-group
        input.form-control.input-search(type="text", v-model="searchText", :placeholder="$t('search')")

      .form
        h2(v-once) {{ $t('filter') }}
        .form-group
          .form-check(
            v-for="category in filterCategories",
            :key="category.key",
          )
            label.custom-control.custom-checkbox
              input.custom-control-input(type="checkbox", v-model="viewOptions[category.key].selected")
              span.custom-control-indicator
              span.custom-control-description(v-once) {{ category.value }}

        div.form-group.clearfix
          h3.float-left(v-once) {{ $t('hidePinned') }}
          toggle-switch.float-right.no-margin(
            :label="''",
            v-model="hidePinned",
          )
    .standard-page
      div.featuredItems
        .background(:class="{opened: seasonal.opened}")
          div.npc
            div.featured-label
              span.rectangle
              span.text Leslie
              span.rectangle
          div.content(v-if="!seasonal.opened")
            div.featured-label.with-border.closed
              span.rectangle
              span.text(v-once, v-html="seasonal.notes")
              span.rectangle
          div.content(v-else-if="seasonal.featured.items.length !== 0")
            div.featured-label.with-border(v-if='!featuredGearBought')
              span.rectangle
              span.text(v-once) {{ $t('featuredset', { name: seasonal.featured.text }) }}
              span.rectangle

            div.items.margin-center
              shopItem(
                v-for="item in seasonal.featured.items",
                :key="item.key",
                :item="item",
                :price="item.value",
                :emptyItem="false",
                :popoverPosition="'top'",
                :showEventBadge="false",
                @click="itemSelected(item)"
              )

      h1.mb-0.page-header(v-once) {{ $t('seasonalShop') }}

      .clearfix(v-if="seasonal.opened")
        h2.float-left
          | {{ $t('classArmor') }}

        div.float-right
          span.dropdown-label {{ $t('sortBy') }}
          b-dropdown(:text="$t(selectedSortItemsBy)", right=true)
            b-dropdown-item(
              v-for="sort in sortItemsBy",
              @click="selectedSortItemsBy = sort",
              :active="selectedSortItemsBy === sort",
              :key="sort"
            ) {{ $t(sort) }}


      div(
        v-for="(groupSets, categoryGroup) in getGroupedCategories(categories)",
      )
        h3.classgroup
          span.svg-icon.inline(v-html="icons[categoryGroup]")
          span.name(:class="categoryGroup") {{ getClassName(categoryGroup) }}

        div.grouped-parent
          div.group(
            v-for="category in groupSets"
          )
            h3 {{ category.text }}
            div.items
              shopItem(
                v-for="item in seasonalItems(category, selectedSortItemsBy, searchTextThrottled, viewOptions, hidePinned)",
                :key="item.key",
                :item="item",
                :price="item.value",
                :emptyItem="false",
                :popoverPosition="'top'",
                :showEventBadge="false",
                @click="itemSelected(item)"
              )
                template(slot="itemBadge", scope="ctx")
                  span.badge.badge-pill.badge-item.badge-svg(
                    :class="{'item-selected-badge': ctx.item.pinned, 'hide': !ctx.item.pinned}",
                    @click.prevent.stop="togglePinned(ctx.item)"
                  )
                    span.svg-icon.inline.icon-12.color(v-html="icons.pin")
</template>

<style lang="scss">
  @import '~client/assets/scss/colors.scss';
  @import '~client/assets/scss/variables.scss';

  .badge-svg {
    left: calc((100% - 18px) / 2);
    cursor: pointer;
    color: $gray-400;
    background: $white;
    padding: 4.5px 6px;

    &.item-selected-badge {
      background: $purple-300;
      color: $white;
    }
  }

  span.badge.badge-pill.badge-item.badge-svg:not(.item-selected-badge) {
    color: #a5a1ac;
  }

  span.badge.badge-pill.badge-item.badge-svg.hide {
    display: none;
  }

  .item:hover {
    span.badge.badge-pill.badge-item.badge-svg.hide {
      display: block;
    }
  }

  .icon-12 {
    width: 12px;
    height: 12px;
  }

  .hand-cursor {
    cursor: pointer;
  }


  .featured-label {
    margin: 24px auto;
  }

  .bordered {
    border-radius: 2px;
    background-color: #f9f9f9;
    margin-bottom: 24px;
    padding: 24px 24px 10px;
  }

  .group {
    display: inline-block;
    width: 50%;
    margin-bottom: 24px;


    .items {
      border-radius: 2px;
      background-color: #edecee;
      display: inline-block;
      padding: 8px;
    }

    .item-wrapper {
      margin-bottom: 0;
    }

    .items > div:not(:last-of-type) {
      margin-right: 16px;
    }
  }

  .seasonal {
    .standard-page {
      position: relative;
    }

    h3.classgroup {
      line-height: 1.5;
      display: flex;
      align-items: center;

      span.svg-icon.inline {
        height: 24px;
        width: 24px;
        margin-right: 8px;
      }
    }

    .healer {
      color: #cf8229;
    }

    .rogue {
      color: #4f2a93;
    }

    .warrior {
      color: #b01515;
    }

    .wizard {
      color: #1f6ea2;
    }

    .featuredItems {
      height: 216px;

      .background {
        background: url('~assets/images/npc/normal/seasonal_shop_closed_background.png');

        background-repeat: repeat-x;

        width: 100%;
        height: 216px;
        position: absolute;

        top: 0;
        left: 0;

        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
      }
      .background.opened {
        background: url('~assets/images/npc/#{$npc_seasonal_flavor}/seasonal_shop_opened_background.png');

        background-repeat: repeat-x;
      }

      .content {
        display: flex;
        flex-direction: column;
      }

      .npc {
        position: absolute;
        left: 0;
        top: 0;
        width: 100%;
        height: 216px;
        background: url('~assets/images/npc/normal/seasonal_shop_closed_npc.png');
        background-repeat: no-repeat;

        .featured-label {
          position: absolute;
          bottom: -14px;
          margin: 0;
          left: 60px;
        }
      }

      .opened .npc{
        background: url('~assets/images/npc/#{$npc_seasonal_flavor}/seasonal_shop_opened_npc.png');
        background-repeat: no-repeat;
      }
    }
  }
</style>


<script>
  import {mapState} from 'client/libs/store';

  import ShopItem from '../shopItem';
  import Item from 'client/components/inventory/item';
  import CountBadge from 'client/components/ui/countBadge';
  import ItemRows from 'client/components/ui/itemRows';
  import toggleSwitch from 'client/components/ui/toggleSwitch';
  import Avatar from 'client/components/avatar';

  import bPopover from 'bootstrap-vue/lib/components/popover';
  import bDropdown from 'bootstrap-vue/lib/components/dropdown';
  import bDropdownItem from 'bootstrap-vue/lib/components/dropdown-item';

  import svgPin from 'assets/svg/pin.svg';
  import svgWarrior from 'assets/svg/warrior.svg';
  import svgWizard from 'assets/svg/wizard.svg';
  import svgRogue from 'assets/svg/rogue.svg';
  import svgHealer from 'assets/svg/healer.svg';

  import _filter from 'lodash/filter';
  import _map from 'lodash/map';
  import _mapValues from 'lodash/mapValues';
  import _forEach from 'lodash/forEach';
  import _sortBy from 'lodash/sortBy';
  import _throttle from 'lodash/throttle';
  import _groupBy from 'lodash/groupBy';
  import _reverse from 'lodash/reverse';

  import isPinned from 'common/script/libs/isPinned';
  import getOfficialPinnedItems from 'common/script/libs/getOfficialPinnedItems';

  import i18n from 'common/script/i18n';

  import shops from 'common/script/libs/shops';

  export default {
    components: {
      ShopItem,
      Item,
      CountBadge,
      ItemRows,
      toggleSwitch,

      bPopover,
      bDropdown,
      bDropdownItem,

      Avatar,
    },
    watch: {
      searchText: _throttle(function throttleSearch () {
        this.searchTextThrottled = this.searchText.toLowerCase();
      }, 250),
    },
    data () {
      return {
        viewOptions: {},
        searchText: null,
        searchTextThrottled: null,

        icons: Object.freeze({
          pin: svgPin,
          warrior: svgWarrior,
          wizard: svgWizard,
          rogue: svgRogue,
          healer: svgHealer,
        }),

        gearTypesToStrings: Object.freeze({ // TODO use content.itemList?
          weapon: i18n.t('weaponCapitalized'),
          shield: i18n.t('offhandCapitalized'),
          head: i18n.t('headgearCapitalized'),
          armor: i18n.t('armorCapitalized'),
          headAccessory: i18n.t('headAccessoryCapitalized'),
          body: i18n.t('body'),
          back: i18n.t('back'),
          eyewear: i18n.t('eyewear'),
        }),

        sortItemsBy: ['AZ'],
        selectedSortItemsBy: 'AZ',

        hidePinned: false,
        featuredGearBought: false,
      };
    },
    computed: {
      ...mapState({
        content: 'content',
        user: 'user.data',
        userStats: 'user.data.stats',
      }),

      usersOfficalPinnedItems () {
        return getOfficialPinnedItems(this.user);
      },

      seasonal () {
        let seasonal = shops.getSeasonalShop(this.user);

        let itemsNotOwned = seasonal.featured.items.filter(item => {
          return !this.user.items.gear.owned[item.key];
        });
        seasonal.featured.items = itemsNotOwned;

        // If we are out of gear, show the spells
        // @TODO: Open this up when they are available.
        // @TODO: add dates to check instead?
        // if (seasonal.featured.items.length === 0) {
        //   this.featuredGearBought = true;
        //   seasonal.featured.items = seasonal.featured.items.concat(seasonal.categories[0].items);
        // }

        return seasonal;
      },
      seasonalCategories () {
        return this.seasonal.categories;
      },
      categories () {
        if (this.seasonalCategories) {
          return _reverse(_sortBy(this.seasonalCategories, (c) => {
            if (c.event) {
              return c.event.start;
            } else {
              return -1;
            }
          }));
        } else {
          return [];
        }
      },
      filterCategories () {
        if (this.content) {
          let equipmentList = _mapValues(this.gearTypesToStrings, (value, key) => {
            return {
              key,
              value,
            };
          });

          _forEach(equipmentList, (value) => {
            this.$set(this.viewOptions, value.key, {
              selected: true,
            });
          });

          return equipmentList;
        } else {
          return [];
        }
      },
    },
    methods: {
      getClassName (classType) {
        if (classType === 'wizard') {
          return this.$t('mage');
        } else {
          return this.$t(classType);
        }
      },
      seasonalItems (category, sortBy, searchBy, viewOptions, hidePinned) {
        let result = _map(category.items, (e) => {
          return {
            ...e,
            pinned: isPinned(this.user, e, this.usersOfficalPinnedItems),
          };
        });

        result = _filter(result, (i) => {
          if (hidePinned && i.pinned) {
            return false;
          }

          if (viewOptions[i.type] && !viewOptions[i.type].selected) {
            return false;
          }

          return !searchBy || i.text.toLowerCase().indexOf(searchBy) !== -1;
        });

        switch (sortBy) {
          case 'AZ': {
            result = _sortBy(result, ['text']);

            break;
          }
        }

        return result;
      },
      getGroupedCategories (categories) {
        let spellCategory = _filter(categories, (c) => {
          return c.identifier === 'spells';
        })[0];

        let setCategories = _filter(categories, 'specialClass');

        let result = _groupBy(setCategories, 'specialClass');

        if (spellCategory) {
          result.spells = [
            spellCategory,
          ];
        }

        return result;
      },
      isGearLocked (gear) {
        if (gear.value > this.userStats.gp) {
          return true;
        }

        return false;
      },
      togglePinned (item) {
        if (!this.$store.dispatch('user:togglePinnedItem', {type: item.pinType, path: item.path})) {
          this.$parent.showUnpinNotification(item);
        }
      },
      itemSelected (item) {
        if (!item.locked) {
          this.$root.$emit('buyModal::showItem', item);
        }
      },
    },
  };
</script>
