<template>
  <div
    class="filters padding-vertical-big padding-left-big padding-right-normal"
    @click="hideLicenseExplanationVisibility()"
  >
    <div class="filters-title" @click.prevent="toggleFilterVisibility">
      <span>{{ title }}</span>
      <button
        :aria-label="'filters list for' + title + 'category'"
        v-if="!filtersExpandedByDefault"
        class="filter-visibility-toggle is-white padding-vertical-small"
      >
        <i
          v-if="areFiltersExpanded"
          class="icon angle-up rotImg is-size-5 has-text-grey-light"
          title="toggle filters visibility"
        />
        <i
          v-else
          class="icon angle-down is-size-5 has-text-grey-light"
          title="toggle filters visibility"
        />
      </button>
    </div>
    <template v-if="areFiltersExpanded && options">
      <div
        v-for="(item, index) in options"
        :key="index"
        class="margin-top-small"
      >
        <label class="checkbox" :for="item.code" :disabled="block(item)">
          <input
            type="checkbox"
            class="filter-checkbox margin-right-small"
            :id="item.code"
            :key="index"
            :checked="item.checked"
            :disabled="block(item)"
            @change="onValueChange"
          />
          <license-icons v-if="filterType == 'licenses'" :license="item.code" />
          {{ item.name }}
        </label>
        <img
          aria-label="license explanation"
          tabindex="0"
          v-if="filterType == 'licenses'"
          src="@/assets/help_icon.svg"
          alt="help"
          class="license-help is-pulled-right padding-top-smallest padding-right-smaller"
          @click.stop="toggleLicenseExplanationVisibility(item.code)"
        />

        <license-explanation-tooltip
          v-if="shouldRenderLicenseExplanationTooltip(item.code)"
          :license="licenseExplanationCode"
        />
      </div>
    </template>
  </div>
</template>

<script>
import { ExperimentData } from '@/abTests/experiments/filterExpansion'
import LicenseIcons from '@/components/LicenseIcons'
import LicenseExplanationTooltip from './LicenseExplanationTooltip'

export default {
  name: 'filter-check-list',
  components: {
    LicenseIcons,
    LicenseExplanationTooltip,
  },
  props: ['options', 'title', 'filterType', 'disabled', 'checked'],
  data() {
    return {
      filtersVisible: false,
      licenseExplanationVisible: false,
      licenseExplanationCode: '',
    }
  },
  computed: {
    /**
     * Check if a filter experiment is active, and if the current case is 'expanded'.
     * Show filters collapsed by default
     */
    filtersExpandedByDefault() {
      const experiment = this.$store.state.experiments.find(
        (exp) => exp.name === ExperimentData.EXPERIMENT_NAME
      )
      return experiment ? experiment.case === ExperimentData.FILTERS : false
    },
    areFiltersExpanded() {
      return this.filtersExpandedByDefault || this.filtersVisible
    },
  },
  methods: {
    onValueChange(e) {
      this.$emit('filterChanged', {
        code: e.target.id,
        filterType: this.$props.filterType,
      })
    },
    toggleFilterVisibility() {
      this.filtersVisible = !this.filtersVisible
    },
    toggleLicenseExplanationVisibility(licenseCode) {
      this.licenseExplanationVisible = !this.licenseExplanationVisible
      this.licenseExplanationCode = licenseCode
    },
    hideLicenseExplanationVisibility() {
      this.licenseExplanationVisible = false
    },
    block(e) {
      if (this.$props.filterType === 'licenseTypes') {
        const nc = this.$store.state.filters.licenses.filter((item) =>
          item.code.includes('nc')
        )
        const nd = this.$store.state.filters.licenses.filter((item) =>
          item.code.includes('nd')
        )
        return (
          (e.code === 'commercial' && nc.some((li) => li.checked)) ||
          (e.code === 'modification' && nd.some((li) => li.checked))
        )
      }

      if (this.$props.filterType === 'licenses') {
        const commercial = this.$store.state.filters.licenseTypes.find(
          (item) => item.code === 'commercial'
        )
        const modification = this.$store.state.filters.licenseTypes.find(
          (item) => item.code === 'modification'
        )
        return (
          (commercial.checked && e.code.includes('nc')) ||
          (modification.checked && e.code.includes('nd'))
        )
      }
      return this.$props.disabled
    },
    shouldRenderLicenseExplanationTooltip(licenseCode) {
      return (
        this.licenseExplanationVisible &&
        this.licenseExplanationCode === licenseCode
      )
    },
  },
}
</script>

<style lang="scss" scoped>
.filters {
  border-bottom: 2px solid rgb(245, 245, 245);
}

.filters-title {
  font-size: 1.25em;
  font-weight: 600;
  font-stretch: normal;
  font-style: normal;
  line-height: 1.5;
  letter-spacing: normal;
  cursor: pointer;
}

.filter-visibility-toggle {
  float: right;
  cursor: pointer;
  background: none;
  border: none;
}

label {
  color: #333333;
}

.license-help {
  cursor: pointer;
}
</style>
