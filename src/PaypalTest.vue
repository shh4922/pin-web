<script setup lang="ts">

import axios from "axios";
import {onMounted, ref} from "vue";

const BASE = "https://api-m.sandbox.paypal.com";
const CLIENT_ID = "AUHCFgRW81GY2iQjQmJF8tHWLjl0-TxGfCjjpmbpXs3AU9_8Ea3DjjiNqFGDoCg7_PkHLP4-wT6t6k9n"
const SECRET    = "EG1G_olUfmpk3lj_y7Pe5ukC4ThNUAhfdkXPloGz55hPZEnIF5_w0ODKRx9sjEQ3wBfIbfI0GEWddv6m"

const paypalProcess = async () => {
  const token = await getToken();
  const products = await listProducts(token);
  console.log("Products:", products);

  const first = products?.products?.[0]?.id;
  if (first) {
    const plans = await listPlans(token, first);
    console.log("Plans for", plans);
    planId.value = plans.plans[0].id
  }
}

async function getToken() {
  const { data } = await axios.post(`${BASE}/v1/oauth2/token`, "grant_type=client_credentials", {
    auth: { username: CLIENT_ID, password: SECRET },
    headers: { "Content-Type": "application/x-www-form-urlencoded" }
  });
  return data.access_token;
}

async function listProducts(token:string) {
  const { data } = await axios.get(`${BASE}/v1/catalogs/products?page_size=20`, {
    headers: { Authorization: `Bearer ${token}` }
  });
  return data;
}

async function savePaypalInfo(email:string, subscriptionId:string) {
  const res = await axios.post("http://localhost:3000/api/subscriptions/confirm",{
    email: email,
    subscriptionId:subscriptionId
  })
  console.info("savePaypalInfo", res)
}

async function listPlans(token:string, productId:string) {
  const { data } = await axios.get(`${BASE}/v1/billing/plans`, {
    headers: { Authorization: `Bearer ${token}` },
    params: { product_id: productId, page_size: 20 }
  });
  return data;
}

// PayPal SDK 로드 함수
function loadPaypalSdk(clientId:string) {
  return new Promise((resolve, reject) => {
    if (window.paypal) return resolve(window.paypal);

    const qs = new URLSearchParams({
      "client-id": clientId,
      "vault": "true",
      "intent": "subscription"
    });

    const script = document.createElement("script");
    script.src = `https://www.paypal.com/sdk/js?${qs.toString()}`;
    script.async = true;
    script.onload = () => resolve(window.paypal);
    script.onerror = () => reject(new Error("PayPal SDK 로드 실패"));
    document.head.appendChild(script);
  });
}

async function renderPaypalButton() {
  approvedId.value = "";
  errorMsg.value = "";

  try {
    const paypal = await loadPaypalSdk(clientId.value);

    const mountEl = document.getElementById("paypal-button-container");
    mountEl.innerHTML = ""; // 중복 방지

    paypal.Buttons({
      style: { shape: "pill", color: "gold", label: "subscribe" },

      createSubscription(data, actions) {
        return actions.subscription.create({
          plan_id: planId.value,
          custom_id: 'customEamil@test.com'
        });
      },

      onApprove(data, actions) {
        approvedId.value = data.subscriptionID;
        console.log("[onApprove]", data);
        savePaypalInfo("test111@test.com",data.subscriptionID)
        alert("구독 승인됨!\nSubscriptionID: " + data.subscriptionID);
      },

      onError(err) {
        console.error("[PayPal Error]", err);
        errorMsg.value = String(err?.message || err);
      }
    }).render("#paypal-button-container");
  } catch (e) {
    console.error(e);
    errorMsg.value = e.message || String(e);
  }
}

onMounted(() => {
  paypalProcess()
})

const clientId = ref(CLIENT_ID); // Sandbox Client ID 입력
const planId = ref("");   // Sandbox Plan ID 입력
const approvedId = ref("");
const errorMsg = ref("");
</script>

<template>
  <div class="paypal-subscribe">
    <h2>PayPal Subscription Test</h2>

    <div class="input-row">
      <label>Client ID</label>
      <input v-model="clientId" placeholder="YOUR_SANDBOX_CLIENT_ID" />
    </div>

    <div class="input-row">
      <label>Plan ID</label>
      <input v-model="planId" placeholder="P-XXXXXXXXXXXX" />
    </div>

    <button :disabled="!clientId || !planId" @click="renderPaypalButton">
      PayPal 버튼 로드
    </button>

    <div id="paypal-button-container" style="margin-top:20px;"></div>

    <div v-if="approvedId" class="ok">
      ✅ 구독 승인됨: {{ approvedId }}
    </div>
    <div v-if="errorMsg" class="err">
      ❌ 에러: {{ errorMsg }}
    </div>

    <button @click="savePaypalInfo('test@test.com','I-1234123')">testPost</button>
  </div>
</template>

<style scoped>
.paypal-subscribe {
  max-width: 500px;
  margin: 24px auto;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif;
}
.input-row {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
}
.input-row label {
  width: 100px;
}
input {
  flex: 1;
  padding: 8px 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
}
button {
  padding: 8px 14px;
  border: none;
  border-radius: 20px;
  background: #ffd140;
  font-weight: bold;
  cursor: pointer;
}
button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}
.ok {
  margin-top: 16px;
  color: green;
}
.err {
  margin-top: 16px;
  color: red;
}
</style>