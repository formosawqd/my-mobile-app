<template>
  <div style="padding: 16px; height: 100%">
    <!-- vant-pull-refresh 默认就是滚动容器，需撑满高度 -->
    <van-pull-refresh
      v-model="refreshing"
      @refresh="onRefresh"
      style="height: 90%; overflow: auto"
      ref="pullRefresh"
    >
      <a-table
        :columns="columns"
        :data-source="data"
        :pagination="false"
        row-key="key"
        :loading="loading"
        @change="onTableChange"
        :scroll="{ x: 1600 }"
        style="min-width: 1600px"
      >
        <template slot="col1" slot-scope="text">
          <span class="copyable" title="点击复制" @click="copyName(text)">
            {{ text }}
          </span>
        </template>

        <template #col10="{ text }">
          <span>{{ formatDate(text) }}</span>
        </template>
      </a-table>
      <div style="text-align: center; padding: 8px 0">
        <a-spin v-if="loadingMore" />
        <span v-else-if="finished">没有更多了</span>
      </div>
    </van-pull-refresh>
  </div>
</template>

<script>
import "ant-design-vue/dist/antd.css";
import { Table, Spin, message } from "ant-design-vue";
import { Toast } from "vant";

// 构造模拟数据
const ALL_MOCK = [];
for (let i = 1; i <= 50; i++) {
  ALL_MOCK.push({
    key: i,
    col1: `用户姓名数据很长很长${i}`,
    col2: i + 10,
    col3: ["男", "女"][i % 2],
    col4: `电话很长：18800${i}88888888888888`,
    col5: i % 5 ? "部门A数据很长" : "部门B数据更长",
    col6: 10 + (i % 10),
    col7: `地址地址${i}数据非常非常长`,
    col8: ["是", "否"][i % 2],
    col9: ["组1超级超级长组名", "组2", "组3", "组4", "组5"][i % 5],
    col10: new Date(Date.now() - i * 86400000),
  });
}

export default {
  name: "PullRefreshTableWithSortNoWrap",
  components: {
    "a-table": Table,
    "a-spin": Spin,
  },
  data() {
    return {
      columns: [
        {
          title: "姓名",
          dataIndex: "col1",
          key: "col1",
          sorter: true,
          scopedSlots: { customRender: "col1" },
        },
        { title: "年龄", dataIndex: "col2", key: "col2", sorter: true },
        { title: "性别", dataIndex: "col3", key: "col3" },
        { title: "电话", dataIndex: "col4", key: "col4" },
        { title: "部门", dataIndex: "col5", key: "col5" },
        { title: "成绩", dataIndex: "col6", key: "col6", sorter: true },
        { title: "地址", dataIndex: "col7", key: "col7" },
        { title: "是否在职", dataIndex: "col8", key: "col8" },
        { title: "分组", dataIndex: "col9", key: "col9" },
        {
          title: "注册日期",
          dataIndex: "col10",
          key: "col10",
          sorter: true,
          scopedSlots: { customRender: "col10" },
        },
      ],
      data: [],
      page: 1,
      pageSize: 10,
      total: ALL_MOCK.length,
      loading: false,
      loadingMore: false,
      finished: false,
      sorter: {
        field: null,
        order: null,
      },
      refreshing: false,
    };
  },
  mounted() {
    this.loadData(true);
    // 监听滚动事件，实现上拉加载更多
    this.$nextTick(() => {
      const pr = this.$refs.pullRefresh;
      if (pr && pr.$el) {
        pr.$el.addEventListener("scroll", this.onScroll, { passive: true });
      }
    });
  },
  beforeDestroy() {
    const pr = this.$refs.pullRefresh;
    if (pr && pr.$el) {
      pr.$el.removeEventListener("scroll", this.onScroll);
    }
  },
  methods: {
    loadData(reset = false) {
      if (reset) {
        this.page = 1;
        this.data = [];
        this.finished = false;
      }
      this.loading = reset;
      this.loadingMore = !reset;
      setTimeout(
        () => {
          let arr = ALL_MOCK.slice(0);
          if (this.sorter.field && this.sorter.order) {
            arr.sort((a, b) => {
              let v1 = a[this.sorter.field];
              let v2 = b[this.sorter.field];
              if (this.sorter.field === "col10") {
                v1 = new Date(v1);
                v2 = new Date(v2);
              }
              if (v1 > v2) return this.sorter.order === "ascend" ? 1 : -1;
              if (v1 < v2) return this.sorter.order === "ascend" ? -1 : 1;
              return 0;
            });
          }

          const start = (this.page - 1) * this.pageSize;
          const end = Math.min(this.page * this.pageSize, this.total);
          const pageData = arr.slice(start, end);
          if (reset) {
            this.data = pageData;
          } else {
            this.data = [...this.data, ...pageData];
          }
          this.finished = this.data.length >= this.total;
          this.loading = false;
          this.loadingMore = false;
          this.refreshing = false;
        },
        reset ? 800 : 1000
      );
    },
    onRefresh() {
      this.loadData(true);
    },
    onScroll(event) {
      if (this.loading || this.loadingMore || this.finished || this.refreshing)
        return;

      const el = event.target;
      const threshold = 40;
      if (el.scrollTop + el.clientHeight + threshold >= el.scrollHeight) {
        this.page += 1;
        this.loadData(false);
      }
    },
    onTableChange(pagination, filters, sorter) {
      console.log(111);

      if (!sorter.order) {
        this.sorter = { field: null, order: null };
      } else {
        this.sorter = {
          field: sorter.field,
          order: sorter.order,
        };
      }
      this.loadData(true);
    },
    formatDate(v) {
      const d = new Date(v);
      return `${d.getFullYear()}-${d.getMonth() + 1}-${String(
        d.getDate()
      ).padStart(2, "0")}`;
    },
    copyName(text) {
      if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard
          .writeText(text)
          .then(() => Toast.success("复制成功"))
          .catch(() => Toast.error("复制失败"));
      } else {
        const input = document.createElement("input");
        input.setAttribute("readonly", "readonly");
        input.value = text;
        document.body.appendChild(input);
        input.select();
        try {
          if (document.execCommand("copy")) {
            Toast.success("复制成功");
          } else {
            Toast.error("复制失败");
          }
        } catch (e) {
          message.error("复制失败");
        }
        document.body.removeChild(input);
      }
    },
  },
};
</script>

<style scoped>
/* 表格滚动容器样式 */
/* .table-scroll-content {
  border: 1px solid #eee;
  border-radius: 5px;
  background: #fff;
} */
/* 表头和内容不换行并超出省略号 */
::v-deep .ant-table-thead > tr > th,
::v-deep .ant-table-tbody > tr > td {
  white-space: nowrap !important;
  /* text-overflow: ellipsis; */
  /* overflow: hidden;
  max-width: 160px;
  min-width: 40px; */
}
/* 文字左对齐 */
/* ::v-deep .ant-table-thead > tr > th,
::v-deep .ant-table-tbody > tr > td {
  text-align: left;
  vertical-align: middle;
} */
/* 横向滚动 */
/* ::v-deep .ant-table-content {
  overflow-x: auto !important;
} */
/* 可复制cell样式 */
.copyable {
  color: #1890ff;
  cursor: pointer;
  text-decoration: underline;
  transition: color 0.2s;
  user-select: all;
}
.copyable:hover {
  color: #40a9ff;
}
</style>
