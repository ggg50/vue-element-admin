<template>
  <el-table :data="innerTableData" style="width: 100%" v-bind="$attrs" v-on="$listeners">
    <el-table-column v-if="hasIndex" type="index" width="50" />
    <template v-for="(item, index) in innerColumnsList">
      <!-- eslint-disable -->
      <el-table-column
        :key="item.title + index"
        :label="item.title"
        :[widthType]="item.width ? item.width : 'auto'"
        v-bind="item.options || {}"
      >
      <!-- eslint-enable -->
        <template #default="{row, $index}">
          <slot :name="item.key" :row="row" :rawRow="tableData[$index]">
            <span>{{ row[item.key] }}</span>
          </slot>
        </template>
      </el-table-column>
    </template>
  </el-table>
</template>

<script>
/*

-description
-usage
-usage-addvence
-awesome
  -el-table prop & event
  -filterDict usage
-disadvantage
-demo data

简单说：
-可以引用全局过滤器自动处理数据
-可以采用最简洁的模板定制表头并控制表单内容顺序
-实在不开心，还可以用命名插槽自己搞事情

# description
a common table component for 'lazy' developer table-building quickly

# usage
|property       |type      |value        |default   |required  |description

|hasIndex       |prop      |Boolean      |true      |-         |whether index show or not
|columnsList    |prop      |Array        |-         |required  |message of table header, read more in addvence-usage
|tableData      |prop      |Array        |-         |required  |array of table data, read more in addvence-usage
|distributeWidth|prop      |Boolean      |true      |-         |whether table auto distribute the space in proportion（是否平均分配剩余空白）
|slots          |slot      |{row, rawRow}|-         |-         |every column have a slot with name equal to it's column key

# usage-addvence

1.columnsList
properties: 'name', 'key', 'width', 'options'
'name' and 'key' in every column item is required, while 'width' is no required
'options' property provide a way for you to custem the el-table-column's props

for 'laziest' person, here is a lazy-template: 'name-key-width'

2.tableData
within this component, data showed in the table is a clone of raw(the input tableData)
if you want to touch the raw row data, use the 'rawRow' property of slot scope

3.slot - {row, rawRow}
row is the filter-handered data
rawRow is the raw data

4.filter related
view more in 'awesome' ...

# awesome

1.every el-table props and event is work in this component (while el-table-column's unwork);

2.global filter
I found that, always we have a requirement to use global filters to handle the expection-unmatch data, these why I introduce a auto template for filter-handling

how to use it?

first, you need to set a unique key(start with $) to every filterDict manually (say one of them is '$t', which means 'globalTimeFormatter');
then, use it in columnsList item's key property(just like 'date$t'), so that every item's date value will be formated;
then, nothing more ...

# disadvantage

1.these is no way for you to touch the el-table-column's event

2.for complex idea, try name slot, or give up this component ...

# demo data
1.columnsList:
[
    { title: 'DDDD', key: 'date$t', width: '100%', },
    { title: 'ADDRESS', key: 'address', width: '200%', },
    { title: 'NAME', key: 'name', width: '100%', },
    { title: "momey", key: 'number$a', width: '100%', }
],

2.columnsList-template:
['DDDD-date$t-100%', 'ADDRESS-address-200%', 'NAME-name-100%', 'momey-number$a-100%']

3.tableData:
[
    { date: '2016-05-02', name: '王小虎1', address: ' 1518 弄', number: '123123123', },
    { date: '2016-05-04', name: '王小虎2', address: '上海市普陀区金沙 弄', number: '123123123', },
    { date: '2016-05-01', name: '王小虎3', address: '上海市普陀区金沙江路 1519 弄' },
    { date: '2016-05-03', name: '王小虎4', address: '上海市普陀区金沙江路 1516 弄' }
],

4.slot
<template #date="{row, rawRow}">
</template>

# unfinished
pageNo(prop)
edit data

# update
created ---- 2020-05-05 15:46:58
dynamic add props to 'el-table-column' through 'columnsList' item's options property ---- 2020-05-11 00:21:27

# raw
hasIndex|prop|Boolean|true|-|whether index show or not
columnsList|prop|Array|-|required|message of table header, read more in addvence-usage
tableData|prop|Array|-|required|array of table data, read more in addvence-usage
distributeWidth|prop|Boolean|true|-|whether table auto distribute the space in proportion（是否平均分配剩余空白）
slots|slot|{row, rawRow}|-|-|every column have a slot with name equal to it's column key
*/

const filterDict = { // Customize your filters dict
  $d: 'dateFormatter',
  $t: 'timeFormatter',
  $m: 'moneyFormatter',
  $a: 'toThousandFilter'
}

export default {
  name: 'CommonTable',
  props: {
    hasIndex: {
      type: Boolean,
      default: true
    },
    pageNo: {
      type: Number,
      default: 1
    },
    columnsList: {
      type: Array,
      required: true,
      validator: function(value) {
        for (const item of value) {
          if (typeof item === 'string') {
            if (item.split('-').length < 2) {
              console.error(
                `check you columnsList template, current ${item}, expect: "title-key$t-100%"`
              )
              return false
            }
          } else {
            if (!item.title || !item.key) {
              console.error(
                'columnsList error, title and key properties are required'
              )
              return false
            }
          }
        }
        return true
      }
    },
    tableData: {
      type: Array,
      required: true
    },
    distributeWidth: {
      type: Boolean,
      default: true
    }
  },

  data() {
    return {
      filterDict,
      globalFilters: this.$options.filters,
      filterReg: /\$.*$/
    }
  },

  computed: {
    innerColumnsList() {
      let list = JSON.parse(JSON.stringify(this.columnsList))
      if (typeof list[0] === 'string') {
        list = list.map(item => {
          const values = item.split('-')
          return {
            title: values[0].trim(),
            key: values[1].trim(),
            width: values[2].trim()
          }
        })
      }

      // remove '$\w' in every column key
      list.forEach(item => (item.key = item.key.replace(this.filterReg, '')))
      console.log(JSON.stringify(list, null, 2))
      return list
    },

    innerTableData() {
      // check filter pre-handle required and edit raw's clone data
      const data = JSON.parse(JSON.stringify(this.tableData))
      this.columnsList.forEach(item => {
        if (this.filterReg.test(item.key)) {
          const match = item.key.match(this.filterReg)[0]
          const key = item.key.replace(this.filterReg, '')
          const globalFilter = this.globalFilters[this.filterDict[match]]

          if (globalFilter) {
            data.forEach(item => (item[key] = globalFilter(item[key])))
          }
        }
      })
      return data
    },

    widthType() {
      return this.distributeWidth ? 'min-width' : 'width'
    }
  },

  created() {
    this.checkFilterExist()
  },

  mounted() {},

  methods: {
    checkFilterExist() {
      for (const key in this.filterDict) {
        if (!this.globalFilters[this.filterDict[key]]) {
          console.error(
            `${this.$options.name} component ckecking error, \n there is no global filter \'${this.filterDict[key]}\' found in current project, you can create it or modify the filterDict's ${key} property within ${this.$options.name} component`
          )
        }
      }
    }
  }
}
</script>
