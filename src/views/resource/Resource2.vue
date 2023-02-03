<template>
  <div>
    <el-table
      :data="tableData"
      :span-method="objectSpanMethod"
      border
      style="width: 100%; margin-top: 20px"
    >
      <el-table-column fixed prop="id" label="ID" width="180">
      </el-table-column>
      <el-table-column prop="name" label="姓名"> </el-table-column>
      <el-table-column prop="amount1" label="数值 1（元）"> </el-table-column>
      <el-table-column prop="amount2" label="数值 2（元）"> </el-table-column>
      <el-table-column prop="amount3" label="数值 3（元）"> </el-table-column>
      <el-table-column fixed="right" label="操作" width="100">
        <template slot-scope="scope">
          <el-button @click="handleClick(scope.row)" type="text" size="small"
            >查看</el-button
          >
          <el-button type="text" size="small">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
export default {
  created() {
    this.getSpanArr(this.tableData);
  },
  data() {
    return {
      spanArr: [],
      tableData: [
        {
          id: "12987121",
          name: "王小虎",
          amount1: "234",
          amount2: "3.2",
          amount3: 10
        },
        {
          id: "12987122",
          name: "王小虎",
          amount1: "165",
          amount2: "4.43",
          amount3: 12
        },
        {
          id: "12987125",
          name: "王小虎",
          amount1: "324",
          amount2: "1.9",
          amount3: 9
        },
        {
          id: "12987125",
          name: "王小虎",
          amount1: "621",
          amount2: "2.2",
          amount3: 17
        },
        {
          id: "12987125",
          name: "王小虎",
          amount1: "539",
          amount2: "4.1",
          amount3: 15
        },
        {
          id: "12987125",
          name: "王小虎",
          amount1: "539",
          amount2: "4.1",
          amount3: 15
        },
        {
          id: "12987125",
          name: "王小虎",
          amount1: "539",
          amount2: "4.1",
          amount3: 15
        },
        {
          id: "12987125",
          name: "王小虎",
          amount1: "539",
          amount2: "4.1",
          amount3: 15
        },
        {
          id: "12987125",
          name: "王小虎",
          amount1: "539",
          amount2: "4.1",
          amount3: 15
        },
        {
          id: "12987125",
          name: "王小虎",
          amount1: "539",
          amount2: "4.1",
          amount3: 15
        }
      ]
    };
  },
  methods: {
    getSpanArr(data) {
      for (let i = 0; i < data.length; i++) {
        if (i === 0) {
          this.spanArr.push(1);
          this.pos = 0;
        } else {
          // 判断当前元素与上一个元素是否相同
          if (data[i].id === data[i - 1].id) {
            this.spanArr[this.pos] += 1;
            this.spanArr.push(0);
          } else {
            this.spanArr.push(1);
            this.pos = i;
          }
        }
      }
    },

    objectSpanMethod({ rowIndex, columnIndex }) {
      if (columnIndex === 0) {
        const _row = this.spanArr[rowIndex];
        const _col = _row > 0 ? 1 : 0;
        return {
          rowspan: _row,
          colspan: _col
        };
      }
    }
  }
};
</script>
