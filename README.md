# gantt-table
一个基于element-ui的甘特图展示组件，不同状态展示不同颜色。

## 组件引入

```vue
<template>
  <div>
    <GanttTable ref="GanttTable">
      <template #tableColumn>
        <el-table-column label="批次" prop="name" width="250" align="center" />
      </template>
    </GanttTable>
  </div>
</template>

<script>
import GanttTable from '@/components/GanttTable/index.vue'
export default {
  name: 'Index',
  components: { GanttTable },
  mounted() {
    this.$refs.GanttTable.fetchData()
  }
}
</script>
<style scoped lang="scss">

</style>

```

##  gantt-table组件的封装

``` vue
<template>
  <el-table :data="tableData" border style="width: 100%">
    <!-- 表格列插槽 -->
    <slot name="tableColumn" />
    <!-- 甘特图时间范围 -->
    <el-table-column v-for="item in timeRange" :key="item" :label="''+item" width="auto">
      <template #default="scope">
        <div :style="isCellShow(scope.row,item)" class="cell-show"> &nbsp;</div>
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
export default {
  name: 'GanttTable',
  props: {
    statusColor: { //  不同状态对应的颜色
      type: Object,
      default: _ => {
        return {
          preview: '#0579FF;',
          optionalAndRefundable: '#8164F8;',
          optional: '#18C59D;',
          refundable: '#FC8357;',
        }
      }
    }
  },
  data() {
    return {
      tableData: [], // 表格数据
      timeRange: [] // 时间范围
    }
  },
  computed: {
    isCellShow() {
      return function(row, item) {
        const rowList = row.list ? JSON.parse(JSON.stringify(row.list)) : []
        const filterItem = rowList.find(i => new Date(i.startDate) <= new Date(item) && new Date(item) <= new Date(i.endDate))
        const isStart = filterItem ? filterItem.startDate === item : false
        const isEnd = filterItem ? filterItem.endDate === item : false
        const bgColor = this.statusColor[filterItem?.status]
        return filterItem ? `background-color:${bgColor};border-radius: ${isStart ? '10px' : 0} ${isEnd ? '10px' : 0} ${isEnd ? '10px' : 0} ${isStart ? '10px' : 0} ` : 'display: none;'
      }
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData(fetchParam) {
      // 获取接口得到数据
      this.tableData = [
        { name: '项目一',
          list: [{ status: 'preview', startDate: '2022-8-1', endDate: '2022-8-2' },
            { status: 'optionalAndRefundable', startDate: '2022-8-4', endDate: '2022-8-5' },
            { status: 'optional', startDate: '2022-8-6', endDate: '2022-8-9' },
            { status: 'refundable', startDate: '2022-8-10', endDate: '2022-8-15' }]
        },
        { name: '项目二', list: [{ status: 'preview', startDate: '2022-8-3', endDate: '2022-8-3' }] }
      ]
      // 最小开始时间与最大结束时间 => 时间范围
      const timeList = this.tableData.map(i => { return i.list }).flat()
      const startTime = timeList.map(i => { return i.startDate }).sort((a, b) => { return new Date(a) - new Date(b) })[0]
      const endTime = timeList.map(i => { return i.endDate }).sort((a, b) => { return new Date(b) - new Date(a) })[0]
      this.timeRange = this.getAllDate(startTime, endTime)
    },
    format(time) {
      const mouth = (time.getMonth() + 1) >= 10 ? (time.getMonth() + 1) : ((time.getMonth() + 1))
      const day = time.getDate() >= 10 ? time.getDate() : (time.getDate())
      return `${time.getFullYear()}-${mouth}-${day}`
    },
    getAllDate(start, end) {
      const dateArr = []
      const startArr = start.split('-')
      const endArr = end.split('-')
      const db = new Date()
      db.setUTCFullYear(startArr[0], startArr[1] - 1, startArr[2])
      const de = new Date()
      de.setUTCFullYear(endArr[0], endArr[1] - 1, endArr[2])
      const unixDb = db.getTime()
      const unixDe = de.getTime()
      let stamp
      const oneDay = 24 * 60 * 60 * 1000
      for (stamp = unixDb; stamp <= unixDe;) {
        dateArr.push(this.format(new Date(parseInt(stamp))))
        stamp = stamp + oneDay
      }
      return dateArr
    }
  }
}
</script>

<style scoped>
.cell-show {
  position: absolute;
  left: 0;
  width: calc(100% + 2px);
  height: 20px;
  top: 50%;
  transform: translate(0, -50%);
}
</style>

```
