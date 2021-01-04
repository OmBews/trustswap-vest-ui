<template>
  <div class="navbar">
    <div class="nav-brand">
      <img src="@/static/images/logo.png" width="80px" />
      <div class="nav-brand-name">TrustSwap Token Claim</div>
    </div>
    <div class="connect-container">
      <button class="round">Connect Wallet</button>
    </div>
  </div>
</template>
<script>
export default {};
</script>
<style lang="scss" scoped>
.navbar {
  padding: 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  .nav-brand {
    display: flex;
    justify-content: space-around;
    align-items: center;
    .nav-brand-name {
      margin-left: 20px;
      color: white;
      font-size: 2rem;
    }
  }
}
</style>
