<template>
  <div class="home-page">
    <div class="container">
      <div class="claim-caption">
        If you have participated in a SWAP private sale, you are eligible to
        collect the vested tokens in an houly basis, or any longer time frame in
        bulk.
      </div>
      <div class="count-indicator">
        <div class="next-claim">Next Claim</div>
        <div class="counter">
          <div v-if="hour">{{ getHour }}:</div>
          <div>{{ getMin }}:</div>
          <div>{{ getSec }}</div>
        </div>
      </div>
      <div class="card-container">
        <div class="card">
          <img src="@/static/images/logo.png" width="80px" />
          <div class="title">Total tokens you've already claimed</div>
          <div class="amount">
            <span>{{ totalClaimed }}</span> SWAP
          </div>
        </div>
        <div class="card">
          <img src="@/static/images/logo.png" width="80px" />
          <div class="title">Available tokens to be claimed</div>
          <div class="amount">
            <span>{{ claimable }}</span> SWAP
          </div>
        </div>
      </div>
      <div class="footer">
        <button class="round claim-but">CLAIM NOW</button>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from "vuex";
import BigNumber from "bignumber.js";
export default {
  data: () => ({
    claimable: 0,
    totalClaimed: 0,
    countDown: 0,
  }),
  computed: {
    ...mapState({
      address: (state) => state.account.address,
      tokenLocker: (state) => state.contract.tokenLocker,
      web3: (state) => state.metamask.web3,
      provider: (state) => state.metamask.provider,
    }),
    getHour() {
      return 0;
    },
    getMin() {
      return 0;
    },
    getSec() {
      return 0;
    },
  },
  watch: {
    async address(value) {
      if (value) await this.loadContract();
    },
  },
  methods: {
    async loadContract() {
      if (!this.tokenLocker) return;
    },
  },
  async mounted() {
    await this.loadContract();
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
