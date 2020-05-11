<template>
  <div>
    <el-table
      :data="tableData"
      style="width: 100%"
    >
      <el-table-column
        label="日期"
        width="180"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span style="margin-left: 10px">{{ scope.row.date }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="姓名"
        width="180"
      >
        <template slot-scope="scope">
          <el-popover trigger="hover" placement="top">
            <p>姓名: {{ scope.row.name }}</p>
            <p>住址: {{ scope.row.address }}</p>
            <div slot="reference" class="name-wrapper">
              <el-tag size="medium">{{ scope.row.name }}</el-tag>
            </div>
          </el-popover>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template #default="scope">
          <el-button
            size="mini"
            @click="handleEdit(scope.$index, scope.row)"
          >编辑</el-button>
          <el-button
            size="mini"
            type="danger"
            @click="handleDelete(scope.$index, scope.row)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <hr>

    <common-table :table-data="testData" :columns-list="testColumn2" @click="abc" />
  </div>
</template>

<script>
import CommonTable from '@/components/Table/CommonTable'
export default {
  components: {
    CommonTable
  },

  data() {
    return {
      testColumn: ['DokD-date$t-100%', 'ADDRESS-address-200%', 'NAME-name-100%', 'momey-number$a-100%'],
      testColumn2: [
        { title: '2DDD', key: 'date$t', width: '100%', attrOptions: { fixed: 'right' }, eventOptions: { click: () => { this.abc() } }},
        { title: 'ADDRESS', key: 'address', width: '200%' },
        { title: 'NAME', key: 'name', width: '100%' },
        { title: 'momey', key: 'number$a', width: '100%' }
      ],

      testData: [
        {
          date: '2016-05-02',
          name: '王虎1',
          address: ' 1518 弄',
          number: '123123123'
        },
        {
          date: '2016-05-04',
          name: '王小虎2',
          address: '上海市普陀区金沙 弄',
          number: '123123123'
        },
        {
          date: '2016-05-01',
          name: '王小虎3',
          address: '上海市普陀区金沙江路 1519 弄'
        },
        {
          date: '2016-05-03',
          name: '王小虎4',
          address: '上海市普陀区金沙江路 1516 弄'
        }
      ],

      tableData: [{
        date: '2016-05-02',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
      }, {
        date: '2016-05-04',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1517 弄'
      }, {
        date: '2016-05-01',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1519 弄'
      }, {
        date: '2016-05-03',
        name: '王小虎',
        address: '上海市普陀区金沙江路 1516 弄'
      }]
    }
  },
  methods: {
    handleEdit(index, row) {
      console.log(index, row)
    },
    handleDelete(index, row) {
      console.log(index, row)
    },
    abc() {
      console.log('abc')
    }
  }
}
</script>
