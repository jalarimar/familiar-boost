<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-img :src="require('../assets/logo.gif')" class="mt-3" contain height="500"/>
      </v-col>
      <v-col class="mb-4">
        <h1 class="display-2 font-weight-bold mb-3 light-text">
          Crafting Helper
        </h1>
        <p class="subheading font-weight-regular light-text">
          Made by RoyalCanary, with logo by ElsaMouse.
        </p>
      </v-col>
    </v-row>

    <v-row class="light-bg">
      <v-col cols="8" offset="2">
        <p>No commas or other characters in any of the fields please! Numbers only.</p>
      </v-col>
    </v-row>
    <v-row class="light-bg">
      <v-col cols="4" offset="2">
        <v-text-field label="A" v-model="a"/>
        <v-text-field label="B" v-model="b"/>
        <v-text-field label="C" v-model="c"/>
        <v-text-field label="D" v-model="d"/>
        <v-text-field label="E" v-model="e"/>
      </v-col>
      <v-col cols="4">
        <v-text-field label="Current tokens" v-model="currentTokens"/>
        <v-text-field label="Current star count" v-model="currentStars"/>
        <v-text-field label="Hours left" v-model="hoursLeft"/>
        <v-text-field label="Minutes left" v-model="minsLeft"/>
        <v-text-field label="Boost multiplier" v-model="boostMultiplier"/>
      </v-col>
    </v-row>
    <v-row class="light-bg">
      <v-col cols="10" offset="1">
        Star goal: {{goalStars}}
        <v-slider 
          v-model="goalStars"
          min="1000"
          max="25000"
          step="1000"
          ticks="always"
          tick-size="2"
        ></v-slider>
      </v-col>
    </v-row>
    <v-row class="light-bg">
      <v-col cols="4" offset="2">
        <p>Numbers to do per batch</p>
        <p>A: {{aPerBatch}}</p>
        <p>B: {{bPerBatch}}</p>
        <p>C: {{cPerBatch}}</p>
        <p>D: {{dPerBatch}}</p>
        <p>E: {{ePerBatch}}</p>
      </v-col>
      <v-col cols="4">
        <p>Craft Cost: {{craftCost.toLocaleString()}}</p>
        <p>Star Cost: {{starCost.toLocaleString()}}</p>
        <p>Total Tokens Required: {{tokensRequiredForGoal.toLocaleString()}}</p>
        <p>Remaining Tokens: {{spareTokens.toLocaleString()}}</p>
        <h4 v-if="spareTokens < 200000 && spareTokens >= 0" class="orange--text">Watch your tokens</h4>
        <h3 v-if="spareTokens < 0" class="red--text">NOT ENOUGH TOKENS</h3>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
  export default {
    data: () => ({
      currentTokens: 0,
      currentStars: 0,
      a: 0,
      b: 0,
      c: 0,
      d: 0,
      e: 0, 
      goalStars: 10000,
      hoursLeft: 10,
      minsLeft: 0,
      boostMultiplier: 1,
    }),
    computed: {
      craftCost: function() {
        let multiplier = parseInt(this.boostMultiplier);

        let aCostPerBatch = this.calcUnboostedCPB(this.aPerBatch, 20, 10) / multiplier;
        let aCostTotal = aCostPerBatch * this.batchesLeft;

        let bCostPerBatch = this.calcUnboostedCPB(this.bPerBatch, 20, 10) / multiplier;
        let bCostTotal = bCostPerBatch * this.batchesLeft;

        let cCostPerBatch = this.calcUnboostedCPB(this.cPerBatch, 20, 10) / multiplier;
        let cCostTotal = cCostPerBatch * this.batchesLeft;

        let dCostPerBatch = this.calcUnboostedCPB(this.dPerBatch, 150, 75) / multiplier;
        let dCostTotal = dCostPerBatch * this.batchesLeft;

        let eCostPerBatch = this.calcUnboostedCPB(this.ePerBatch, 200, 100) / multiplier;
        let eCostTotal = eCostPerBatch * this.batchesLeft;

        return Math.floor(aCostTotal + bCostTotal + cCostTotal + dCostTotal + eCostTotal);
      },
      starCost: function() {
        if (this.currentStars > this.goalStars) {
          return 0;
        }
        return (this.goalStars - this.currentStars) * 100;
      },
      tokensRequiredForGoal: function() {
        return this.starCost + this.craftCost;
      },
      spareTokens: function() {
        return this.currentTokens - this.tokensRequiredForGoal;
      },
      batchesLeft: function() {
        let totalMinsLeft = parseInt(this.hoursLeft * 60) + parseInt(this.minsLeft);
        return Math.floor(totalMinsLeft / 15) + 1;
      },
      aUsed: function() {
        return (parseInt(this.currentStars) * 1/3) + ((parseInt(this.d) + this.dUsed) * 3);
      },
      bUsed: function() {
        return (parseInt(this.currentStars) * 1/3) + ((parseInt(this.d) + this.dUsed) * 2) + ((parseInt(this.e) + this.eUsed) * 4);
      },
      cUsed: function() {
        return (parseInt(this.currentStars) * 1/3) + ((parseInt(this.d) + this.dUsed) * 2) + ((parseInt(this.e) + this.eUsed) * 4);
      },
      dUsed: function() {
        return (parseInt(this.e) + this.eUsed) * 2;
      },
      eUsed: function() {
        return parseInt(this.currentStars) * 1/3; 
      },
      aPerBatch: function() {
        let aRequiredTotal = parseInt(this.goalStars) * 7/3;
        let aToMake = aRequiredTotal - this.aUsed - parseInt(this.a);
        if (aToMake < 1) {
          return 0;
        }
        if (aToMake < this.batchesLeft) {
          return 1;
        }
        return Math.floor(aToMake / this.batchesLeft);
      },
      bPerBatch: function() {
        let bRequiredTotal = parseInt(this.goalStars) * 9/3;
        let bToMake = bRequiredTotal - this.bUsed - parseInt(this.b);
        if (bToMake < 1) {
          return 0;
        }
        if (bToMake < this.batchesLeft) {
          return 1;
        }
        return Math.floor(bToMake / this.batchesLeft);
      },
      cPerBatch: function() {
        let cRequiredTotal = parseInt(this.goalStars) * 9/3;
        let cToMake = cRequiredTotal - this.cUsed - parseInt(this.c);
        if (cToMake < 1) {
          return 0;
        }
        if (cToMake < this.batchesLeft) {
          return 1;
        }
        return Math.floor(cToMake / this.batchesLeft);
      },
      dPerBatch: function() {
        let dRequiredTotal = parseInt(this.goalStars) * 2/3;
        let dToMake = dRequiredTotal - this.dUsed - parseInt(this.d);
        if (dToMake < 1) {
          return 0;
        }
        if (dToMake < this.batchesLeft) {
          return 1;
        }
        return Math.floor(dToMake / this.batchesLeft);
      },
      ePerBatch: function() {
        let eRequiredTotal = parseInt(this.goalStars) * 1/3;
        let eToMake = eRequiredTotal - this.eUsed - parseInt(this.e);
        if (eToMake < 1) {
          return 0;
        }
        if (eToMake < this.batchesLeft) {
          return 1;
        }
        return Math.floor(eToMake * 2 / this.batchesLeft);
      },
    },
    methods: {
      calcUnboostedCPB(batchSize, firstCost, costJump) {
        let r = batchSize - 1;
        let p = firstCost;
        let q = costJump;
        return (r*q)*((r*q)+q)/(q*2) + (r+p)*(r+p+1)/2 - r*(r+1)/2 - (p-1)*p/2;
      },
    }
  }
</script>

<style scoped>
.light-bg {
  background-color: #eeeeee;
}
.light-text {
  color: #eeeeee;
}
</style>