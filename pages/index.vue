<template lang="pug">
div.root
  button.download(@click="downloadOpml" :disabled="markedRows.length == 0" ref="downloadBtn") Download OPML
  v-client-table(:columns="columns" :data="data" :options="options" ref="table")
    template(slot="cover" slot-scope="props")
      cover.cover(:channel="props.row.key" @click.native="toggleChildRow(props.row.key)" title="Click to show detail")
    template(slot="title" slot-scope="props")
      span(@click.self="toggleChildRow(props.row.key)") {{ props.row.title }}
    template(slot="lastEpisodeDate" slot-scope="props")
      a(v-if="props.row.lastEpisodeLink" :href="props.row.lastEpisodeLink" target="_blank" rel="noopener")
        span.new(v-if="isIn(props.row.lastEpisodeDate, newThreshold1)") New!
        | {{ props.row.lastEpisodeDate | formatDate }}
    template(slot="durationMedian" slot-scope="props")
      duration(:duration="props.row.durationMedian")
    template(slot="tweets" slot-scope="props")
      a(v-if="props.row.hashtag" :href="tweetsLink(props.row.tweets_query)" target="_blank" rel="noopener")
        b(v-if="props.row.tweets") {{ props.row.tweets | addPlus }}
        small {{ props.row.hashtag }}
    template(slot="followers" slot-scope="props")
      a(v-if="props.row.twitter" :href="twitterLink(props.row.twitter)" target="_blank" rel="noopener")
        b(v-if="props.row.followers") {{ props.row.followers }}
        small {{ props.row.twitter }}
    template(slot="firstEpisodeDate" slot-scope="props")
      a(v-if="props.row.firstEpisodeLink" :href="props.row.firstEpisodeLink" target="_blank" rel="noopener")
        span.new(v-if="isIn(props.row.firstEpisodeDate, newThreshold2)") New!
        | {{ props.row.firstEpisodeDate | formatDate }}
      template(v-else="props.row.firstEpisodeLink") {{ props.row.firstEpisodeDate | formatDate }}
    template(slot="download" slot-scope="props")
      input(type="checkbox" :value="props.row.key" v-model="markedRows")
    template(slot="child_row" slot-scope="props")
      p.description(v-if="props.row.desciprtion" v-html.raw="props.row.desciprtion")
      p.description(v-else) No description
      button-text(v-if="props.row.link" :text="props.row.link" :buttonText="'Open Web'" buttonAction="'open'")
      button-text(:text="props.row.feed" :buttonText="'Copy RSS'")
</template>

<style lang="sass" scoped>
$color_new: #e100ff

.download
  margin-right: 20px
  position: absolute
  width: 150px
  top: 20px
  right: 0

.new
  $color: #f7ff00
  background: $color
  font-weight: bold
  font-size: 10px
  display: flex
  justify-content: center
  align-items: center
  width: 50px
  height: 20px
  border-radius: 10px
  margin-right: 7px
  position: absolute
  top: -40px
  right: -30px
  color: #47525d
  &:after
    content: '▼'
    font-size: 12px
    line-height: 1em
    position: absolute
    display: block
    color: $color
    bottom: -7px
    left: calc(50% - 8px)
    transform: rotate(19deg)

.root /deep/
  padding-top: 20px
  padding-bottom: 20px
  -webkit-overflow-scrolling: touch
  overflow-scrolling: touch
  position: relative
  table
    border-collapse: collapse
    border-spacing: 0
    width: 100%
  th
    white-space: nowrap
    &.last
      text-align: right
  th,td
    text-align: left
    vertical-align: top
    padding: 10px
    &:first-child:not(:last-child)
      display: none
    &:nth-child(2)
      padding-left: 20px
    &.duration,
    &.tweets,
    &.followers
      text-align: center
  thead
    color: #ccc
    font-size: 12px
    th
      font-weight: normal
  tbody
    th,td
      font-weight: 500
      font-size: 13px
      vertical-align: middle
    td.artwork
      position: relative
    td.title
      font-weight: bold
      font-size: 15px
      span
        cursor: pointer
        &:hover
          color: lighten(#444, 10%)
    td.followers,
    td.tweets
      text-align: center
      b
        font-weight: 500
        display: block
        font-size: 18px
      small
        display: block
        font-size: 10px
        color: #ccc
    td.total
      text-align: right
      font-size: 18px
    td.last,
    td.first
      >a
        position: relative
        display: flex
        align-items: center
        justify-content: flex-end
    tr
      &:first-child
        border-top: 1px solid #ccc
      &:not(.VueTables__child-row)
        border-top: 1px solid #ccc
      &.VueTables__child-row
        border-top: 1px dotted #eee
        background-image: url(/img/slant-bg.png)
        background-color: #e4e4e4
        background-size: auto 168px
        td
          padding: 20px
          line-height: 1.8em
        p
          max-width: calc(100vw - 30px)
          &:first-child
            margin-top: 0
    button
      font-size: 10px
      padding: 5px 10px
      min-width: initial
  .VueTables
    .row
      padding-left: 20px
      padding-right: 20px
    .table-responsive
      overflow: auto
      width: 100%
      margin-top: 15px
    .cover
      cursor: pointer
      transition-duration: 0.2s
      overflow: hidden
      &:before
        content: 'info\A▼'
        white-space: pre
        color: white
        font-size: 10px
        line-height: 1.3em
        font-weight: bold
        background-color: rgba(0,0,0,0.4)
        display: flex
        justify-content: center
        align-items: center
        text-align: center
        width: 100%
        height: 100%
        opacity: 0
        transition-duration: 0.2s
      &:hover
        &:before
          opacity: 1
          transition-duration: 0.2s
      &:active
        &:before
          opacity: 0
  .VueTables__search-field
    margin-bottom: 20px
    input
      padding: 8px
      outline: none
      font-size: 13px
      border: 1px solid #ddd
      width: 300px
      &:placeholder-shown
        color: #ccc
      &::-webkit-input-placeholder
        color: #ccc
      &::-moz-placeholder
        color: #ccc
  .VueTables__search
    float: left
    width: calc(100% - 270px)
  .VueTables__columns-dropdown
    float: right
    width: 100px
    .dropdown-menu
      position: absolute
      right: 20px
      padding: 15px
      margin-top: 6px
      margin-bottom: 0
      background: #050935
      border-radius: 3px
      list-style: none
      z-index: 10
      a
        color: white
        font-size: 12px
      input[type=checkbox]
        margin-right: 1em
  .VueTables__limit
    display: none
  .VueTables__sortable
    cursor: pointer
    &:hover
      color: #7f00ff
  .glyphicon-chevron-down
    &:before
      content: '▼'
      margin-left: 10px
      font-size: 0.7em
  .glyphicon-chevron-up
    &:before
      content: '▲'
      margin-left: 10px
      font-size: 0.7em
  .VuePagination
    .text-center
      margin-left: 0

.small
  .root /deep/
    padding-top: 15px
    padding-bottom: 15px
    table
      margin-top: 15px
    tbody
      th,td
        font-size: 11px
      td.total
        font-size: 14px
      td.followers,
      td.tweets
        b
          font-size: 14px
    button
      font-size: 10px
    .download
      position: relative
      margin-left: 15px
      top: initial
      left: initial
      height: 38px
    .VueTables
      margin-top: 15px
      .row
        padding-left: 15px
        padding-right: 15px
      // hoverが解除されないので、打ち消す
      .cover:hover:before
        opacity: 0
    .VueTables__columns-dropdown
      .dropdown-menu
        right: 15px
    .VueTables__search
      width: 100%
    .VueTables__search-field
      margin-bottom: 0
      input
        width: calc(100% - 20px)
        padding: 9px
        font-size: 16px
    .VueTables__columns-dropdown
      clear: left
    th,td
      &:nth-child(2)
        padding-left: 15px
    tr
      &.VueTables__child-row
        td
          padding: 15px
    .description
      max-width: calc(100vw - 30px)
</style>

<script>
import axios from 'axios'
import moment from 'moment'
import xml2js from '~/lib/xml2js-promise'
import rss from '~/data/rss.json'
import build_info from '~/static/downloads/build_info.json'
import opml from 'opml-generator'
import { saveAs } from 'file-saver'

export default {
  components: {
    'button-text': require('~/components/button-text.vue').default,
    'cover': require('~/components/cover.vue').default,
    'duration': require('~/components/duration.vue').default
  },
  head() {
    return {
      title: 'Podcast Freaks - Japanese techie podcast archive'
    }
  },
  data: function() {
    return {
      // 行ごとに何度も作成しないように予め作る
      newThreshold1: moment().subtract(3, 'days').startOf('date'),
      newThreshold2: moment().subtract(30, 'days').startOf('date'),

      allMarked: false,
      markedRows: [],
      columns: [
        'cover',
        'title',
        'total',
        'durationMedian',
        'tweets',
        'followers',
        'firstEpisodeDate',
        'lastEpisodeDate',
        'download'
      ],
      options: {
        columnsClasses: {
          cover: 'artwork',
          title: 'title',
          tweets: 'tweets',
          followers: 'followers',
          total: 'total',
          firstEpisodeDate: 'first',
          lastEpisodeDate: 'last',
          durationMedian: 'duration'
        },
        orderBy: {
          ascending: false,
          column: 'lastEpisodeDate'
        },
        perPage: 9999,
        headings: {
          cover: '▽ Click',
          title: 'Title',
          followers: 'Twitter followers',
          tweets: 'Tweets in a week',
          total: 'Episodes',
          firstEpisodeDate: 'First episode',
          lastEpisodeDate: 'Last episode',
          durationMedian: 'Duration',
          download: function(h){
            const self = this;
            return h('input', {
              attrs: { type: 'checkbox', checked: true},
              domProps: { value: self.value },
              class: 'form-control check-all',
              on: {
                change: self.toggleAllCheckbox,
                input: function(event) {
                  self.value = event.target.value
                  self.$emit('input', event.target.value)
                }
              }
            })
          }
        },
        headingsTooltips: {
          title: 'クリックすると詳細情報が確認できます',
          durationMedian: '収録時間の中央値',
          followers: '番組公式Twitterアカウントのフォロワー数',
          download: 'ダウンロードするためにチェックしてください'
        },
        sortable: [
          'title',
          'followers',
          'tweets',
          'total',
          'firstEpisodeDate',
          'lastEpisodeDate',
          'durationMedian'
        ],
        descOrderColumns: [
          'followers',
          'tweets',
          'total',
          'lastEpisodeDate',
          'durationMedian'
        ],
        texts: {
          filter: '',
          filterPlaceholder: 'Search'
        },
        uniqueKey: 'key'
      },
      // customSorting: {
      //   twitter: function (ascending) {
      //     return function (a, b) {
      //       var lastA = a.name[a.name.length - 1].toLowerCase();
      //       var lastB = b.name[b.name.length - 1].toLowerCase();

      //       if (ascending)
      //           return lastA >= lastB ? 1 : -1;

      //       return lastA <= lastB ? 1 : -1;
      //     }
      //   }
      // },
      data: Object.values(build_info.channels)
    }
  },
  mounted: function(){
    this.toggleAllCheckbox()
  },
  methods: {
    toggleChildRow: function(key){
      this.$refs.table.toggleChildRow(key)
    },
    twitterLink: function(str) {
      if(str != null) {
        return `https://twitter.com/${str.replace('@','')}`
      }
      return ''
    },
    tweetsLink: function(query) {
      if(query != null) {
        return `https://twitter.com/search?q=${query.replace('#','%23')}`
      }
      return ''
    },
    toggleAllCheckbox: function() {
      this.markedRows = this.allMarked ? [] : Object.keys(rss)
      this.allMarked = !this.allMarked
    },
    isIn: function(date, threshold){
      return moment(date, 'YYYY.MM.DD').isAfter(threshold)
    },
    downloadOpml: function(){
      const header = {
        "title": "podcast-freaks channel list",
        "dateCreated": new Date(),
        "ownerName": "podcast-freaks"
      }
      const outlines = this.markedRows.map((channelName)=>{
        const channel = rss[channelName]
        return {
          text: "txt",
          title: channelName,
          type: "rss",
          "xmlUrl": channel.feed
        }
      })
      var blob = new Blob([opml(header, outlines)], {type: "text/plain;charset=utf-8"})
      saveAs(blob, "podcast-freaks.opml")
    }
  }
}
</script>
