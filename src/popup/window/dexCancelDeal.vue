<template>
  <div class="cdp">
    <div class="content">
      <h5 class="titleHeader">{{$t('window.cdp.qxjy')}}</h5>
      <div class="cell">
        <p class="cellName">{{$t('account.transDetail.hashLabel')}}</p>
        <p class="cellValue coin-card-copy" style="padding-right:20px">
          {{cutMiddleStr(dealNum,12)}}
          <img class="copyImg" src="../static/copy-icon-blue.svg" />
        </p>
      </div>
      <div class="cell" style="padding-top:0">
        <div class="cellName" style="width:100%;line-height:50px;border-top: 1px #eeeeee solid;">
          {{$t('window.cdp.ddxq')}}
          <div class="subcell">
            <p class="cellName">{{$t('window.cdp.lx')}}</p>
            <p class="cellValue">{{formatNewTxType(dexType)}}</p>
          </div>
          <div class="subcell">
            <p class="cellName">{{dexType == "" ? '' : confirmType(dexType)[1]}}({{danweiStr1}})</p>
            <p class="cellValue">{{wiccNum}}</p>
          </div>
          <div class="subcell">
            <p class="cellName">{{dexType == "" ? '' : confirmType(dexType)[2]}}({{danweiStr2}})</p>
            <p class="cellValue">{{price}}</p>
          </div>
          <div v-if="dexType.indexOf('MARKET')==-1" class="subcell">
            <p class="cellName">{{dexType == "" ? '' : confirmType(dexType)[3]}}({{danweiStr3}})</p>
            <p class="cellValue">{{dexType.indexOf('Market') > -1 ? $t('window.cdp.sjcjwz') : (wiccNum * price).toFixed(8)}}</p>
          </div>
        </div>
      </div>
      <div class="bar"></div>
    </div>

    <div class="feesView">
      <select class="feesName" name="WICC" id v-model="feesName">
        <option value="WICC">WICC</option>
        <option value="WUSD">WUSD</option>
      </select>
      <fees-slider v-model="fees" type="dex" :feeName="feesName"></fees-slider>
    </div>
    <div class="bottom_btn">
       <div class="btn" @click="cancel">{{$t('window.cdp.qx')}}</div>
      <div class="btn sure" @click="sure">{{$t('window.cdp.qd')}}</div>
    </div>
  </div>
</template>
<script>
import transUtil from '../account/components/trans-util';
import CopyMixin from "../components/copy-mixin";
import FeesSlider from "../components/fees-slider";
import API from "../api";
import WindowMixin from "./mixin";
import formatError from "../api/format-error";
export default {
  mixins: [WindowMixin, CopyMixin],
  components: { FeesSlider },
  data() {
    return {
      urlQuery: "",
      fees: 0.001,
      cdpTxId: "",
      wiccNum: 0,
      wusdNum: 0,
      price: 0,
      feesName: "WICC",
      dexType: "",
      dealNum: "",
      clipboardSelector: ".coin-card-copy",
      assetType:'',
      coinType:'',
      danweiStr1:"",
      danweiStr2:"",
      danweiStr3:"",
      Tiles: {
        limitTitle1: [
          this.$t('window.cdp.xjmc'),
          this.$t('window.cdp.slwicc'),
          this.$t('window.cdp.jgwusd'),
          this.$t('window.cdp.hdslwusd'),
        ],
        limitTitle2: [
          this.$t('window.cdp.xjmr'),
          this.$t('window.cdp.slwusd'),
          this.$t('window.cdp.jgwusd'),
          this.$t('window.cdp.hdslwicc'),
        ],
        marketTitle1: [
          this.$t('window.cdp.sjmc'),
          this.$t('window.cdp.slwicc'),
          this.$t('window.cdp.dqscjg'),
          this.$t('window.cdp.yjhdwusd'),
        ],
        marketTitle2: [
          this.$t('window.cdp.sjmr'),
          this.$t('window.cdp.slwusd'),
          this.$t('window.cdp.dqscjg'),
          this.$t('window.cdp.yjhdwicc'),
        ]
      }
    };
  },
  ///type : 0 限价卖出，1 限价买入， 2 市价卖出，3市价买入
  created() {
    const query = this.$router.currentRoute.query;
    this.callbackId = query.callbackId;
    this.dealNum = query.dealNum;
    console.log(query);
    this.getDetailInfo();
  },
  methods: {
    formatNewTxType: transUtil.formatNewTxType,
    confirmType(dexType) {
      if (dexType == "DEX_LIMIT_SELL_ORDER_TX") {
        this.danweiStr1 = this.assetType
        this.danweiStr2 = this.coinType
        this.danweiStr3 = this.coinType
        return this.Tiles.limitTitle1;
      }
      if (dexType == "DEX_LIMIT_BUY_ORDER_TX") {
        this.danweiStr1 = this.assetType
        this.danweiStr2 = this.coinType
        this.danweiStr3 = this.coinType
        return this.Tiles.limitTitle2;
      }
      if (dexType == "DEX_MARKET_SELL_ORDER_TX") {
        this.danweiStr1 = this.assetType
        this.danweiStr2 = this.coinType
        this.danweiStr3 = this.coinType
        return this.Tiles.marketTitle1;
      }
      if (dexType == 'DEX_MARKET_BUY_ORDER_TX') {
        this.danweiStr1 = this.coinType
        this.danweiStr2 = this.coinType
        this.danweiStr3 = this.assetType
        return this.Tiles.marketTitle2;
      }
    },
    sure() {
      this.$loading(this.$t("window.cdp.zzqxjy")); //this.$t("window.transfer.confirmLoading")

      let param = {
        fees: parseFloat(this.fees) * Math.pow(10, 8),
        orderId: this.dealNum,
        address: this.address,
        feeType: this.feesName
      };
      this.callRaw("dexCancel", param);
    },
    callRaw(method, param) {
      API.callRaw(method, { info: param }).then(
        res => {
          console.log(res);
          this.$loading.close();
          this.$toast(res, {
            type: "center",
            duration: 5000,
            wordWrap: true
          });
          if (this.callbackId) {
            API.callPageCallback(this.callbackId, null, res);
            this.$toast("Success", {
              type: "center",
              duration: 1000,
              wordWrap: true
            });
            setTimeout(() => {
              window.close();
            }, 300);
          }
        },
        error => {
          this.$loading.close();
          this.$toast(
            this.$t("window.transfer.createFailure") + " " + formatError(error),
            {
              type: "center",
              duration: 5000,
              wordWrap: true
            }
          );
        }
      );
    },

    getDetailInfo(){
        let param = {
          "hash": this.dealNum
        };
        API.callRaw('getDetailInfo',{info:param}).then(
        res => {
          console.log(res)
          this.dexType = res.txtype;
          this.price = res.price ? res.price/100000000 :this.$t('window.cdp.sjcjwz') ;
          this.wiccNum = res.assetamount ? res.assetamount/100000000 : res.coinamount/100000000;
          this.assetType = res.assetsymbol;
          this.coinType = res.coinsymbol;
        },
        error => {
          this.$loading.close();
          this.$toast(
            this.$t("window.transfer.createFailure") + " " + formatError(error),
            {
              type: "center",
              duration: 5000,
              wordWrap: true
            }
          );
        }
      );
    },

    getCopyText() {
      return this.dealNum;
    }
  }
};
</script>
<style lang="scss" scoped>
.cdp {
  .titleHeader {
    line-height: 64px;
    text-align: center;
    border-bottom: 1px solid rgba($color: #b4bccc, $alpha: 0.3);
    font-size: 18px;
    margin-bottom: 0;
  }
  p {
    margin-bottom: 0;
    line-height: 18px;
  }
}
.content {
  height: 447px;
  position: relative;
  .bar {
    position: absolute;
    height: 10px;
    bottom: 0;
    width: 100%;
    background: #f2f5fc;
  }
}
.cell {
  position: relative;
  display: flex;
  justify-content: space-between;
  padding: 20px 16px;
  width: 100%;
  box-sizing: border-box;

  .subcell {
    display: flex;
    justify-content: space-between;
    margin-top: 10px;
  }
  .cellName {
    color: #b4bccc;
    font-size: 13px;
  }
  .cellValue {
    color: #5b5f67;
    font-size: 13px;
    position: relative;
  }
  .copyImg {
    position: absolute;
    right: 0px;
    top: 0px;
  }
}

.feesView {
  padding-top: 47px;
  position: relative;
  .feesName {
    border: none;
    position: absolute;
    top: 10px;
    right: 20px;
  }
}
.bottom_btn {
  display: flex;
  justify-content: flex-start;
  margin-top: 16px;
  .btn {
    width: calc(50% - 24px);
    margin-left: 16px;
    text-align: center;
    font-size: 16px;
    line-height: 48px;
    border-radius: 4px;
    border: 1px #b4bccc solid;
    cursor: pointer;
  }
  .sure {
    background: -webkit-linear-gradient(
      #3c78ea,
      #004eec
    ); /* Safari 5.1 - 6.0 */
    background: -o-linear-gradient(#3c78ea, #004eec); /* Opera 11.1 - 12.0 */
    background: -moz-linear-gradient(#3c78ea, #004eec); /* Firefox 3.6 - 15 */
    background: linear-gradient(#3c78ea, #004eec); /* 标准的语法 */
    color: white;
    border: none;
  }
}
</style>
