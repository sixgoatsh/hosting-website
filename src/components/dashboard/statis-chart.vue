<template>
  <div class="pos-r">
    <div class="pos-a top-0 w100p">
      <h4>{{ title }}</h4>
    </div>
    <div ref="chart" style="height: 200px"></div>
  </div>
</template>

<script>
import * as echarts from "echarts";

export default {
  props: {
    title: String,
    type: String,
    appId: String,
  },
  data() {
    return {
      timeLimit: "HOUR_24", //DAY_7 DAY_30
      list: [],
    };
  },
  watch: {
    appId() {
      this.getData();
    },
    timeLimit() {
      this.getData();
    },
  },
  mounted() {
    this.getData();
  },
  methods: {
    async getData() {
      try {
        const {
          data: { data },
        } = await this.$http.get("/analytics/user/view/data", {
          params: {
            viewType: this.type,
            projectId: this.appId,
            timeLimit: this.timeLimit,
          },
        });
        // console.log(data)
        // const now = new Date()
        const list = [];
        const xArr = [];
        const yArr = [];
        for (const key in data) {
          list.push({
            time: new Date(key * 1e3),
            num: data[key],
          });
        }
        list.sort((a, b) => {
          return a.time - b.time;
        });
        this.list = list;
        for (const it of list) {
          const fmt = /hour/i.test(this.timeLimit) ? "HH:mm" : "MM-dd";
          xArr.push(it.time.format(fmt));
          // it.num = (Math.random() * 20) | 0;
          yArr.push(it.num);
        }
        this.setData(xArr, yArr);
      } catch (error) {
        //
      }
    },
    setData(xArr, yArr) {
      const el = this.$refs.chart;
      // console.log(el, echarts)
      const chart = echarts.init(el);
      const option = {
        // title: {
        //   text: this.title,
        // },
        xAxis: {
          type: "category",
          boundaryGap: false,
          data: xArr,
        },
        yAxis: {
          type: "value",
          minInterval: 1,
          splitLine: {
            show: false,
          },
        },
        tooltip: {
          trigger: "axis",
          formatter: (params) => {
            const { dataIndex: idx } = params[0];
            const obj = this.list[idx];
            return `${obj.time.format().replace(/:00$/, "")}<br><b>${
              obj.num
            }</b>`;
          },
        },
        series: [
          {
            data: yArr,
            type: "line",
            symbol: "none",
            areaStyle: {
              color: "rgba(52, 169, 255, 0.2)",
            },
            itemStyle: {
              color: "rgba(52, 169, 255, 0.2)",
            },
          },
        ],
      };
      chart.setOption(option);
    },
  },
};
</script>