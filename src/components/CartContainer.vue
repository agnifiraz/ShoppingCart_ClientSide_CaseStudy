<template>
  <div class="text-center">
    <q-avatar class="q-mt-md" size="xl" square>
      <img :src="`/img/logo.png`" />
    </q-avatar>
    <div class="text-h6 text-center text-primary">Cart Contents</div>
    <q-avatar class="q-mt-md" size="xl" square>
      <img :src="`cart.png`" />
    </q-avatar>
    <div class="text-h6 text-positive">{{ state.status }}</div>
  </div>

  <div
    v-if="state.cart.length > 0"
    style="width: 90vw; margin-left: 5vw; margin-top: 1vh"
  >
    <div>
      <q-item style="bottom: -2vh">
        <q-item-section
          class="col-2 q-ml-lg text-accent"
          style="font-weight: bold; font-size: large"
          >Name</q-item-section
        >
        <q-item-section
          class="col-2 q-ml-lg text-accent"
          style="font-weight: bold; font-size: large"
          >Qty</q-item-section
        >
        <q-item-section
          class="col-2 q-ml-lg text-accent"
          style="font-weight: bold; font-size: large"
          >MSRP</q-item-section
        >
        <q-item-section
          class="q-ml-md text-accent"
          style="font-weight: bold; font-size: large"
          >Extended</q-item-section
        >
      </q-item>
      <q-scroll-area style="height: 40vh">
        <q-card class="q-ma-md">
          <q-list separator>
            <q-item v-for="item in state.cart" :key="item.id" clickable>
              <q-item-section class="col-2">
                {{ item.item.productName }}
              </q-item-section>
              <q-item-section class="text-center">
                {{ item.qty }}
              </q-item-section>
              <q-item-section class="text-center">
                {{ formatCurrency(item.item.msrp) }}
              </q-item-section>
              <q-item-section class="text-right">
                {{ formatCurrency(item.item.msrp * item.qty) }}
              </q-item-section>
            </q-item>
            <q-item in state :key="state.id" clickable>
              <q-item-section class="col-2">
                {{ "" }}
              </q-item-section>
              <q-item-section class="text-center">
                {{ "" }}
              </q-item-section>
              <q-item-section
                class="text-center"
                style="font-weight: bold; margin-left: 10px"
              >
                {{ "Sub:" }}
              </q-item-section>
              <q-item-section class="text-center">
                {{ formatCurrency(state.sub) }}
              </q-item-section>
            </q-item>
            <q-item in state :key="state.id" clickable>
              <q-item-section class="col-2">
                {{ "" }}
              </q-item-section>
              <q-item-section class="text-center">
                {{ "" }}
              </q-item-section>
              <q-item-section class="text-center" style="font-weight: bold">
                {{ "Tax(13%):" }}
              </q-item-section>
              <q-item-section class="text-center">
                {{ formatCurrency(state.tax) }}
              </q-item-section>
            </q-item>

            <q-item in state :key="state.id" clickable>
              <q-item-section class="col-2">
                {{ "" }}
              </q-item-section>
              <q-item-section class="text-center">
                {{ "" }}
              </q-item-section>
              <q-item-section
                class="text-center text-primary"
                style="font-weight: bold"
              >
                {{ "Total:" }}
              </q-item-section>
              <q-item-section class="text-center text-primary">
                {{ formatCurrency(state.total) }}
              </q-item-section>
            </q-item>
          </q-list>
        </q-card>
      </q-scroll-area>

      <div class="text-center q-mt-md q-mb-lg">
        <q-btn
          class="q-mr-sm"
          color="primary"
          label="Save Cart"
          :disable="state.cart.length < 1"
          @click="saveCart()"
        />
        <q-btn color="primary" label="Clear Cart" @click="clearCart()" />
      </div>
    </div>
  </div>
</template>

<script>
import { reactive, onMounted } from "vue";
import { formatCurrency } from "../utils/formatutils";
import { poster } from "../utils/apiutil";

export default {
  setup() {
    let state = reactive({
      status: "",
      sub: 0,
      tax: 0,
      total: 0,
      cart: [],
    });

    onMounted(() => {
      if (sessionStorage.getItem("cart")) {
        state.cart = JSON.parse(sessionStorage.getItem("cart"));
        state.cart.forEach((cartItem) => {
          state.sub += cartItem.item.msrp * cartItem.qty;
          state.tax = state.sub * 0.13;
          state.total = state.sub + state.tax;
        });
      } else {
        state.cart = [];
      }
    });

    const clearCart = () => {
      sessionStorage.removeItem("cart");
      state.cart = [];
      state.status = "Cart emptied";
    };

    const saveCart = async () => {
      let customer = JSON.parse(sessionStorage.getItem("customer"));
      let cart = JSON.parse(sessionStorage.getItem("cart"));
      try {
        state.status = "sending cart info to server";
        let orderHelper = { email: customer.email, selections: cart };
        let payload = await poster("order", orderHelper);
        if (payload.indexOf("not") > 0) {
          state.status = payload;
        } else {
          clearCart();
          state.status = payload;
        }
      } catch (err) {
        console.log(err);
        state.status = `Error add cart: ${err}`;
      }
    };

    return {
      state,
      clearCart,
      formatCurrency,
      saveCart,
    };
  },
};
</script>
