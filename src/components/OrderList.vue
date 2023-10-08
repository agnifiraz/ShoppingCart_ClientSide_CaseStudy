<template>
  <div class="text-center">
    <q-avatar class="q-mb-md" size="xl" square>
      <img :src="`/img/logo.png`" />
    </q-avatar>
    <div class="text-h5 q-mt-md">Your Orders</div>
    <div class="text-positive text-h6 q-mt-lg q-mb-lg">
      {{ state.status }}
    </div>
  </div>
  <q-card class="q-ma-sm">
    <q-list highlight>
      <q-item>
        <q-item-section>
          <q-item-label>ID</q-item-label>
        </q-item-section>
        <q-item-section>Date Created</q-item-section>
        <!-- <q-item-section>
          <q-item-label>Total Cals.</q-item-label>
        </q-item-section> -->
      </q-item>
      <q-item
        clickable
        v-for="order in state.orders"
        :key="order.id"
        @click="selectOrder(order.id)"
      >
        <q-item-section>
          {{ order.id }}
        </q-item-section>
        <q-item-section>
          {{ formatDate(order.dateCreated) }}
        </q-item-section>
        <!-- <q-item-section>
          {{ order.totalCalories }}
        </q-item-section> -->
      </q-item>
    </q-list>
  </q-card>

  <q-dialog
    v-model="state.selectedAOrder"
    transition-show="rotate"
    transition-hide="rotate"
  >
    <q-card>
      <q-card-actions align="right">
        <q-btn flat label="X" color="primary" v-close-popup class="text-h5" />
      </q-card-actions>
      <q-card-section>
        <div class="text-h5 text-center">
          Order: #{{ state.orderDetails[0].orderId }}
        </div>
      </q-card-section>
      <q-card-section class="q-pa-none text-center">
        <q-item-section
          style="
            font-weight: bold;
            font-size: small;
            text-align: center;
            padding: 10px;
            color: palevioletred;
          "
        >
          {{ formatDate(state.orderDetails[0].dateCreated) }}
        </q-item-section>

        <q-avatar>
          <img :src="`cart.png`" />
        </q-avatar>
      </q-card-section>
      <q-card-section class="q-mb-none">
        <q-list>
          <q-item style="bottom: -2vh">
            <q-item-section
              class="col-3 q-sm-lg text-accent text-left"
              style="
                font-weight: bold;
                font-size: medium;
                text-align: left;
                padding-right: 5px;
                padding-left: -5px;
              "
              >Name</q-item-section
            >
            <q-item-section
              class="col-2 q-ml-sm text-accent text-center"
              style="
                font-weight: bold;
                font-size: medium;
                text-align: center;
                padding: 10px;
              "
              >Quantities</q-item-section
            >
            <q-item-section
              class="q-ml-md text-accent text-right"
              style="font-weight: bold; font-size: medium"
              >Extended</q-item-section
            >
          </q-item>

          <q-item v-for="detail in state.orderDetails" :key="detail.orderId">
            <q-item-section class="text-left col-3">
              {{ detail.productName }}
            </q-item-section>
            <q-item-section class="text-left col-2">
              {{ detail.qtySold }}
            </q-item-section>
            <q-item-section class="text-left col-2">
              {{ detail.qtyOrdered }}
            </q-item-section>
            <q-item-section class="text-left col-2">
              {{ detail.qtyBackOrdered }}
            </q-item-section>
            <q-item-section class="text-left col-2">
              ${{ detail.sellingPrice * detail.qty }}
            </q-item-section>
          </q-item>
          <!-- <q-item in state :key="order.id" clickable> -->
          <q-item v-for="order in state.orders" :key="order.id">
            <q-item-section>
              Sub-Total: ${{ order.orderAmount }}
            </q-item-section>
            <q-item-section>
              Tax(13%): ${{ order.orderAmount * 0.13 }}
            </q-item-section>
            <q-item-section>
              Total: ${{ order.orderAmount + order.orderAmount * 0.13 }}
            </q-item-section>
          </q-item>
          <!-- <q-item v-for="order in state.orders" :key="order.id">
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
              {{ order.orderAmount }}
            </q-item-section>
           
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
              {{ order.orderAmount * 0.13 }}
            </q-item-section>
        
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
              {{ order.orderAmount * order.orderAmount * 0.13 }}
            </q-item-section>
          </q-item> -->
        </q-list>
      </q-card-section>
      <q-card-section
        class="text-center text-positive text-h6 q-mb-xs q-mt-lg q-pa-none"
      >
        {{ state.dialogStatus }}
      </q-card-section>
      <q-card-section class="q-pa-none text-center q-ma-xs">
        <div
          style="
            font-weight: bold;
            font-size: larger;
            margin-top: 0vh;
            text-align: center;
          "
        ></div>
      </q-card-section>
    </q-card>
  </q-dialog>
</template>
<script>
import { reactive, onMounted } from "vue";
import { fetcher } from "../utils/apiutil";
import { formatDate } from "../utils/formatutils";

export default {
  setup() {
    let state = reactive({
      status: "",
      dialogStatus: "",
      selectedAOrder: false,
      orderDetails: [],
      orderSelected: {},
    });

    onMounted(() => {
      loadOrders();
      if (sessionStorage.getItem("order")) {
        state.order = JSON.parse(sessionStorage.getItem("order"));
        state.order.forEach((orderLineItem) => {
          state.sub += orderLineItem.detail.qty * orderLineItem.qty;
          state.tax = state.sub * 0.13;
          state.total = state.sub + state.tax;
        });
      } else {
        state.order = [];
      }
    });

    const loadOrders = async () => {
      try {
        let customer = JSON.parse(sessionStorage.getItem("customer"));
        const payload = await fetcher(`order/${customer.email}`);
        if (payload.error === undefined) {
          state.orders = payload;
          state.status = `loaded ${state.orders.length} orders`;
        } else {
          state.status = payload.error;
        }
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    const selectOrder = async (orderId) => {
      try {
        let customer = JSON.parse(sessionStorage.getItem("customer"));
        const payload = await fetcher(`order/${orderId}/${customer.email}`);
        state.orderDetails = payload;
        state.dialogStatus =
          `details for order ` + state.orderDetails[0].orderId;
        state.selectedAOrder = true;
        state.orderSelected = state.orders.find(
          (order) => order.id === orderId
        );
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    return {
      state,
      formatDate,
      selectOrder,
    };
  },
};
</script>
