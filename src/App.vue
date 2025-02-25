<!--
 * @Author: wdl
 * @Email: wangdelei4406@yundasys.com
 * @Date: 2025-02-17 10:05:11
 * @LastEditors: wdl
 * @LastEditTime: 2025-02-25 09:10:47
 * @Description: 
-->
<template>
  <div id="app">
    <div>
      <span>充值总金额：{{ money }}</span>
      <span>抽奖钥匙：{{ key }}</span>
      <span>钥匙碎片：{{ shard }}</span>
      <span>获得的宝箱：{{ box }}</span>
      <span>获得的积分：{{ score }}</span>
    </div>

    <div>
      <button @click="addkey(1)">充值十块</button>
      <button @click="addkey(11)">充值一百</button>
      <button @click="oneClick">抽奖一次</button>
      <button @click="tenClick">抽奖十次</button>
      <button @click="exchange" :disabled="isDisabled">宝箱兑换钥匙碎片</button>
      <button @click="shardChange" :disabled="isDisabled">
        钥匙碎片兑换抽奖钥匙
      </button>
    </div>

    <div v-if="!rewardList.length">获得的奖励：0</div>
    <div v-else>
      获得的奖励：<span
        v-for="(item, index) in rewardList"
        :key="index"
        @click="salvage(item, index)"
        >{{ item.title }}</span
      >
    </div>
    <div></div>
    <div></div>
    <div class="curbox">
      <div
        v-for="(item, index) in curRewardList"
        :key="index"
        :class="item.type === 1 ? 'goods' : ''"
      >
        {{ `${item.title}(${item.min}-${item.max})-${item.random}` }}
      </div>
    </div>
  </div>
</template>

<script>
// active20250205
// active20250211
import { active20250101 } from "./config";
export default {
  name: "App",
  data() {
    return {
      money: 0,
      count: 0,
      key: 0,
      box: 0,
      shard: 0,
      rewardList: [],
      curRewardList: [],
      score: 0,
      active: [],
      isDisabled: false,
    };
  },
  mounted() {
    this.initActive(active20250101);
  },
  methods: {
    initActive(list) {
      let stemp = 0;
      this.active = list.map((item) => {
        const newItem = {
          ...item,
          min: stemp,
          max: stemp + item.rate * 100,
        };
        if (item.isDisabled) {
          this.isDisabled = true;
        }
        stemp = stemp + item.rate * 100;
        return newItem;
      });
      this.$nextTick(() => {
        const item = this.active[this.active.length - 1];
        if (item.max !== 10000) {
          alert("活动概率配置错误！");
        }
      });
    },
    exchange() {
      if (this.box < 1) {
        alert("宝箱数量不足");
        return;
      }
      this.box -= 1;
      this.shard += 3;
    },
    shardChange() {
      if (this.shard < 10) {
        alert("钥匙碎片数量不足");
        return;
      }
      this.shard -= 10;
      this.key++;
    },
    salvage(item, index) {
      if (item.type !== 1) {
        return;
      }
      const newList = this.rewardList.filter((re, i) => i !== index);
      this.rewardList = newList;
      if (item.countType === "score") {
        this.score += item.count;
      } else {
        this.key += item.count;
      }
    },
    addkey(num) {
      if (num === 1) {
        this.money += 10;
      } else {
        this.money += 100;
      }
      this.key += num;
    },
    getReward() {
      const random = Math.ceil(Math.random() * 10000);
      const curItem = this.active.filter((item) => {
        return item.min < random && item.max >= random;
      })[0];
      if (curItem.type === 1) {
        this.rewardList = [...this.rewardList, curItem];
      } else if (curItem.type === 2) {
        this.box++;
      } else {
        this.score += curItem.value;
      }
      curItem.random = random;
      this.curRewardList = [...this.curRewardList, curItem];
    },
    oneClick() {
      if (this.key < 1) {
        alert("抽奖钥匙不足");
        return;
      }
      this.curRewardList = [];
      this.key--;
      this.getReward();
    },
    tenClick() {
      if (this.key < 10) {
        alert("抽奖钥匙不足");
        return;
      }
      this.curRewardList = [];
      this.key -= 10;
      for (let i = 0; i < 10; i++) {
        this.getReward();
      }
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  .curbox {
    border-top: 1px solid greenyellow;
  }
  span,
  button {
    display: inline-block;
    margin: 0 5px;
  }
  div {
    margin: 5px 0;
  }
  .goods {
    color: red;
  }
}
</style>
