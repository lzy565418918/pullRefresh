<template>
  <div id="list">
    <div class="header">
      <span>图标</span>
      <span>名称</span>
      <span>技能</span>
    </div>
    <van-pull-refresh
      v-model="isRefresh"
      @refresh="onRefresh"
      success-text="刷新成功"
      :error.sync="errRefresh"
    >
      <van-list
        v-model="isLoading"
        :finished="isFinished"
        finished-text="我是有底线的~"
        @load="onLoad"
        :error.sync="isError"
        :offset="offset"
      >
        <div class="content" v-for="(item,index) in list" :key="index">
          <img class="heroIcon" :src="item.icon" :alt="item.name" />
          <span class="heroName">{{item.name}}</span>
          <span class="heroSkill">{{item.skill}}</span>
        </div>
      </van-list>
    </van-pull-refresh>
  </div>
</template>
<script>
import { List, PullRefresh } from "vant";
export default {
  components: {
    [List.name]: List,
    [PullRefresh.name]: PullRefresh
  },
  data() {
    return {
      // 是否正在加载中
      isLoading: false,
      // 是否加载已完成
      isFinished: false,
      // 是否加载出错
      isError: false,
      // 数据源
      list: [],
      // 当前页
      currentPage: 1,
      // 一页有多少条
      pageSize: 150,
      // 总页数
      totalPage: 1,
      // 滚动条距离底部多少像素触发加载更多
      offset: 100,
      // 是否正处于刷新状态
      isRefresh: false,
      // 刷新是否失败
      errRefresh: false
    };
  },
  computed: {},
  watch: {},
  beforeCreate() {},
  created() {},
  beforeMount() {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {},
  methods: {
    // 加载更多
    onLoad() {
      // 如果当前页大于总页数，则加载已完成
      if (this.currentPage > this.totalPage) {
        this.isFinished = true;
        return;
      }
      // 无网络直接请求失败
      if (!navigator.onLine) {
        this.isError = true;
        return
      }
      this.$axios
        .get("https://autumnfish.cn/api/cq/page", {
          params: {
            pageNum: this.currentPage,
            pageSize: this.pageSize
          }
        })
        .then(res => {
          // 记录总页数
          this.totalPage = res.data.totalPage;
          // 累加数据
          this.list = [...this.list, ...res.data.list];
          this.currentPage++;
          // 需要置为false才能继续加载
          this.isLoading = false;
          console.log(this.list);
        })
        .catch(() => {
          this.isError = true;
        });
    },
    onRefresh() {
      if (!navigator.onLine) {
        this.errRefresh = true;
        return;
      }
      // 初始化当前页数
      this.currentPage = 1;
      this.$axios
        .get("https://autumnfish.cn/api/cq/page", {
          params: {
            pageNum: this.currentPage,
            pageSize: this.pageSize
          }
        })
        .then(res => {
          //  记录总页数
          this.totalPage = res.data.totalPage;
          // 只获取第一页数据
          this.list = res.data.list;
          this.currentPage++;
          this.isRefresh = false;
          console.log(this.list);
        });
    }
  }
};
</script>
<style lang="less" scoped>
#list {
  .header {
    display: flex;
    justify-content: space-between;
    border: 1px solid #aaaaaa;
    height: 30px;
    line-height: 30px;
    padding: 0 16px;
  }
  .content {
    display: flex;
    justify-content: space-between;
    padding: 0 16px;
    span {
      display: inline-block;
      font-size: 18px;
      color: red;
      height: 40px;
      line-height: 40px;
    }
    .heroIcon {
      width: 30px;
      height: 30px;
      vertical-align: center;
    }
  }
}
</style>