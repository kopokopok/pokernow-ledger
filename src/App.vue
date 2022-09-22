<script>
export default {
  data() {
    return {
      url: "",
      urlList: [],
      payment: {}
    }
  },

  methods: {
    getLedger() {
      let paramIndex = this.url.indexOf("?");
      let rawUrl = (paramIndex > -1) ? this.url.substring(0, paramIndex) : this.url;
      let requestUrl = `/ledger?url=${rawUrl}`;

      let xhttp = new XMLHttpRequest();
      xhttp.open("GET", requestUrl, true);
      xhttp.onreadystatechange = (function (res, data) {
        return function () {
          if (res.readyState === XMLHttpRequest.DONE && res.status === 200) {
            data.payment = data.updatePayment(res.responseText);
          }
        }
      })(xhttp, this);
      xhttp.send();

      this.url = "";
    },

    updatePayment(res) {
      let data = JSON.parse(res).playersInfos;
      let players = data.playersInfos
      for (let player of Object.values(players)) {
        if (player.net != 0) {
          if (player.names[0] in payment) {
            payment[player.names[0]] += player.net;
          }
          else {
            payment[player.names[0]] = player.net;
          }
        }
      }
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
          let min = Object.keys(tmp).reduce((a, b) => tmp[a] > tmp[b] ? a : b);
          let max = Object.keys(tmp).reduce((a, b) => tmp[a] > tmp[b] ? b : a);
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
        return res.join("\n");
      }
    }
  }
}
</script>

<template>
  <div>
    <h1>Enter Poker Now URL below:</h1>
    <input v-model="url" type="text" placeholder="Poker Now URL">
    <button @click="getLedger">Add</button>
    <button @click="reset">Reset</button>
  </div>
  <div style="marginTop:20px">{{ allPayments }}</div>
</template>
