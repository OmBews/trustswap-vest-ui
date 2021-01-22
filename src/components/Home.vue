<template>
  <div class="home-page">
    <div class="container">
      <div class="claim-caption">
        If you have participated in a SWAP private sale, you are eligible to
        collect the vested tokens in an houly basis, or any longer time frame in
        bulk.
      </div>
      <div v-if="isStarted" class="count-indicator">
        <div class="next-claim">Next Claim</div>
        <div class="counter">
          <!-- <div v-if="getHour">{{ getHour }}:</div> -->
          <div>{{ getMin }}:</div>
          <div>{{ getSec }}</div>
        </div>
      </div>
      <div v-else class="not-started-caption">
        <div class="caption">Your vesting time is not started yet</div>
        <div class="timeformat">Start at: {{ startTimeFormat }}</div>
      </div>
      <div class="card-container">
        <div class="card">
          <img src="@/static/images/logo.png" width="80px" />
          <div class="title">Max amount you can claim</div>
          <div class="amount">
            <span>{{ fromWei(amount) }}</span> SWAP
          </div>
        </div>
        <div class="card">
          <img src="@/static/images/logo.png" width="80px" />
          <div class="title">Remaining amount you can claim</div>
          <div class="amount">
            <span>{{ fromWei(remainAmount) }}</span> SWAP
          </div>
        </div>
        <div class="card">
          <img src="@/static/images/logo.png" width="80px" />
          <div class="title">Total tokens you've already claimed</div>
          <div class="amount">
            <span>{{ fromWei(claimedAmount) }}</span> SWAP
          </div>
        </div>
        <div class="card">
          <img src="@/static/images/logo.png" width="80px" />
          <div class="title">Available tokens to be claimed</div>
          <div class="amount">
            <span>{{ fromWei(claimable) }}</span> SWAP
          </div>
        </div>
      </div>
      <div class="footer">
        <button class="round claim-but" @click="claimNow">CLAIM NOW</button>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import BigNumber from "bignumber.js";
import moment from "moment";
export default {
  data: () => ({
    claimedAmount: 0,
    claimable: 0,
    countDown: 0,
    amount: 0,
    startTimestamp: 0,
    startTimeFormat: "",
    lockHours: 0,
    remainAmount: 0,
    isStarted: false,
  }),
  computed: {
    ...mapState({
      address: (state) => state.account.address,
      tokenLocker: (state) => state.contract.tokenLocker,
      web3: (state) => state.metamask.web3,
      provider: (state) => state.metamask.provider,
    }),
    getHour() {
      const hour = Math.floor(this.countDown / 60 / 60) % 24;
      return hour > 9 ? hour : "0" + hour;
    },
    getMin() {
      const min = Math.floor(this.countDown / 60) % 60;
      return min > 9 ? min : "0" + min;
    },
    getSec() {
      const sec = this.countDown % 60;
      return sec > 9 ? sec : "0" + sec;
    },
  },
  watch: {
    async address(value) {
      if (value) await this.loadContract();
    },
  },
  methods: {
    async claimNow() {
      try {
        const { transactionHash } = await this.tokenLocker.methods
          .claimToken(this.claimable)
          .send({
            from: this.address,
          });
        const tx = await this.web3.eth.getTransactionReceipt(transactionHash);
        if (tx) {
          this.$snotify.success(
            `Successfully claimed ${this.fromWei(this.claimable)} SWAP tokens`
          );
          await this.loadContract(false);
        }
      } catch (error) {
        console.error(error);
        this.$snotify.error(error.message);
      }
    },
    fromWei(data) {
      if (!this.web3) return 0;
      return new BigNumber(
        this.web3.utils.fromWei(this.web3.utils.toBN(data))
      ).toFixed(0);
    },
    startCountDown() {
      const currentTimestamp = Math.round(new Date().getTime() / 1000);
      const leftSecs = currentTimestamp - this.startTimestamp;
      this.isStarted = leftSecs > 0;
      if (leftSecs < 0) {
        this.startTimeFormat = moment
          .unix(this.startTimestamp)
          .utc()
          .format("YYYY/MM/DD hh:mm z");
        return;
      }
      const passedHours = Math.floor(leftSecs / 60 / 60);
      const offset = leftSecs - passedHours * 3600;
      this.countDown = 3600 - offset;
      if (passedHours >= this.lockHours)
        this.claimable = new BigNumber(this.amount)
          .minus(this.claimedAmount)
          .toFixed(0);
      else
        this.claimable = new BigNumber(this.amount)
          .dividedBy(this.lockHours)
          .multipliedBy(passedHours)
          .minus(this.claimedAmount)
          .toFixed(0);
      this.remainAmount = new BigNumber(this.amount).minus(this.claimedAmount);
      setTimeout(() => this.startCountDown(), 1000);
    },
    async loadContract(startTimer) {
      if (!this.tokenLocker || !this.web3) return;
      const res = await this.tokenLocker.methods
        .getLockData(this.address)
        .call();
      this.amount = res[0];
      this.startTimestamp = res[1];
      this.lockHours = res[2];
      this.claimedAmount = res[3];
      if (startTimer) this.startCountDown();
    },
  },
  async mounted() {
    await this.loadContract(true);
  },
};
</script>
<style lang="scss" scoped>
.home-page {
  padding: 2rem;
  .container {
    box-shadow: 0 5px 10px rgb(58 149 214 / 90%),
      0 15px 40px rgb(51 75 169 / 50%);
    padding: 40px;
    border-radius: 40px;
    width: 80%;
    margin: auto;
  }
  .claim-caption {
    text-align: center;
    font-size: 1.3rem;
  }
  .count-indicator {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 1rem;
    font-size: 2rem;
    .next-claim {
      margin-right: 1rem;
      font-size: 1.5rem;
    }
    .counter {
      display: flex;
      align-items: center;
      font-size: 3rem;
      margin-right: 10px;
    }
    &.blue {
      color: #0d1365;
    }
  }
  .not-started-caption {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 1rem;
    font-size: 1.2rem;
    gap: 10px;
    .caption {
      color: #0d1365;
    }
  }
  .card-container {
    padding: 3rem 0;
    display: flex;
    justify-content: center;
    .card {
      &:not(:last-child) {
        margin-right: 20px;
      }
      padding: 5rem 2rem;
      border-radius: 15px;
      max-width: 200px;
      box-shadow: 0 0px 21px rgb(173 221 255 / 90%),
        0 0px 53px rgb(40 123 245 / 17%);
      background: #1519253d;
      display: flex;
      flex-direction: column;
      align-items: center;
      text-align: center;
      .title {
        margin-top: 20px;
      }
      .amount {
        margin-top: 10px;
        font-size: 1.5rem;
        word-break: break-word;
        span {
          font-size: 2.5rem;
          font-weight: 500;
        }
      }
    }
  }
  .footer {
    justify-content: center;
    display: flex;
    .claim-but {
      font-size: 2rem;
    }
  }
}
</style>
