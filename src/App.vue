<template>
  <div id="app">
    <head>
      <link href='https://fonts.googleapis.com/css?family=Roboto:100,300,400,500,700,900|Material+Icons' rel="stylesheet">
    </head>
    <div class="top-div">The real-time currency data coming from an external API: </div>
    <div class="box animated bounceInLeft">
      <div v-for="currency in currencies" :key="currency.code">
        <span>{{ currency.description }} ({{ currency.code }}):</span>
        <span class="highlight right">
          <span v-html="currency.symbol"></span>{{ currency['rate_float'] | currencydecimal }}
        </span>
      </div>
    </div>

    <div class="bottom-div">The CALCULATOR: </div>
    <div class="bottom-box animated bounceInLeft">
      
      <div>convert from</div>
      <ol id="convert-from">
        <li v-for="currency in currencies" :key="currency.description" @click="convertFrom">
          {{ currency.description }}
        </li>
      </ol>
      <div>convert to</div>
      <ol id="convert-to">
        <li v-for="currency in currencies" :key="currency.description" @click="convertTo">
          {{ currency.description }}
        </li>
      </ol>
      <input class="currency-input" type="number" v-model.number="amount" placeholder="Enter Amount" @keyup="convertCurrency">
      <div>Result is: {{ result }} </div>
    </div>
  </div>

</template>

<script>

import axios from 'axios';

export default {
  name: 'app',

  data () {
    return {
      currencies: null,
      updated: null,
      loading: true,
      errored: false,
      result: 0,
      from: 0,
      to: 0,
      amount: 0
    }
  },

  filters: {
    currencydecimal (value) {
      return value.toFixed(2)
    }
  },

  methods: {
    convertCurrency() {
      // base currency is bitcoin in this API!
      switch(this.from) {
         case "United States Dollar":
           if (this.to == "British Pound Sterling") {
             this.result = parseInt(this.amount) * parseInt(this.GBPrate);
           }
           if (this.to == "Euro") {
             this.result = parseInt(this.amount) * parseInt(this.EURrate);
           }
           break;
         case "British Pound Sterling":
           if (this.to == "United States Dollar") {
             this.result = parseInt(this.amount) * parseInt(this.USDrate);
           }
           if (this.to == "Euro") {
             this.result = parseInt(this.amount) * parseInt(this.EURrate);
           }
           break;
         case "Euro":
           if (this.to == "United States Dollar") {
             this.result = parseInt(this.amount) * parseInt(this.USDrate);
           }
           if (this.to == "British Pound Sterling") {
             this.result = parseInt(this.amount) * parseInt(this.GBPrate);
           }
           break;
       }
    },

    convertFrom(value) {
      this.from = value.srcElement.innerText;
    },

     convertTo(value) {
      this.to = value.srcElement.innerText;
    }
  },

  mounted () {
    axios
      .get('https://api.coindesk.com/v1/bpi/currentprice.json')
      .then(response => {
          this.currencies = response.data['bpi'];
          this.updated    = response.data.time.updated;

          let crc = Object.values(this.currencies);
          this.USDrate = crc[0].rate;
          this.GBPrate = crc[1].rate;
          this.EURrate = crc[2].rate;
        })
        .catch(error => { // Executes if an error occurs if code is not >= 200 && < 300
          this.showError = true;
          this.error     = error;
        })
      .finally(() => this.loading = false)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.currency-input{
	width: 20%;
	margin-right: 5px;
	font-size: 14px;
	height: 30px;
	border-radius: 5px;
	border: 1px solid #41b883;
	padding-left: 10px;
}

.box {
  width: 360px;
  padding: 1.6em;
  background: rgb(233, 233, 233);
  border-radius: 6px;
  color: rgb(66, 66, 66);
  margin: 0 auto;
  text-align: left;
  line-height: 1.6em;
}

.bottom-box {
  width: 600px;
  padding: 1.6em;
  background: rgb(233, 233, 233);
  border-radius: 6px;
  color: rgb(66, 66, 66);
  margin: 0 auto;
  text-align: left;
  line-height: 1.6em;
}

.highlight {
  color: #41b883;
}

.right {
  float: right;
}

.updated {
  font-size: medium;
  line-height: 2em;
  color: #35495e;
}

h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.top-div {
  margin-bottom: 30px;
  width: 100%;
  text-align: center;
}

.bottom-div {
  margin: 30px 0;
  width: 100%;
  text-align: center;
}

</style>

