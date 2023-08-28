<template>
  <div>
    <div>
      <h1>Ezo book ATM</h1>
    </div>
    <div class="atm-container">
      <div class="section">
        <h2>Current Available denomination in Machine</h2>
        <div class="denomination-table">
          <div v-for="(count, denom) in denomination" :key="denom" class="denom-row">
            <span class="denom-label">{{ denom }}:</span>
            <input type="number" v-model="denomination[denom]" min="0" />
          </div>
        </div>
        <div>Total: {{ calculateTotalDeposit() }}</div>
        <button class="button" @click="depositMoney">Deposit</button>
      </div>

      <div class="section">
        <h2>Current Available denomination in Machine for withdrawl</h2>
        <div class="denomination-table">
        <div v-for="(count, denom) in availableDenomination" :key="denom" class="denom-row">
          <span class="denom-label">{{ denom }}:</span>
          <span class="denom-count">{{ count }}</span>
        </div>
        <h2>Enter amount for withdrawal</h2>
        <input type="number" v-model="withdrawAmount" />
        <button class="button" @click="withdrawMoney">Withdraw</button>
      </div>
      </div>

      <div class="section">
        <h2>Log</h2>
        <div v-for="logItem in log" :key="logItem" class="log-item">{{ logItem }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import './styles.css';

export default {
  setup() {
    // note denomination
    const denomination = ref({
      2000: 0,
      500: 0,
      200: 0,
      100: 0,
    });
    const availableDenomination = ref({
      2000: 0,
      500: 0,
      200: 0,
      100: 0,
    });

    // refrence to store reactive data
    const withdrawAmount = ref(0);
    const log = ref([]);
    const ledger = ref(0);

    // this will give current timestamp
    let currentTimestamp = () => new Date().toString();

    // this will return total deposit to be calculated
    const calculateTotalDeposit = () => {
      let total = 0;
      for (const denom of Object.keys(denomination.value)) {
        total += denomination.value[denom] * parseInt(denom);
      }
      return total;
    };
    // this function will withdraw money and aslo print logs
    const withdrawMoney = () => {
      if (ledger.value < 0) {
        log.value.push(`[${currentTimestamp()}] Insufficient funds. Money cannot be withdrawn.`);
      } else if (withdrawAmount.value <= 0) {
        log.value.push(`[${currentTimestamp()}] Invalid withdrawal amount.`);
      } else if (ledger.value < withdrawAmount.value) {
        log.value.push(`[${currentTimestamp()}] Not enough funds in the ledger.`);
      } else {
        let remainingAmount = withdrawAmount.value
        // sorting denom notes in assending order
        for (const denom of Object.keys(availableDenomination.value).sort((a, b) => b - a)) {
          const notesToWithdraw = Math.floor(remainingAmount / denom);
          if (notesToWithdraw > 0 && availableDenomination.value[denom] >= notesToWithdraw) {
            availableDenomination.value[denom] -= notesToWithdraw;
            remainingAmount -= notesToWithdraw * denom;
          }
        }
        if (remainingAmount === 0) {
          ledger.value -= withdrawAmount.value;
          log.value.push(`[${currentTimestamp()}] Withdraw successful of ${withdrawAmount.value}`);
        } else {
          log.value.push(`[${currentTimestamp()}] Unable to withdraw exact amount.`);
        }
      }
    };



    // this function will deposit money in the account
    const depositMoney = () => {
      let totalAmount = calculateTotalDeposit();
      if (totalAmount <= 0) {
        log.value.push(`[${currentTimestamp()}] Deposit amount must be greater than 0.`);
        return;
      }
      ledger.value += totalAmount;
      log.value.push(`[${currentTimestamp()}] Deposit of 2000:${denomination.value[2000]},500:${denomination.value[500]},200:${denomination.value[200]},100:${denomination.value[100]} with total of : ${totalAmount} sucessfull`);
      for (const denom of Object.keys(denomination.value)) {
        availableDenomination.value[denom] += denomination.value[denom];
      }

      log.value.push(`[${currentTimestamp()}] Available denomination: 2000:${availableDenomination.value[2000]},500:${availableDenomination.value[500]},200:${availableDenomination.value[200]},100:${denomination.value[100]} `);
    };


    return {
      denomination,
      withdrawAmount,
      availableDenomination,
      log,
      calculateTotalDeposit,
      withdrawMoney,
      depositMoney,
    };
  },
};
</script>
