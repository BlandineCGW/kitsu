<template>
<div class="page">
  <div class="page-header">
    <div class="level">
      <div class="level-left">
        <div class="level-item">
          <entity-thumbnail
            class="shot-thumbnail"
            :entity="currentShot"
            v-if="currentShot"
          >
          </entity-thumbnail>
        </div>
        <div class="level-item">
          <page-title :text="title"></page-title>
        </div>
      </div>
    </div>
  </div>

  <div class="columns">
    <div class="column">
    <page-subtitle :text="$t('shots.tasks')"></page-subtitle>
    <entity-task-list
      :entries="currentShot ? currentShot.tasks : []"
      :is-loading="!currentShot"
      :is-error="false"
    ></entity-task-list>
    </div>
    <div class="column">
      <page-subtitle :text="$t('main.info')"></page-subtitle>
      <table class="table">
        <tbody>
          <tr v-if="currentShot && currentShot.data.fps">
            <td class="field-label">{{ $t('shots.fields.fps') }}</td>
            <td>
              {{ currentShot ? currentShot.data.fps : '' }}
            </td>
          </tr>

          <tr v-if="currentShot && currentShot.data.frame_in">
            <td class="field-label">{{ $t('shots.fields.frame_in') }}</td>
            <td>
              {{ currentShot ? currentShot.data.frame_in : '' }}
            </td>
          </tr>

          <tr v-if="currentShot && currentShot.data.frame_out">
            <td class="field-label">{{ $t('shots.fields.frame_out') }}</td>
            <td>
              {{ currentShot ? currentShot.data.frame_out : '' }}
            </td>
          </tr>

          <tr>
            <td class="field-label">{{ $t('shots.fields.description') }}</td>
            <td>
              {{ currentShot ? currentShot.description : '' }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <div>
    <page-subtitle :text="$t('shots.casting')"></page-subtitle>
    <div v-if="currentShot">
      <div
        class="type-assets"
        v-for="typeAssets in currentShot.castingAssetsByType"
        v-if="currentShot.castingAssetsByType[0].length > 0"
      >
        <div class="asset-type">
          {{ typeAssets[0] ? typeAssets[0].asset_type_name : '' }}
        </div>
        <div class="asset-list">
          <router-link
            class="asset-link"
            :key="asset.id"
            :to="{
              name: 'asset',
              params: {
                production_id: currentProduction.id,
                asset_id: asset.asset_id
              }
            }"
            v-for="asset in typeAssets"
          >
            <entity-thumbnail
              :entity="asset"
              :square="true"
              :empty-width="100"
              :empty-height="100"
              :with-link="false"
            >
            </entity-thumbnail>
            <div>
              <span>{{ asset.name }}</span>
              <span v-if="asset.nb_occurences > 1">
                ({{ asset.nb_occurences }})
              </span>
            </div>
          </router-link>
        </div>
      </div>
      <div v-else>
        {{ $t('shots.no_casting') }}
      </div>
    </div>
    <table-info
      :is-loading="casting.isLoadin"
      :is-error="casting.isError"
      v-else
    >
    </table-info>
  </div>
</div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex'

import PageTitle from './widgets/PageTitle'
import PageSubtitle from './widgets/PageSubtitle'
import EntityThumbnail from './widgets/EntityThumbnail'
import EntityTaskList from './lists/EntityTaskList'
import TableInfo from './widgets/TableInfo'

export default {
  name: 'shot',
  components: {
    EntityThumbnail,
    EntityTaskList,
    PageSubtitle,
    PageTitle,
    TableInfo
  },

  data () {
    return {
      currentShot: null,
      casting: {
        isLoading: false,
        isError: false
      }
    }
  },

  created () {
    this.clearSelectedTasks()

    this.currentShot = this.getCurrentShot()

    this.casting.isLoading = true
    this.casting.isError = false

    if (!this.currentShot) {
      this.loadShot({
        shotId: this.route.params.shot_id,
        callback: (err) => {
          if (!err) {
            this.currentShot = this.getCurrentShot()
            this.loadShotCasting({
              shot: this.currentShot,
              callback: (err, casting) => {
                if (err) {
                  this.casting.isError = true
                } else {
                  this.casting.isError = false
                }
                this.casting.isLoading = true
              }
            })
          }
        }
      })
    } else {
      this.loadShotCasting({
        shot: this.currentShot,
        callback: (err, casting) => {
          if (err) {
            this.casting.isError = true
          } else {
            this.casting.isError = false
          }
          this.casting.isLoading = true
        }
      })
    }
  },

  computed: {
    ...mapGetters([
      'currentProduction',
      'route',
      'shotMap'
    ]),

    title () {
      if (this.currentShot) {
        if (this.currentShot.episode_name) {
          return `${this.currentShot.project_name} / ` +
                 `${this.currentShot.episode_name} / ` +
                 `${this.currentShot.sequence_name} / ` +
                 `${this.currentShot.name}`
        } else {
          return `${this.currentShot.project_name} / ` +
                 `${this.currentShot.sequence_name} / ` +
                 `${this.currentShot.name}`
        }
      } else {
        return 'Loading...'
      }
    },

    shotThumbnailPath () {
      const previewId = this.currentShot.preview_file_id
      return `/api/pictures/originals/preview-files/${previewId}.png`
    },

    isPreview () {
      return this.currentShot &&
        this.currentShot.preview_file_id &&
        this.currentShot.preview_file_id.length > 0
    }
  },

  methods: {
    ...mapActions([
      'loadShot',
      'loadShotCasting',
      'clearSelectedTasks'
    ]),
    changeTab (tab) {
      this.selectedTab = tab
    },
    getCurrentShot () {
      return this.shotMap[this.route.params.shot_id] || null
    }
  },

  watch: {
    $route () { this.handleModalsDisplay() }
  },

  metaInfo () {
    return {
      title: `${this.title} - Kitsu`
    }
  }
}
</script>

<style scoped>
.shot-thumbnail {
  max-width: 100px;
}

.asset-link .thumbnail-picture {
  margin-bottom: 0.5em;
}

.asset-type {
  text-transform: uppercase;
  font-size: 1.2em;
  color: #999;
  margin-top: 2em;
  margin-bottom: 0.4em;
}

.asset-list {
  display: flex;
  flex-wrap: wrap;
}

.asset-link {
  color: inherit;
  margin-right: 1em;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-size: 0.8em;
}

.asset-link div {
  max-width: 100px;
}

.asset-link span {
  word-wrap: break-word;
}

.field-label {
  font-weight: bold;
  width: 140px;
}
</style>
