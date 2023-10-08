<template>
  <q-page>
    <div class="text-center q-mt-lg">
      <q-avatar class="q-mb-md" size="xl" square>
        <img :src="`/img/logo.png`" />
      </q-avatar>
      <div class="text-h3">Brands</div>
      <div class="text-positive q-mt-lg">{{ state.status }}</div>

      <q-select
        class="q-mt-lg q-ml-lg"
        v-if="state.brands.length > 0"
        style="width: 50vw; margin-bottom: 4vh; background-color: #fff"
        :option-value="'id'"
        :option-label="'name'"
        v-model="state.selectedBrandId"
        :options="state.brands"
        label="Select a Brand"
        emit-value
        map-options
        @update:model-value="loadProducts()"
      />

      <div
        class="text-h6 text-bold text-center text-primary"
        v-if="state.products.length > 0"
      >
        {{ state.selectedBrand.name }} ITEMS
      </div>
      <q-scroll-area style="height: 55vh">
        <q-card class="q-ma-md">
          <q-list separator>
            <q-item
              v-for="item in state.products"
              :key="item.id"
              clickable
              @click="selectProduct(item.id)"
            >
              <q-item-section avatar>
                <q-avatar style="height: 125px; width: 90px" square>
                  <!-- img -->
                  <img :src="`/img/${item.graphicName}`" />
                  <!-- <img :src="`img/burger.png`" /> -->
                </q-avatar>
              </q-item-section>
              <q-item-section class="text-left">
                {{ item.productName }}
              </q-item-section>
            </q-item>
          </q-list>
        </q-card>
      </q-scroll-area>
    </div>

    <q-dialog
      v-model="state.itemSelected"
      transition-show="rotate"
      transition-hide="rotate"
    >
      <q-card>
        <q-card-actions align="right">
          <q-btn flat label="X" color="primary" v-close-popup class="text-h5" />
        </q-card-actions>

        <q-card-section class="q-pa-none text-center">
          <img
            :src="`/img/${state.selectedProduct.graphicName}`"
            class="item-image"
          />
        </q-card-section>

        <q-card-section>
          <div class="text-h4 text-center" style="color: teal">
            {{ state.selectedProduct.productName }}-{{
              formatCurrency(state.selectedProduct.msrp)
            }}
          </div>
        </q-card-section>

        <q-card-section>
          <div class="text-subtitle2 text-center">
            {{ state.selectedProduct.description }}
          </div>
        </q-card-section>

        <q-card-section class="q-pa-none text-center">
          <div
            style="
              font-weight: bold;
              font-size: larger;
              margin-top: 3vh;
              text-align: center;
            "
          ></div>
          <!-- {{ formatCurrency(state.selectedProduct.msrp) }} -->
        </q-card-section>

        <q-card-section>
          <div class="row">
            <div class="col-2 q-mr-sm">
              <q-input
                v-model.number="state.qty"
                type="number"
                filled
                style="max-width: 15vw"
                placeholder="qty"
                hint="Qty"
                dense
              />
            </div>
            <div class="col-4 q-mr-sm">
              <q-btn
                color="primary"
                label="Add To Cart"
                :disable="state.qty < 0"
                @click="addToCart()"
              />
            </div>
            <div class="col-3">
              <q-btn
                color="primary"
                label="View Cart"
                :disable="state.cart.length < 1"
                @click="viewCart()"
              />
            </div>
          </div>
        </q-card-section>

        <q-card-section class="text-center text-positive">
          {{ state.dialogStatus }}
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-page>
</template>
<script>
import { formatCurrency } from "../utils/formatutils";
import { reactive, onMounted } from "vue";
import { fetcher } from "../utils/apiutil";
import { useRouter } from "vue-router";

export default {
  setup() {
    onMounted(() => {
      loadBrands();
    });

    const router = useRouter();

    let state = reactive({
      status: "",
      brands: [],
      products: [],
      selectedBrand: {},
      selectedBrandId: "",
      selectedProduct: {},
      dialogStatus: "",
      itemSelected: false,
      qty: 0,
      cart: [],
    });

    const loadBrands = async () => {
      try {
        state.status = "loading brands...";
        const payload = await fetcher(`Brand`);
        if (payload.error === undefined) {
          state.brands = payload;
          state.status = `loaded ${state.brands.length} brandss`;
        } else {
          state.status = payload.error;
        }
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    const loadProducts = async () => {
      try {
        state.selectedBrand = state.brands.find(
          (brand) => brand.id === state.selectedBrandId
        );
        state.status = `finding products for brand ${state.selectedBrand}...`;
        state.products = await fetcher(`Product/${state.selectedBrand.id}`);
        state.status = `loaded ${state.products.length} menu items for 
${state.selectedBrand.name}`;
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    const selectProduct = async (productid) => {
      try {
        // q-item click sends us the id, so we need to find the object
        state.selectedProduct = state.products.find(
          (item) => item.id === productid
        );
        state.itemSelected = true;
        state.dialogStatus = "";
        if (sessionStorage.getItem("cart")) {
          state.cart = JSON.parse(sessionStorage.getItem("cart"));
        }
      } catch (err) {
        console.log(err);
        state.status = `Error has occured: ${err.message}`;
      }
    };

    const addToCart = () => {
      let index = -1;
      if (state.cart.length > 0) {
        index = state.cart.findIndex(
          // is item already on the cart
          (item) => item.id === state.selectedProduct.id
        );
      }
      if (state.qty > 0) {
        index === -1 // add
          ? state.cart.push({
              id: state.selectedProduct.id,
              qty: state.qty,
              item: state.selectedProduct,
            })
          : (state.cart[index] = {
              // replace
              id: state.selectedProduct.id,
              qty: state.qty,
              item: state.selectedProduct,
            });
        state.dialogStatus = `${state.qty} product(s) added`;
      } else {
        index === -1 ? null : state.cart.splice(index, 1); // remove
        state.dialogStatus = `product(s) removed`;
      }
      sessionStorage.setItem("cart", JSON.stringify(state.cart));
      state.qty = 0;
    };

    const viewCart = () => {
      router.push("cart");
    };
    return {
      state,
      loadProducts,
      selectProduct,
      formatCurrency,
      addToCart,
      viewCart,
    };
  },
};
</script>
