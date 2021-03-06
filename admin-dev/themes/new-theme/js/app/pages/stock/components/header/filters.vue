<!--**
 * 2007-2017 PrestaShop
 *
 * NOTICE OF LICENSE
 *
 * This source file is subject to the Open Software License (OSL 3.0)
 * that is bundled with this package in the file LICENSE.txt.
 * It is also available through the world-wide-web at this URL:
 * https://opensource.org/licenses/OSL-3.0
 * If you did not receive a copy of the license and are unable to
 * obtain it through the world-wide-web, please send an email
 * to license@prestashop.com so we can send you a copy immediately.
 *
 * DISCLAIMER
 *
 * Do not edit or add to this file if you wish to upgrade PrestaShop to newer
 * versions in the future. If you wish to customize PrestaShop for your
 * needs please refer to http://www.prestashop.com for more information.
 *
 * @author    PrestaShop SA <contact@prestashop.com>
 * @copyright 2007-2017 PrestaShop SA
 * @license   https://opensource.org/licenses/OSL-3.0 Open Software License (OSL 3.0)
 * International Registered Trademark & Property of PrestaShop SA
 *-->
<template>
  <div>
    <button class="search-input collapse-button" type="button" data-toggle="collapse" data-target="#filters">
      <i class="material-icons m-r-1">filter_list</i>
      <i class="material-icons pull-right ">keyboard_arrow_down</i>
      {{trans('button_advanced_filter')}}
    </button>
    <div class="collapse" id="filters">
      <div class="row">
        <div class="col-md-6">
          <div v-if="isOverview" class="p-y-2 p-x-2">
            <h2>{{trans('filter_suppliers')}}</h2>
            <FilterComponent
              :placeholder="trans('filter_search_suppliers')"
              :list="this.$store.getters.suppliers"
              itemID="supplier_id"
              label="name"
              @active="onFilterActive"
            />
          </div>
          <div v-else class="p-y-2 p-x-2">
            <h2>{{trans('filter_movements_type')}}</h2>
            <PSSelect :items="movementsTypes" itemID="id_stock_mvt_reason" itemName="name" @change="onChange">
              {{trans('none')}}
            </PSSelect>
            <h2 class="m-t-2">{{trans('filter_movements_employee')}}</h2>
            <PSSelect :items="employees" itemID="id_employee" itemName="name" @change="onChange">
             {{trans('none')}}
            </PSSelect>
            <h2 class="m-t-2">{{trans('filter_movements_period')}}</h2>
            <form class="row">
              <div class="col-md-6">
                <label>{{trans('filter_datepicker_from')}}</label>
                <PSDatePicker @dpChange="onDpChange" data-type="sup"/>
              </div>
              <div class="col-md-6">
                <label>{{trans('filter_datepicker_to')}}</label>
                <PSDatePicker @dpChange="onDpChange" data-type="inf" />
              </div>
            </form>
          </div>
        </div>
        <div class="col-md-6">
          <div class="p-y-2 p-x-2">
            <h2>{{trans('filter_categories')}}</h2>
            <FilterComponent
              :placeholder="trans('filter_search_category')"
              :list="categoriesList"
              itemID="id_category"
              label="name"
              @active="onFilterActive"
            />
          </div>
        </div>
        <PSButton type="button" class="pull-right m-y-2 m-x-2" :primary="true" :disabled="disabled" @click="onClick">
          <i class="material-icons m-r-1">filter_list</i>
          {{trans('button_apply_advanced_filter')}}
        </PSButton>
      </div>
    </div>
  </div>
</template>

<script>
  import FilterComponent from './filters/filter-component';
  import PSSelect from 'app/widgets/ps-select';
  import PSButton from 'app/widgets/ps-button';
  import PSDatePicker from 'app/widgets/ps-datepicker';
  import _ from 'lodash';

  export default {
    computed: {
      isOverview() {
        return this.$route.name === 'overview';
      },
      employees() {
        return this.$store.state.employees;
      },
      movementsTypes() {
        let displayName;
        const movements = [];

        return this.$store.state.movementsTypes.filter((movement) => {
          if (displayName !== movement.name) {
            displayName = movement.name;
            movements.push(movement);
            return movements;
          }
          const item = _.find(movements, {
            name: displayName,
          });
          const clone = Object.assign(item, {
            id_stock_mvt_reason: [item.id_stock_mvt_reason]
          });

          return clone.id_stock_mvt_reason.push(movement.id_stock_mvt_reason);
        });
      },
      categoriesList() {
        return this.$store.getters.categories;
      },
    },
    methods: {
      onClick() {
        this.applyFilter();
      },
      onFilterActive(list, type) {
        if (type === 'supplier') {
          this.suppliers = list;
        } else {
          this.categories = list;
        }
        this.disabled = !this.suppliers.length && !this.categories.length;
        if (!list.length) {
          this.applyFilter();
        }
      },
      applyFilter() {
        this.$store.dispatch('isLoading');
        this.$emit('applyFilter', {
          suppliers: this.suppliers,
          categories: this.categories,
          id_stock_mvt_reason: this.id_stock_mvt_reason,
          id_employee: this.id_employee,
          date_add: this.date_add,
        });
      },
      onChange(item) {
        if (item.itemID === 'id_stock_mvt_reason') {
          this.id_stock_mvt_reason = item.value === 'default' ? [] : item.value;
        } else {
          this.id_employee = item.value === 'default' ? [] : item.value;
        }
        this.applyFilter();
      },
      onDpChange(event) {
        const type = $(event.currentTarget).data('type');
        this.date_add[type] = event.date.unix();
        if (event.oldDate) {
          this.applyFilter();
        }
      },
    },
    components: {
      FilterComponent,
      PSSelect,
      PSButton,
      PSDatePicker,
    },
    mounted() {
      this.$store.dispatch('getSuppliers');
      this.$store.dispatch('getCategories');
    },
    data: () => ({
      disabled: true,
      suppliers: [],
      categories: [],
      id_stock_mvt_reason: [],
      id_employee: [],
      date_add: {},
    }),
  };
</script>

<style lang="sass" scoped>
  @import "~PrestaKit/scss/custom/_variables.scss";
  #filters {
    background: white;
    border-radius: 2px;
    box-shadow: 1px 2px 3px 0 rgba(108, 134, 142, 0.3);
    border: solid 1px #b9cdd2;
  }
  .collapse-button {
    width: 100%;
    text-align: left;
    .material-icons {
      vertical-align: bottom;
      font-size: 20px;
      color: $gray-medium;
    }
  }
</style>
