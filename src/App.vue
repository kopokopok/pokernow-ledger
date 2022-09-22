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
      this.urlList.push(url.href);

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
      this.payments = {};
      this.urlList = [];
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
          res.push("\nGames played:");
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
  margin-top: 20px;
  white-space: pre-wrap;
}
</style>


<template>
  <div>
    <h1>Enter Poker Now URL below:</h1>
    <input v-model="urlInput" type="text" placeholder="Poker Now URL">
    <button @click="getLedger">Add</button>
    <button @click="reset">Reset</button>
  </div>
  <div id="payments">{{ allPayments }}</div>
</template>
