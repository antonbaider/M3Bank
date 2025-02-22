<template>
  <div class="payment-page">
    <div class="payment-card">
      <h2 class="payment-title">Make a Payment</h2>
      <form @submit.prevent="handlePayment">
        <div class="form-group">
          <label for="senderCardNumber">Sender Card Number</label>
          <select
              id="senderCardNumber"
              v-model="selectedSenderCard"
              required
          >
            <option value="" disabled>Select a card</option>
            <option
                v-for="card in availableCards"
                :key="card.cardNumber"
                :value="card.cardNumber"
            >
              {{ card.cardNumber }} - {{ card.balance }} {{ card.currency }}
            </option>
          </select>
        </div>

        <div class="form-group">
          <label for="receiverCardNumber">Receiver Card Number</label>
          <input
              type="text"
              id="receiverCardNumber"
              v-model="receiverCardNumber"
              placeholder="Enter receiver's card number"
              required
          />
        </div>

        <div class="form-group">
          <label for="amount">Amount</label>
          <input
              type="number"
              id="amount"
              v-model="amount"
              placeholder="Enter amount"
              min="1"
              required
          />
        </div>

        <!-- Adding @click for direct binding as fallback -->
        <button
            type="submit"
            class="payment-button"
            :disabled="!isFormValid || loading"
            @click="handlePayment"
        >
          <span v-if="loading">Processing...</span>
          <span v-else>Send Payment</span>
        </button>
      </form>

      <!-- Display success message -->
      <div v-if="successMessage" class="success-message">
        {{ successMessage }}
      </div>

      <!-- Display error message -->
      <div v-if="errorMessage" class="error-message">
        {{ errorMessage }}
      </div>

      <div v-if="transaction" class="transaction-info">
        <h3>Transaction Details</h3>
        <ul>
          <li><strong>Transaction ID:</strong> {{ transaction.transactionId }}</li>
          <li><strong>Sender Card:</strong> {{ transaction.senderCardNumber }}</li>
          <li><strong>Receiver Card:</strong> {{ transaction.receiverCardNumber }}</li>
          <li><strong>Amount:</strong> {{ transaction.amount }} {{ transaction.currency }}</li>
          <li><strong>Balance After:</strong> {{ transaction.balanceAfter }}</li>
          <li><strong>Timestamp:</strong> {{ formatDate(transaction.timestamp) }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue';
import api from '@/services/api';

export default {
  name: 'Payment',
  setup() {
    const availableCards = ref([]);
    const selectedSenderCard = ref('');
    const receiverCardNumber = ref('');
    const amount = ref('');
    const loading = ref(false);
    const successMessage = ref('');
    const errorMessage = ref('');
    const transaction = ref(null);

    // Fetch available cards for dropdown
    const fetchAvailableCards = async () => {
      try {
        const response = await api.get('/api/accounts');
        availableCards.value = response.data.data;
      } catch (error) {
        console.error('Error fetching accounts:', error);
      }
    };

    // Handle payment submission
    const handlePayment = async () => {
      console.log('handlePayment triggered'); // Debugging to ensure the method runs
      loading.value = true;
      successMessage.value = '';
      errorMessage.value = '';
      transaction.value = null;

      try {
        const response = await api.post('/api/transactions/transferByCard', {
          senderCardNumber: selectedSenderCard.value,
          receiverCardNumber: receiverCardNumber.value,
          amount: parseFloat(amount.value),
        });

        successMessage.value = 'Congratulations! Transfer by card was successful.';
        transaction.value = response.data.data;
      } catch (error) {
        console.error('Error during payment:', error); // Log error for further insight
        errorMessage.value = error.response?.data?.message || 'Payment failed. Please try again.';
      } finally {
        loading.value = false;
      }
    };

    // Check if form fields are valid to enable the submit button
    const isFormValid = computed(() => {
      return selectedSenderCard.value && receiverCardNumber.value && amount.value > 0;
    });

    const formatDate = (timestamp) => {
      return new Date(timestamp).toLocaleString();
    };

    onMounted(fetchAvailableCards);

    return {
      availableCards,
      selectedSenderCard,
      receiverCardNumber,
      amount,
      loading,
      successMessage,
      errorMessage,
      transaction,
      isFormValid,
      formatDate,
      handlePayment,
    };
  },
};
</script>

<style scoped>
.payment-page {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  padding: 20px;

}

.payment-card {
  padding: 25px 30px;
  border-radius: 16px;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
  max-width: 500px;
  width: 100%;
  text-align: center;
  background-color: #f5f7fa;
}

.payment-title {
  margin-bottom: 20px;
  color: #333;
  font-size: 1.8rem;
  font-weight: 600;
}

.form-group {
  margin-bottom: 20px;
  text-align: left;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #555;
}

.form-group input, .form-group select {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 1rem;
}

.payment-button {
  width: 100%;
  padding: 12px;
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 25px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

.payment-button:hover {
  background-color: #357ab8;
  transform: translateY(-2px);
}

.payment-button:disabled {
  background-color: #a5d6a7;
  cursor: not-allowed;
}

.success-message {
  margin-top: 20px;
  font-size: 1rem;
  color: #28a745;
}

.error-message {
  margin-top: 20px;
  font-size: 1rem;
  color: #e74c3c;
}

.transaction-info {
  margin-top: 30px;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  max-width: 500px;
  width: 100%;
  text-align: left;
}

.transaction-info h3 {
  margin-bottom: 15px;
  color: #333;
  font-size: 1.5rem;
  font-weight: 600;
}

.transaction-info ul {
  list-style: none;
  padding: 0;
}

.transaction-info li {
  margin-bottom: 10px;
  font-size: 1rem;
}
</style>