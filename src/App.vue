<script>
export default {
  data() {
    return {
      urlInput: "",
      urlList: [],
      payment: {}
    }
  },

  methods: {
    getLedger() { 
      let url = new URL(this.urlInput);
      this.urlInput = "";
      if (this.urlList.includes(url.href)) { return; }
      this.urlList.push(`${url.origin}${url.pathname}`);

      let requestUrl = `api${url.pathname}/players_sessions`;

      let xhttp = new XMLHttpRequest();
      xhttp.open("GET", requestUrl, true);
      xhttp.onreadystatechange = (function (res, data) {
        return function () {
          if (res.readyState === XMLHttpRequest.DONE && res.status === 200) {
            data.updatePayment(res.responseText);
          }
        }
      })(xhttp, this);
      xhttp.send();
    },

    updatePayment(res) {
      let data = JSON.parse(res);
      let players = data.playersInfos;
      for (let player of Object.values(players)) {
        if (player.net != 0) {
          if (player.names[0] in this.payment) {
            this.payment[player.names[0]] += player.net;
          }
          else {
            this.payment[player.names[0]] = player.net;
          }
        }
      }
    },

    reset() {
      this.payment = {};
      this.urlList = [];
    },

    async copyAllPayments() {
      await navigator.clipboard.writeText(this.allPayments);
      alert("Copied");
    }
  },

  computed: {
    allPayments() {
      if (Object.values(this.payment).reduce((a, b) => a + b, 0) != 0) {        
        return "Ledger does not sum to 0";
      } else {
        let res = [];
        let tmp = JSON.parse(JSON.stringify(this.payment))
        while (Object.keys(tmp).length > 0){
          let max = Object.keys(tmp).reduce((a, b) => tmp[a] > tmp[b] ? a : b);
          let min = Object.keys(tmp).reduce((a, b) => tmp[a] > tmp[b] ? b : a);
          let amt = Math.min(tmp[max], Math.abs(tmp[min]))
          tmp[max] -= amt
          tmp[min] += amt
          if (tmp[max] === 0){
            delete tmp[max]
          }
          if (tmp[min] === 0){
            delete tmp[min]
          }
          res.push(`${min} pays ${max} $${(amt / 100).toFixed(2)}`);
        }

        if (res.length !== 0) {
          res.push(`\nGame${this.urlList.length === 1 ? "" : "s"} played:`);
          res.push.apply(res, this.urlList);
        }

        return res.join("\n");
      }
    }
  }
}
</script>


<style scoped>
#payments {
  white-space: pre-wrap;
  text-align: left;
}

#copyBtn {
  padding: 2px 6px;
  position: absolute;
  right: 4px;
  top: 4px;
}

.center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.snippet {
  margin-top: 20px;
  position: relative;
  overflow: visible;
  display: inline-block;
}

button {
  position: relative;
  display: inline-block;
  padding: 6px 12px;
  font-size: 13px;
  font-weight: 700;
  line-height: 20px;
  color: #333;
  white-space: nowrap;
  vertical-align: middle;
  cursor: pointer;
  background-color: #eee;
  background-image: linear-gradient(#fcfcfc,#eee);
  border: 1px solid #d5d5d5;
  border-radius: 3px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  -webkit-appearance: none;
}
</style>


<template>
  <h2>Poker Now Payments</h2>
  <div class="center">
    <!-- <div class="form-floating row-cols-auto">
      <input v-model="urlInput" v-on:keyup.enter="getLedger" type="text" class="form-control" id="floatingInput" placeholder="Poker Now URL">
      <label for="floatingInput">Poker Now URL</label>
    </div> -->
    <input v-model="urlInput" v-on:keyup.enter="getLedger" type="text" placeholder="Poker Now URL">
    <button @click="getLedger" title="Add a ledger">
      <i class="fa-solid fa-plus"></i>
    </button>
    <button @click="reset" title="Reset">
      <i class="fa-solid fa-rotate-left"></i>
    </button>
  </div>
  <div class="snippet">
    <button v-if="allPayments" @click="copyAllPayments" id="copyBtn" title="Copy">
      <i class="fa-regular fa-copy"></i>
    </button>
    <div id="payments">{{ allPayments }}</div>
  </div>
</template>
