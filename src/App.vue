<template lang="pug">
  .top
    h1 新型コロナウィルス都道府県別脅威度
    .about 新型コロナウィルスの人口10万人あたりの各指標の都道府県別のリストです。
    v-data-table.table(
      :headers="tableHeaders",
      :items="tableData",
      :items-per-page="100",
      :hide-default-footer="true",
      :mobile-breakpoint="0"
      :sort-by.sync="sortBy"
      :sort-desc.sync="sortDesc")
    .template(v-for="header in tableHeaders")
      .grid(v-if="header.value !== 'prefecture'")
        .grid-caption {{ header.text }}
        table
          tr(v-for="row in getGridMapData()")
            template(v-for="data in row")
              td(:style="getCellStyle(data, header.value)")
                .prefecture {{ data.prefecture }}
                .value {{ getValue(data, header.value) }}
    .last-update 最終更新日: {{ lastUpdate }}
    .project-home Project home: 
      a(href="https://github.com/ApplePedlar/covid-19-severity-jp" target="_new") GitHub ApplePedlar/covid-19-severity-jp
    .data-source 情報ソース: 
      a(href="https://www.stopcovid19.jp/data/covid19japan.json" target="_new") https://www.stopcovid19.jp/data/covid19japan.json
</template>

<script>

import axios from "axios"
import populations from "./populations.json"

export default {
  data () {
    return {
      sourceUrl: 'https://www.stopcovid19.jp/data/covid19japan.json',
      sortBy: 'currentPatientsPerPop',
      sortDesc: true,
      tableHeaders: [
        { text: "都道府県", value: "prefecture", width: "90px", sortable: false },
        { text: "累計感染者数", value: "totalPatients" },
        { text: "現在患者数", value: "currentPatients" },
        { text: "累計死亡者数", value: "totalDeaths" },
        { text: "10万人あたりの累計感染者数", value: "totalPatientsPerPop" },
        { text: "10万人あたりの現在患者数", value: "currentPatientsPerPop" },
        { text: "10万人あたりの累計死亡者数", value: "totalDeathsPerPop" }
      ],
      tableData: [],
      populations: populations,
      lastUpdate: ""
    }
  },
  mounted () {
    axios
      .get(this.sourceUrl)
      .then(response => {
        let records = response.data
        this.lastUpdate = records.lastUpdate
        let areas = records.area
        areas.forEach(area => {
          let totalPatients = area.npatients
          let totalDeaths = area.ndeaths
          let currentPatients = area.ncurrentpatients
          let prefecture = area.name_jp

          let population = this.populations[prefecture]
          let totalPatientsPerPop = Math.round(totalPatients / (population / 100) * 1000) / 1000
          let totalDeathsPerPop = Math.round(totalDeaths / (population / 100) * 1000) / 1000
          let currentPatientsPerPop = Math.round(currentPatients / (population / 100) * 1000) / 1000
          this.tableData.push({
            prefecture: prefecture,
            population: population,
            totalPatients: totalPatients,
            totalDeaths: totalDeaths,
            currentPatients: currentPatients,
            totalPatientsPerPop: totalPatientsPerPop,
            totalDeathsPerPop: totalDeathsPerPop,
            currentPatientsPerPop: currentPatientsPerPop
          })
        })
      })
  },
  methods: {
    getGridCaption () {
      let sortByHeader = this.tableHeaders.find(header => header.value == this.sortBy)
      if (sortByHeader) {
        return sortByHeader.text
      }
    },
    getGridMapData () {
      let prefectureSequence = [ "島根県", "鳥取県", "京都府", "新潟県", "青森県", "北海道", "広島県", "兵庫県", "滋賀県", "石川県", "秋田県", "岩手県", "山口県", "岡山県", "大阪府", "富山県", "山形県", "宮城県", "佐賀県", "福岡県", "奈良県", "福井県", "群馬県", "福島県", "長崎県", "大分県", "三重県", "長野県", "栃木県", "茨城県", "熊本県", "宮崎県", "和歌山県", "岐阜県", "山梨県", "千葉県", "鹿児島県", "愛媛県", "香川県", "愛知県", "埼玉県", "東京都", "沖縄県", "高知県", "徳島県", "静岡県", "神奈川県" ]
      let gridMapData = []
      let row = []
      prefectureSequence.forEach(prefecture => {
        let data = this.tableData.find(d => d.prefecture == prefecture)
        if (data) {
          row.push(data)
          if (row.length == 6) {
            gridMapData.push(row)
            row = []
          }
        }
      })
      if (row.length > 0) {
        gridMapData.push(row)
      }
      return gridMapData
    },
    getValue (data, field) {
      return data[field]
    },
    getCellStyle (data, field) {
      let mean = this.tableData.reduce((acc, cur) => acc + cur[field], 0) / this.tableData.length
      let value = this.getValue(data, field)

      let red = 255
      let green = 255
      let blue = 255

      if (value >= mean) {
        blue = 0
        green = blue = 127 - parseInt((value - mean) / mean * 128)
        if (green < 0) {
          green = blue = 0
        }
      } else {
        green = blue = 255 - parseInt(value / mean * 128)
      }

      return `background-color: rgb(${red}, ${green}, ${blue})`

    }
  },
  watch: {
    sortDesc (newSortDesc, oldSortDesc) {
      this.$nextTick(() => {
        this.sortDesc = true
      })
    }
  }
}
</script>

<style lang="sass">
.top
  max-width: 800px
  margin: 30px auto
  h1
    font-size: 24px
  .about
    margin: 20px
  .table
    margin: 30px auto
    border: 1px silver solid
  .grid
    margin: 30px auto
    .grid-caption
      font-weight: bold
    table
      width: 100%
      td
        border: 1px solid black
        width: 16%
        text-align: center
        padding: 3px 0
        &.danger
          background-color: #FF3030
        &.warn
          background-color: #FF9090
        &.alert
          background-color: #FFFFA0
        font-size: 14px
  .project-home, .data-source, .last-update
    font-size: 12px
</style>
