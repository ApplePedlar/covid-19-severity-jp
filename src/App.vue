<template lang="pug">
  .top
    h1 新型コロナウィルス脅威度(日本)
    .about 新型コロナウィルスの人口10万人あたりの死亡者数の都道府県別のリストです。
    v-data-table.table(
      :headers="tableHeaders",
      :items="tableData",
      :items-per-page="100",
      :hide-default-footer="true",
      :sort-by="['totalNumberOfDeathsPerPop']",
      :sort-desc="[true]",
      :mobile-breakpoint="0")
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
      tableHeaders: [
        { text: "都道府県", value: "prefecture" },
        { text: "人口(千人)", value: "population" },
        { text: "累計死亡者数", value: "totalNumberOfDeaths" },
        { text: "10万人あたりの累計死亡者数", value: "totalNumberOfDeathsPerPop" }
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
        for (let i = 0; i < areas.length; i++) {
          let area = areas[i]
          let totalNumberOfDeaths = area.ndeaths
          let prefecture = area.name_jp

          if (totalNumberOfDeaths > 0) {
            let population = this.populations[prefecture]
            let totalNumberOfDeathsPerPop = Math.round(totalNumberOfDeaths / (population / 100) * 1000) / 1000
            this.tableData.push({
              prefecture: prefecture,
              population: population,
              totalNumberOfDeaths: totalNumberOfDeaths,
              totalNumberOfDeathsPerPop: totalNumberOfDeathsPerPop
            })
          }
        }
      })
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
    max-width: 500px
    margin: 30px auto
    border: 1px silver solid
  .project-home, .data-source, .last-update
    font-size: 12px
</style>
