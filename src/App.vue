<template>
  <div class="card-form">
    <div class="card-form-title">
      <img class="bank-card" src="./assets/bank-card.svg"/>
      <h2 class="title">Додати картку</h2>
    </div>
    <div :class="['card-form-main', cardType + '-fade']">
      <div class="card-number-data">
        <input
            class="card-form-number"
            type="text"
            placeholder="Номер картки"
            v-model="cardNumber"
            maxlength="19"
            @input="formatCardNumber"
        />
        <img class="card-logo" v-if="cardType === 'visa'" src="./assets/visa.svg"/>
        <img class="card-logo" v-if="cardType === 'mastercard'" src="./assets/mastercard.svg"/>
      </div>
      <div class="thick-separator1"></div>
      <div class="card-data-row">
        <input
            class="card-form-expiry"
            type="text"
            placeholder="ММ / РР"
            v-model="expiry"
            maxlength="7"
            @input="formatExpiry"
        />
        <input
            class="card-form-cvc"
            type="password"
            placeholder="CVC"
            v-model="cvc"
            maxlength="3"
        />
      </div>
      <div class="thick-separator-row">
        <div class="thick-separator2"></div>
        <div class="thick-separator3"></div>
      </div>
      <div class="info">
        <img class="notification" src="./assets/notification.svg"/>
        <p class="info-text"> Задля перевірки вашої карти ми спишемо 1 грн </p>
      </div>
      <button :class="buttonClass" @click="addCard">Оплатити</button>
      <div v-if="expiryError" :class="['error', errorColor]">{{ expiryError }}</div>
      <div v-if="cvcError" :class="['error', errorColor]">{{ cvcError }}</div>
      <div v-if="cardNumberError" :class="['error', errorColor]">{{ cardNumberError }}</div>
    </div>
    <div class="card-form-notification">
      <img class="tick" src="./assets/tick.svg"/>
      <div class="success-message" v-if="message">{{ message }}</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cardNumber: '',
      expiry: '',
      cvc: '',
      message: 'Безпечне підключення',
      defaultCardType: 'visa',
      cardNumberError: '',
      expiryError: '',
      cvcError: ''
    };
  },
  computed: {
    isCardValid() {
      const cardNumber = this.cardNumber.replace(/\s/g, '');
      return this.luhnCheck(cardNumber);
    },
    cardType() {
      const number = this.cardNumber.replace(/\s/g, '');
      if (/^4[0-9]{0,15}$/.test(number)) {
        return 'visa';
      } else if (/^5[1-5][0-9]{0,14}$/.test(number)) {
        return 'mastercard';
      }
      return this.defaultCardType;
    },
    buttonClass() {
      return this.cardType === 'visa' ? 'visa-button' : this.cardType === 'mastercard' ? 'mastercard-button' : '';
    },
    errorColor() {
      return this.cardType === 'visa' ? 'visa-error' : this.cardType === 'mastercard' ? 'mastercard-error' : '';
    }
  },
  methods: {
    formatCardNumber() {
      this.cardNumber = this.cardNumber
          .replace(/\D/g, '')
          .replace(/(.{4})/g, '$1 ')
          .trim();
    },
    formatExpiry() {
      this.expiry = this.expiry
          .replace(/\D/g, '')
          .replace(/(\d{2})(\d{1,2})/, '$1 / $2')
          .slice(0, 7);
    },
    luhnCheck(cardNumber) {
      let sum = 0;
      let shouldDouble = false;

      for (let i = cardNumber.length - 1; i >= 0; i--) {
        let digit = parseInt(cardNumber[i]);

        if (shouldDouble) {
          digit *= 2;
          if (digit > 9) digit -= 9;
        }

        sum += digit;
        shouldDouble = !shouldDouble;
      }

      return sum % 10 === 0;
    },
    validateCardNumber() {
      const cardNumber = this.cardNumber.replace(/\s/g, '');
      if (!this.luhnCheck(cardNumber)) {
        this.cardNumberError = 'Некоректний номер картки. Спробуйте ще раз';
        return false;
      }
      this.cardNumberError = '';
      return true;
    },
    validateExpiry() {
      const expiry = this.expiry.replace(/\s/g, '');
      const [month, year] = expiry.split('/').map(Number);
      const currentYear = new Date().getFullYear() % 100;
      const currentMonth = new Date().getMonth() + 1;

      if (!/^\d{2}\/\d{2}$/.test(expiry) || month < 1 || month > 12 || year < currentYear || (year === currentYear && month < currentMonth)) {
        this.expiryError = 'Некоректний термін дії. Спробуйте ще раз';
        return false;
      }
      this.expiryError = '';
      return true;
    },
    validateCvc() {
      if (!/^\d{3}$/.test(this.cvc)) {
        this.cvcError = 'Некоректний CVC. Спробуйте ще раз';
        return false;
      }
      this.cvcError = '';
      return true;
    },
    addCard() {
      const isCardNumberValid = this.validateCardNumber();
      const isExpiryValid = this.validateExpiry();
      const isCvcValid = this.validateCvc();

      if (isCardNumberValid && isExpiryValid && isCvcValid) {
        window.location.href = '/merchant';
      } else {
        this.message = 'Будь ласка, виправте помилки';
      }
    }
  }
};
</script>

<style>
@font-face {
  font-family: 'Geist Mono';
  src: url('./assets/fonts/GeistMono-Regular.woff2') format('woff2'),
  url('./assets/fonts/GeistMono-Regular.woff') format('woff'),
  url('./assets/fonts/GeistMono-Regular.ttf') format('truetype');
  font-weight: normal;
  font-style: normal;
}

body {
  background-color: #f4f4f4;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
  font-family: 'Geist Mono', monospace;
}

.card-form {
  width: 100%;
  max-width: 400px;
  margin: 20px;
}

.card-form-title,
.card-form-notification {
  background: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  text-align: left;
  margin-top: 10px;
  margin-bottom: 10px;
  display: flex;
  align-items: center;
  gap: 20px;
}

.bank-card {
  height: 50px;
}

.tick {
  height: 30px;
}

.card-logo {
  height: 90%;
}

.card-form-main {
  background: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.card-number-data {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 70px;
}

.card-form-number {
  width: calc(100% - 100px);
  padding: 10px;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  outline: none;
}

.card-data-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.card-form-expiry,
.card-form-cvc {
  width: 43%;
  padding: 10px;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  outline: none;
  margin-top: 10px;
  gap: 10px;
}

.card-form-main button {
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  font-size: 16px;
  cursor: pointer;
  width: 100%;
}

.card-form-main button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.visa-button {
  background-color: #2566af;
}

.mastercard-button {
  background-color: #D9222A;
}

.visa-error {
  color: #2566af;
}

.mastercard-error {
  color: #D9222A;
}

.info {
  font-size: 12px;
  color: #777;
  margin-top: 20px;
  margin-bottom: 20px;
  text-align: left;
  display: flex;
  align-items: center;
  gap: 10px;
}

.info-text {
  color: grey;
  font-size: 13px;
}

.success-message {
  color: black;
  font-size: 19px;
}

.notification {
  height: 30px;
  width: 30px;
}

.thick-separator-row {
  display: flex;
  justify-content: space-between;
}

.thick-separator1 {
  width: 100%;
  height: 1px;
  background-color: #ccc;
}

.thick-separator2,
.thick-separator3 {
  width: 48%;
  height: 1px;
  background-color: #ccc;
}

.error {
  margin-top: 20px;
  font-size: 12px;
}
</style>