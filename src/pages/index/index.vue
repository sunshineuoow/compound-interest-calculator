<template>
  <div class="container">
    <div class="calculate-form">
      <div class="input-item">
        <label for="principal">本金</label>
        <input type="number" v-model="principal" name="principal" placeholder="0">
        <span>元</span>
      </div>
      <div class="input-item">
        <label for="rate">利率</label>
        <input type="digit" v-model="rate.value" name="rate" placeholder="0">
        <select name="rate" :value="rate.period" :range="timePeriod" @change="handleChange('rate', $event)">
          <option>%/{{ timePeriod[rate.period] }}</option>
        </select>
      </div>
      <div class="input-item">
        <label for="kind">复利方式</label>
        <select name="kind" :value="kind" :range="timePeriod" @change="handleChange('kind', $event)">
          <option>{{ timePeriod[kind] }}</option>
        </select>
      </div>
      <div class="input-item">
        <label for="amount">存</label>
        <input type="number" v-model="amount.value" name="amount" placeholder="0">
        <select name="amount" :value="amount.period" :range="timePeriod" @change="handleChange('amount', $event)">
          <option>元/{{ timePeriod[amount.period] }}</option>
        </select>
      </div>
      <div class="input-item">
        <label for="time">期限</label>
        <input type="number" v-model="time.value" name="time" placeholder="0">
        <select name="time" :value="time.period" :range="timePeriod" @change="handleChange('time', $event)">
          <option>{{ timePeriod[time.period] }}</option>
        </select>
      </div>
    </div>

    <p>计算结果</p>

    <div class="calculate-result">
      <div class="input-item">
        <label>本金</label>
        <input type="number" v-model="result.principal" disabled>
      </div>
      <div class="input-item">
        <label>利息</label>
        <input type="number" v-model="result.profit" disabled>
      </div>
      <div class="input-item">
        <label>合计</label>
        <input type="number" v-model="result.value" disabled>
      </div>
    </div>

    <div class="calculate-btn">
      <button @click="calculateProfit()">开始计算</button>
    </div>
  </div>
</template>

<script>

export default {
  data () {
    return {
      timePeriod: ['年', '月', '日'],
      principal: '',
      time: {
        value: '',
        period: 0
      },
      amount: {
        value: '',
        period: 0
      },
      rate: {
        value: '',
        period: 0
      },
      kind: 0,
      result: {
        principal: 0,
        profit: 0,
        value: 0
      }
    }
  },

  methods: {
    clickHandle (msg, ev) {
      console.log('clickHandle:', msg, ev)
    },
    handleChange (type, ev) {
      const value = ev.mp.detail.value
      if (type === 'kind') {
        this[type] = Number(value)
      } else {
        this[type]['period'] = Number(value)
      }
    },
    transferPeriod (type) {
      const period = this.kind
      if (period === 0) {
        switch (this[type].period) {
          case 0:
            return 1
          case 1:
            return 1 / 12
          case 2:
            return 1 / 365
        }
      } else if (period === 1) {
        switch (this[type].period) {
          case 0:
            return 12
          case 1:
            return 1
          case 2:
            return 1 / 30
        }
      } else {
        switch (this[type].period) {
          case 0:
            return 365
          case 1:
            return 30
          case 2:
            return 1
        }
      }
    },
    calculateProfit () {
      // 转换成 复利计算方式的期限
      // 复利 = 本金(1 + 利率)^期限 - 本金
      const rate = this.rate.value / (this.transferPeriod('rate') * 100) // 利率(每期复利)
      const payTime = this.transferPeriod('time') * this.time.value // 投资时间
      const calTime = this.transferPeriod('amount') // 本金增加间隔
      const count = payTime / calTime

      if (count >= 1) {
        let principal = Number(this.principal)
        let total = principal * Math.pow(1 + rate, payTime)
        for (let i = 0; i < count; i++) {
          total += Number(this.amount.value * Math.pow(1 + rate, (payTime - (i + 1) * calTime)))
          principal += Number(this.amount.value)
        }
        this.result.principal = principal.toFixed(2)
        this.result.value = total.toFixed(2)
        this.result.profit = (total - principal).toFixed(2)
      } else {
        this.result.principal = this.principal
        const total = this.principal * Math.pow(1 + rate, payTime)
        this.result.profit = (total - this.principal).toFixed(2)
        this.result.value = total.toFixed(2)
      }
    }
  }
}
</script>

<style scoped>

.userinfo {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.userinfo-avatar {
  width: 128rpx;
  height: 128rpx;
  margin: 20rpx;
  border-radius: 50%;
}

.userinfo-nickname {
  color: #aaa;
}

.usermotto {
  margin-top: 150px;
}

p {
  font-family: PingFang-SC-Regular;
  font-size: 12px;
  color: #6A6A6A;
  text-align: center;
}

.calculate-form {
  width: 100%;
  padding: 12px 12px 4px 12px;
  box-sizing: border-box;
}

.calculate-result {
  width: 100%;
  padding: 8px 12px 12px;
  box-sizing: border-box;
}

.input-item {
  display: flex;
  width: 100%;
  margin-bottom: 4px;
  padding: 14px 28px 14px 14px;
  background-color: #fff;
  box-sizing: border-box;
}

.input-item label {
  min-width: 20%;
  font-family: PingFangSC-Bold;
  font-size: 14px;
  color: #333333;
  line-height: 20px;
}

.input-item input {
  width: 60%;
  min-height: 20px;
  height: 20px;
  font-family: PingFangSC-Bold;
  font-size: 14px;
  color: #333333;
  line-height: 20px;
  text-align: center;
}

.input-item span,
.input-item select {
  width: 20%;
  text-align: right;
  font-family: PingFangSC-Bold;
  font-size: 14px;
  color: #333333;
}

.calculate-form .input-item:nth-child(3) label {
  margin-right: 13%;
}

.calculate-btn {
  position: absolute;
  bottom: 0;
  left: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 14px;
  width: 100%;
  height: 72px;
  border-top: 1px solid #eee;
  background-color: #fff;
  box-sizing: border-box;
}

.calculate-btn button {
  width: 100%;
  font-family: PingFangSC-Regular;
  font-size: 18px;
  color: #FFFFFF;
  text-align: center;
  background-color: #FE9614;
  border-radius: 5px;
}

</style>
