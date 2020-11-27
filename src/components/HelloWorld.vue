<template>
  <div>
    <v-data-table
      dark
      :headers="headers"
      :items="datas"
      :items-per-page=100
      :footer-props="{
        itemsPerPageOptions: [50, 100, 150, 200, 250, 300, -1]
      }"
      @click:row="selectItem"
      @contextmenu:row="scroll"
      fixed-header
      height="700"
      mobile-breakpoint="300"
      ref="dataTable"
      @update:page="updatePage"
    >
      <template v-slot:[`item.checked`]="{ item }">
        <v-simple-checkbox
          v-model="item.checked"
          disabled
          off-icon=""
          on-icon="mdi-check"
        ></v-simple-checkbox>
      </template>
      <template v-slot:[`item.en`]="{ item }">
        <span class="en text-h6">  {{ item.en }}</span>
      </template>
      <template v-slot:[`item.ja`]="{ item }">
        <div 
          class="ja text-subtitle-1"
          @mouseover="speechSpeak(item.en)"
          @mouseleave="speechCancel"
        >
          {{ item.ja }}
        </div>
      </template>
      <template v-slot:[`item.pos`]="{ item }">
        <div 
          class="grey--text text-overline"
        >
          {{ item.pos }}
          <span>
            ({{ ( item.ja.toString().match( /；/g ) || [] ).length + 1 }})
          </span>
        </div>
      </template>
    </v-data-table>
  </div>
</template>

<script>
import dbData from '@/assets/ngsl.json'
import path from 'path'
import { remote } from 'electron'
const Database = require("nedb")
const DB = new Database({
  autoload: true,
  filename: path.join(remote.app.getPath('userData'), '/data.db')
})
export default {
  data: () => {
    return {
      headers: [
        { text: 'M', value: 'checked', width: '10%', align: 'left' },
        { text: 'EN', value: 'en', width: '38%' },
        { text: 'JA', value: 'ja', sortable: false, width: '42%' },       
        { text: 'POS', value: 'pos', sortable: false, width: '10%' },       
      ],
      datas: [],
      selected: []
    }
  },
  created () {
    //データ全削除
    // DB.remove({}, { multi: true }, function(err, numRemoved){ console.log(err, numRemoved) });

    this.getItems()
  },
  methods: {
    getItems () {
      DB.find({}, (err, doc) => {
        if (Object.keys(doc).length === 0) {
          DB.insert(dbData)
        }
        this.datas = doc
      })
    },
    selectItem (e) {
      this.datas.forEach((obj, index) => {
        if (obj._id === e._id) {
          this.$set(this.datas[index], 'checked', !obj.checked)
        }
      })
      
      // console.log(e._id, e.checked)
      DB.update(
        { "_id": e._id },
        { 
          $set: { 
            "checked": e.checked
          } 
        },
        {},
        // function (error, numOfDocs) {
        //   console.log(error, numOfDocs)
        // }
      )
    },
    speechSpeak (word) {
      const uttr = new SpeechSynthesisUtterance(word)
      uttr.lang = "en-US"
      speechSynthesis.speak(uttr)
    },
    speechCancel () {
      speechSynthesis.cancel()
    },
    updatePage () {
      // const table = this.$refs.dataTable
      const table = this.$refs['dataTable']
      const wrapper = table.$el.querySelector('div.v-data-table__wrapper')

      this.$vuetify.goTo(table)
      this.$vuetify.goTo(table, {
        container: wrapper,
        duration: 0
      })
    },
    scroll (e) {
      const table = this.$refs['dataTable']
      const wrapper = table.$el.querySelector('div.v-data-table__wrapper')

      this.$vuetify.goTo(table)
      this.$vuetify.goTo(table, {
        container: wrapper,
        offset: -(e.target.parentNode.offsetTop) + e.target.parentNode.offsetHeight - 15,
        duration: 200
      })
      // console.log(e.target.parentNode)
    }
  }
}
</script>

<style>
.v-data-footer {
  position: fixed;
  bottom: 0;
  width: 100%;
  border-top: 1px solid grey !important;
  background-color: black;
}
.v-data-footer__select>.v-select {
    margin: 0 0 3px 34px !important;
}
.v-data-table__wrapper {
  margin-bottom: 36px;
}


.ja {
  opacity: 0;
}
.ja:hover {
  opacity: 1;
}
</style>