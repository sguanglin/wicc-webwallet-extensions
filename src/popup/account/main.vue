<template>
  <main-layout>
    <main-header></main-header>

    <coin-card
      style="margin: 16px;"
      :show-register-button="false"
      name="wicc"
      :address="activeAddress"
      :value="$t('common.accountLabel') + `${activeAccount ? activeAccount.index + 1 : 1}`"
    ></coin-card>

    <token-list
      slot="body"
      @active-token-change="handleActiveTokenChange"
      :active-reg-id="null"
      :active-account="activeAccount"
      :active-address="activeAddress"
      :network="network"
      :tokens="tokens"
    ></token-list>
  </main-layout>
</template>

<style scoped lang="scss">
.button-wrapper {
  > button {
    flex: 1 0 0;
  }

  > button:first-child {
    margin-right: 10px;
  }
}
</style>

<script type="text/jsx">
import API from "../api";
import { DrawerLayout } from "vue-drawer-layout";
import TokenList from "./components/token-list";
import CoinCard from "./components/coin-card";
import TransHistory from "./components/trans-history";
import StateWatcher from "./state-watcher";
import MainLayout from "./components/main-layout";
import MainHeader from "./components/main-header";
export default {
  mixins: [StateWatcher],

  components: {
    MainLayout,
    DrawerLayout,
    TokenList,
    CoinCard,
    TransHistory,
    MainHeader
  },

  methods: {
    handleActiveTokenChange(token) {
      if (this.isRealNum(token.num)){
        this.$router.push({ name: "WiccRecord",query:{coinNum:token.num,coinName:token.name}});
        return
      }
      if (token) {
        this.$router.push({
          name: "tokenMain",
          query: {
            name: token.name,
            regId: token.regId
          }
        });
      } 
    },

    toggleDrawer() {
      this.$refs.drawerLayout.toggle();
    },
    isRealNum(val){
      if (typeof val !== 'number'){
        return false
      }
      if (!isNaN(val)){
        return true
      }else{
        return false
      }
    }
  },

  data() {
    return {
      loading: false,

      activeAccountInfo: null,
      transactions: null
    };
  }
};
</script>
